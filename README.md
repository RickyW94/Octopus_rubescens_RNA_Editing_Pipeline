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
