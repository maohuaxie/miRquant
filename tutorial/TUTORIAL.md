# miRquant Tutorial
Last update to README: 12/6/16

# miRquant 2.0  

1. Information
2. Installation
  1. Requirements
  2. Setup
3. Running miRquant
  1. Configuration file input
  2. Alignment
  3. Sorting results
  4. Final analysis

## Tutorial info
This tutorial is for two mouse samples.  The mouse samples are were prepared by TruSeq.

## miRquant Installation  
### Requirements
##### Software
miRquant 2.0 can be downloaded as a zip file ([click here](https://github.com/Sethupathy-Lab/miRquant/archive/master.zip)) or cloned from the [miRquant GitHub page](https://github.com/Sethupathy-Lab/miRquant).  

In addition to these scripts, miRquant 2.0 requires the following software for various steps of the pipeline.

* python v2.7.6
* pip 
* bedtools v2.26.0  
* bowtie v1.1.0  
* SHRiMP v2.2.2  
* R v3.2.2 

Install these programs with in your system path.

##### Resources

miRquant is currently set up to work with human, mouse and rat, with fruitfly support coming.

The specific genome releases used in miRquant are:

human - [hg19](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/)  
mouse - [mm9](http://hgdownload.cse.ucsc.edu/goldenPath/mm9/bigZips/), [mm10](http://hgdownload.cse.ucsc.edu/goldenPath/mm9/bigZips/)  
rat - [rn4](http://hgdownload.cse.ucsc.edu/goldenPath/rn4/bigZips/)  
fruitfly - dm3  
dog - [canFam3](http://hgdownload.cse.ucsc.edu/goldenPath/canFam3/bigZips/)

Download the appropriate genomes and the chromosome sizes for that genome release (\<release\>.chrom.sizes) 

### Setup

Once python/2.7.6 and pip are installed, change to the miRquant directory and type:

```
pip install --user -r requirements.txt
```

Change the genome fasta name to \<prefix\>.fa and the chromosome sizes file to \<prefix\>.chromSizes.  The prefixes for each species is as follows:

human - hg19  
mouse - mm9, mm10  
rat - rn4  
fruitfly - dm3  
dog - canFam3

Store the genomes and the chromosome size files *in the same location*.

Build Bowtie genome indexes for each genome.  Information on this can be found in the [Bowtie tutorial](http://bowtie-bio.sourceforge.net/tutorial.shtml).


## Running miRQuant

#### Enter run parameters into the miRquant configuration file


The configuration directory contains two configuration files.

1. conf_system.yml
  * Configuration file for the cluster you are working on, currently filled out for lsf job scheduler.
2. conf_miRquant.yml
  * Configuration that will be edited on a project by project basis

The system configuration (conf_system.yml) is as follows:
```
# System configuration for submitting jobs
job:
    bsub:
        -q: week

job_quick:
    bsub:
        -q: hour

# This will be edited to fit your job scheduler (eg slurm, lsf, ect)
```

The system configuration file is setup for a lsf job scheduler.  The queue names should be changed to match the queues on the cluster you are working.  Additional options for can be added as single lines in the form \<option flag\>: value.  For example, if we wanted to add to output the log to a file named Log.txt instead of recieveing an email (the -o bsub option), we would add `-o: Log.txt` under the -q option *at the same indentation*.  If you are not working with a job scheduler, **delete the conf_system.yml file.**

The miRquant configuration file (conf_miRquant.yml) is as follows:
```
# Directory locations
paths:
    genome:
        /path/to/genome/
    mirquant:
        /path/to/miRquant/
    output:
        /path/to/output/
    project:
        /path/to/fastqs/

# miRquant parameters
parameters:
    genome_release:
        mm9           <- prefix for genome release (eg, mouse release 9)
    species:
        mmu           <- species, currently hsa, mmu, or rno
# Load in options for cutAdapt
cutadapt:
    adapter:
        'TGGAATTCTCGGGTGCCAAGGAACTCCAGTCACXXXXXXATCTCGTATGCCGTCTTCTGCTTG'           <- TruSeq adapter (Xs denote barcode)
    overlap:
        10
    error:
        1
    Minimum_Read_Length:
        14
# Load in options for bowtie
bowtie:
    quality:
        33
# Load in options for SHRiMP
shrimp:
    path:
        /proj/.test/roach/miRNA/SHRiMP_2_2_2/
    dependencies:
        python:
            /proj/.test/roach/miRNA/lib/python/
    quality:
        33
```

*How to fill out the miRquant configuration file:*
* Directory locations
  - genome - path to the genome and chromosome sizes directory
  - mirquant - path to the miRquant program directory
  - output - path to where outputs will be saved.  Directory will be created if it does not exist.
  - project - location of the small RNA sequencing results.  All fastqs in location will be processed by miRquant.
* miRquant parameters
  - genome release - appropriate prefix for the species (see prefixes in setup)
  - species - hsa, mmu, or rno for human, mouse, or rat, respectively.
* cutadapt options
  - adapter - 3' adapter sequence; if barcode present, replace with Xs; if degenerate bases present at 5' end, add as Ns
  - overlap - number of overlapping nucleotides for trimming to occur
  - error - error tolerance (1 = 0.1 or 10%)
  - minimum read length - minumum length of trimmed read to be included
* Bowtie options
  - quality - quality cutoff for Bowtie alignment
* SHRiMP options
  - path - path to SHRiMP executables
  - quality - quality cutoff for SHRiMP
    
For the tutorial, only the paths section of conf_mirquant.yml will have to be altered.
  
Copy the configuration directory to the directory containing the small RNA-seq fastqs.
```
cp -r /path/to/miRquant/configuration /path/to/fastq_containing_directory
```

##### Notes on adaptors

miRquant2.0 is able to extract the barcode index sequence from fastq files in the Cassava 1.8 or later format. An example of the Cassava 1.8 format, where the six nucleotide index sequence can be seen at the end of the sequence ID line.
```
@EAS139:136:FC706VJ:2:2104:15343:197393 1:Y:18:ATCACG
```

If your fastqs are not in this format, the index sequences can be supplied in a tab-separated file called _barcodes.txt_, formated like this:
```
SampleA.fastq    ATGTCA
SampleB.fastq    CGTCCG
```
where the first column is the name of the fastq file and the second column is the index sequence.  The _barcodes.txt_ file should be placed in the same directory as the fastq files.


##### **_OPTIONAL_** Condition and comparison files
While not a part of this tutorial, a condition and comparison file can also be provided. Providing these files will allow for a more thorough final report. These files should be placed in the directory containing the sequencing files and be named **conditions.csv** and **comparisons.csv**. 

The conditions file will be a two columned comma-separated file where the first column has the header 'Sample' and contains the names of the sample fastqs without the extension (eg. SampleA.fastq -> SampleA). The second column has the header 'Condition' and assigns a condition to each sample. Supplying a conditions file has the folling benefits: 1) Condition information will be added to the sample correlation heatmap 2) Condition information will be added to the principal component analysis 3) A statistical analysis will be performed to find the average expression within each condition. Here is an example of a **conditions.csv** file:
```
Sample,Condition
SampleA,Control
SampleB,Treatment
SampleC,Treatment
SampleD,Control
```
The comparisons file will be a two columned comma-separated file used for statistical comparisons. Each column must be one of the conditions from the conditions file. These comparisons will be used for the generation of fold changes and p-values. The condition in the first column with be the numerator for the fold change calculation while the second will be the denominator (eg Treatment expression / Control expression). Here is an example of a **comparisons.csv** file:
```
Treatment,Control
```

#### Run the miRquant script:
From the miRquant directory:
```
$ cd /path/to/miRquant

$ python miRquant.py path/to/configuration/
```

#### Once all jobs have finished:
Once the chain submission has finished, check for any errors in the log file.  Each sample will have a multiple output directories setup in the output directory specified in the miRquant configuration file, in the following structure.
```
SAMPLE_NAME
  -logs
  -output
  -temp
```

The logs from each step of miRquant will be in the logs directory.  
The outputs will be saved in the output directory.
Temporary files generated during a miRquant run will be stored in the temp directory.

In your project directory, there will be a directory for each \<SAMPLE\>.fastq (called \<SAMPLE\>.)

In that directory, there will be an IntermediateFiles subdirectory and a \<SAMPLE\>.stats file.

The \<SAMPLE\>.stats will have statistics to inform on the degree of trimming and aligning, type:
```
$ cat /path/to/fastqs/*/*.stats

file:/path/to/fastqs/<SAMPLE>.fastq
TotReads:100000
TrimmReads:90000
ShortReads:8500
EMhits:40000
EMmiss:45000

TotReads = Total number of reads for this file
TrimmReads = # of reads successfully trimmed of 3’ adapter
ShortReads = # of reads too short after trimming (< 13 )
EMhits =  # of reads with an exact alignment to the genome
EMmiss = # of reads that fail to exactly align to genome
```

#### Run the next stage to collect results:
From your pipeline directory (/path/to/miRquant):
```
$ python runC.py path/to/configuration
```

#### Run the next stage to generate TAB separated files:
```
$ python process_summary_to_tab.py path/to/configuration
```
After run finishes, you should see:
```
$ cd path/to/fastqs
$ cat */*.stats

file:path/to/fastqs/<SAMPLE>.fastq
TotReads:6149484.00000000000000000000
TrimmReads:3730081.00000000000000000000
ShortReads:1938220.00000000000000000000
EMhits:2509867
EMmiss:1220214
Mapped: 2881057.09082651
miRMapped: 1104952.30513136
```

Mapped and miRMapped indicate the number of reads mapped to the genome and to miRNAs respectively.

The log file above will also contain a table that you can use to put together the mapping stats for your project.

Outputs:
For each Sample:
  TAB_3p_summary.txt       -   3'-end differences
  TAB_3p_summary_miR.txt   -   3'-end differences (miRNA loci only)
  TAB_ed_summary.txt       -   central differences
  TAB_lenDist_summary.txt  -   length differences
  Shrimp_results.bed       -   bed file containing all results

#### Final processing
To produce the final summary files, run:
```
$ python final_processing.py path/to/configuration
```
This will produce the mapping statistics, read length distribution, expression correlation heatmap, reads per million mapped (RPMM), and reads per million miRs mapped (RPMMM).

These final outputs will be in the output folder specified in the configuration file, in a directory named year_month_day_miRquant_num, where the year, month, and day refer to the date and the num will correspond to how many times miRquant had been run on that day.
