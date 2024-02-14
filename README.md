# abstract
Detailed transcription maps of bacteriophages are not usually explored, limiting our understanding of molecular phage biology and restricting their exploitation and engineering. With ONT-cappable-seq, primary transcripts are specifically capped, enriched, and prepared for long-read sequencing on the nanopore sequencing platform. This enables end-to-end sequencing of unprocessed transcripts covering both phage and host genome, thus providing insight on their operons. The subsequent analysis pipeline makesit possible to rapidly identify the most important transcriptional features such as transcription start and stop sites. The obtained data can thus provide a comprehensive overview of the transcription by your phage and/or bacterium of interest.

# ONT-cappable-seq2

We describe here a general overview of the updated ONT-cappable-seq data analysis pipeline, which is now fully automated. For a more detailed overview of individual steps, see the old ONT-cappable-seq repository. The automated ONT-cappable-seq data analysis tool (v2) is implemented in the workflow management system Snakemake. 

### **I. Installation**
Using Conda, create a separate environment (here called ONT-cappable-seq) where we will install the tools and run the pipeline.

```bash
conda create -n ONT-cappable-seq python=3.9
conda activate ONT-cappable-seq
```

The pipeline uses eight external tools (pychopper, cutadapt, minimap2, samtools, samclip, bedtools, termseq-peaks, r-dplyr). All tools, except for termseq-peaks and dplyr, are automatically installed in their ownenvironment upon running the Snakemake pipeline. In the ONT-cappable-seq environment, navigate to the ONT-cappable-seq2 project folder and manually install the termseq-peaks package using the following commands:

```bash
cd ONT-cappable-seq2/
git clone https://github.com/NICHD-BSPC/termseq-peaks
cd termseq-peaks
conda install -c bioconda --file requirements.txt
python setup.py install
termseq_peaks #check to see if the installation worked
```

Afterwards, install the Snakemake tool and the rdplyr package in the same environment:
```bash
conda install -c bioconda -c conda-forge snakemake
conda install -c conda-forge r-dplyr
```

### **II. Project navigation and data preparation**
After downloading, the ONT-cappable-seq project folder is organized as follows:

```bash
ONT-cappable-seq2/
├── workflow/
 ├── envs/
 ├── rules/
 └── snakefile
├── termseq-peaks/
├── config/
 └── config.yaml
├── input/
 ├── fastq_data/
 └── genome_data/
└── peak_clustering.r
```

Retrieve high-quality FASTA files of the reference genomes of your organisms of interest and store them in theinput/genome_data directory. For clarity, we recommend to rename the FASTA files to the respective IDs of the phage (or host)(=sampleID). The sequence of the FLuc control plasmid used for in vitro transcription of the RNA control spike-in can be downloaded from the manufacturer’s website 

```bash
ONT-cappable-seq2/
…
├── input/
 └── genome_data/
  ├── phageID.fasta
  ├── hostID.fasta
  └── in-vitro-RNA.fasta
```

Place the raw sequencing files in the input/fastq_data directory and rename them to ensure they contain information on sampleID and treatment (enriched or control sample). For each individual sample, it is important to have a single decompressed.fastq file that contains sufficient reads with an appropriate mean read length (>200 bp).

```bash
ONT-cappable-seq2/
…
├── input/
 ├─ fastq_data/
  ├── sampleID_enriched.fastq
  └── sampleID_control.fastq
└── genome_data/
  ├── phageID.fasta
  ├── hostID.fasta
  └── in-vitro-RNA.fasta
```

### **III. Setting up the configuration file**

Navigate to the config directory in the ONT-cappable-seq project folder and open the config.yaml file. This file contains the paths to your input files and the different parameters used for annotation of the transcriptional boundaries.

where:

–**Sample name**: contains the name of your organism of interest (sampleID, e.g., LUZ19).

–**Fasta file**: contains the path to your reference genome of interest (input/genome_data/sampleID.fasta, e.g.,input/genome_data/LUZ19.fasta).

