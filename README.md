# RNA-Seq Feb 25, 2021 Workshop

Hackmd link can be found here: https://hackmd.io/O8iR_G7OSmG226E9QlAsGA?view

## Table of Contents

[TOC]

## Session-1

### 1. Login to HTC
a. Connect to Pulse Secure. Instructions can be found [here](https://www.technology.pitt.edu/help-desk/how-to-documents/pittnet-vpn-pulse-secure-connect-pulse-secure-client).
b. Open Terminal (Mac)/ [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) (Windows)
c. Use your pitt id to login:

```
$ ssh abc46@htc.crc.pitt.edu
Warning: the ECDSA host key for 'htc.crc.pitt.edu' differs from the key for the IP address '136.142.217.118'
Offending key for IP in /Users/Anish/.ssh/known_hosts:9
Matching host key in /Users/Anish/.ssh/known_hosts:12
Are you sure you want to continue connecting (yes/no)? yes
abc46@htc.crc.pitt.edu's password: 
Last login: Thu Oct 22 11:19:07 2020 from sremote-10-195-22-76.vpn.pitt.edu
```

If you need more details on HTC login, you can use [this](https://hackmd.io/0M53ndGMTh2Y-8wT8JwRpQ#1-HTC-login) handout from the first part of the workshop.
### 2. Transfer fastq files to your login node
a.	Go to your login node and then create a folder called rna_wkshop for analysis. Under the folder create the following sub-folders as well.
```
abc46: ~$ cd
abc46: ~$ mkdir rna_wkshop
abc46: ~$ cd rna_wkshop/
abc46: rna_wkshop$ mkdir Counts Data Jobs Mapping QC
abc46: rna_wkshop$ ls
Counts  Data  Jobs  Mapping  QC
```
b.	Move to the Data folder and transfer the raw fastq files using cp -r command.

```
abc46: rna_wkshop$ cd Data/
abc46: Data$ cp -r /bgfs/genomics/workshops/2021s/RNASeq_data_analysis/Data/Raw/ .
abc46: Data$ ls
Raw
abc46: Data$ cd Raw/
abc46: Raw$ ls
1_S4_DMSO_chr21_1.fastq.gz      5_S8_Al-10-49_chr21_1.fastq.gz
1_S4_DMSO_chr21_2.fastq.gz      5_S8_Al-10-49_chr21_2.fastq.gz
2_S5_DMSO_chr21_1.fastq.gz      6_S9_Al-10-49_chr21_1.fastq.gz
2_S5_DMSO_chr21_2.fastq.gz      6_S9_Al-10-49_chr21_2.fastq.gz
3_S6_DMSO_chr21_1.fastq.gz      md5sum.txt
3_S6_DMSO_chr21_2.fastq.gz      practice
4_S7_Al-10-49_chr21_1.fastq.gz  test.txt
4_S7_Al-10-49_chr21_2.fastq.gz
```
c.	Validate your transfer using md5sum.
```
abc46: Raw$ md5sum -c md5sum.txt 
1_S4_DMSO_chr21_1.fastq.gz: OK
1_S4_DMSO_chr21_2.fastq.gz: OK
2_S5_DMSO_chr21_1.fastq.gz: OK
2_S5_DMSO_chr21_2.fastq.gz: OK
3_S6_DMSO_chr21_1.fastq.gz: OK
3_S6_DMSO_chr21_2.fastq.gz: OK
4_S7_Al-10-49_chr21_1.fastq.gz: OK
4_S7_Al-10-49_chr21_2.fastq.gz: OK
5_S8_Al-10-49_chr21_1.fastq.gz: OK
5_S8_Al-10-49_chr21_2.fastq.gz: OK
6_S9_Al-10-49_chr21_1.fastq.gz: OK
6_S9_Al-10-49_chr21_2.fastq.gz: OK
```
### 3. Copy analysis job scripts.
a.	Move to your Jobs folder under rna_wkshop. 
b.	Instead of cp -r, we can also use rsync to transfer files. It takes care of md5sum for us as well. 
```
abc46: Raw$ cd ~/rna_wkshop/Jobs/
abc46: Jobs$ ls
abc46: Jobs$ rsync -avh /bgfs/genomics/workshops/2021s/RNASeq_data_analysis/Jobs/ .
sending incremental file list
./
Cutadapt/
Cutadapt/cutadapt.job
Cutadapt/OUT/
FastQC/
FastQC/fastqc.job
FastQC/OUT/
HT-Seq/
HT-Seq/htseq.job
HT-Seq/OUT/
Hisat2/
Hisat2/hisat2.job
Hisat2/OUT/
Salmon/
Salmon/salmon.job
Salmon/OUT/
featureCounts/
featureCounts/fCounts.job
featureCounts/OUT/

sent 5.36K bytes  received 185 bytes  3.69K bytes/sec
total size is 4.59K  speedup is 0.83
abc46: Jobs$ ls
Cutadapt  FastQC  featureCounts  Hisat2  HT-Seq  Salmon
```




### 4. Run FastQC
a.	Move to FastQC folder under Jobs and open fastqc.job using vim editor.
```
abc46: Jobs$ cd FastQC/
abc46: FastQC$ ls
fastqc.job  OUT
abc46: FastQC$ vim fastqc.job 
```

fastqc.job
```gherkin=
#!/bin/bash
#SBATCH -J fastqc
#SBATCH -c 1
#SBATCH -t 1:00:00
#SBATCH -o OUT/fastqc-%A_%a.out
#SBATCH --array=1-6 # job array index
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=<email>
###########

####### set-up fastqc
module load fastqc/0.11.7

set -x

################

fastq=
out=

#################


mkdir -p $out

fastqc -o $out $fastq/${SLURM_ARRAY_TASK_ID}*_1.fastq.gz
fastqc -o $out $fastq/${SLURM_ARRAY_TASK_ID}*_2.fastq.gz
```
:::info
1. When you open Vim for the first time, you will be in command mode. 
2. To make changes to the document using Vim, you need to go *insert* mode, by hitting the key **'i'**
:::

b.	Modify the following lines in the script
* *line 8:* Add your email id in the script (this should be done for all scripts) 
* *lines 18&19:* Modify the fastq and out variables, to include the correct paths.
```
fastq=~/rna_wkshop/Data/Raw/
out=~/rna_wkshop/QC/FastQC/Raw
```
:::info
To save the changes you made and quit Vim, do the following:
1. Hit <Esc> key, to go to command mode in Vim
2. Type :wq and then hit <enter>
:::

More command options for Vim are available [here](https://hackmd.io/0M53ndGMTh2Y-8wT8JwRpQ#5-Unix-Tutorial-Vim-Editor) from the first session of the RNA-Seq workshop.

c.	Run the script using sbatch command and you can check the status by using squeue command.
```
abc46: FastQC$ ls
fastqc.job  OUT
abc46: FastQC$ sbatch fastqc.job 
Submitted batch job 1197691
abc46: FastQC$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197691_[1-6]       htc   fastqc    abc46 PD       0:00      1 (None)
abc46: FastQC$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
         1197691_1       htc   fastqc    abc46  R       0:00      1 htc-n0
         1197691_2       htc   fastqc    abc46  R       0:00      1 htc-n0
         1197691_3       htc   fastqc    abc46  R       0:00      1 htc-n0
         1197691_4       htc   fastqc    abc46  R       0:00      1 htc-n1
         1197691_5       htc   fastqc    abc46  R       0:00      1 htc-n1
         1197691_6       htc   fastqc    abc46  R       0:00      1 htc-n1
```
### 5. Run MultiQC
a.	Move to your FastQC results folder and load your MultiQC module on HTC.
```
abc46: FastQC$ cd ~/rna_wkshop/QC/FastQC/Raw/
abc46: Raw$ ls
1_S4_DMSO_chr21_1_fastqc.html  4_S7_Al-10-49_chr21_1_fastqc.html
1_S4_DMSO_chr21_1_fastqc.zip   4_S7_Al-10-49_chr21_1_fastqc.zip
1_S4_DMSO_chr21_2_fastqc.html  4_S7_Al-10-49_chr21_2_fastqc.html
1_S4_DMSO_chr21_2_fastqc.zip   4_S7_Al-10-49_chr21_2_fastqc.zip
2_S5_DMSO_chr21_1_fastqc.html  5_S8_Al-10-49_chr21_1_fastqc.html
2_S5_DMSO_chr21_1_fastqc.zip   5_S8_Al-10-49_chr21_1_fastqc.zip
2_S5_DMSO_chr21_2_fastqc.html  5_S8_Al-10-49_chr21_2_fastqc.html
2_S5_DMSO_chr21_2_fastqc.zip   5_S8_Al-10-49_chr21_2_fastqc.zip
3_S6_DMSO_chr21_1_fastqc.html  6_S9_Al-10-49_chr21_1_fastqc.html
3_S6_DMSO_chr21_1_fastqc.zip   6_S9_Al-10-49_chr21_1_fastqc.zip
3_S6_DMSO_chr21_2_fastqc.html  6_S9_Al-10-49_chr21_2_fastqc.html
3_S6_DMSO_chr21_2_fastqc.zip   6_S9_Al-10-49_chr21_2_fastqc.zip
```
b.	Run multiqc on the above FastQC files to summarize the results. But, first we load the module.
```
abc46: Raw$ module spider multiqc

---------------------------------------------------------------------------
  multiqc:
---------------------------------------------------------------------------
    Description:
      Aggregate results from bioinformatics analyses across many samples
      into a single report.

     Versions:
        multiqc/1.7
        multiqc/1.8

---------------------------------------------------------------------------
  For detailed information about a specific "multiqc" module (including how to load the modules) use the module's full name.
  For example:

     $ module spider multiqc/1.8
---------------------------------------------------------------------------

abc46: Raw$ module load multiqc/1.8
```
c. Run the following multiqc command to summarize the results
```
abc46: Raw$ module load multiqc/1.8
abc46: Raw$ multiqc *.zip
[WARNING]         multiqc : MultiQC Version v1.9 now available!
[INFO   ]         multiqc : This is MultiQC v1.8
[INFO   ]         multiqc : Template    : default
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/1_S4_DMSO_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/1_S4_DMSO_chr21_2_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/2_S5_DMSO_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/2_S5_DMSO_chr21_2_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/3_S6_DMSO_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/3_S6_DMSO_chr21_2_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/4_S7_Al-10-49_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/4_S7_Al-10-49_chr21_2_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/5_S8_Al-10-49_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/5_S8_Al-10-49_chr21_2_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/6_S9_Al-10-49_chr21_1_fastqc.zip
[INFO   ]         multiqc : Searching   : /ihome/uchandran/abc46/rna_wkshop/QC/FastQC/Raw/6_S9_Al-10-49_chr21_2_fastqc.zip
[INFO   ]          fastqc : Found 12 reports
[INFO   ]         multiqc : Compressing plot data
[INFO   ]         multiqc : Report      : multiqc_report.html
[INFO   ]         multiqc : Data        : multiqc_data
[INFO   ]         multiqc : MultiQC complete
abc46: Raw$ 
abc46: Raw$ ls
1_S4_DMSO_chr21_1_fastqc.html      4_S7_Al-10-49_chr21_1_fastqc.zip
1_S4_DMSO_chr21_1_fastqc.zip       4_S7_Al-10-49_chr21_2_fastqc.html
1_S4_DMSO_chr21_2_fastqc.html      4_S7_Al-10-49_chr21_2_fastqc.zip
1_S4_DMSO_chr21_2_fastqc.zip       5_S8_Al-10-49_chr21_1_fastqc.html
2_S5_DMSO_chr21_1_fastqc.html      5_S8_Al-10-49_chr21_1_fastqc.zip
2_S5_DMSO_chr21_1_fastqc.zip       5_S8_Al-10-49_chr21_2_fastqc.html
2_S5_DMSO_chr21_2_fastqc.html      5_S8_Al-10-49_chr21_2_fastqc.zip
2_S5_DMSO_chr21_2_fastqc.zip       6_S9_Al-10-49_chr21_1_fastqc.html
3_S6_DMSO_chr21_1_fastqc.html      6_S9_Al-10-49_chr21_1_fastqc.zip
3_S6_DMSO_chr21_1_fastqc.zip       6_S9_Al-10-49_chr21_2_fastqc.html
3_S6_DMSO_chr21_2_fastqc.html      6_S9_Al-10-49_chr21_2_fastqc.zip
3_S6_DMSO_chr21_2_fastqc.zip       multiqc_data
4_S7_Al-10-49_chr21_1_fastqc.html  multiqc_report.html
```
### 6. FileZilla

Next, we will try to view the html file results using FileZilla. If you don't have it installed already, please use this [link](https://filezilla-project.org/download.php?type=client) to download.

1. Please make sure your sremote is active.
2. Login using your Pitt credentials.
![](https://i.imgur.com/5t5J7B4.png)

## Session-2

### 1. Run Cutadapt
a.	Move to Cutadapt folder under your Jobs folder.
```
abc46: Raw$ cd ~/rna_wkshop/Jobs/Cutadapt/
abc46: Cutadapt$ ls
cutadapt.job  OUT
abc46: Cutadapt$ vim cutadapt.job
```
cutadapt.job
```gherkin=
#! /bin/bash
#SBATCH -N 1
#SBATCH -J cutadapt
#SBATCH -c 2
#SBATCH -t 1:00:00
#SBATCH -o OUT/cutadapt-%A_%a.out
#SBATCH --array=1-6 # job array index 
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=<email>
########################################

## Cutadapt set-up
module load cutadapt/1.17

set -x
#########################

fastq=
out=

##########################


mkdir -p $out

### Cutadapt TruSeq trim

sample=$fastq/${SLURM_ARRAY_TASK_ID}*_1.fastq.gz
file_base=`basename $sample`


cutadapt -m 25 -q 15 \
   -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
   -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
   -o $out/${file_base%_1.fastq.gz}_1.cutadapt.fastq.gz \
   -p $out/${file_base%_1.fastq.gz}_2.cutadapt.fastq.gz \
   $fastq/${file_base} $fastq/${file_base%_1.fastq.gz}_2.fastq.gz
```
b.	Modify the following lines in the script
* *line9:* Add your email id in the script (this should be done for all scripts) 
* *lines 18&19:* Modify the fastq and out variables, to include the correct paths.
```
fastq=~/rna_wkshop/Data/Raw
out=~/rna_wkshop/Data/Cutadapt
```

c.	Run script using sbatch command.
```
abc46: Cutadapt$ sbatch cutadapt.job 
Submitted batch job 1197703
abc46: Cutadapt$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197703_[1-6]       htc cutadapt    abc46 PD       0:00      1 (None)
```
d.	You can re-run FastQC on the trimmed fastq files to check for its quality.

### 2. Run Hisat2
a.	Move to Hisat2 folder under your Jobs folder.

```
abc46: Cutadapt$ cd ~/rna_wkshop/Jobs/Hisat2/
abc46: Hisat2$ ls
hisat2.job  OUT
abc46: Hisat2$ vim hisat2.job
```
hisat2.job
```gherkin=
#! /bin/bash
#SBATCH -N 1
#SBATCH -J HISAT2
#SBATCH -t 1:00:00
#SBATCH -c 3
#SBATCH -o OUT/hisat2-%A_%a.out
#SBATCH --array=1-6 # job array index 
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<email>
#############################


## HISAT2 set-up
module load gcc/8.2.0
module load hisat2/2.1.0
module load samtools/1.9

set -x

################################

trimfastq=
out=

ref=/bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/GRCh38_index_chr21/GRCh38_index_chr21

####################################

mkdir -p $out

# take input fastq file
fastqfile=$trimfastq/${SLURM_ARRAY_TASK_ID}*_1.cutadapt.fastq.gz
# obtain only the file name and exclude the full path
filename=`basename $fastqfile`
# get sample name
sample=${filename%_1.cutadapt.fastq.gz}

hisat2 -x $ref \
        -S $out/${sample}.sam \
        -p 3 \
        --dta \
        -1 $fastqfile \
        -2 ${fastqfile%_1.cutadapt.fastq.gz}_2.cutadapt.fastq.gz

samtools view -@ 3 -h -o $out/${sample}.bam $out/${sample}.sam
```
b.	Modify the following lines in the script
* *line 9:* Add your email id in the script (this should be done for all scripts) 
* *lines 22&23:* Modify the trimfastq and out variables, to include the correct paths.
```
trimfastq=~/rna_wkshop/Data/Cutadapt
out=~/rna_wkshop/Mapping/Hisat2
```
c.	Run the script using sbatch command.
```
abc46: Hisat2$ sbatch hisat2.job 
Submitted batch job 1197720
abc46: Hisat2$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197720_[1-6]       htc   HISAT2    abc46 PD       0:00      1 (Priority)
abc46: Hisat2$ 
```
d.	Go to the out directory and search for overall alignment rate for your samples.
```
abc46: Hisat2$ cd OUT/
abc46: OUT$ grep "overall alignment rate" *.out
hisat2-1197720_1.out:99.09% overall alignment rate
hisat2-1197720_2.out:99.36% overall alignment rate
hisat2-1197720_3.out:99.40% overall alignment rate
hisat2-1197720_4.out:99.43% overall alignment rate
hisat2-1197720_5.out:99.46% overall alignment rate
hisat2-1197720_6.out:99.42% overall alignment rate
```

### 3. Working with BAM/SAM files using Samtools.
a.	Move to your Hisat2 results folder. This folder has the BAM/SAM files for you to work with.
```
abc46: OUT$ cd ~/rna_wkshop/Mapping/Hisat2/
abc46: Hisat2$ ls -lh
total 9.4G
-rw-r--r-- 1 abc46 uchandran  81M Oct 23 13:31 1_S4_DMSO_chr21.bam
-rw-r--r-- 1 abc46 uchandran 549M Oct 23 13:31 1_S4_DMSO_chr21.sam
-rw-r--r-- 1 abc46 uchandran 132M Oct 23 13:31 2_S5_DMSO_chr21.bam
-rw-r--r-- 1 abc46 uchandran 1.1G Oct 23 13:31 2_S5_DMSO_chr21.sam
-rw-r--r-- 1 abc46 uchandran 143M Oct 23 13:31 3_S6_DMSO_chr21.bam
-rw-r--r-- 1 abc46 uchandran 1.2G Oct 23 13:31 3_S6_DMSO_chr21.sam
-rw-r--r-- 1 abc46 uchandran 147M Oct 23 13:32 4_S7_Al-10-49_chr21.bam
-rw-r--r-- 1 abc46 uchandran 1.3G Oct 23 13:31 4_S7_Al-10-49_chr21.sam
-rw-r--r-- 1 abc46 uchandran 139M Oct 23 13:31 5_S8_Al-10-49_chr21.bam
-rw-r--r-- 1 abc46 uchandran 1.2G Oct 23 13:31 5_S8_Al-10-49_chr21.sam
-rw-r--r-- 1 abc46 uchandran 142M Oct 23 13:31 6_S9_Al-10-49_chr21.bam
-rw-r--r-- 1 abc46 uchandran 1.2G Oct 23 13:31 6_S9_Al-10-49_chr21.sam
```
b.	Load samtools module using the following command
```
abc46: Hisat2$ module load gcc/8.2.0 samtools/1.9
```
c.	You can display the header of the BAM file using the following command
As you can see above, this BAM file is unsorted.

```
abc46: Hisat2$ samtools view -H 1_S4_DMSO_chr21.bam 
@HD     VN:1.0  SO:unsorted
@SQ     SN:21   LN:46709983
@PG     ID:hisat2       PN:hisat2       VN:2.1.0        CL:"/ihome/crc/install/hisat2/hisat2-2.1.0/hisat2-align-s --wrapper basic-0 -x /bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/GRCh38_index_chr21/GRCh38_index_chr21 -S /ihome/uchandran/abc46/rna_wkshop/Mapping/Hisat2/1_S4_DMSO_chr21.sam -p 3 --dta -1 /tmp/6191.inpipe1 -2 /tmp/6191.inpipe2"
```
d.	To sort the BAM by coordinate and read names you can use the following commands respectively.
**Coordinate**
```
abc46: Hisat2$ samtools sort 1_S4_DMSO_chr21.bam -o 1_S4_DMSO_chr21.sorted.bam
```
**Query name**
```
abc46: Hisat2$ samtools sort 1_S4_DMSO_chr21.bam -n -o 1_S4_DMSO_chr21.sorted.query.bam
```
e.	You can check if the sorting worked, by viewing the header of these newly generated BAM files.
**Coordinate**
```
abc46: Hisat2$ samtools view -H 1_S4_DMSO_chr21.sorted.bam 
@HD     VN:1.0  SO:coordinate
@SQ     SN:21   LN:46709983
@PG     ID:hisat2       PN:hisat2       VN:2.1.0        CL:"/ihome/crc/install/hisat2/hisat2-2.1.0/hisat2-align-s --wrapper basic-0 -x /bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/GRCh38_index_chr21/GRCh38_index_chr21 -S /ihome/uchandran/abc46/rna_wkshop/Mapping/Hisat2/1_S4_DMSO_chr21.sam -p 3 --dta -1 /tmp/6191.inpipe1 -2 /tmp/6191.inpipe2"
```
**Query name**
```
abc46: Hisat2$ samtools view -H 1_S4_DMSO_chr21.sorted.query.bam 
@HD     VN:1.0  SO:queryname
@SQ     SN:21   LN:46709983
@PG     ID:hisat2       PN:hisat2       VN:2.1.0        CL:"/ihome/crc/install/hisat2/hisat2-2.1.0/hisat2-align-s --wrapper basic-0 -x /bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/GRCh38_index_chr21/GRCh38_index_chr21 -S /ihome/uchandran/abc46/rna_wkshop/Mapping/Hisat2/1_S4_DMSO_chr21.sam -p 3 --dta -1 /tmp/6191.inpipe1 -2 /tmp/6191.inpipe2"
```

f.	We can index the BAM files using the index command in samtools. BAM files need to be indexed for some tools like IGV browser. It will generate a .bai file.
```
abc46: Hisat2$ samtools index 1_S4_DMSO_chr21.sorted.bam 
abc46: Hisat2$ ls -lh 1_S4_DMSO_chr21.*
-rw-r--r-- 1 abc46 uchandran  81M Oct 23 13:31 1_S4_DMSO_chr21.bam
-rw-r--r-- 1 abc46 uchandran 549M Oct 23 13:31 1_S4_DMSO_chr21.sam
-rw-r--r-- 1 abc46 uchandran  82M Oct 23 13:35 1_S4_DMSO_chr21.sorted.bam
-rw-r--r-- 1 abc46 uchandran  41K Oct 23 13:38 1_S4_DMSO_chr21.sorted.bam.bai
-rw-r--r-- 1 abc46 uchandran  81M Oct 23 13:36 1_S4_DMSO_chr21.sorted.query.bam
```
### 4. Run Salmon
a.	Move to Salmon folder under Jobs.
```
abc46: Hisat2$ cd ~/rna_wkshop/Jobs/Salmon/
abc46: Salmon$ ls
OUT  salmon.job
abc46: Salmon$ vim salmon.job
```
**salmon.job**
```gherkin=
#! /bin/bash
#SBATCH -N 1
#SBATCH -J SALMON
#SBATCH -o OUT/salmon-%A_%a.out
#SBATCH -t 2:00:00
#SBATCH --array=1-6 # job array index 
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<email>
#SBATCH --cpus-per-task=6



## modules set-up
module load salmon/0.13.1


set -x
#######################################


fastq=
out=


ref=/bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/Homo_sapiens.GRCh38.cdna.all.fa
transcript_ind=/bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/Salmon_Index

#######################################

mkdir -p $out

filepath=$fastq/${SLURM_ARRAY_TASK_ID}*_1.cutadapt.fastq.gz
sample=`basename $filepath`

#salmon index -t $ref -i $transcript_ind -k 21 

filebase=${sample%_1.cutadapt.fastq.gz}
salmon quant -i $transcript_ind -l A -1 $filepath \
   -2 ${filepath%_1.cutadapt.fastq.gz}_2.cutadapt.fastq.gz \
    --validateMappings \
    -o $out/${filebase}.counts
```
b.	Modify the following lines in the script
* *line 9:* Add your email id in the script (this should be done for all scripts) 
* *lines 21 & 22:* Modify the fastq and out variables, to include the correct paths.

```
fastq=~/rna_wkshop/Data/Cutadapt
out=~/rna_wkshop/Mapping/Salmon
```
c.	Run the script using sbatch.

```
abc46: Salmon$ sbatch salmon.job 
Submitted batch job 1197834
abc46: Salmon$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197834_[1-6]       htc   SALMON    abc46 PD       0:00      1 (Priority)
abc46: Salmon$ 
```

## Session-3

### 1. Run HT-Seq counts
a.	Move to HT-Seq folder under Jobs
```
abc46: Salmon$ cd ~/rna_wkshop/Jobs/HT-Seq/
abc46: HT-Seq$ ls
htseq.job  OUT
abc46: HT-Seq$ vim htseq.job 
```
htseq.job
```gherkin=
#! /bin/bash
#SBATCH -N 1
#SBATCH -t 1:00:00
#SBATCH -J htseq
#SBATCH -c 6
#SBATCH -o OUT/htseq-%A_%a.out
#SBATCH --array=1-6 # job array index 
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<email>
######################################


############ htseq set-up
module load htseq/0.11.2

set -x

###########################

BAMs=
out=

gtf=/bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/Homo_sapiens.GRCh38.97.gtf

#############################

mkdir -p $out


sample=$BAMs/${SLURM_ARRAY_TASK_ID}*_chr21.bam
file=`basename $sample`

htseq-count -f bam \
    -s no \
    -t exon \
    -m union \
    -i gene_id \
    $sample \
    $gtf > $out/${file%.bam}.counts.txt                   
```

b.	Modify the following lines in the script
* *line 9:* Add your email id in the script (this should be done for all scripts) 
* *lines 20&21:* Modify the fastq and out variables, to include the correct paths.
```
BAMs=~/rna_wkshop/Mapping/Hisat2
out=~/rna_wkshop/Counts/HT-Seq
```
c.	Run the htseq.job script using sbatch command 
```
abc46: HT-Seq$ sbatch htseq.job 
Submitted batch job 1197848
abc46: HT-Seq$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197848_[1-6]       htc    htseq    abc46 PD       0:00      1 (Priority)
abc46: HT-Seq$ 
```



### 2. Run featureCounts
a.	Move to featureCounts folder under Jobs
```
abc46: HT-Seq$ cd ~/rna_wkshop/Jobs/featureCounts/
abc46: featureCounts$ ls
fCounts.job  OUT
abc46: featureCounts$ vim fCounts.job 
```
fCounts.job
```gherkin=
#! /bin/bash
#SBATCH -N 1
#SBATCH -J featCounts
#SBATCH -o OUT/featCounts-%A_%a.out
#SBATCH -t 2:00:00
#SBATCH --array=1-6 # job array index 
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<email>
#SBATCH --cpus-per-task=6

#######################################
module load gcc/8.2.0
module load subread/1.6.2

set -x

########################################

out=
BAM=

gtf=/bgfs/genomics/workshops/2020s/RNASeq_data_analysis/Refs/Homo_sapiens.GRCh38.97.gtf

#########################################


file=$BAM/${SLURM_ARRAY_TASK_ID}*_chr21.bam
sample=`basename $file`


mkdir -p $out


featureCounts -t exon \
              -g gene_id \
              -s 0 \
              -a $gtf \
              -o $out/${sample%.bam}.txt \
              -T 2 \
              $file

```
b.	Modify the following lines in the script
* *line 8:* Add your email id in the script (this should be done for all scripts) 
* *lines 19&20:* Modify the out and BAM variables, to include the correct paths.
```
out=~/rna_wkshop/Counts/FCounts
BAM=~/rna_wkshop/Mapping/Hisat2
```
c.	Run the fCounts.job script using sbatch command 
```
abc46: featureCounts$ sbatch fCounts.job 
Submitted batch job 1197856
abc46: featureCounts$ squeue -u abc46
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1197856_[1-6]       htc featCoun    abc46 PD       0:00      1 (Priority)
```
### 3. IGV Browser

1. If you do not have IGV browser, you can download it from this [link](https://software.broadinstitute.org/software/igv/download).
2. Open Filezilla to transfer the sorted bam and bai files.
3. Open IGV browser and load you reference genome

![](https://i.imgur.com/MOie2HU.png)

