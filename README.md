#  Developing a novel pipeline for detecting genomic compressions caused by whole genome misassembly.

## Synopsis

Genomic Compression Detection Pipeline (GCDP) is a pipeline designed to efficiently detect and estimate genomic compressions in 		whole genome sequence created by genome misassembly. Genomic compressions are assembly artifacts characterized by collapse or merge of near identical repeats in genome sequence. GCDP uses Read Depth Coverage (R.D.C) generated by pair-end (or single end) short reads (example: Illumina) after getting mapped to these sequencec, for locating these problematic regions across entire genome sequence as well as user-defined gene/genomic co-	ordinates. Currently it runs for haploid genomes only and requires co-ordinates of user-defined single-copy genes/genomic segments for normalizing background read coverage and calculating Compression Detection Threshold required for detecting and estimating compressions. It runs in UNIX command prompt as well as on cluster (HPC).

## Workflow

	The pipeline works in 2 steps :
	(i)  Mapping reads to reference genome sequence and generating depth file from uniquely mapped reads. 
	(Currently works for pair-end reads only)
	(ii) Using depth files generated from step (i) and user-provided single copy gene/ genomic co-ordinates
	to identify and estimate genomic compressions across entire genome sequence as well as specific genomic co-ordinates.
	
	User can skip step (i) by providing pre-mapped depth file generated from mapping either single or pair-end reads to the 
	reference genome sequence. [See Input files & Examples for details] 
	
	Step (ii) consists of 4 phases:
	Phase 1. This step calculates average read depth coverage across all user-provided single copy gene/genomic segments and uses 
	its distribution and deviation to filter out false positives and background noise. It further generates Compression Detection 
	Threshold, a value corresponding to 3 times standard deviation of average read depth, which is used in subsequent phases for
	calculation and estimation of compressions. 
	
	Phase 2. In this step, compressions of user-provided genomic co-ordinates are calculated using Compression Detection Threshold from
	phase 1. This step is particularly useful when it is required to calculate copy number of specific genes or intergenic sequences that
	are believed to be compressed in the given reference sequence. This phase is optional and runs parallelly with phase 3 & 4.  
	
	Phase 3. The Compression Detection Threshold is used to estimate compression / copy number for each base across entire genome sequence
	to seperate regions with detected compressions from non-compressed regions. It works by clustering these compressed regions and 
	defining boundaries between segments that are atleast 50bp apart (by default setting) . This phase provides genomic segments that represent 
	compressions with a wider genomic resolution and copy number which has been averaged over relatively longer sequence ength. 
	
	Phase 4. This step uses the output from phase 3 and sharpens the resolution of the detected compressions by sub-segmenting the 	
	compressed genomic segments based on difference in copy number ( copy number >2 by default setting)  between 2 consecutive bases. Thus it redefines boundaries of compressed
	segments detected from previous step and generates a new list of genomic co-ordinates that represents genome locations that are 
	enriched in compressions relative to its adjacent regions. 
	
		
	
	
	
	         

## Prerequisites

	Following programs are required for GCDP to run :
	(i)   SAMTools 
	(ii)  BWA-MEM 
	(iii) R (V3.1 or above)
	
	All of the above should be made available in the PATH BEFORE running the pipeline. To do that, use : 
	export PATH=$PATH:/path/to/folder_containing_program 	
	e.g: If R is located at usr/local/apps/R/3.3.1/bin/R, use
	export PATH=$PATH:/usr/local/apps/R/3.3.1/bin

## Input files
	
	The following files will be required to run GCDP: 
	For step (i) : 			 1.	reference_genome.fasta 
				         2.	pair_end_read_1.fastq 
				         3.	pair_end_read_2.fastq 
				         4.	chromosome_list.txt
				  
	For step (ii): 			 1. 	depth_file.depth 
				         2. 	chromosome_list.txt
				         3. 	single_copy_co-ordinates.txt 
				         4. 	all_co-ordinates.txt
				   
	Location and name of files required in step (ii) must be written in parameter.file [see example] 
	Then only the parameter.file will be required for running his step.
	
## Details of input files:
	
	reference_genome.fasta: Genome sequence in fasta format for which the genomic compressions are calculated. Please only include 
	chromosomes and not broken contigs for optimum use.
	pair_end_read_1.fastq : Forward reads in fastq format. 
	pair_end_read_2.fastq : Reverse reads in fastq format. 
	Reads should be ideally clean ( using trimmomatic for example ) before using.If using multiple libraries they can be concatenated into   total forward and reverse read fastq files.							                   
    
	User can skip all the above files required for step (i) if starting with pre-mapped depth file
	
	depth_file.depth: 	        Depth file generated from step (i) or the one generated by user from mapping single/pair-end reads to reference_genome.fasta 
	single_copy_co-ordinates.txt:	A tab delimited co-ordinates of genes / genomic segments believed to be present in single copy in genome sequence
	all_co-ordinates.txt:	        A tab delimited co-ordinates of genes / genomic segments for which compressions need to be calculated (optional)
	chromosome_list.txt:	        A text file listing name of chromosomes (one in each line). Make sure the name matches the header    representing chromosomes in input reference_genome.fasta file.
	parameter.file:			A text file detailing name and locations of the files required for running step (ii)
	
	                [See Example for details on the format and content of the input files] 									
	
	
##  Brief description of GCDP script files
	
	Scripts required for step (i) & (ii) are located inside Depth_Maker and GCDP folders respectively. 
	Inside Depth_Maker
			depth_maker.sh : Main script file that generates depth file following mapping pair_end_read_1/2.fastq reads to reference_genome.fasta. 
							
	Inside GCDP 
			gcdp.sh: Main script file that detects and estimates compression in entire reference_genome.fasta (using entire_chrom.sh & entire_chrom_subsegment.sh) as well as those provided in all_co-ordinates.txt (using cords.sh). 
	
##  Output files 
	