–**Enriched fastq**: contains the path to the raw sequencing file of your enriched sample (input/fastq_data/sampleID_enriched.fastq, e.g., input/fastq_data/LUZ19_enriched.fastq).

–**Control fastq**: contains the path to the raw sequencing file of your control sample (input/fastq_data/sampleID_control.fastq, e.g., input/fastq_data/LUZ19_control.fastq).

–**ID**: additional identifier to classify your samples (cannot be empty), e.g., timepoint, specific condition, replicate, etc.

–**Termseq alpha**: peak calling threshold value used by the termseq-peak algorithm (-t flag).

–**Cluster width**: peak positions within the specified distance are clustered. The position with the highest number of reads is takenas the representative of the cluster.

–**Minimum coverage**: absolute number of reads required at this position to be considered as candidate TSS/TTS position. Thiscan be specified for the enriched and the control sample separately.

–**Peak alignment error**: positional difference (n) allowed between peak positions identified in the enriched and the control dataset used to calculate the enrichment ratio at a specific genomic position i, based on the read count per million mapped reads (RPM) at that peak position (see Putzeys et al. 2022).

-**TSS threshold**: enrichment ratio value that needs to be surpassed to annotate 5′ peak position as a TSS.

–**TTS threshold**: minimum read reduction to annotate 3′ peak position as TTS, determined by calculating the coverage dropacross the putative TTS, averaged over a 20-bp region up- and downstream the TTS.

–**TSS/TTS sequence extraction**: selection of promoter and terminator region for which the user wants to extract the DNA sequence, defined by the number of up- and downstream nucleotides relative to the TSS and TTS.

It is important that the sampleIDs and names of the FASTA and FASTQ files in the config file are modified according to the input files provided. The default settings for TSS and TTS identification can be adjusted to optimize for your specific organism of interest.

### **IV. Running the Pipeline**
In the terminal, change directory to the ONT-cappable-seq project folder and execute the Snakemake pipeline, which will run according to the settings specified in the configuration file.

```bash
cd ONT-cappable-seq2
snakemake --cores N --use-conda
```
where N is the number of cores available on your computer to run the pipeline. In-house, generally 8 cores are used. All commandline options can be printed by calling Snakemake -h.

### **V. Result Interpretation**

After running the pipeline, you can find the results in the newly created Results directory, which contains the subdirectories processed_fastq, alignments, and transcript_boundaries.

Notes:

1. The raw sequencing files of the enriched and control sample are processed in two stages, in which they are first oriented with pychopper and trimmed using cutadapt. The **processed sequencing FASTQ (.fq) files** and tool-specific metadata can be retrieved in their respective processed_fastq folder.
   
2. The **sorted alignment files (.BAM)** together with their index files (.BAI) of the enriched and the control samples can be found in the alignments/BAM_files_sampleID folder. These files can be uploaded in Integrative Genomics Viewer (IGV) to visually inspectthe full-length transcriptome of the input genome/sequence you provided.
   
3. For each sampleID and additional ID provided, **TSS identification results** are deposited in their respective TSS_sampleID_ID subfolder within the transcript_boundaries folder. This folder contains a .BED file and FASTA (.fa) file with the promoter regionsand sequences associated with the identified TSS, respectively, as specified in the config file. In addition, one can find two strand-specific .CSV files that specify the absolute and relative read counts of each annotated TSS position on the specified
strand, as well as the enrichment ratio.

4. Similarly, **TTS identification results** are deposited in their respective TTS_sampleID_ID subfolder within the transcript_boundaries folder. This folder contains a .BED file and FASTA (.fa) file that respectively contain the terminator regions and sequences associated with the TTS positions defined by ONT-cappable-seq, as specified in the config file. In addition, two strand-specific .CSV files can be found, which indicate the read count reduction across each annotated TSS position on the specified strand.
   
