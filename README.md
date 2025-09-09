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




## TOGA analysis
 ## will add this code really soon


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

