# RNA Editing Pipeline
This was adapted from Jaydee Sereewit's pipeline [found here](https://github.com/asereewit/RNA-Editing-in-Octopus-rubescens-in-Response-to-Ocean-Acidification-Methods/tree/0.1.0)
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

### Fastp
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
### rCorrector
Install rcorrector
There are multiple ways to install rcorrector. I am using the instructions on the [rcorrector github](https://github.com/mourisl/Rcorrector)
1. Clone the GitHub repo, e.g. with ```git clone https://github.com/mourisl/rcorrector.git```
2. Run ```make``` in the repo directory. During the ```make``` procedure, the script will check whether you have jellyfish2 in $PATH. If not, it will download and compile jellyfish2 from its repository.

Run rcorrector
Installation using git clone will get you the perl file you need to run rcorrector, just make sure the perl file is in the same directory as your fastp outputs.
```
perl run_rcorrector.pl \
  -1 \ # denotes the first items in the paired end files
  R4c_1_trimmed.fq.gz,R6c_1_trimmed.fq.gz,R7c_1_trimmed.fq.gz,R8c_1_trimmed.fq.gz,R9c_1_trimmed.fq.gz,R10c_1_trimmed.fq.gz \
  -2 \ # denotes the second items in the paired end files
  R4c_2_trimmed.fq.gz,R6c_2_trimmed.fq.gz,R7c_2_trimmed.fq.gz,R8c_2_trimmed.fq.gz,R9c_2_trimmed.fq.gz,R10c_2_trimmed.fq.gz
```
### FilterUncorrectabledPEfastq.py
Now we filter the uncorrectable reads using a terrible script. You can still find it [here](https://github.com/harvardinformatics/TranscriptomeAssemblyTools/tree/master/utilities) on github.
A copy is currently located in ```/media/work/Ricky_Sequencing/RNA/Working_RNA/FastpTrimmedRNAReads``` and it is named 'FilterUncorrectabledPEfastq.py'
<details>
  <summary>If this doesn't work</summary>
  There is another version of this python script located in 
  [/media/work/Ricky_Sequencing/RNA/Working_RNA/FastpTrimmedRNAReads/unzipped_rcorrected/trinity_bowtie_results]
  
  It may be the one I actually used, I can't remember and for that I apologize. I updated the structure of line 78 from “R2.next()” to “next(R2)”
  
  _Also, it might be easier to try the script from the github, but I still don't know if the clowns that made it have updated it to work in python3. I could test it but I don't wanna._
</details>

I'm so sorry the folder structure is so terrible. You can copy it from there to anywhere else, just navigate to the folder and use the 'cp' command
```
cp FilterUncorrectabledPEfastq.py /media/work/[a reasonable folder location somewhere else]
```
The above command template should work once you figure out where you want to put stuff. I recommend keeping most of the outputs in the same folder, it will get cluttered, but putting things in separate folders can be even worse sometimes
The script itself needs to be executed in the same folder as the trimmed reads. Also, the script was written before python 3, so you'll need to run it using a python V2.7 environment. I created one in conda, and while I don't remember the exact command, it should have looked something like this
```
conda create -n python2_env python=2.7
```
'-n' is the name parameter, and python2_env is the argument you assign to that parameter
The above command will create a new conda environment with python V2.7 installed. Then to activate it you would use the following command
```
conda activate python2_env
```
Instead of seeing '(base)' at the beginning of the bash prompt like you have up until now, you should see '(python2_env)'
And before you can run this script, you will need to decompress the files. Up until now the applications have been able to execute on compressed files, but this script is not that cool :P. The following code should work as a template to decompress (AKA unzip) the files.
```
gunzip -c filename.fq.gz > filename.fq
```
<details>
  <summary>Extra</summary> 
  The command is probably supposed to be pronounced out loud as 'jee-unzip' but I just pronounce it as spelled with a hard 'g'. It does the opposite of the 'gzip' command, which compresses files.
  The '-c' parameter means that it will leave the original file alone and print the output to the command line. That's good because we want to keep the intermediate files, but bad because it'll just spit gigabytes of fastq data onto the screen for several minutes and  accomplish nothing. This is why, after we specify the filename.fq.gz to unzip, we then pipe that output into a text file named filename.fq (all we've done is dropped the '.gz' extension on the name). You could probably easily write the command to unzip all the files you want to at once and rename them accordingly, but if you do it wrong you'll create unzipped copies of every zipped file in the folder and you probably have ones you'd rather not unzip. If you want to do it all at once, ask Kirt how to format a command for that, he'll show you a simple regex tip, otherwise you'll have to run the command once for every single file, which doesn't take super long so it's not a huge deal.

</details>

Now that you've properly activated the python 2 conda environment and have unzipped all the files, you are ready to run the script. Unfortunately you will have to run separately for each pair, which is slightly extra typing work, but if you just open a bunch of terminals and do them all concurrently it might save a little time. I don't remember how long this step takes, but this script didn't actually remove any uncorrectable reads, because there were none. I just used it because it was part of the supposed due diligence Jaydee built in to the pipeline. If you can't get it to work, you can probably skip it.
Anyway here's an example command that you'll need to duplicate for each pair
```
python3 FilterUncorrectabledPEfastq.py \
-1 R4c_1_trimmed.fq \ # I've added line breaks to make things easier to read, but since I used '\' at each break it won't affect you copying and pasting the code... I think
  -2 R4c_2_trimmed.fq \
  -s R4c_trimmed_corrected_log
```

### rRNA removal
Our sequence data should still have a bunch of rRNA because the original extraction certainly didn't remove it, so that got sequenced too. Thankfully, rRNA is pretty conserved, so we can use an rRNA dataset from a database to match to ours our rRNA and remove it using an alignment tool called Bowtie2. I created a conda environment with bowtie2 using the same method I showed above for creating evironments
```
conda create -n bowtie_environment bowtie2
```
I didn't specify a version, and neither should you. My bowtie version at the time was 2.5.0, and if you can't replicate my results you can try going back to that version, but otherwise you should just let conda find the latest version by default when it creates the environment. I used the same rRNA datasets as Jaydee did, which were _Octopus vulgaris_ 18s rRNA FJ617439 and _Octopus cyanea_ 28s rRNA from the SILVA database.
Unfortunately, only the _O. vulgaris_ dataset is available on SILVA. I couldn't find the _O. cyanea_ dataset online, so I used Jaydee's files that he kept from his run, and as far as I know this isn't a problem. These files are named 'Ocyanea28srRNA.fasta' and 'Ovulgaris18srRNA.fasta'. You'll want to find these fastas and copy them from [/media/work/Ricky_Sequencing/RNA/Working_RNA/FastpTrimmedRNAReads/unzipped_rcorrected/trinity_bowtie_results] into the directory containing your trimmed and filtered read files (remember these are named something like 'R4c_1_trimmed.fq'). Once you have everything in that directory and have activated the bowtie2 environment using 'conda activate [environment name]', you are ready to start building the bowtie index with the following command.
```
bowtie2-build -f \# this is the initial command, '-f' tells it that we're using fasta format
Ovulgaris18srRNA.fasta,Ocyanea28srRNA.fasta \# these are the two rRNA datasets in fasta format, separated by a comma, no space
Ovulgaris18sOcyanea28srRNA # this is the base name which will be included in all of the index files it outputs, they will each have their own filename additions such as '.rev.1.bt2' tacked on
```
Removing rRNA using bowtie2. You'll have to run the command once for every single file, you can't even combine the single pairs. But you could just open a bunch of terminal windows and start them all at once :P
```
bowtie2 --quiet --very-sensitive-local --phred33 \# 'quiet' makes it so that it won't vomit millions of lines of text into your terminal, the other 2 are technical alignment parameters which can be found on bowtie2's documentation site
  -x Ovulgaris18sOcyanea28srRNA \# '-x' is the index call parameter, and we feed it the name we gave to all of our index files
  -U unfixrm_R4c_1_trimmed.fq \#
  --threads 15 \
  --met-file blacklist_unpaired_unfixrm_R4c_1_trimmed_bowtie2_metrics.txt \# this is the metrics file it produces and I've never checked them but might be useful for troubleshooting and recordkeeping
  --al blacklist_unpaired_aligned_unfixrm_R4c_1_trimmed.fq \# this output file will be all the reads that aligned to the rRNA index, meaning we don't want these reads because we don't want rRNA
  --un blacklist_unpaired_unaligned_unfixrm_R4c_1_trimmed.fq \# the other output file, these reads didn't align to the rRNA index, which is good and we will continue with this file in the next steps 
```
# Transcriptome Assembly using TrinityRNASeq
## Install Trinity
Trinity is already installed on the genomics computer, but here are the instructions for a fresh install
```
sudo apt update
sudo apt upgrade
sudo apt install -y trinityrnaseq
```
Guides for trinity installation include many dependencies such as bowtie2 and samtools, and those are all currently installed on the genomics computer as well. I don't have the trinity version number which I used (bad note taking), but the current version is up to date as of Jan 21, 2024.
## Running Trinity
The below command is a template to work off of. Make sure to delimit the list of input files by either commas or spaces, e.g. 'leftfile1,leftfile2,leftfile3' or 'leftfile1 leftfile2 leftfile3'. Definitely don't waste weeks trying to troubleshoot it while using both commas AND spaces...
```
sudo Trinity \
  --seqtype fq \# denotes fastq as input sequence
  --left \# first list of input files grouped by orientation
  blacklist_unpaired_unaligned-unfixrm_R4c_1_trimmed.fq \# this list of files is delimited using spaces
  ...list_of_files_separated_by_spaces.fqs \
  --left \# second list of input files grouped by orientation and in same order as first list
   blacklist_unpaired_unaligned-unfixrm_R4c_2_trimmed.fq \
  second_list_of_files_separated_by_spaces.fqs \
  --max_memory 200G \
  --CPU 15 \
  --output $(pwd)/trinity_out_dir > run.log 2>&1 # I believe this creates the output directory, and it also creates the run.log file, but I never bothered to figure out why the run.log gets piped to '2>&1'
```
# Transcriptome analysis
## Busco
Install busco. I used V5.4.2 but the installation command has been lost to time.
```
busco -i trinity_out_dir.Trinity.fasta -l mollusca_odb10 -o rubescens_tran_busco -m tran -c 18 -f
```
# ORFfinder
Get ORFfinder [here](https://ftp.ncbi.nlm.nih.gov/genomes/TOOLS/ORFfinder/linux-i64/)
The command may fail to find any ORFs and require troubleshooting. I generated a reverse complement but that didn't help. The main thing is to make sure your transcriptome assembled in some sensible fashion.
This command will ignore ORFs nested in larger ORFs
```
~/ORFfinder -in trinity_out_dir.Trinity.fasta -out rubescens_transcriptome_ORF_ignore_nested.fasta -n true -outfmt 1 > rubescens_transcriptome_ORFfinder_log.txt
```
# Swissprot
This next section blasts the ORFs against the swissprot database. **It should be noted that these results were later determined to be very unreliable. If the swissprot annotation says it's an ATPase, it might be, it might not be.**
## Installing blast
This should already be installed by now, but here's the instructions as I used them
```
https://ftp.ncbi.nlm.nih.gov/blast/executables/LATEST/ncbi-blast-2.13.0+-x64-arm-linux.tar.gz
tar zxvpf ncbi-blast-2.13.0+-x64-arm-linux.tar.gz
export PATH=$PATH:$HOME/ncbi-blast-2.13.0+
```
Decompress Swissprot
```
perl ~/ncbi-blast-2.13.0+/bin/update_blastdb.pl --decompress swissprot
```
## Running blast
```blastx -query rubescens_transcriptome_ORF_ignore_nested.fasta -out rubescens_transcriptome_ORF_swissprot_blastx_1bestalignment.txt -db ~/ncbi-blast-2.13.0+/bin/blastdb/swissprot -evalue 1e-6 -max_target_seqs 1 -subject_besthit -outfmt "6 qaccver saccver pident bitscore evalue" -num_threads 16
```
## Running swissprot analysis
Download this python script [here](https://github.com/asereewit/RNA-Editing-in-Octopus-rubescens-in-Response-to-Ocean-Acidification-Methods/blob/0.1.0/swissprot_blastx_results_analysis.py)
This script wasn't tested after upload, so you'll need to change line 9 of this text file using nano. Change 'row[1]' to 'row[0]' because somebody mixed up python and R indexing.
## Pull swissprot blasted ORFs from original transcriptome
These will be collated into a file titled 'swissprotORF.fasta
But first install seqtk. I used V1.3-r106, again the installation method is lost to time, so you'll have to google it.
```
seqtk subseq rubescens_transcriptome_ORF_ignore_nested.fasta swissprot_ORF_seqid.lst > swissprotORF.fasta
```
## Remove duplicate sequences and linebreaks from swissprotORF.fasta
This step wasn't included in the original pipeline, but I have a feeling it must have been done at some point because it's impossible to proceed with duplicates in the file. I also have no idea why it includes duplicate sequences but maybe it stems from me following Jaydee's exact seqtk command without properly understanding it. The same applies for linebreaks. Don't know why they're there, 
Rename 'swissprotORF.fasta' to 'swissprotORF_with_dupes.fasta'
```
seqkit rmdup -s < swissprotORF_with_dupes.fasta > swissprotORF.fasta
```
Rename 'swissprotORF.fasta' to 'swissprotORF_withlinebreaks.fasta'
```
seqtk seq -l 0 swissprotORF_withlinebreaks.fasta > swissprotORF.fasta
```
## Generate stats on swissprotORF.fasta using TrinityStats
This script is included in the 'util' folder in the trinity installation folder.
```
perl ~/trinityrnaseq-v2.14.0/util/TrinityStats.pl swissprotORF.fasta > swissprotORFstats.txt
```
# Prep index of swissprotORF.fasta for the later steps
```
samtools faidx swissprotORF.fasta # makes index for editing sites detection script
bowtie2-build -f swissprotORF.fasta swissprotORF # builds bowtie index for the next mapping steps
```
# DNA QC
Trimming the raw DNA reads

We will used '--paired' to let us feed it consecutive pairs. Note these pairs are consecutive, not mirrored. So we feed it one pair at a time. This is different than feeding it all reverse followed by all forward reads, don't do that.
```
trim_galore --fastqc --paired \
  D4_CKDN220050988-1A_H7NK3DSX5_L1_1.fq.gz \
  D4_CKDN220050988-1A_H7NK3DSX5_L1_2.fq.gz \
  D6_CKDN220050989-1A_H7NK3DSX5_L1_1.fq.gz \
  D6_CKDN220050989-1A_H7NK3DSX5_L1_2.fq.gz \
  D6_CKDN220050989-1A_H7NKKDSX5_L1_1.fq.gz \
  D6_CKDN220050989-1A_H7NKKDSX5_L1_2.fq.gz \
  D7_CKDN220050990-1A_H7MMJDSX5_L1_1.fq.gz \
  D7_CKDN220050990-1A_H7MMJDSX5_L1_2.fq.gz \
  D8_CKDN220050991-1A_H7MMJDSX5_L1_1.fq.gz \
  D8_CKDN220050991-1A_H7MMJDSX5_L1_2.fq.gz \
  D9b_CKDN220056888-1A_HJN2JDSX5_L2_1.fq.gz \
  D9b_CKDN220056888-1A_HJN2JDSX5_L2_2.fq.gz \
  D9b_CKDN220056888-1A_HM2NKDSX5_L3_1.fq.gz \
  D9b_CKDN220056888-1A_HM2NKDSX5_L3_2.fq.gz \
  D10b_CKDN220056889-1A_HJN2JDSX5_L2_1.fq.gz \
  D10b_CKDN220056889-1A_HJN2JDSX5_L2_2.fq.gz \
  D10b_CKDN220056889-1A_HM2NKDSX5_L1_1.fq.gz \
  D10b_CKDN220056889-1A_HM2NKDSX5_L1_2.fq.gz
```
## Concatenate all DNA reads into two files
Take all the 'forward' reads and 'reverse' reads and concatenate them into two respective files. Leave them zipped
```
cat \
  D10b_CKDN220056889-1A_HJN2JDSX5_L2_1_val_1.fq.gz \
  D10b_CKDN220056889-1A_HM2NKDSX5_L1_1_val_1.fq.gz \
  D4_CKDN220050988-1A_H7NK3DSX5_L1_1_val_1.fq.gz \
  D6_CKDN220050989-1A_H7NK3DSX5_L1_1_val_1.fq.gz \
  D6_CKDN220050989-1A_H7NKKDSX5_L1_1_val_1.fq.gz \
  D7_CKDN220050990-1A_H7MMJDSX5_L1_1_val_1.fq.gz \
  D8_CKDN220050991-1A_H7MMJDSX5_L1_1_val_1.fq.gz \
  D9b_CKDN220056888-1A_HJN2JDSX5_L2_1_val_1.fq.gz \
  D9b_CKDN220056888-1A_HM2NKDSX5_L3_1_val_1.fq.gz \
  > /media/data/rwright/pooled_DNA_reads/pooled_trimmed_reads_1.fq.gz
```
```
cat \
  D10b_CKDN220056889-1A_HJN2JDSX5_L2_2_val_2.fq.gz \
  D10b_CKDN220056889-1A_HM2NKDSX5_L1_2_val_2.fq.gz \
  D4_CKDN220050988-1A_H7NK3DSX5_L1_2_val_2.fq.gz \
  D6_CKDN220050989-1A_H7NK3DSX5_L1_2_val_2.fq.gz \
  D6_CKDN220050989-1A_H7NKKDSX5_L1_2_val_2.fq.gz \
  D7_CKDN220050990-1A_H7MMJDSX5_L1_2_val_2.fq.gz \
  D8_CKDN220050991-1A_H7MMJDSX5_L1_2_val_2.fq.gz \
  D9b_CKDN220056888-1A_HJN2JDSX5_L2_2_val_2.fq.gz \
  D9b_CKDN220056888-1A_HM2NKDSX5_L3_2_val_2.fq.gz \
  > /media/data/rwright/pooled_DNA_reads/pooled_trimmed_reads_2.fq.gz
```
# Concatenate RNA read pairs into 1 file each
The final python script only accepts 6 files and concatenating the paired reads doesn't matter since their paired nature mostly helped for transcriptome assembly, not editing detection
```
cat \
  blacklist_unpaired_unaligned_unfixrm_R4c_1_trimmed.fq \
  blacklist_unpaired_unaligned_unfixrm_R4c_2_trimmed.fq \
  > blacklist_unpaired_unaligned_unfixrm_R4c_trimmed.fq
```
# Map all the reads to swissprotORF.fasta
## Mapping, sorting, and indexing the RNA
Map each read file to swissprotORF using the index created by bowtie2
```
bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R4c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R4c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R4c_rna_orf_alignment.bam
```
The above command maps the reads using the bowtie index, then outputs that mapping to standard out. The standard output is piped to samtools using the '|' operator. Samtools puts the output into a bam file for use in the final step.
The bam file outputs then get sorted using samtools' sort function
<details>
  <summary>The rest of the commands</summary>
  ```
  bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R6c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R6c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R6c_rna_orf_alignment.bam
  ```

  ```
  bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R7c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R7c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R7c_rna_orf_alignment.bam
  ```
  ```
  bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R8c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R8c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R8c_rna_orf_alignment.bam
  ```
  ```
  bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R9c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R9c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R9c_rna_orf_alignment.bam
  ```
  ```
  bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t \
  --met-file R4c_rna_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U blacklist_unpaired_unaligned_unfixrm_R10c_1_trimmed.fq,blacklist_unpaired_unaligned_unfixrm_R10c_2_trimmed.fq \
  | samtools view -b \
    -F 260
    --threads 15 > \
    R10c_rna_orf_alignment.bam
  ```
</details>
```
samtools \
  sort \
  -@ 15 \# this specifies number of threads. I think '-t' was already used for another parameter so they used an '@' symbol instead :p
  -o R4c_rna_orf_alignment_sorted.bam \# this is the name of the output file
  R4c_rna_orf_alignment.bam # this is the input file. I don't know why the command was structured this way but just keep this in mind and don't get confused
```
Again you'll have to repeat the above command for each bam file to generate sorted bams for each.
Lastly you'll index the bam files using samtools again, generating bam index files '.bai'
```
samtools \
  index \
  -b \
  -@ 15 \
  R4c_rna_orf_alignment_sorted.bam
```
Repeat for each file.
## Mapping, sorting, and indexing the DNA
I believe this command works for both DNA files at once. I haven't run it in quite a while but it should work.
```
bowtie2 \
  --local \
  --threads 15 \
  --quiet \
  -t --met-file pooled_gDNA_orf_alignment_bowtie2_metrics.txt \
  -q \
  -x swissprotORF \
  -U pooled_trimmed_reads_1.fq,pooled_trimmed_reads_2.fq \
  | samtools view -b \
    -h \
    -F 260 \
    -q 10 \
    -@ 15 \
    -o octo_dna_orf_alignment.bam
```
Now sort them
```
samtools \
  sort \
  -@ 15 \
  -o octo_dna_orf_alignment_sorted.bam \
  octo_dna_orf_alignment.bam
```
```
samtools \
  index \
  -b \
  -@ 15 \
  octo_dna_orf_alignment_sorted.bam
```

## Count mapped reads
RNA
```
samtools view -c -F 260 R4c_1_2_alignment.bam
samtools view -c -F 260 R6c_1_2_alignment.bam
samtools view -c -F 260 R7c_1_2_alignment.bam
samtools view -c -F 260 R8c_1_2_alignment.bam
samtools view -c -F 260 R9c_1_2_alignment.bam
samtools view -c -F 260 R10c_1_2_alignment.bam
```
DNA
```
samtools view -c -F 260 octo_dna_orf_alignment.bam
```
# That should be it
Run Jaydee's python script 'editing_sites_screening.py' located in /media/work/Ricky_Sequencing/trinity_working_dir
You'll need all the bam files and bam index files in the same folder. Hopefully everything will be in the same folder from start to finish.
I'm rerunning "editing_sites_screening.py" in /media/data/rwright/final_screening
