# Workflow details for the _Candida Albicans_ RNA sequence analysis
## Goals and Experimental Setup
The primary goal of this analysis was to identify genes that are differentially expressed in response to the presence or absence of thiamine in _Candida Albicans_ growth environment. The datasets include RNA sequencing reads from samples treated with thiamine and samples grown without thiamine. By comparing the gene expression from both conditions, the analysis aims to reveal what genes are necessary for survival in an environment lacking thiamine. Normalized read counts were used for statistical analysis. 
### Research question:
How does the presence or absence of thiamine in the growth environment of _Candida Albicans_ affect the expression of genes? What genes are upregulated in a thiamine-deprived growth environment?
### Hypothesis: 
The absence of thiamine in _Candida Albicans_ growth environment induces differential gene expression, with specific genes being upregulated or downregulated to adapt to the thiamine deficiency. This hypothesis reflects the expectation that _Candida Albicans_ can activate certain metabolic pathways to compensate for the lack of thiamine, revealing survival mechanisms in the presence of environmental stressors. 

## Upstream workflow
### Data Accession 
Genomic data for _Candida Albicans_ grown in the presence and absence of thiamine was obtained using GTF and FNA files from NCBI's genome database. The link to access these datasets is [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000182965.3/) . The .fna file is linked [here](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/182/965/GCF_000182965.3_ASM18296v3/GCF_000182965.3_ASM18296v3_cds_from_genomic.fna.gz). The .gtf file is linked [here](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/182/965/GCF_000182965.3_ASM18296v3/GCF_000182965.3_ASM18296v3_genomic.gtf.gz). 
Before cleaning, FastQC was performed on forward and reverse reads to use later as comparison of how effective cleaning was. For reference here are the FastQC reads before cleaning was performed: [Forward read](https://www.dropbox.com/scl/fi/9h4zneet703n2zmz2ndh8/WTA2_1_fastqc-1.html?rlkey=954we5578wmit1ganxf5a49gj&st=m9u4tygp&dl=0) ...
[Reverse read](https://www.dropbox.com/scl/fi/rr2k5qlw67rpk8niwkgmb/WTA2_2_fastqc-1.html?rlkey=288ugauug8pjvu510f5zumrgs&st=yveofmbb&dl=0) .
### Preprocessing and Quality Control 
All preprocessing and quality control was performed on SSH key of google cloud grapes controller. Trimmomatic (version 0.39) was used to trim reads based on quality. 

#### Trimmomatic
Trimmomatic script: [linked here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/trimmomatic_run1) .

We removed specific [adapters sequences](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/TruSeq3-PE_adapter_sequence) for Illumina. We used trimmomatic filters of:
- ILLUMINACLIP:2:30:10 to trim adapter sequences. 
- HEADCROP:10 to remove the first 10 bases. 
- TRAILING:10 to remove the last 10 bases. 
- SLIDINGWINDOW:4:15 to improve data quality. This scans over 4 bases at a time, and if the quality score is not at least an average of 15, trimmomatic will trim that read.
- MINLEN:75 to ensure that only reads with at least 75 bases are kept after trimming.

FastQC (version 0.12.0) was then performed on the forward and reverse trimmed paired-end reads to visualize the quality control of data.
Output for [forward paired ends trimmomatic read](https://www.dropbox.com/scl/fi/rzh0t2qiaelko2gv9n4ik/output_R1_trPE_fastqc.html?rlkey=gij3a5ajcvy38s1kumq2lexzm&st=ru98zeuk&dl=0). Output for [reverse paired ends trimmomatic read](https://www.dropbox.com/scl/fi/ids13arvlixc2rd8ohebt/output_R2_trPE_fastqc.html?rlkey=yma67idaneljhaotspfmzyw33&st=uo0bpm9m&dl=0).

| FILE           | # reads before cleaning | # reads AFTER cleaning | % retained after cleaning | per-base sequence quality AFTER cleaning | per-base sequence content before cleaning | per-base sequence content AFTER cleaning | sequence duplication before cleaning | sequence duplication AFTER cleaning | Did minor adapter contamination go away AFTER cleaning (Y/N)? |
|----------------|--------------------------|-------------------------|---------------------------|------------------------------------------|--------------------------------------------|--------------------------------------------|-------------------------------------|-------------------------------------|------------------------------------------------------------|
| WTA2_1.fq.gz   | 21,972,519              | 21,037,734             | 0.9574566303             | G                                        | B                                          | G                                          | B                                   | B                                   | Y                                                          |
| WTA2_2.fq.gz   | 21,972,519              | 21,037,734             | 0.9574566303             | G                                        | B                                          | G                                          | B                                   | B                                   | Y                                                          |


#### Interpretations of Trimmomatic outputs
Prior to cleaning, per-base sequence content and sequence duplication levels were flagged as concerning by FastQC. Per base sequence quality and per sequence GC content were also flagged as possibly concerning. 
After running Trimmomatic, all of these areas were improved except for sequence duplication. It is important to note that sequence duplication may represent highly expressed transcripts, and de-duplicating may skew the relative proportions of transcripts and thus impact interpretations. For RNAseq reads, it is the standard to not de-duplicate while cleaning reads. Also, adapter contamination was eliminated during cleaning with Trimmomatic. You can find a [master Google sheet](https://docs.google.com/spreadsheets/d/1AOa-XaTzR_PKMIRQDmu8oDTmawXXnkIwEjKOQkNC7Vs/edit?gid=0#gid=0) that includes details of the number of reads per library. Most strains retained about 95% of genomic data after cleaning. 

#### Bowtie
Bowtie2 (version 2.5.3) was used on forward and reverse paired-end reads obtained from Trimmomatic to align the data to a reference 'de novo' sequence. [Reference sequence](https://api.ncbi.nlm.nih.gov/datasets/v2/genome/accession/GCF_000182965.3/download?include_annotation_type=GENOME_FASTA&include_annotation_type=GENOME_GFF&include_annotation_type=RNA_FASTA&include_annotation_type=CDS_FASTA&include_annotation_type=PROT_FASTA&include_annotation_type=SEQUENCE_REPORT&hydrated=FULLY_HYDRATED) GCF_000182965.3 for SC5314 genome assembly was obtained from NCBI genome datasets. You can find a [master Google sheet](https://docs.google.com/spreadsheets/d/1fa-FXVMlCXOZkbHSx_mMg0OXLMy9BeBJg8uWrEMpKGo/edit?gid=0#gid=0) that shows the bowtie alignment results for all of the data files analyzed by fellow students. About 89% of data from the files were aligned concordantly exactly 1 time, while about 6% were aligned concordantly more than one time. The overall alignment rate was about 98% for all of the data files. 

[Bowtie 2 script](https://github.com/graceobrien2002/RNAseqProject/blob/e8db7edae80fbf20142463c10ea4e06051486a41/scripts1/bowtie_script) used the forward and reverse Trimmomatic-cleaned reads as input, with the output being a .sam file. 
LINK HTML OUTPUT HERE. DESCRIBE FINDINGS HERE. 
#### BAM File Processing 
Once the .SAM file was generated from Bowtie2, it was converted into a .BAM file and further processed using Samtools (version 1.17). These steps were necessary to ensure the file was ready for downstream analysis. The .SAM file was converted to a .BAM file using the command: Samtools view -S -b WTA2.sam > WTA2.bam on the HPC terminal. The resulting .BAM file was sorted to arrange the genes by their genomic coordinates using the command: Samtools sort WTA2.bam -o sorted_reads.bam. An index file was created using the command: Samtools index sample.srt.bam. 

### Counting Reads per Gene Model using HTSeq
Using HTseq (version 2.0.3), the reads were quantified and analyzed by assigning aligned reads from the .bam to annotated genes from the .gtf file. The SBATCH script used for HTSeq is linked [here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/SBATCH_HTSeq). The output HTSeq files for all of the strains of _C. Albicans_ is linked [here](https://www.dropbox.com/scl/fi/0yr4fqgl82yem36ad4co7/htseq_counts.zip?rlkey=9cgdpvpf38oqnf21xws6r05lq&st=xancdhul&dl=0), which shows the gene counts. 

## Downstream workflow

### DESeq analysis
All downstream analyses done in R (VERSION).
Using DESeq2 (VERSION) LINK SCRIPT HERE, differential expression analysis was performed. 

### Gene Ontology enrichment analysis


# Biological analysis of data

