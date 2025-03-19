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

## References
Andrews S. (2010). FastQC: a quality control tool for high throughput sequence data. Available online at: http://www.bioinformatics.babraham.ac.uk/projects/fastqc.

Ewels P, Magnusson M, Lundin S, Kaller M (2016) MultiQC: summarize analysis results for multiple tools and samples in a single report. *Bioinformatics* **32**: 3047-3048.
