# transcriptome-properties
Calculate various properties of an input transcriptome (GTF/knownGenes)

This program calculates various biological paramters regarding a set of transcript regions. Some properties are universal (length, GC content, etc) and some are specific to certain transcript regions (e.g. rare codons are only defined for a coding sequence).

#### Input 

Two inputs are mandatory: 

-i: A .bed file describing transcript regions over which to calculate properties
-g: A FASTA file of the genome from which the transcriptome is derived. 

Further inputs determine the properties to compute of the input transcriptome

#### Output 

The "-o" flag specifies the output basename, and a series of CSV files will be generated with this basename for each property supplied on the command line. 

#### Example usage
```
./transcriptome_properties.py -i grch38_exons.bed -g genome.fa -o grch38_tx --nt 4 --exonct --length --gc
```

#### Example output

```
==> grch38_tx_exonct.csv <==
transcriptID,exonct
ENST00000618880_exon,4

==> grch38_tx_gc_content.csv <==
transcriptID,gc_content
ENST00000618880_exon,0.64

==> grch38_tx_length.csv <==
transcriptID,length
ENST00000618880_exon,582
```

#### Dependencies

The non-python dependencies are variable depending on the properties one wishes to compute. 

##### Software

ViennaRNA - https://www.tbi.univie.ac.at/RNA/index.html

##### External data

Transcriptome region BED files - generated from https://github.com/stephenfloor/extract-transcript-regions
Whole genome fasta file - downloaded from UCSC/Ensembl or similar
TargetScan miRNA scores - downloaded from http://www.targetscan.org/cgi-bin/targetscan/data_download.cgi?db=vert_70
 -- download the Summary_Counts file
 -- can extract human-only with: awk '$4 == 9606 {print $0}' Summary_Counts.txt > human_mir_targets.txt
Ensembl/UCSC-to-refseq mapping - required for calculation of TargetScan scores per transcript region

#### Notes

Multithreading is currently only used during the calculation of RNA folding energies, which is by far the most time consuming step. Using RNALfold
is considerably faster than the sliding window RNAfold procedure implemented in this program, but RNALfold only returns the minimum free energy and as such it may be desirable to use RNAfold depending on the application. 

#### Full usage 
```
./transcriptome_properties.py -h
usage: transcriptome_properties.py [-h] -i INPUT -g GENOME [--gc] [--length]
                                   [--exonct] [--nt NT] [-o OUTPUT]
                                   [--window WINDOW]
                                   [--convtorefseq CONVTOREFSEQ]
                                   [--targetscanfile TARGETSCANFILE]
                                   [--deltag] [--lfold] [--cap-structure]
                                   [--kozak] [--uorf-count] [--uorf-overlap]
                                   [--start-codon] [--rare-codons]
                                   [--mirna-sites] [--au-elements]

Calculate transcriptome-wide properties

optional arguments:
  -h, --help            show this help message and exit

Global arguments:
  -i INPUT, --input INPUT
                        The transcriptome region (BED format)
  -g GENOME, --genome GENOME
                        The genome for the input transcriptome
  --gc                  Calculate GC content
  --length              Calculate length
  --exonct              Count # of exons
  --nt NT               Number of threads (default is 8 or 4 for lfold)
  -o OUTPUT, --output OUTPUT
                        Output basename (e.g. CDS)
  --window WINDOW       Window size for sliding window calculations (default
                        75)
  --convtorefseq CONVTOREFSEQ
                        Filename to convert input annotations to refseq (for
                        targetscan; e.g. knownToRefSeq.txt)
  --targetscanfile TARGETSCANFILE
                        Filename of targetscan scores (e.g.
                        Summary_Counts.txt)
  --deltag              Calculate min deltaG in sliding window of size
                        --window over region
  --lfold               Use RNALfold to calculate MFE rather than RNAfold
                        (faster but does not compute centroid,MEA)

5' UTR specific arguments:
  --cap-structure       Calculate structure at the 5' end

Start-codon-specific arguments:
  --kozak               Calculate Kozak context score
  --uorf-count          Calculate number of 5' UTR uORFs (starting with
                        [ACT]TG)
  --uorf-overlap        Overlap of uORF with start codon (implies --uorf-
                        count)
  --start-codon         Record the start codon used (ATG or other)

CDS-specific arguments:
  --rare-codons         Calculate codon usage properties

3' UTR specific arguments:
  --mirna-sites         Compile miRNA binding site info from targetscan
  --au-elements         Count number of AU-rich elements in the 3' UTR