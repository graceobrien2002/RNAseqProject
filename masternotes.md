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
All preprocessing and quality control was performed on SSH key of Google cloud grapes controller. Trimmomatic (version 0.39) was used to trim reads based on quality. 

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


##### Interpretations of Trimmomatic outputs
Prior to cleaning, per-base sequence content and sequence duplication levels were flagged as concerning by FastQC. Per base sequence quality and per sequence GC content were also flagged as possibly concerning. 
After running Trimmomatic, all of these areas were improved except for sequence duplication. It is important to note that sequence duplication may represent highly expressed transcripts, and de-duplicating may skew the relative proportions of transcripts and thus impact interpretations. For RNAseq reads, it is the standard to not de-duplicate while cleaning reads. Also, adapter contamination was eliminated during cleaning with Trimmomatic. You can find a [master Google sheet](https://docs.google.com/spreadsheets/d/1AOa-XaTzR_PKMIRQDmu8oDTmawXXnkIwEjKOQkNC7Vs/edit?gid=0#gid=0) that includes details of the number of reads per library. Most strains retained about 95% of genomic data after cleaning. 

#### Bowtie
Bowtie2 (version 2.5.3) was used on forward and reverse paired-end reads obtained from Trimmomatic to align the data to a reference 'de novo' sequence. [Reference sequence](https://api.ncbi.nlm.nih.gov/datasets/v2/genome/accession/GCF_000182965.3/download?include_annotation_type=GENOME_FASTA&include_annotation_type=GENOME_GFF&include_annotation_type=RNA_FASTA&include_annotation_type=CDS_FASTA&include_annotation_type=PROT_FASTA&include_annotation_type=SEQUENCE_REPORT&hydrated=FULLY_HYDRATED) GCF_000182965.3 for SC5314 genome assembly was obtained from NCBI genome datasets. Sequence alignment maps (.SAM files) were the output of running Bowtie2 using this 
[Bowtie 2 script](https://github.com/graceobrien2002/RNAseqProject/blob/e8db7edae80fbf20142463c10ea4e06051486a41/scripts1/bowtie_script) used the forward and reverse Trimmomatic-cleaned reads as input. Index sequences were made, and samtools were used on the compute node to convert the alignment map into a sorted file, as detailed in the BAM file processing below. 

| Alignment file (.sam) | # cleaned reads after Trimmomatic | # read pairs detected by Bowtie2 | % aligned concordantly exactly 1 time | % aligned concordantly > 1 time | Overall alignment rate |
|------------------------|-----------------------------------|-----------------------------------|----------------------------------------|----------------------------------|-------------------------|
| WTA2.sam              | 21,037,734                       | 21,037,734                       | 88.79%                                 | 7.15%                           | 98.39%                 |

You can find a [master Google sheet](https://docs.google.com/spreadsheets/d/1fa-FXVMlCXOZkbHSx_mMg0OXLMy9BeBJg8uWrEMpKGo/edit?gid=0#gid=0) that shows the bowtie alignment results for all of the data files analyzed by fellow students. About 89% of data from the files were aligned concordantly exactly 1 time, while about 6% were aligned concordantly more than one time. The overall alignment rate was about 98% for all of the data files. 

#### BAM File Processing 
Once the .SAM file was generated from Bowtie2, it was converted into a binary format - .BAM file - and further processed using Samtools (version 1.17). These steps were necessary to ensure the file was ready for downstream analysis. The .SAM file was converted to a .BAM file using the command: Samtools view -S -b WTA2.sam > WTA2.bam on the HPC terminal. The resulting .BAM file was sorted to arrange the genes by their genomic coordinates using the command: Samtools sort WTA2.bam -o sorted_reads.bam. An index file was created using the command: Samtools index sample.srt.bam. 

### Counting Reads per Gene Model using HTSeq
Using HTseq (version 2.0.3), the reads were quantified and analyzed by assigning aligned reads from the .bam to annotated genes from the .gtf file. The SBATCH script used for HTSeq is linked [here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/SBATCH_HTSeq). The output HTSeq files for all of the strains of _C. Albicans_ is linked [here](https://www.dropbox.com/scl/fi/0yr4fqgl82yem36ad4co7/htseq_counts.zip?rlkey=9cgdpvpf38oqnf21xws6r05lq&st=xancdhul&dl=0), which shows the gene counts. 

## Downstream workflow

### DESeq analysis
All downstream analyses done in R Studio (Version 3.6.0). 
Using DESeq2 (Version 3.20) differential expression analysis was performed using [this script](https://github.com/graceobrien2002/RNAseqProject/commit/3ebf9a866b5af5965c7af0c6eced0322795b27f2). DESeq2 used the 6 HTSeq output files (LINK HERE) to analyze differential expression of various genes, filtered for the occurrence of greater than 10 reads across all libraries of strains that were cultivated in environments with and without thiamine. 13 genes were found to be significant, linked in this [file](https://docs.google.com/spreadsheets/d/1NIFbaL8KOtdYOHXUA1CmjdkcB021oKFD6jgzvrTFf4I/edit?gid=116329557#gid=116329557). 

### Gene Ontology enrichment analysis
The names and numerical IDs of significant genes were obtained from the GTF file using the command: grep -wFf signif_geneIDs GCF_000182965.3_ASM18296v3_genomic.gtf | grep "protein_coding" | cut -f9 | cut -d ";" -f1,3,5 > signif_gene_annot_info. Using the gene names or numerical ID's, research was conducted from the [_C. Albicans_ genome database](http://www.candidagenome.org/), [UniProt](https://www.uniprot.org/), and [NCBI Datasets](https://www.ncbi.nlm.nih.gov/datasets/taxonomy/5476/) to determine the biological function and significance of these genes and their involvement in thiamine starvation. Annotations were put into the [google sheet](https://docs.google.com/spreadsheets/d/1NIFbaL8KOtdYOHXUA1CmjdkcB021oKFD6jgzvrTFf4I/edit?gid=116329557#gid=116329557) with the 13 significant genes - these genes were deemed significant as they experienced increased expression in an environment lacking thiamine. While 13 genes were significant, 4 were uncharacterized, so databases may lack information about the function of these genes.  

# Results and Biological analysis of data
## PCA Plot
![PCA plot](https://github.com/graceobrien2002/RNAseqProject/blob/main/outputs/TH-vTH+_pcaplot.png?raw=true)
Figure 1. PCA Plot for significant differentially expressed _C. Albicans_ genes in the presence or absence of Thiamine. The x-axis PC1 shows the largest proportion of variance in the dataset, while the y-axis PC2 captures the second-largest proportion of variance in the dataset. The blue dots represent the THI+ (thymine present in growth media) samples, which are clustered in the top right quadrant. The red dots represent the THI- (thymine absent in growth media) which are clustered in the bottom left quadrant.

The x-axis PC1 captures the largest proportion of variance in the data, which is 88%, while PC2 (y-axis) captures the second largest proportion of variance which is 9%. Each sample is represented by a blue dot (THI+) or a red dot (THI-). The THI+ samples are clustered tightly in the top right quadrant which indicates high similarity in gene expression profiles across replicates under the presence of thiamine. The THI- samples are clustered tightly in the bottom left quadrant, which suggests distinct gene expression patterns in the absence of thiamine. The expression of the genes in this plot is likely very sensitive to the presence/absence of thiamine. 

## Volcano Plot
![Volcano plot](https://github.com/graceobrien2002/RNAseqProject/blob/main/outputs/volcano_TH-vTH_.png?raw=true)
Figure 2. Volcano plot for significant differentially expressed _C. Albicans_ genes in the presence or absence of Thiamine. The y-axis - Log2 fold change - represents the magnitude of change in gene expression between the two conditions - THI+ or THI-. The y-axis - negative log10 of the P-value represents statistical significance. 

The genes to the right of the dashed lines indicate upregulated genes in the absence of thiamine. Genes to the left (negative x-values) are downregulated in the absence of thiamine. Higher y-values represent greater significance (smaller p-values). So, based on these criteria, 13 genes (13 red dots in the volcano plot) were found to be upregulated in a thiamine-deprived environment. 

## Gene Ontology
#### Adaptation to Nutrient Limitation 
Many of the 13 significantly upregulated genes play roles in vitamin biosynthesis pathways, particularly those that are related to thiamine and pyridoxine (vitamin B6). This shows that _C. Albicans_ can adapt to thiamine depletion. SNZ1 and SNO1 are involved in pyridoxine metabolism, which emphasizes a metabolic shift to sustain cellular processes in the absence of Thiamine. ERG20 and THI4 are involved in biosynthetic pathways, including sterol synthesis, suggesting an adaptive mechanism to maintain membrane integrity. 

#### Biofilm Formation 
Several genes are induced during Spider biofilm formation, a key survival and gene virulence strategy for _C. Albicans_. 

#### DNA Damage and Mitosis
THI4 is upregulated, and its activity as a 'suicide enzyme' emphasizes its importance in addressing DNA damage. Also, DUO1, a component of the Dam1 complex, is upregulated which is critical for chromosome segregation during mitosis. This may indicate a need for increased control of the cell cycle. 

## Conclusion 
The upregulation of the 13 identified significant genes during thiamine depletion represent a coordinate response of _C. Albicans_ to metabolic stress. _C. Albicans_, by modifying its gene expression, can respond to environmental stressors such as the absence of thiamine, and promote survival, biofilm formation, and virulence. These insights can offer potential therapeutic targets for antifungal treatments - disrupting key pathways could impair _C. Albican's_ ability to survive. Further studies should focus on those 4 uncharacterized proteins that are upregulated to determine their specific role in the response to thiamine depletion. 
