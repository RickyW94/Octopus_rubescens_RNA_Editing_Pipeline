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
  Also, it might be easier to try the script from the github, but I still don't know if the clowns that made it have updated it to work in python3. I could test it but I don't wanna.
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
Ovulgaris18sOcyanea28srRNA # this is the base name which will be included in all of the index files it outputs, they will each have their own additions such as '.rev.1.bt2' tacked on
```

