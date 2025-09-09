# MongoliaBirds
Code for Emily's thesis 


# README: nf-core/rnaseq Analysis 

This document will guide you through preparing data and running the [nf-core/rnaseq](https://nf-co.re/rnaseq/) pipeline to analyze RNA-seq data. There will be one run for those reads that will be mapped to Great Tit reference genome and another run for those reads that will be mapped to the other one.. Then we will filter for one2one orthologs using TOGA... [code coming soon].   

******* !!!! All the following code were examples from what I used while running on my antbird (Thamnophilus) data. Be careful to change according to your files and paths !!!! *******

---

## Table of Contents

1. [Introduction](#introduction)
2. [Set up Folders](#1-set-up-folders)
3. [Get Your Data Ready](#2-get-your-data-ready)
4. [Install Nextflow & Set Up Environment](#3-install-nextflow--set-up-environment)
5. [Download the Reference Genome](#4-download-the-reference-genome)
6. [Create the Sample Sheet](#5-create-the-sample-sheet)
7. [Check Your File Paths](#6-check-your-file-paths)
8. [Test the Pipeline](#7-test-the-pipeline)
9. [Submit a Batch Job](#8-submit-a-batch-job)
10. [Understanding Output and Warnings](#9-understanding-output-and-warnings)
11. [Common Problems and Tips](#10-common-problems-and-tips)

---

## 1. Set up Folders

```bash
# Make a folder for your project:
mkdir -p /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna

# "cd" means "change directory". Move into your new folder:
cd /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna

# Print your current folder. (You should see the path above.)
pwd
```

---

## 2. Get Your Data Ready

**Copy your FASTQ files** (the big RNA-seq files ending in `.fastq.gz`) into your working folder.
This also keeps originals untouched!

First, **find where your data lives** (for example, `/n/netscratch/.../fastq`).  
Then **copy** all files:

```bash
# Change the FILE_PATH to where your data actually is
cp /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/RNA_round1_re_demultiplex/fastq/*.fastq.gz .
```

---

## 3. Install Nextflow & Set Up Environment

We'll use **Conda** (an environment manager) so that all the software will work together.

Load python:
```bash
module load python/3.10.9-fasrc01
```

Create and activate a new environment **(copy/paste each line!)**:

```bash
conda create --name env_nf_rna nextflow -c conda-forge -y
conda activate env_nf_rna
```

**Test Nextflow:**
```bash
nextflow run nf-core/rnaseq --help
```
If you see help output, it's working

---

## 4. Download the Reference Genome

We need a **reference genome** and a **gene annotation file (GTF)**.  
Let’s use NCBI’s download tool.

First install it:

```bash
conda install -c conda-forge ncbi-datasets-cli -y
```

Now, make a reference folder and download your genome:

```bash
mkdir sakLuc_ref
cd sakLuc_ref

# Download genome, annotation, and supporting files with the NCBI tool:
datasets download genome accession GCA_013396695.1 --include gtf,gff3,rna,cds,protein,genome,seq-report

# Unzip the files you downloaded (they will be in ncbi_dataset.zip):
unzip ncbi_dataset.zip

# Look at the contents. (You should see .fna and .gtf files)
ls ncbi_dataset/data/GCA_013396695.1/
```
**You are looking for:**
- A genome file (`*.fna`)
- An annotation file (`genomic.gtf`)

**Go BACK to your main folder:**

```bash
cd ..
```

---

## 5. Create the Sample Sheet

The nf-core/rnaseq pipeline needs a “sample sheet” (a simple **CSV spreadsheet** telling it what files to use).  
**You can make this in Excel or Google Sheets and download as CSV**, or use a text editor.

**You must have these columns:**  
- `sample` (unique name for each sample, e.g. 22-394_Brain)
- `fastq_1` (R1 file name)
- `fastq_2` (R2 file name)
- `strandedness` (just use `auto` unless someone tells you otherwise)

**Example (you can copy and paste this in Excel, then Save As CSV):**

```csv
sample,fastq_1,fastq_2,strandedness
22-394_Brain,P504_KLope19066_A07v1_22-394_Brain_S49_L004_R1_001.fastq.gz,P504_KLope19066_A07v1_22-394_Brain_S49_L004_R2_001.fastq.gz,auto
22-394_Heart,P504_KLope19066_F06v1_22-394_Heart_S46_L004_R1_001.fastq.gz,P504_KLope19066_F06v1_22-394_Heart_S46_L004_R2_001.fastq.gz,auto
```

**Name this file `Nextflow_sakLuc_test.csv` and put it in your project folder.**

---

## 6. Check Your File Paths

This is important! Double-check you have the following and know their **full path** (use `pwd` to print your location):

- FASTQ files (e.g. `P504_KLope19066_...fastq.gz`)
- Reference genome:  
  `/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/GCA_013396695.1_ASM1339669v1_genomic.fna`
- Annotation file (GTF):  
  `/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/genomic.gtf`
- Sample sheet:  
  `/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/Nextflow_sakLuc_test.csv`

---

## 7. Test the Pipeline

This command will process all your samples.  
Start by pasting these lines to set up your paths.

```bash
workdir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna"
samples="Nextflow_sakLuc_test.csv"
fasta="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/GCA_013396695.1_ASM1339669v1_genomic.fna"
gtf="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/genomic.gtf"
out_dir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/output_test_1"

cd ${workdir}
```

**Dry run/test run (replace with actual run later):**
```bash
nextflow run nf-core/rnaseq \
    --input ${samples} \
    --outdir ${out_dir} \
    --gtf ${gtf} \
    --fasta ${fasta} \
    -profile singularity
```
> If you see warnings about the GTF "biotype" attribute, you can ignore them for simple analyses.

---

## 8. Submit a Batch Job

Put this into a file called `nf_core_rna_test1_sakLuc.sh`:

```bash
#!/bin/bash
#SBATCH -p edwards,shared
#SBATCH -c 16
#SBATCH -t 0-12:00
#SBATCH -o nf_core_rna_test1_sakLuc_%j.out
#SBATCH -e nf_core_rna_test1_sakLuc_%j.err
#SBATCH --mem=150000
#SBATCH --mail-type=END

workdir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna"
samples="Nextflow_sakLuc_test.csv"
fasta="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/GCA_013396695.1_ASM1339669v1_genomic.fna"
gtf="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/sakLuc_ref/ncbi_dataset/data/GCA_013396695.1/genomic.gtf"
out_dir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/nextflow_rna/output_test_1"

cd ${workdir}

nextflow run nf-core/rnaseq \
    --input ${samples} \
    --outdir ${out_dir} \
    --gtf ${gtf} \
    --fasta ${fasta} \
    -profile singularity
```

**Submit your job to the queue:**
```bash
sbatch nf_core_rna_test1_sakLuc.sh &
```

---

## 9. Understanding Output and Warnings

All output will appear in your output folder (in this example `$out_dir`).  
Look for the **MultiQC report** file; it gives an overview

**Warning about “gene_biotype” not found**  
You can safely ignore this for most species/genomes – it only affects some summary plots, not the main results.

---

## 10. Common Problems

- If you see:  
  `Missing required parameter(s): input, outdir`  
  → Double-check you gave the right sample CSV and output folder.
- If you get a “file not found” error, check your file paths with `ls some_path`.
- If your job fails with “not enough memory,” increase the `--mem` value in your script (e.g. 150000 → 200000).
---

## Additional Resources

- [nf-core/rnaseq documentation](https://nf-co.re/rnaseq/usage)

---

# Running TOGA to find one2one orthologs 


## Installation, configuring, and running test data
```bash
### trying to run toga 

https://github.com/hillerlab/TOGA

# enter python environment with my user specific alias 
py_env


# make a new python environment for toga. i had to do this to run toga cause i guess it relies on netflow (from the TOGA github insttructinos) 


conda create --name nf_toga nextflow              # Create a new environment
conda activate nf_toga                         

# change into thamnophilus directory 
cd /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus

# clone the repository
git clone https://github.com/hillerlab/TOGA.git
cd TOGA


# install necessary oython packages using pip 

python3 -m pip install -r requirements.txt --user


# configure and check that toga worked 
./configure.sh
./run_test.sh micro



# final test

wget https://hgdownload.cse.ucsc.edu/goldenpath/hg38/bigZips/hg38.2bit
wget https://hgdownload.cse.ucsc.edu/goldenpath/mm10/bigZips/mm10.2bit
# testing that toga is working





test_toga.sh

#!/bin/bash
#SBATCH -p edwards 
#SBATCH -c 48 
#SBATCH -t 0-12:00 # request 12 hours 
#SBATCH -o test_toga_%j.out # file name to write output to
#SBATCH -e test_toga_%j.err # file name to write errors to 
#SBATCH --mem=100000 # memory requested (100Gb) 
#SBATCH --mail-type=END # send me an email when my job is done running 

#py_env

# don't forget to be in nf_toga

#conda activate nf_toga

path_to_human_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/hg38.2bit"
path_to_mouse_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/mm10.2bit"
workdir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA"
path_to_nextflow_config_dir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/nextflow_config_files"

cd ${workdir}

./toga.py test_input/hg38.mm10.chr11.chain test_input/hg38.genCode27.chr11.bed \
${path_to_human_2bit} ${path_to_mouse_2bit} \
--kt --pn test -i supply/hg38.wgEncodeGencodeCompV34.isoforms.txt \
--nc ${path_to_nextflow_config_dir} --cb 3,5 --cjn 500 \
--u12 supply/hg38.U12sites.tsv --ms


# oof i could have requested WAY less resources but i wanted it to run fast. 
[kelsielopez@boslogin08 TOGA]$ seff 30343763
Job ID: 30343763
Cluster: odyssey
User/Group: kelsielopez/edwards_lab
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 48
CPU Utilized: 00:00:53
CPU Efficiency: 0.12% of 11:55:12 core-walltime
Job Wall-clock time: 00:14:54
Memory Utilized: 1.37 GB
Memory Efficiency: 1.41% of 97.66 GB (97.66 GB/node)
[kelsielopez@boslogin08 TOGA]$
# try requesting like 2 hrs and 16 Gb when you run the test. TBD how much memory we should request when we run on the actual samples


```

## Now that the test is one you have to make these files of your own 
## TOGA analysis - Step 1: Make lastz chains 
# https://github.com/hillerlab/make_lastz_chains 
# https://github.com/lastz/lastz

```bash
# before TOGA you need to prepare a chain file from the program 'makelastzchains'


#i had to make another nextflow environment... so many... lol 

module load python/3.12.8-fasrc01
conda create --name nf_lastz bioconda::nextflow python=3.12
conda activate nf_lastz


# change into scratch directory 
cd /n/netscratch/edwards_lab/Lab/kelsielopez

# i downloaded this from github and moved it to my scratch using filezilla. 

# go here, 
https://github.com/lastz/lastz

# then on the right, navigate to 'Releases'
https://github.com/lastz/lastz/releases

# under 'Generally inconsequential update.' download the Source code (tar.gz) file 


/n/netscratch/edwards_lab/Lab/kelsielopez/lastz-1.04.52

tar -xvf lastz-1.04.52.tar


cd src
make
make install

# test that it was installed

make test

# the path to this needs to be added to the .bashrc folder in your home directory

/n/netscratch/edwards_lab/Lab/kelsielopez/lastz-1.04.52/src

# it means adding a line like this to the .bashrc folder (but make sure to replace it with your path!!!) 

# change to home directory 
cd ~

# edit .bashrc folder with nano
nano .bashrc

export PATH=/n/netscratch/edwards_lab/Lab/kelsielopez/lastz-1.04.52/src:$PATH
source .bashrc


# now change to scratch and download make lastz chains
cd /n/netscratch/edwards_lab/Lab/kelsielopez

git clone https://github.com/hillerlab/make_lastz_chains.git
cd make_lastz_chains
# install python packages (just one actually for now)
pip3 install -r requirements.txt

# it says to do this ./install_dependencies.py but skip it because we are goin got use mamba to install the dependencies. this threw me the biggest issue with make lastz chains!

mamba install ucsc-chainscore
mamba install ucsc-twoBitToFa
mamba install ucsc-faToTwoBit
mamba install ucsc-axtChain
mamba install ucsc-chainAntiRepeat
mamba install ucsc-chainMergeSort
mamba install ucsc-chainSort
mamba install ucsc-chainNet
mamba install ucsc-axtToPsl
mamba install ucsc-chainFilter
mamba install ucsc-pslSortAcc
mamba install ucsc-chainCleaner

# test that they all were downloaded  by calling each program
twoBitToFa
faToTwoBit
pslSortAcc
axtChain
chainAntiRepeat
chainMergeSort
chainCleaner
chainSort
chainScore
chainNet
axtToPsl
chainFilter



# now when i had make_lastz_chains downloaded i had to edit this file becaus of some issue with it not being compatibile with the version of nextflow i was running. i got rid of the squiggly brackets "{ }"

sed -i \
  -e 's|process.memory = { 4.GB \* task.attempt }|process.memory = '\''4 GB'\''|g' \
  -e 's|process.time = {{ {self.time} \* task.attempt }}|process.time = '\''2h'\''|g' \
  /n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/parallelization/nextflow_wrapper.py

# i also had to edit this to allocate more time! 4 Gb per job and 2 hrs is not enough. 

nano /n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/parallelization/nextflow_wrapper.py

# go in and change '4' to '32' Gb and '2h' to '6h'.. then i got it to run!

# here is my script for running it! but before you run this, you have to rename your reference genomes because they have a long name that makes the program crash...
# for example, this is what the labels on the reference genome look like

(python_env1) [kelsielopez@holy8a26602 Lab]$ ls /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3
cds_from_genomic.fna					      genomic.genePred			  genomic.gtf			  rna.fna
GCF_001522545.3_Parus_major1.1_genomic_abreviated_naming.fna  genomic.genePred.bed12.bed	  genomic_renamed_toga.bed	  sequence_report.jsonl
GCF_001522545.3_Parus_major1.1_genomic.fna		      genomic.genePred.bed12.renamed.bed  genomic_renamed_toga_BED12.bed
genomic.bed						      genomic.gff			  protein.faa
_major1.1_genomic.fna | headoly8a26602 Lab]$ grep -e ">" /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/GCF_001522545.3_Parus_
>NC_031768.1 Parus major isolate Abel chromosome 1, Parus_major1.1, whole genome shotgun sequence
>NC_031769.1 Parus major isolate Abel chromosome 2, Parus_major1.1, whole genome shotgun sequence
>NC_031770.1 Parus major isolate Abel chromosome 3, Parus_major1.1, whole genome shotgun sequence
>NC_031771.1 Parus major isolate Abel chromosome 4, Parus_major1.1, whole genome shotgun sequence
>NC_031772.1 Parus major isolate Abel chromosome 4A, Parus_major1.1, whole genome shotgun sequence
>NC_031773.1 Parus major isolate Abel chromosome 1A, Parus_major1.1, whole genome shotgun sequence
>NC_031774.1 Parus major isolate Abel chromosome 5, Parus_major1.1, whole genome shotgun sequence
>NC_031775.1 Parus major isolate Abel chromosome 6, Parus_major1.1, whole genome shotgun sequence
>NC_031776.1 Parus major isolate Abel chromosome 7, Parus_major1.1, whole genome shotgun sequence
>NC_031777.1 Parus major isolate Abel chromosome 8, Parus_major1.1, whole genome shotgun sequence
(python_env1) [kelsielopez@holy8a26602 Lab]$

# all those extra comments made the program crash 

parus_major_genome="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/GCF_001522545.3_Parus_major1.1_genomic.fna"
parus_major_genome_abreviated_fasta_naming="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/GCF_001522545.3_Parus_major1.1_genomic_abreviated_naming.fna"


awk '{if($0 ~ /^>/){split($1, a, ">"); print ">"a[2]} else {print $0}}' ${parus_major_genome} > ${parus_major_genome_abreviated_fasta_naming}
# i renamed it like this. repeat for ficedula albicollis. then input the renamed fasta when you run make lastz chains 



test5_lastz_chains_mongolia_data.sh

#!/bin/bash
#SBATCH -p shared
#SBATCH -c 1
#SBATCH -t 3-00:00
#SBATCH -o test5_lastz_chains_mongolia_data_%j.out
#SBATCH -e test5_lastz_chains_mongolia_data_%j.err
#SBATCH --mem=64000
#SBATCH --mail-type=END
  
cd /n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains

target_genome_id="parMaj" # i'm going to simplify the IDs for downstream purposes
query_genome_id="ficAlb"
target_genome_sequence="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/GCF_001522545.3_Parus_major1.1_genomic_abreviated_naming.fna"
query_genome_sequence="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/ficedula_albicollis/data/GCF_000247815.1/GCF_000247815.1_FicAlb1.5_genomic_abreviated_naming.fna"

./make_chains.py ${target_genome_id} ${query_genome_id} \
${target_genome_sequence} ${query_genome_sequence} \
--project_dir test5_v2_reDownload \
--job_time_req 6h --executor slurm \
--cluster_queue shared



# when this program works, you should have a .chain.gz file 
```


## TOGA - step 2 - preparing and running with your own data  
## https://github.com/hillerlab/TOGA

```bash

# enter environment which you downloaded TOGA in

conda activate nf_toga

# lets prepare the input files for toga
# make the input bed file. use the genomic.gtf file from great tit that was downloaded from ncbi to make a bed file.

# download programs gtfToGenePred and genePredToBed
mamba install gtfToGenePred
mamba install genePredToBed

# test that they were downloaded by calling the programs 
(nf_toga) [kelsielopez@holy8a26502 kelsielopez]$ genePredToBed
genePredToBed - Convert from genePred to bed format. Does not yet handle genePredExt
usage:
   genePredToBed in.genePred out.bed
options:
   -tab - genePred fields are separated by tab instead of just white space
   -fillSpace - when tab input, fill space chars in 'name' with underscore: _
   -score=N - set score to N in bed output (default 0)
(nf_toga) [kelsielopez@

input="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/genomic"


# 1. GTF to genePred
gtfToGenePred -ignoreGroupsWithoutExons ${input}.gtf ${input}.genePred


# 2. genePred to BED12
genePredToBed ${input}.genePred ${input}.genePred.bed12.bed


# also need to rename this file to match the output from make_lastz_chains. make_lastz_chains renamed everything from the reference genomes 
# into these files parMaj_chrom_rename_table and ficAlb_chrom_rename_table. again its just something weird with the naming requirements for the programs
# make lastz chains removed the '.1' at the end of the name of the chromosomes 

python_env1) [kelsielopez@holy8a26602 make_lastz_chains]$ cd test5_v2_reDownload
(python_env1) [kelsielopez@holy8a26602 test5_v2_reDownload]$ ls
ficAlb_chrom_rename_table.tsv  parMaj_renamed_chrom.fa	 query.chrom.sizes     steps.json	   target_partitions.txt     temp_fill_chain	    temp_lastz_run
ficAlb_renamed_chrom.fa        pipeline_parameters.json  query_partitions.txt  target.2bit	   temp_chain_run	     temp_kent
parMaj_chrom_rename_table.tsv  query.2bit		 run.log	       target.chrom.sizes  temp_concat_lastz_output  temp_lastz_psl_output
(python_env1) [kelsielopez@holy8a26602 test5_v2_reDownload]$ pwd
/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test5_v2_reDownload
(python_env1) [kelsielopez@holy8a26602 test5_v2_reDownload]$ head parMaj_chrom_rename_table.tsv
NC_031768.1	NC_031768
NC_031769.1	NC_031769
NC_031770.1	NC_031770
NC_031771.1	NC_031771
NC_031772.1	NC_031772
NC_031773.1	NC_031773
NC_031774.1	NC_031774
NC_031775.1	NC_031775
NC_031776.1	NC_031776
NC_031777.1	NC_031777
(python_env1) [kelsielopez@holy8a26602 test5_v2_reDownload]$ head ficAlb_chrom_rename_table.tsv
NC_021671.1	NC_021671
NW_004775877.1	NW_004775877
NC_021672.1	NC_021672
NW_004775878.1	NW_004775878
NW_004775879.1	NW_004775879
NC_021673.1	NC_021673
NW_004775880.1	NW_004775880
NC_021674.1	NC_021674
NC_021675.1	NC_021675
NW_004775881.1	NW_004775881
(python_env1) [kelsielopez@holy8a26602 test5_v2_reDownload]$ 

# so we are renaming the bed file to match this output from lastz chains.

input="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/genomic"

awk 'BEGIN{FS=OFS="\t"} 
    NR==FNR {map[$1]=$2; next} 
    {if ($1 in map) $1=map[$1]; print}' \
    parMaj_chrom_rename_table.tsv ${input}.genePred.bed12.bed \
    > ${input}.genePred.bed12.renamed.bed



#the next input flie we need is 2bit files. this is another weird file format that is like a fasta, but binary, so i guess it is smaller.
# the program faToTwoBit was downloaded when we ran lastz chains, so you can change into that environment

conda activate nf_lastz

ficAlb_fa="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/ficAlb_renamed_chrom"
parMaj_fa="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/parMaj_renamed_chrom"


nf_lastz) [kelsielopez@holy8a26602 TOGA]$ faToTwoBit
faToTwoBit - Convert DNA from fasta to 2bit format
usage:
   faToTwoBit in.fa [in2.fa in3.fa ...] out.2bit
options:
   -long            use 64-bit offsets for index.   Allow for twoBit to contain more than 4Gb of sequence. 
                    NOT COMPATIBLE WITH OLDER CODE.
   -noMask          Ignore lower-case masking in fa file.
   -stripVersion    Strip off version number after '.' for GenBank accessions.
   -ignoreDups      Convert first sequence only if there are duplicate sequence
                    names.  Use 'twoBitDup' to find duplicate sequences.
   -namePrefix=XX.  add XX. to start of sequence name in 2bit.
(nf_lastz) [kelsielopez@holy8a26602 TOGA]$ 



faToTwoBit ${ficAlb_fa}.fa ${ficAlb_fa}.2bit
faToTwoBit ${parMaj_fa}.fa ${parMaj_fa}.2bit


# now these are the paths to our two bit files that we will input into toga. 
ficAlb_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/ficAlb_renamed_chrom.2bit"
parMaj_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/parMaj_renamed_chrom.2bit"


# lastly, its necessary to prepare an isoform file
# this tells us all the transcripts that correspond to each gene. put the path to the reference .gtf file for parus major 

awk '$3=="transcript" && /transcript_id/ && /gene_id/' /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/genomic.gtf \
  | awk '{
      match($0, /gene_id "([^"]+)"/, g)
      match($0, /transcript_id "([^"]+)"/, t)
      if(length(g) && length(t)) print g[1] "\t" t[1]
    }' | sort | uniq > isoforms.tsv

# this makes a file liek this, but it needs a header..
(python_env1) [kelsielopez@holy8a26602 TOGA]$ head isoforms.tsv
A1CF	XM_015633798.2
A1CF	XM_015633799.2
A1CF	XM_015633801.2
A1CF	XM_015633802.2
A1CF	XM_015633803.2
A4GNT	XM_015637125.3
AAAS	XM_015615913.3
AAAS	XM_033511510.1
AACS	XM_015644309.3
(python_env1) [kelsielopez@holy8a26602 TOGA]$

# open the file with nano to add tab separated 'geneID' and 'transcriptID'

(python_env1) [kelsielopez@holy8a26602 TOGA]$ head isoforms.tsv
geneID	transcriptID
A1CF	XM_015633798.2
A1CF	XM_015633799.2
A1CF	XM_015633801.2
A1CF	XM_015633802.2
A1CF	XM_015633803.2
A4GNT	XM_015637125.3
AAAS	XM_015615913.3
AAAS	XM_033511510.1
AACS	XM_015644309.3
(python_env1) [kelsielopez@holy8a26602 TOGA]$ 

# now we have a path to our isoforms data 
isoform_file="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/isoforms.tsv"

# lastly, we need to specify the path to our chain file output from make lastz chains!
chain="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/parMaj.ficAlb.final.chain.gz"


# this is the script i used to run TOGA on the parus major and ficedulla albicollis

nano test_toga_2.sh

#!/bin/bash
#SBATCH -p edwards 
#SBATCH -c 16 
#SBATCH -t 3-00:00 #  
#SBATCH -o test_toga_2_%j.out # file name to write output to
#SBATCH -e test_toga_2_%j.err # file name to write errors to 
#SBATCH --mem=100000 # memory requested (100Gb) 
#SBATCH --mail-type=END # send me an email when my job is done running 

#py_env

# don't forget to be in nf_toga

#conda activate nf_toga


ficAlb_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/ficAlb_renamed_chrom.2bit"
parMaj_2bit="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/parMaj_renamed_chrom.2bit"
workdir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA"
path_to_nextflow_config_dir="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/nextflow_config_files"
#renamed_for_TOGA_bed="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/genomic_renamed_toga_BED12.bed"
chain="/n/netscratch/edwards_lab/Lab/kelsielopez/make_lastz_chains/test6_v2_reDownload/parMaj.ficAlb.final.chain.gz"
renamed_for_TOGA_bed="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/parus_major/data/GCF_001522545.3/genomic.genePred.bed12.renamed.bed"
isoform_file="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/isoforms.tsv"

cd ${workdir}

./toga.py ${chain} ${renamed_for_TOGA_bed} \
${parMaj_2bit} ${ficAlb_2bit} \
--kt \
--nc ${path_to_nextflow_config_dir} --cb 3,5 --cjn 500 \
--ms \
--isoforms ${isoform_file} \
--project_dir test2



# this is the important output from toga !

/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/test2/orthology_classification.tsv

```




## Post TOGA - getting one2one orthologs for differential expression analysis 

```bash

# i copied this stuff over from your nextflow directories! 

cp /n/netscratch/edwards_lab/Lab/enickhes/nfcore_reads/output_pauroreus_2/star_salmon/salmon.merged.gene_counts.tsv /n/netscratch/edwards_lab/Lab/kelsielopez/emily/pauroreus

cp /n/netscratch/edwards_lab/Lab/enickhes/nfcore_reads/output_pm_m_3/star_salmon/salmon.merged.gene_counts.tsv /n/netscratch/edwards_lab/Lab/kelsielopez/emily/parus_m_m



### merging data sets by correct toga one2one orthologs

# output file with orthology of each gene 
/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/test2/orthology_classification.tsv

# path to 
/n/netscratch/edwards_lab/Lab/kelsielopez/emily/pauroreus/salmon.merged.gene_counts.tsv
/n/netscratch/edwards_lab/Lab/kelsielopez/emily/parus_m_m/salmon.merged.gene_counts.tsv



# results of toga are in this directory /n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/test2
orthology="/n/netscratch/edwards_lab/Lab/kelsielopez/Thamnophilus/TOGA/test2/orthology_classification.tsv"

pauroreus_counts="/n/netscratch/edwards_lab/Lab/kelsielopez/emily/pauroreus/salmon.merged.gene_counts.tsv"

parus_m_m_counts="/n/netscratch/edwards_lab/Lab/kelsielopez/emily/parus_m_m/salmon.merged.gene_counts.tsv"




# put all one2ones into this tsv file. 
grep 'one2one' orthology_classification.tsv > orthology_classification_one2one.tsv


# now filter a list of all these genes .. unique orthologs

cut -f1 orthology_classification_one2one.tsv | sort | uniq > one2one_t_genes.txt


# there are 13,036 one2one orthologs 
kelsielopez@boslogin06 test2]$ wc -l one2one_t_genes.txt
13036 one2one_t_genes.txt
[kelsielopez@boslogin06 test2]$ head one2one_t_genes.txt
A1CF
A4GNT
AAAS
AACS
AADAC
AADAT
AAGAB
AAK1
AAMDC
AAMP
[kelsielopez@boslogin06 test2]$ 



```



## Now run DESeq2_TOGA_all_samples.R

