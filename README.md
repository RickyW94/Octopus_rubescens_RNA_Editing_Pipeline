# RNA Editing Pipeline
## Perform QC on raw RNA reads
Navigate to folder with raw reads
```
/media/work/Ricky_Sequencing/RNA/Working_RNA
```
Octopuses were numbered 1 through 12 in the original experiment. Numbers 4,6,7,8,9,10 were chosen for WGS and RNASeq.
Numbers 4,9,10 were controls, not subjected to elevated pCO2
Numbers 6,7,8 were treatments, subjected to elevated pCO2
The raw reads were named according to the following format:
  R[octopus_number]c_[1 or 2].fq.gz
  '1' or '2' corresponds to the forward or reverse read file for that particular octopus
  'R10c_2.fq.gz' is the second of two read files, the reverse read file is named 'R10c_1.fq.gz'
  The '.fq' extension denotes the fastq format, while '.gz' means it is a 'gzipped' file, AKA compressed

## Fastp
[Install Fastp](https://anaconda.org/bioconda/fastp) (I used V0.20.1)
```
conda install bioconda::fastp
```
This will install Fastp in whatever conda environment is currently active, which will most likely be the default 'base' denoted by the '(base)' at the beginning of the bash prompt

Then run Fastp on each pair using the '-i' argument for paired files
```
fastp -i R4c_1.fq.gz -I R4c_2.fq.gz -o FastpTrimmedRNAReads/R4c_1_trimmed.fq.gz -O FastpTrimmedRNAReads/R4c_2_trimmed.fq.gz
# '-o' is followed by the name of the output file corresponding to R4c_1.fq.gz
# '-O' is followed by the name of the output file corresponding to R4c_2.fq.gz
```
## rCorrector
Install rcorrector
There are multiple ways to install rcorrector. I am using the instructions on the [rcorrector github](https://github.com/mourisl/Rcorrector)
1. Clone the GitHub repo, e.g. with ```git clone https://github.com/mourisl/rcorrector.git```
2. Run ```make``` in the repo directory. During the ```make``` procedure, the script will check whether you have jellyfish2 in $PATH. If not, it will download and compile jellyfish2 from its repository.

Run rcorrector
Installation using git clone will get you the perl file you need to run rcorrector, just make sure the perl file is in the same directory as your fastp outputs.
```
perl run_rcorrector.pl
