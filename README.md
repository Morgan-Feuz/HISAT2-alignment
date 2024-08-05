# Aligning RNA-seq Reads to a Reference Genome with HISAT2

### Background
HISAT2 (hierarchical indexing for spliced alignment of transcripts 2) is a popular, fast, and sensitive alignment program for mapping next-generation sequencing reads. The HISAT2 main page is available [here](https://daehwankimlab.github.io/hisat2/manual/).  

HISAT2 can be used in three steps: (1) Download the HISAT2 binaries and system set up, (2) build or download an appropriate index, (3) aliging next get reads to the indexed genome. 

-----------------------------------------------------------------------------------------------------------
### 1 - Linux Installation and Set-up 
```
# Download HISAT2 binaries and unzip in an appropriate location
$ wget https://cloud.biohpc.swmed.edu/index.php/s/oTtGWbWjaxsQ2Ho/download
$ unzip download

# Make executable (both hisat2 and hisat2-build)
$ sudo ln -s /path/to/file/hisat2 /usr/bin
$ sudo ln -s /path/to/file/hisat2-build /usr/bin
```

-----------------------------------------------------------------------------------------------------------
### 2 - Download or Build a Reference Genome Index
The [HISAT2 downloads](https://daehwankimlab.github.io/hisat2/download/) page contains pre-build indexes for H.sapiens, M. musculus. R. norvegicus, D. melanogaster, C. elegans, and S. cerevisiae, which are hosted on Amazon Web Services (AWS). Specifically, for M. musculus, UCSC mm10. To use a prebuild index, simply download the files. 

However, a new index will need to be build if one wanted to use the updated GRCm39 mus musculus genome annotation. Here, the GRCm39 genome annotation was downloaded from [GENCODE](https://www.gencodegenes.org/mouse/). Note that this should be a FASTA file. 


```
# Basic usage to create a small index
$ hisat2-build GRCM39.genome.fa genome
```

`hisat2-build` builds a HISAT2 index from a set of DNA sequences. `hisat2-build` outputs a set of 8 different files with suffixes `.1.ht2` - `.8.ht2`. In the case of a large index, these suffixes will have a `ht2l` termination. These files together constitute the index: they are all that is needed to align reads to that reference. The original sequence FASTA files are no longer used by HISAT2 once the index is built. 

<ins>Note:</ins> If you use `--snp`, `--ss`, and/or `exon`, `hisat2-build` will need about 200GB RAM for the human genome size as index building invovles a graph construction. Otherwise, you will be able to build an index on your desktop with 8GB RAM. 

----------------------------------------------------------------------------------------------------------

### 3 - Sequencing Read Alignment to the Indexed Genome

By default, HISAT2 may soft-clip reads near their 5' and 3' ends. Users can control this behavior by setting different penalties for soft-clipping (`--sp`) or by disallowing soft-clipping (`--no-softclip`). 

If your computer ahs multiple processors/cores, use `-p`. The `-p` option causes HISAT2 to launch a specified number of parallel search threads.

Default output of HISAT2 is a SAM file, and [Samtools](https://www.htslib.org/) can be used to convert the SAM to a BAM and sort. 

<ins>Single-end Reads:</ins>
```
# single-end basic working example for one sample
$ hisat2 -q --rna-strandness R -x /path/GRCm39_GENCODE/genome -U fastq_R1_001.fastq.gz --summary-file file.txt | samtools sort -o file.bam
```

<ins>Paired-end Reads:</ins>
```
# paired-end basic working example for one sample
$ hisat2 -q -x /path/GRCm39/genome -1 fastq_R1_001.fastq.gz -2 fastq_R2_001.fastq.gz --summary-file file.txt | samtools sort -o file.bam
```

+ `-q`: Reads are FASTQ files
+ `--rna-strandness R`: Reverse stranded (default is unstraded)
+ `-x`: The basename of the index for the reference genome

