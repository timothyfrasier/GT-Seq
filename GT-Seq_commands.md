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

Before we run this command, however, we need to organize things a bit for better processing and organization.

First, let's make a new directory for our output, and then navigate to it. I'll call it *filtered*.
```
mkdir filtered

cd filtered
```

Then, we need to make a list of all of the directory names that we can use to loop through, but only keep those that refer to samples. We will do this all in one line by piping the results from the `ls` command to a `grep` command that will just keep those lines that start with `Sample`.
```
ls ../ | grep "Sample_*" > files
```

Then we need to create a list of the sample files within each of these directories. To do this, create and open a new file, which we'll call *samples.sh* that will hold a script that we will run to do this.
```
nano samples.sh
```

What we want this script to do is go into each of the directories and list the contents. However, we only want the files that are associated with our samples. These can be differentiated because the have `_R1_` or `_R2_` in them (we just need one though, so we will select those with `R1`). The script should look like this.
```
#!/bin/bash

while read FILES;
do
	ls ../$FILES | grep "_R1_" >> names;
done < files
```

What this will do is list the contents of each of the directories listed in the *files* file, and the pipe those results to `grep`, which will search for just those lines that contain either `_R1_`. The output will be concatenated into a new file called *names*.

We next need to make this file an executable file.
```
chmod a+x samples.sh
```

Then we can run it.
```
./samples.sh
```

Take a look at the new *names* file to see that our script worked as expected.
```
less names
```

You will see that it is close to what we need, but that the `.md5` files are also included. We can remove them with the following command, which uses `grep` and the `-v` flag to keep all lines that **do not** contain the search term.
```
grep -v ".md5" names > names_kept
```

Take a look at this file and make sure it looks how you expect it to. You should have one line for each sample.
```
less names_kept

wc -l names_kept
```

For our next script to work, we need to capture just the part of the sample names prior to the `_R1`. There are probably a few ways that we can do this, but I will use `sed`. First I will replace `_R1_` with a tab.
```
sed -i "s/_R1_/\t/g" names_kept
```

Check that this did what you expected.
```
head names_kept
```

Then use the `cut` command to just keep the first column.
```
cut -f 1 names_kept > names_used
```

Check that this did what you expected.
```
head names_used
```


What we need to do next is write a script that will go into each of these directories and run **fastp** to perform the necessary filtering and merging actions, but then store the results in our new *filtered* directory.

First, create and open a new file (we'll call it "processing.sh") within which we will write our script.
```
nano processing.sh
```

Then, write the following lines within that file. Note that some of these names may change over time, so you should check every time and change as needed.
```
#!/bin/bash

#--- Loop through each sample in the file ---#
while read SAMPLE;
do
  SUB=$(echo "$SAMPLE" | awk -F _S '{ print $1 }')
  
	fastp --detect_adapter_for_pe --merge --merged_out ${SUB}_merged.fastq.gz --out1 ${SUB}_out1.fastq.gz --out2 ${SUB}_out2.fastq.gz --correction --dont_eval_duplication --average_qual 30 --trim_poly_g --trim_poly_x --html ${SUB}_test.html --thread 2 --in1 ../Sample_${SUB}/${SAMPLE}_R1_001.fastq.gz --in2 ../Sample_${SUB}/${SAMPLE}_R2_001.fastq.gz;
done < names_used
```

This is a bit complicated, so let's walk through it. The names being passed to this loop (from our *names_used* file) have the following format: `EGL00003-2_FRA2523A5-1_S46_L001`. This is the prefix for the read files for each sample. For this sample, they would be `EGL00003-2_FRA2523A5-1_S46_L001_R1_001.fastq.gz` and `EGL00003-2_FRA2523A5-1_S46_L001_R2_001.fastq.gz`. So we will need to add on the latter bits when we refer to the specific files. However, the directories that each of these file names is in is the first part of the file name (up until the `_S46` for this sample, but the number is different for different samples), but with `Sample_` as a prefix. So, for this sample, the directory that the files are in is called `Sample_EGL00003-2_FRA2523A5-1`.

The `while read SAMPLE...< names_used` part of this script is saying that for every line in the `names_used` file, refer to the text in that line as `SAMPLE`.

The `SUB=$(echo "$SAMPLE" | awk -F _S '{ print $1 }')` line uses the `awk` command to separate the text for each line at the point where there is a `_S`. The text *before* that occurs is considered the first column of the text, and we are printing that text to a new variable that we are calling `SUB`. In this way, for each step in the loop we have the general sample name `SUB`, as well as the fuller read file name other than the `_R1_001.fastq.gz` and `_R2_001.fastq.gz` suffixes, which is referred to as `SAMPLE`. 

The `fastp` line then calls **fastp**, with all of the arguments discussed above, naming the output files after the shorter sample name `SUB`, and then using these variable prefixes to process the correct files in the correct directories.


Then, make this script and executable file by typing the command below.
```
chmod a+x processing.sh
```

Then you can let it rip by typing
```
./processing.sh
```
   

## References
Andrews S. (2010). FastQC: a quality control tool for high throughput sequence data. Available online at: http://www.bioinformatics.babraham.ac.uk/projects/fastqc.

Chen S, Zhou Y, Chen Y, Gu J (2018) fastp: an ultra-fast all-in-one FASTQ preprocessor. *Bioinformatics* **34**: i884--i890.

Ewels P, Magnusson M, Lundin S, Kaller M (2016) MultiQC: summarize analysis results for multiple tools and samples in a single report. *Bioinformatics* **32**: 3047-3048.
