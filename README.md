# popscle
`popscle` is a suite of population scale analysis tools for single-cell genomics data. The key software tools in this repository includes `demuxlet` (version 2) and `freemuxlet`, a genotyping-free method to deconvolute barcoded cells by their identities while detecting doublets. 

### Quick Overview

With `popscle`, we recommend analyzing single cell RNA-seq (and other single cell genomic) dataset in two steps.
1. Use `dsc-pileup` to generate pileups around known variants from aligned sequence reads.
2. Use `demuxlet` (with genotypes) or `freemuxlet` (without genotypes) to deconvolute the identities of barcoded cells. 

Read the tutorial at https://github.com/statgen/popscle/wiki , if you would like to learn how to run software tools in `popscle` by example.

Read the documentation below if you want a comprehensive documentation about these tools.

### Introduction

#### Overview

`demuxlet` and `freemuxlet` are two software tools to deconvolute sample identity and identify multiplets when multiple samples are pooled by barcoded single cell sequencing. If external genotyping data for each sample is available (e.g. from SNP arrays), `demuxlet` would be recommended. On the other hand, if external genotyping data is not available, the genotyping-free version demuxlet, `freemuxlet`, would be recommended. You still need variant site list (in VCF) even if you intend to use `freemuxlet` in order to generate pileups.

You need to run `dsc-pileup` before running `demuxlet` and `freemuxlet`. `dsc-pileup` is a software tool to pileup reads and corresponding base quality for  each overlapping SNPs and each barcode. By using pileup files, it would allow us to run demuxlet/freemuxlet pretty fast multiple times without going over the BAM file again. 

`dsc-pileup` requires the following input files:

1. a SAM/BAM/CRAM file produced by the standard 10x sequencing platform, or any other barcoded single cell RNA-seq (with proper `--tag-UMI` and `--tag-group`) options.
2. A VCF/BCF files containing (AC) and (AN) from referenced population (e.g. 1000g).

`demuxlet` require the following input files:

1. Pileup files (CEL,VAR and PLP) produced by `dsc-pileup`.
2. a VCF/BCF file containing the genotype (GT), posterior probability (GP), or genotype likelihood (GL) to assign each barcode to a specific sample (or a pair of samples) in the VCF file.

Alternatively, `dsc-pileup` could also directly take SAM file without running `dsc-pileup`. In this case, `dsc-pileup` would require the following files:

1. a SAM/BAM/CRAM file produced by the standard 10x sequencing platform, or any other barcoded single cell RNA-seq (with proper `--tag-UMI` and `--tag-group`) options.
2. a VCF/BCF file containing the genotype (GT), posterior probability (GP), or genotype likelihood (GL) to assign each barcode to a specific sample (or a pair of samples) in the VCF file.

`freemuxlet` require the following input: 

1. Pileup files (CEL, PLP and VAR) from dsc-pileup 
2. Number of samples

### Tips for running
* If external reference sequence vcf file is available, **_demuxlet_** is recommended
* Default setting alpha as 0.5, which assumes the expected proportion of 50% genetic mixture from two individuals, to get better estimates of doublets.
* Set `--group-list` to a list of barcodes (i.e. barcodes.tsv from 10X) in `dsc-pileup` to speed things up and only get demultiplexing for cells called by other methods.
* To reproduce the results presented in Figure 2 of the demuxlet paper, please use the original version of demuxlet, with the data downloadable at https://github.com/yelabucsf/demuxlet_paper_code/tree/master/fig2 . If you want to learn how to perform similar analysis with `popscle`, please go to https://github.com/statgen/popscle/wiki .
* Check tutorial README.md for more detailed tutorial with example data
* If you start process in docker, use cmdline `docker run <imagename> "<popscle-arguments>"`(e.g. `docker run popscle "freemuxlet"`) to run docker tasks.
### Installing demuxlet/freemuxlet

<pre>
$ mkdir build

$ cd build

$ cmake ..
</pre>

In case any required libraries is missing, you may specify customized installing path by replacing "cmake .." with:

<pre>
For libhts:
  - $ cmake -DHTS_INCLUDE_DIRS=/hts_absolute_path/include/  -DHTS_LIBRARIES=/hts_absolute_path/lib/libhts.a ..

