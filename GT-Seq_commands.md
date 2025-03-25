# Analysing GT-Seq Data

## First Look
Sequences will come back from the sequencing facility with the results for each sample located in a different directory, named based on the sample names. In addition to the actual sequences, McGill also runs **fastqc** (Andrews 2010) on the files, and provides the resulting files.

Each of these directories will contain **10 files**:     
    1-2. The `fastq.gz` files for the two index reads (one for each direction of the paired-end reads, "I1" and "I2").     
    3-4. The `fastq.gz` files for the forward and reverse reads for that sample ("R1" and "R2").     
    5-6. The `.md5` files for the forward and reverse reads, for ensuring that transfered files are correct.     
    7-8. The **fastqc** files for the reads of the two indices.     
    9-10. The **fastqc** files for the forward and reverse reads.      

The **fastqc** files can be looked at independently, and provide a range of information on the quality of the reads for each sample. However, we don't want to click on a different file for each sample, so we will use available tools (the program **multiqc** (Ewels et al. 2016)) to summarize the information across all samples. 

To run **multiqc**, navigate into the parent directory containing the different directory for each sample. Then type the following command. Note that we only want to evaluate the quality information for the samples themselves, not the indexes, so we'll ignore those.
```
multiqc . --ignore "*_I1_*" --ignore "*_I2_*"
```

This will search all subdirectories for quality information (**fastqc** files in this case) and summarize the resulting information. The summary information will be presented in several formats that are useful. First, a `html` file will be created that summarizes all of the information. It is worth transferring this to your own computer and taking a look through it to see how each sample performed. 

However, one thing that we are really interested in is the number of reads per sample, and this is summarized only broadly in the `html` file. We can actually view the underlying data by navigating into the `multiqc_data` directory that is created, and then typing the following command. This will show us the total number of reads for each sample, as well as the number poor quality reads for each.
```
less multiqc_fastqc.txt | cut -f 1,5,6
```

Make note of any samples for which it looks like there were sequencing issues, such as low read counts or any other issues.


## Preparing Samples for Genotyping
There are a few things that we need to do next, that we can luckily all do at the same time using the program **fastp** (Chen et al. 2018). Below are some of the arguments that we will use, and an explanation of what they do.     

     * `-merge`: for paired-end input, merge each pair of reads into a single read if they are overlapped.     
     * `-merged_out`: specifies a file in which to store the merged reads.     
     * `-out1` `-out2`: Reads that can't be successfully merged, but which pass all filters.     
     * `-dont_eval_duplication`: We expect a lot of duplicates because these are amplicons, so we will disable evaluation of duplicates.     
     * `-detect_adapter_for_pe`: The adapter sequence auto-detection is enabled for single-end data only. Turn on this option to enable it for paired-end data.     
     * `average_qual 30`: Set the quality score threshold for reads. If read's average quality score is less than 30, read will be discarded.     
     * `thread 2`: Use 2 threads (default is 3)
     * `-correction`: Enable base correction in overlapped regions (only for paired-end data), default is disabled.     
     * `-trim_poly_x`: Enable polyX trimming in 3' ends. This setting is useful for trimming the tails that have poliX (e.g., polyA).     
     * `-trim_poly_g`: Force polyG tail trimming. By default trimming is automatically enabled for Illumina NextSeq/NovaSeq data.     


   

## References
Andrews S. (2010). FastQC: a quality control tool for high throughput sequence data. Available online at: http://www.bioinformatics.babraham.ac.uk/projects/fastqc.

Chen S, Zhou Y, Chen Y, Gu J (2018) fastp: an ultra-fast all-in-one FASTQ preprocessor. *Bioinformatics* **34**: i884--i890.

Ewels P, Magnusson M, Lundin S, Kaller M (2016) MultiQC: summarize analysis results for multiple tools and samples in a single report. *Bioinformatics* **32**: 3047-3048.
