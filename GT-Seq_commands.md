# Analysing GT-Seq Data
Sequences will come back from the sequencing facility with the results for each sample located in a different directory, named based on the sample names.

To run **multiqc**, navigate into the parent directory containing the different directory for each sample. Then type the following command.
```
multiqc .
```

This will search all subdirectories for quality information (**fastqc** files in this case) and summarize the resulting information. The summary information will be presented in several formats that are useful. First, a `html` file will be created that summarizes all of the information. It is worth transferring this to your own computer and taking a look through it to see how each sample performed. 

However, one thing that we are really interested in is the number of reads per sample, and this is summarized only broadly in the `html` file. We can actually view the underlying data by navigating into the `multiqc_data` directory that is created, and then typing the following command. This will show us the total number of reads for each sample, as well as the number poor quality reads for each.
```
less multiqc_fastqc.txt | cut -f 1,5,6
```