For bzip2:
  - $ cmake -DBZIP2_INCLUDE_DIRS=/bzip2_absolute_path/include/ -DBZIP2_LIBRARIES=/bzip2_absolute_path/lib/libbz2.a ..

For lzma:
  - $ cmake -DLZMA_INCLUDE_DIRS=/lzma_absolute_path/include/ -DLZMA_LIBRARIES=/lzma_absolute_path/lib/liblzma.a ..
</pre>

Finally, to build the binary, run

<pre>
$ make
</pre>

### Using demuxlet and freemuxlet
All softwares use a self-documentation utility. You can run each utility with -man or -help option to see the command line usages. Also, we offer some general practice with an example in tutorial (data is available here: https://drive.google.com/drive/folders/1wfnn132vMbZhicpWOZVbR_36YpIiojug?usp=sharing).

#### demuxlet
<pre>
$(POPSCLE_HOME)/bin/popscle dsc-pileup --sam /data/$bam --vcf /data/$ref_vcf --out /data/$pileup
$(POPSCLE_HOME)/bin/popscle demuxlet --plp /data/$pileup --vcf /data/$external_vcf --field $(GT or GP or PL) --out /data/$filename
</pre>

Or, demuxlet could directly take SAM file as input:
<pre>
$(POPSCLE_HOME)/bin/popscle demuxlet --sam /data/$sam --vcf /data/$external_vcf --field $(GT or GP or PL) --out /data/$filename
</pre>

#### freemuxlet
<pre>
$(POPSCLE_HOME)/bin/popscle dsc-pileup --sam /data/$bam --vcf /data/$ref_vcf --out /data/$pileup
$(POPSCLE_HOME)/bin/popscle freemuxlet --plp /data/$pileup --out /data/$filename --nsample $n
</pre>

The detailed usage is also pasted below.

#### dsc-pileup
<pre>
Options for input SAM/BAM/CRAM 
   --sam [STR: ] : Input SAM/BAM/CRAM file. Must be sorted by coordinates and indexed
   --tag-group [STR: CB] : Tag representing readgroup or cell barcodes, in the case to partition the BAM file into multiple groups. For 10x genomics, use CB
   --tag-UMI [STR: UB] : Tag representing UMIs. For 10x genomiucs, use UB

Options for input VCF/BCF
   --vcf [STR: ] : Input VCF/BCF file, containing the AC and AN field
   --sm [V_STR: ] : List of sample IDs to compare to (default: use all)
   --sm-list [STR: ] : File containing the list of sample IDs to compare

Output Options
   --out [STR: ]  : Output file prefix
   --sam-verbose [INT: 1000000] : Verbose message frequency for SAM/BAM/CRAM
   --vcf-verbose [INT: 10000] : Verbose message frequency for VCF/BCF
   --skip-umi [FLG: OFF] : Do not generate [prefix].umi.gz file, which stores the regions covered by each barcode/UMI pair

SNP-overlapping Read filtering Options
   --cap-BQ [INT: 40] : Maximum base quality (higher BQ will be capped)
   --min-BQ [INT: 13] : Minimum base quality to consider (lower BQ will be skipped)
   --min-MQ [INT: 20] : Minimum mapping quality to consider (lower MQ will be ignored)
   --min-TD [INT: 0] : Minimum distance to the tail (lower will be ignored)
   --excl-flag [INT: 3844] : SAM/BAM FLAGs to be excluded

Cell/droplet filtering options
   --group-list [STR: ] : List of tag readgroup/cell barcode to consider in this run. All other barcodes will be ignored. This is useful for parallelized run
   --min-total [INT: 0] : Minimum number of total reads for a droplet/cell to be considered
   --min-uniq [INT: 0] : Minimum number of unique reads (determined by UMI/SNP pair) for a droplet/cell to be considered
   --min-snp [INT: 0] : Minimum number of SNPs with coverage for a droplet/cell to be considered
</pre>

#### demuxlet
<pre>
Options for input SAM/BAM/CRAM
   --sam [STR: ] : Input SAM/BAM/CRAM file. Must be sorted by coordinates and indexed
   --tag-group [STR: CB] : Tag representing readgroup or cell barcodes, in the case to partition the BAM file into multiple groups. For 10x genomics, use CB
   --tag-UMI [STR: UB] : Tag representing UMIs. For 10x genomiucs, use UB

Options for input Pileup format
   --plp [STR: ] : Input pileup format

Options for input VCF/BCF
   --vcf [STR: ] : Input VCF/BCF file, containing the individual genotypes (GT), posterior probability (GP), or genotype likelihood (PL)
   --field [STR: GP] : FORMAT field to extract the genotype, likelihood, or posterior from
   --geno-error-offset [FLT: 0.10] : Offset of genotype error rate. [error] = [offset] + [1-offset]*[coeff]*[1-r2]
   --geno-error-coeff  [FLT: 0.00] : Slope of genotype error rate. [error] = [offset] + [1-offset]*[coeff]*[1-r2]
   --r2-info [STR: R2] : INFO field name representing R2 value. Used for representing imputation quality
   --min-mac [INT: 1] : Minimum minor allele frequency
   --min-callrate [FLT: 0.50] : Minimum call rate
   --sm [V_STR: ] : List of sample IDs to compare to (default: use all)
   --sm-list [STR: ] : File containing the list of sample IDs to compare

Output Options
   --out [STR: ] : Output file prefix
   --alpha [V_FLT: ] : Grid of alpha to search for (default is 0.1, 0.2, 0.3, 0.4, 0.5)
   --doublet-prior [FLT: 0.50] : Prior of doublet
   --sam-verbose [INT: 1000000] : Verbose message frequency for SAM/BAM/CRAM
   --vcf-verbose [INT: 10000] : Verbose message frequency for VCF/BCF

Read filtering Options
   --cap-BQ [INT: 40] : Maximum base quality (higher BQ will be capped)
   --min-BQ [INT: 13] : Minimum base quality to consider (lower BQ will be skipped)
   --min-MQ [INT: 20] : Minimum mapping quality to consider (lower MQ will be ignored)
   --min-TD [INT: 0] : Minimum distance to the tail (lower will be ignored)
   --excl-flag [INT: 3844] : SAM/BAM FLAGs to be excluded

Cell/droplet filtering options
   --group-list [STR: ] : List of tag readgroup/cell barcode to consider in this run. All other barcodes will be ignored. This is useful for parallelized run
   --min-total [INT: 0] : Minimum number of total reads for a droplet/cell to be considered
   --min-uniq [INT: 0] : Minimum number of unique reads (determined by UMI/SNP pair) for a droplet/cell to be considered
   --min-snp [INT: 0] : Minimum number of SNPs with coverage for a droplet/cell to be considered
</pre>

#### freemuxlet
<pre>
Options for input pileup
--plp [STR: ] : Prefix of input files generated by dsc-pileup
--init-cluster [STR: ] : Input file containing the initial cluster information

Output Options
--out [STR: ] : Output file prefix
--nsample [INT: 0] : Number of samples multiplexed together
--aux-files [FLG: OFF] : Turn on writing auxiliary output files
--verbose [INT: 100] : Turn on verbose mode with specific verbosity threshold. 0: fully verbose, 100 : no verbose messages

Options for statistical inference
--doublet-prior [FLT: 0.50] : Prior of doublet
--bf-thres [FLT: 5.41] : Bayes Factor Threshold used in the initial clustering
--frac-init-clust [FLT: 0.50] : Fraction of droplets to be clustered in the very first round of initial clustering procedure
--iter-init [INT: 10] : Iteration for initial cluster assignment (set to zero to skip the iterations)
--keep-init-missing [FLG: OFF] : Keep missing cluster assignment as missing in the initial iteration

Read filtering Options
--cap-BQ [INT: 40] : Maximum base quality (higher BQ will be capped)
--min-BQ [INT: 13] : Minimum base quality to consider (lower BQ will be skipped)

Cell/droplet filtering options
--group-list [STR: ] : List of tag readgroup/cell barcode to consider in this run. All other barcodes will be ignored. This is useful for parallelized run
--min-total [INT: 0] : Minimum number of total reads for a droplet/cell to be considered
--min-uniq [INT: 0] : Minimum number of unique reads (determined by UMI/SNP pair) for a droplet/cell to be considered
--min-snp [INT: 0] : Minimum number of SNPs with coverage for a droplet/cell to be considered
</pre>

### Interpretation of output files

#### dsc-pileup
**_dsc-pileup_** generates multiple output file, such as  `[prefix].cel`, `[prefix].var`, `[prefix].plp` and `[prefix].umi`, and these three files would be the input for **_freemuxlet_** and **_demuxlet_**.

* The `[prefix].cel` file contains the relation between numerated barcode ID and barcode. Also, it contains the number of SNP and number of UMI for each barcoded droplet.  
* The `[prefix].plp` file contains the overlapping SNP and the corresponding read and base quality for each barcode ID.
* The `[prefix].var` file contains the position, reference allele and allele frequency for each SNP. 
* The `[prefix].umi` file contains the position covered by each umi.


#### demuxlet/freemuxlet
In this package, both **_demuxlet_** and **_freemuxlet_** generate output file contains the best guess of the sample identity, with detailed statistics to reach to the best guess. It is called  `[prefix].best` for **_demuxlet_** and  `[prefix].clust1.samples.gz` for **_freemuxlet_**

Both `[prefix].best` and `[prefix].clust1.samples.gz` file contains the following 19 columns of content, but `[prefix].clust1.samples.gz` contains one additional INT_ID column showing the numerated BARCODE ID.

 1. BARCODE - Cell barcode for the cell that is being assigned in this row.
 2. NUM.SNPS - The total number of variants overlapping with any read in the droplet.
 3. NUM.READS - The total number of reads overlapping with variant sites for each droplet.
 4. DROPLET.TYPE - Inferred droplet type: {SNG:singlet, DBL:doublet, AMB: ambiguous}.
 5. BEST.GUESS    - The best assignment for sample ID. e.g. <sample ID1>,<sampleID2> for DBL.
 6. BEST.LLK - The log(likelihood that the ID from BEST.GUESS is the correct assignment).
 7. NEXT.GUESS - The next best assignment for sample ID.
 8. NEXT.LLK - The log(likelihood that the ID from NEXT.GUESS is the correct assignment).
 9. DIFF.LLK.BEST.NEXT - The log-likelihood difference between NEXT.LLK and BEST.LLK.
 10. BEST.POSTERIOR - The posterior probability of the best assignment.
 11. SNG.POSTERIOR - The posterior probability of being singlet.
 12. SNG.BEST.GUESS - The best singlet assignment for sample ID.
 13. SNG.BEST.LLK   - The log(likelihood that the ID from SNG.BEST.GUESS is the correct singlet assignment).
 14. SNG.NEXT.GUESS   - The next best singlet assignment for sample ID.
 15. SNG.NEXT.LLK    - The log(likelihood that the ID from SNG.NEXT.GUESS is the correct singlet assignment).
 16. SNG.ONLY.POSTERIOR    - The posterior probabiltiy of the singlet assignment given it is singlet.
 17. DBL.BEST.GUESS   - The best doublet assignment for sample ID.
 18. DBL.BEST.LLK   - The log(likelihood that the ID from DBL.BEST.GUESS is the correct assignment).
 19. DIFF.LLK.SNG.DBL   - The log-likelihood difference between SNG.BEST.LLK and DBL.BEST.LLK.

#### Additional output files from freemuxlet
**_freemuxlet_** generates additional output files, such as `[prefix].clust1.vcf.gz`, `[prefix].lmix`, and optionally `[prefix].clust0.samples.gz`, `[prefix].clust0.vcf.gz` and `[prefix].ldist.gz` (with `--aux-files` argument). Each file contains the following information. As the auxilary files are experimental and may subject to change, use the information at your own risk.

* The `[prefix].clust1.vcf.gz` file is the vcf file for each sample inferred and clustered from `freemuxlet`
* The `[prefix].lmix` file contains basic statistics for each barcode
* The `[prefix].clust0.samples.gz` files contains the best sample identity assuming all droplets are singlets
* The `[prefix].clust0.vcf.gz` files is the vcf similar to `[prefix].clust1.vcf.gz` but assuming all droplets are singlets 
* The `[prefix].ldist.gz` files contains the pairwise Bayes factor for each possible pair of droplets.  
  

