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

