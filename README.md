# abstract
Detailed transcription maps of bacteriophages are not usually explored, limiting our understanding of molecular phage biology and restricting their exploitation and engineering. With ONT-cappable-seq, primary transcripts are specifically capped, enriched, and prepared for long-read sequencing on the nanopore sequencing platform. This enables end-to-end sequencing of unprocessed transcripts covering both phage and host genome, thus providing insight on their operons. The subsequent analysis pipeline makesit possible to rapidly identify the most important transcriptional features such as transcription start and stop sites. The obtained data can thus provide a comprehensive overview of the transcription by your phage and/or bacterium of interest.

# ONT-cappable-seq2

We describe here a general overview of the updated ONT-cappable-seq data analysis pipeline, which is now fully automated. For a more detailed overview of individual steps, see the old ONT-cappable-seq repository. The automated ONT-cappable-seq data analysis tool (v2) is implemented in the workflow management system Snakemake. 

### **I. Installation**
Using Conda, create a separate environment (here called ONT-cappable-seq) where we will install the tools and run the pipeline.

> $ conda create -n ONT-cappable-seq python=3.9
> $ conda activate ONT-cappable-seq

The pipeline uses eight external tools (pychopper, cutadapt, minimap2, samtools, samclip, bedtools, termseq-peaks, r-dplyr). All tools, except for termseq-peaks and dplyr, are automatically installed in their ownenvironment upon running the Snakemake pipeline. In the ONT-cappable-seq environment, navigate to the ONT-cappable-seq2 project folder and manually install the termseq-peaks package using the following commands:

> $ cd ONT-cappable-seq2/
> $ git clone https://github.com/NICHD-BSPC/termseq-peaks
> $ cd termseq-peaks
> $ conda install -c bioconda --file requirements.txt
> $ python setup.py install

Check if the installation worked by typing:
> $ termseq_peaks

Afterwards, install the Snakemake tool and the rdplyr package in the same environment:
> $ conda install -c bioconda -c conda-forge snakemake
> $ conda install -c conda-forge r-dplyr

### **II. Data navigation**


### **III. Setting up the configuration file**
