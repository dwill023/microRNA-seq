# microRNA-seq
Small RNAs or microRNAs are around 22 nt long and have been shown to regulate gene expression, mainly through suppressing translation of their target mRNA. microRNAs (miRs) have been implicated in many cellular processes from differentiation and cancer [Gebert & MacRae, 2018](https://www.nature.com/articles/s41580-018-0045-7). 
microRNA sequencing is used to analyze the repertoire of these miRs within cells and how they may be differentially between conditions, such as healthy and cancerous cells. 

MicroRNAs get named sequentially. If a microRNA has the following name/identifier:
hsa-mir-121-1 and hsa-mir-121-2: Identical mature miR that come from different genomic loci
has-miR-142-5p or hsa-miR-142-3p: 5p=sense strand and 3p=antisense strand

Sequencing can be perfomed single-ended with at least 2 million alined reads per sample with two or more replicates. See [ENCODE](https://www.encodeproject.org/microrna/microrna-seq/).

What you need before you begin:
- gtf/gff file with coordinates of all microRNAs in the human genome. Downloaded from [mirbase](http://www.mirbase.org/ftp.shtml). 
- Genomic DNA human genome reference (GRCh38/hg38) downloaded from ensembl or UCSC.

Below is a schematic of the small RNA library prep and the adapters that get ligated on. 
![experiment](https://github.com/dwill023/microRNA-seq/blob/main/pics/experiment.JPG)

## Pre-processing of reads
In the terminal perfom the following checks within the directory containing the sequencing files(.fastq).

1. Simple check to see how many reads contain the adapter:
```
grep -c AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC miRNA_reads.fastq
```
2. Graph the sequence length distribution to get an overview of the dataset.
```
zcat miRNA_reads.fastq.gz | awk ’{if(NR%4==2) print length($1)}’ | sort -n | uniq -c > miRNA_readLength.txt
```
3. Check the overall sequence quality with [MapQC](https://multiqc.info/)
```
conda activate bioinfo
multiqc .
```

## Alignment of reads
Use the [Rsubread/Subread aligner](http://subread.sourceforge.net/) which has instructions specifically for miRNA-seq. Reads do not need to be trimmed  before feeding them to Subread aligner since Subread soft clips sequences in the reads that can not be properly mapped. 

1. Build the index files:
```
module load subread/1.6.2

subread-buildindex -F -B -o /human/subread_index/HS_index /data/human/Homo_sapiens.GRCh38.dna.primary_assembly.fa
```
2. Alignment:
If a threshold of 4 consensus subreads was used, length of miRNA sequences that can be detected is 19 bp or longer. With this threshold, there will be at least 19 perfectly matched bases present in each reported alignment.

| Threshold  | Length of miRNA detected |
|  :---: |  :---: |
| 2  | 17 bp  |
| 4  | 19 bp  |
| 7 | 22 bp  |

Note that ‘-t’ option should have a value of 1 since miRNA-seq reads are more similar to gDNA-seq reads than mRNA-seq reads from the read mapping point of view.
-n: number of subreads extracted for mapping
-m: consensus subreads required for reporting a hit or threshold in above table.
-M: max number of mismatched bases allowed in alignment. 3 by default.
-I: number of indel bases allowed in the mapping. 5 by default.
-t: type of input sequencing data. 0 (RNA-seq), 1(DNA-seq). 
-T: number of threads/CPUs used for mapping. 1 by default.
--multiMapping: Multi-mapping reads will also be reported in the mapping output. Number of alignments reported for each multimapping read is determined by the ‘-B’ option.

```
subread-align -t 1 -i /human/subread_index/HS_index -n 35 -m 4 -M 3 -T 10 -I 0 --multiMapping -B 10 -r miRNA_reads.fastq -o result.sam
```

## Quantification of microRNAs
miRNA reads can be counted using bedtools intersect or featureCounts. It is important to allow multiple hits of each read when mapping, since there are multiple copies of some microRNAs in the genome and if it is not allowed, the results might be misleading, or wrong.
Align the small RNAs to mature miRNA sequences(hsa.gff3) from  http://www.mirbase.org/ allowing for no mismatches.
featureCounts -t miRNA -g Name -O -s 1 -M -a <mirbasefile> -o <outfile> <samfiles>

```
featureCounts -t miRNA -g Name -O -s 1 -M -a /data/human/miRBase/hsa.gff3 -o miR_counts.txt result.sam
```
Here we only look at loci that are “miRNA”, and we use the “Name” attribute to name the loci. The -O flag tells the program that reads that map to several overlapping microRNAs should be assigned to all of them. The -s 1 flag tells the program to only count reads that map to the same strand as the microRNA, and the -M flag makes sure we count multi mapping reads. The output will be a list with the number of reads mapping to each microRNA.

### Count normalization

From the paper, [Preprocessing of miRNA-seq data](https://paperpile.com/shared/2IJnFZ) they recommend:
UQ or TMM for miRNA count normalization for comprehensive miRNA profiling studies.

This can be done using the R package edgeR that calculates the TMM normalization and can be used to assses differential microRNA expression between the samples.
See the attached R file for these steps.

