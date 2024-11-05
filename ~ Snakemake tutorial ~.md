
#### Directory Structure

my_pipeline/
	└── config.yaml                       # Configuration file
	└── Snakefile                           # Main Snakemake file
	└── scripts/
		   └── clean_data.py       # Python script to clean data
	└── data/
		   └── sample1.tsv           # Sample data files (more will be downloaded)
	└── results/
		   └── cleaned/                 # Directory for cleaned files
		   └── merged_data.tsv   # Final output file

**Step 1: Config File (config.yaml)**

The config.yaml file is a convenient way to store parameters or paths that may change, allowing you to adjust them without modifying the Snakefile.

```yaml
data_dir: "data"                # Directory to store raw data files
cleaned_dir: "results/cleaned"  # Directory to store cleaned data files
output_dir: "results"           # Directory to store the final output file

samples:
  - "sample1"
  - "sample2"
  - "sample3"
```
  
**Step 2: Python Script for Cleaning Data (scripts/clean_data.py)**

This script will take an input file and produce a cleaned output file. 

```python
#!/usr/bin/env python

import sys

input_file = sys.argv[1]
output_file = sys.argv[2]

# Simulate cleaning the data
with open(input_file, 'r') as infile, open(output_file, 'w') as outfile:
    for line in infile:
        # Example cleaning step: remove leading/trailing whitespace
        cleaned_line = line.strip()
        outfile.write(cleaned_line + "\n")
```

**Step 3: Write the Snakefile**

This file defines the rules for each step of the pipeline. We’ll use wildcards to handle multiple files dynamically and config.yaml to make the pipeline configurable.

```python
# Load configuration
configfile: "config.yaml"

# Rule to download sample data files (simulated here as creating empty files)
rule download_data:
    output:
        expand("{data_dir}/{sample}.tsv", data_dir=config["data_dir"], sample=config["samples"])
    shell:
        """
        # Simulating download by creating empty files
        for f in {output}; do touch $f; done
        """

# Rule to clean data files
rule clean_data:
    input:
        "{data_dir}/{sample}.tsv"
    output:
        "{cleaned_dir}/{sample}_cleaned.tsv"
    params:
        cleaned_dir=config["cleaned_dir"]
    script:
        "scripts/clean_data.py {input} {output}"

# Rule to merge all cleaned files into one
rule merge_data:
    input:
        expand("{cleaned_dir}/{sample}_cleaned.tsv", cleaned_dir=config["cleaned_dir"], sample=config["samples"])
    output:
        "{output_dir}/merged_data.tsv"
    params:
        output_dir=config["output_dir"]
    shell:
        """
        cat {input} > {output}
        """

# Define the default target (final output)
rule all:
    input:
        "{output_dir}/merged_data.tsv"
```

**Explanation of the Snakefile**


• configfile: "config.yaml" loads the configuration from config.yaml.

• **Rule** download_data: This rule simulates downloading data files by creating empty files in the data directory. It uses expand() to generate file names dynamically based on the sample names in config.yaml.

• **Rule** clean_data: This rule calls the clean_data.py script to process each sample file. {sample} is a wildcard that matches each sample name from config.yaml.

• **Rule** merge_data: This rule merges all cleaned data files into a single file. The expand() function generates the list of cleaned files, and they are concatenated using the cat command.

• **Rule** all: This is the default rule, defining the final target as the merged file. Running snakemake will execute this rule and trigger all dependent rules.


**Step 4: Run the Pipeline**

1. **Initialize the pipeline**: Navigate to the my_pipeline directory.

2. **Run Snakemake**: Execute the following command to start the pipeline:

```bash
snakemake --cores 1
```

3. **Dry Run**: To see the planned steps without running them, use:

```bash
snakemake --n
```

4. **Force**: Force the creation of output file even if it already exists

  ```bash
snakemake --n --force
```

5. **Additional Command-Line Options for Flexibility**: 

```bash
snakemake --cores 1 --config data_dir="custom_data" output_dir="custom_results"
```

  
Snakemake allows you to override config file parameters directly from the command line if needed. For example:

## **Summary of Key Snakemake Concepts**
  
• **Wildcards**: {sample} is a wildcard that allows Snakemake to handle multiple files dynamically. Use clear wildcard names that describe the variable (e.g., {sample}, {chromosome}).

• **Config Files**: config.yaml stores paths and variables, making the pipeline more flexible and maintainable.

• **Rules**: Each rule defines a step in the pipeline with input, output, and the command to run (shell or script). Use intermediate files if you have multi-step processing, but mark them with temp() if you don’t need them after the pipeline finishes. This helps Snakemake manage storage efficiently by automatically deleting temporary files.

• expand(): Generates multiple file names based on patterns and lists, useful for batch processing.

• Use resources to define custom requirements (e.g., resources: mem_mb=1000 for memory).

• Add threads to specify how many CPU cores each rule should use.

• Use log files in each rule to capture standard output and error (stdout and stderr).

• Organizing large workflows into separate Snakefiles, using include: "other_file.smk" to import them. This is helpful if you have multiple sub-workflows that can be shared across projects.

• **Debugging**: Use --printshellcmds to see what shell commands are being run, and -p to print each command as it’s executed. Both can help you diagnose issues when they occur.

• Use snakemake --report report.html to create an HTML report that summarizes the run, showing each rule’s input, output, and runtime.

In a normal Snakemake rule, all inputs and outputs must be defined at the start of the workflow execution. However, if some inputs depend on the output of a previous step and we don't know about it, this becomes a **dynamic dependency** that Snakemake cannot resolve at the start. **Checkpoint rules** provide a way to handle this by allowing you to dynamically re-evaluate the workflow after the checkpoint rule completes.
