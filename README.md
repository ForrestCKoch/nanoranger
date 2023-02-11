# nanoranger

*nanoranger* is a processing tool for long-read single-cell and spatial transcriptomics data as described here (future link to paper). 

The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, sequenced with Oxford Nanopore Technologies (ONT) or Pacific Biosciences devices. It is inspired by *cellranger*. 

## Background

One of the main challenges of ONT data analysis for single-cell applications has been higher sequencing error compared to Illumina data the variable location of cell barcodes and molecular identifiers (UMI) within each sequenced transcript. 

To overcome this challenge *nanoranger* introduces **two innovations**:

- The processing pipeline starts with alignment of reads to a transcriptome reference. This initial transcriptome alignment step enables orientation and extraction of 'subread' components - the transcript and part of the read upstream or downstream of the transcript which contains barcode and UMI. <br><br>
By extracting flanking (soft-clipped) portions of a transcript it is possible to reliably assign cell barcodes to their transcript. This also limits the search space from a usually 200nt region at both ends of a read to a small 50nt part. This not only speeds up barcode matching, it also reduces the chance of assigning wrong barcodes to transcripts. <br><br>
Another feature automatically enabled by this approach is recovery and reliable quantification of fused/follow-on reads (also called informatic chimeras) generated abundantly in newer ONT chemistries (LSK112 and LSK114) by processing all supplementary transcript alignments for each read. This occasionally leads to extraction of 100s of supplementary transcripts from a single read. By accounting for such events we can recover as high as %50 more usable transcripts from the raw reads. A natural extension of this feature is processing and deconcatenation of libraries which are generated using concatenation methods, such as [MAS-ISO-seq](https://www.biorxiv.org/content/10.1101/2021.10.01.462818v1.full).<br><br>
- To perform barcode matching while accounting for indels and mismatches, *nanoranger* uses an aligner-based technique by aligning barcode components of the subreads against a reference of known barcodes (such as 737K whitelist for 10x Genomics 5' libraries included here). <br>Compared to techniques which solely rely on adapter identification, this approach can avoid missed or erroneously assigned barcodes due to frameshifts introduced by errors in flanking adapters. To achieve this *nanoranger* uses STAR with a number of changes to the default options. The primary modification is change of the alignment mode to EndToEnd instead of the default softclipping in the unaligned ends of a read to force all bases of the barcode candidates to be mapped to the reference. Simultaneously, *nanoranger* pads the whitelist of barcodes with unknown nucleotides to avoid penalizing the adapter and UMI sequences which are kept in the barcode candidate reads.

![schema](https://raw.githubusercontent.com/mehdiborji/nanoranger/main/data/schema.png)

There are different quantification 'modes' available for different libraries structures and tasks and the transcriptome reference can be modified accordingly. For whole transcriptome gene expression analysis a GENCODE transcriptome reference can be used . For 5' immune profiling this can be reduced to a reference of V transcripts and similarly for 3' immune profiling this can be a reference of C transcripts. If a set of targets is used for enrichment from cDNA, to speed up analysis one can only use a reference for those transcripts that are expected to be present.

nanoranger has been primarily tested on targeted libraries generated using 10X 5' Chromium and slide-seq 3' platforms. It can be used for immune profiling and genotyping from other library types with minimal modifications. 

Further developments for generating count matrices for whole transcriptome libraries as well as addition of other chemistry types are currently underway.

## Software Dependencies 
This tool has been tested on Python 3.7.10 under Centos and Ubuntu systems.

The following programs are also assumed to be in path when running the tool. Please refer to the provided link for each to install them prior to start of your data analysis using this tool. Alternatively they are available as bioconda packages.

[STAR](https://github.com/alexdobin/STAR) is used for barcode correction against a set of known barcodes. By certain input parameter changes we use STAR in a Smith-Waterman-like mode.

[minimap2](https://github.com/lh3/minimap2) is used for initial alignment of raw nanopore reads to a transcriptome and (subsequently based on operation mode) alignment to a genome. 

[SAMtools](http://www.htslib.org/download/) is used for sorting and indexing BAM files

[pigz](https://zlib.net/pigz/) is used for compressing output and intermediate fasta and fastq files.

[MiXCR](https://github.com/milaboratory/mixcr) is used for VDJ alignment and clonotype extraction.

[SeqKit](https://bioinf.shenwei.me/seqkit/) is used for splitting input fastq files in case of very large libraries or libraries prepared with cDNA concatenation. Deconcatenation speed-up is achieved by parallel processing of splitted input files. To enable this step set the optional boolean flag --split.

## Download and Install
```
git clone https://github.com/mehdiborji/nanoranger.git
cd nanoranger
chmod -R +x *
pip install -r requirements.txt

```
## Sample Input Command 
The following can perform analysis of sample mitochondrial reads in 5-mer concatenation form, from a 10x 5' library:
```
python pipeline.py --o outdir --c 4 --i data/5mer.gz --e Mito --m 5p10XGEX --s
```
