# Workflow details for the _Candida Albicans_ RNA sequence analysis
## Upstream workflow
### Data Accession 
Data for Candida Albicans was obtained using GTF and FNA files from NCBI's genome database. The link to access these datasets is [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000182965.3/) .
Before cleaning, FastQC was performed on forward and reverse reads to use later as comparison of how effective cleaning was. For reference here are the FastQC reads before cleaning was performed: [Forward read](https://www.dropbox.com/scl/fi/9h4zneet703n2zmz2ndh8/WTA2_1_fastqc-1.html?rlkey=954we5578wmit1ganxf5a49gj&st=m9u4tygp&dl=0) ...
[Reverse read](https://www.dropbox.com/scl/fi/rr2k5qlw67rpk8niwkgmb/WTA2_2_fastqc-1.html?rlkey=288ugauug8pjvu510f5zumrgs&st=yveofmbb&dl=0) .
### Preprocessing and Quality Control 
All preprocessing and quality control was performed on SSH key of google cloud grapes controller. Trimmomatic (version 0.39) was used to trim reads based on quality. 
Trimmomatic script: [linked here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/trimmomatic_run1) .
We removed specific [adapters sequences](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/TruSeq3-PE_adapter_sequence) for Illumina. We used trimmomatic filters of:
ILLUMINACLIP:2:30:10 to trim adapter sequences. 
HEADCROP:10 to remove the first 10 bases. 
TRAILING:10 to remove the last 10 bases. 
SLIDINGWINDOW:4:15 to improve data quality. This scans over 4 bases at a time, and if the quality score is not at least an average of 15, trimmomatic will trim that read.
MINLEN:75 to ensure that only reads with at least 75 bases are kept after trimming.
FastQC (version 0.12.0) was then performed on the forward and reverse trimmed paired-end reads to visualize the quality control of data.
Output for [forward paired ends trimmomatic read](https://www.dropbox.com/scl/fi/rzh0t2qiaelko2gv9n4ik/output_R1_trPE_fastqc.html?rlkey=gij3a5ajcvy38s1kumq2lexzm&st=ru98zeuk&dl=0). Output for [reverse paired ends trimmomatic read](https://www.dropbox.com/scl/fi/ids13arvlixc2rd8ohebt/output_R2_trPE_fastqc.html?rlkey=yma67idaneljhaotspfmzyw33&st=uo0bpm9m&dl=0).
DESCRIBE FINDINGS HERE.
### Sequence Mapping
Bowtie2 (version 2.5.3) was used on forward and reverse paired-end reads obtained from trimmomatic to map the data to reference sequence. (Reference sequence)[https://api.ncbi.nlm.nih.gov/datasets/v2/genome/accession/GCF_000182965.3/download?include_annotation_type=GENOME_FASTA&include_annotation_type=GENOME_GFF&include_annotation_type=RNA_FASTA&include_annotation_type=CDS_FASTA&include_annotation_type=PROT_FASTA&include_annotation_type=SEQUENCE_REPORT&hydrated=FULLY_HYDRATED] GCF_000182965.3 for SC5314 genome assembly was obtained from NCBI genome datasets. 
[Bowtie 2 script](https://github.com/graceobrien2002/RNAseqProject/blob/e8db7edae80fbf20142463c10ea4e06051486a41/scripts1/bowtie_script) used the forward and reverse trimmomatic-cleaned reads as input, with the output being a .sam file. 
DESCRIBE FINDINGS HERE. LINK OUTPUT HERE.

### Counting Reads per Gene Model
Using HTseq (VERSION)

## Downstream workflow
All downstream analyses done in R (VERSION)
Using DESeq2 (VERSION) LINK SCRIPT HERE.
