# Genome annotation with DANTE_LTR and TideCluster on RepeatExplorer Galaxy server

This protocol describe step by step how to use DANTE, DANTE_LTR and TideCluster tools to annotated LTR-retrotransposons and tandem repeats in genome assemblies on the RepeatExplorer Galaxy server (https://repeatexplorer-elixir.cerit-sc.cz/). DANTE_LTR was developed for annotation of LTR-RTs in Viridiplantae genomes. TideCluster can be used for annotation of tandem repeats in any genome assembly.


The annotation include following steps (see figure below). 
- Identification of individual LTR-RT protein domains using DANTE tool (**A**)
- Identification of complete LTR-retrotransposons using DANTE_LTR tool based on DANTE domain annotation (**B**)
- Creation of library of representative LTR-retrotransposons from complete LTR-retrotransposons(**C**)
- Library of LTR-retrotransposons is then used to annotate remaining LTR-RTs in genome assembly using similarity based approach. This allows to annotate also LTR-RTs sequences which are not complete elements and could not be identified in step directly by DANTE_LTR (**D**).
- Structure base annotation of tandem repeats using TideCluster tool (**E**). 
- Preparation of library of tandem repeats from TideCluster output (**F**). 
- Annotation of tandem repeats in genome assembly using library based approach. This allows to annotate tandem repeats which were too divergent to be identified directly by TideCluster (**G**). This step can be used to fill in gaps in tandem repeat annotation provided by TideCluster.
- Annotation of tandem repeats can be then used to improve LTR-RT annotation by substracting tandem repeat annotation from LTR-RT annotation (**H**). This step is recommended as some tandem repeats in the genome share similarity with LTR-RTs and could be misannotated as LTR-RTs when library based approach is used.
![workflow](workflow.png)


# 1. Input data
Upload your genome assembly in FASTA format to the Galaxy server. The genome assembly could be either plain FASTA file or compressed in GZIP format.

# 2. Annotation of LTR-retrotransposons
## 2.1. Annotation of transposable elements protein domain with DANTE tools
- Tool: *Domain based ANnotation of Transposable Elements - DANTE*
- Input data
  - *Sequence in fasta format*: uploaded genome assembly in FASTA format
- Parameters
  - *Select REXdb database*: Viridiplantae_v3.0
  - Other parameters: default

This tool will annotate LTR-RT protein coding domains using similarity search against REXdb database. The output include following files:
- GFF3 file with full DANTE output 
- Filtered GFF3 file with domains which passed the filtering criteria like minimum identity(35%), minimum similarity(45%), minimum alignment length(80% of domain length) and maximum number of interruptions (3 frameshifts or stop codons).
- FASTA file with protein sequences of annotated domains based on filtered GFF3 file.

## 2.2. Annotation of complete LTR-retrotransposons with DANTE_LTR
- Tool: *DANTE_LTR retrotransposon identification*
- Input data
  - *GFF3 output from DANTE pipeline - full output* - Use DANTE ouput from previous step
  - *Reference sequence matching DANTE output* - Use genome assembly in FASTA format
- Parameters
  - *Maximum number of missing protein domains to tolerate in full length retrotransposon*: 1

This tool will identify complete LTR-retrotransposons based on the DANTE output. The output include following files:
- GFF3 file with full DANTE_LTR output
- Statistics of identified LTR-retrotransposons
- Graphical report in HTML format

## 2.3. Using complete LTR-retrotransposons GFF3 to create library of similarity based annotation
- Tool: *Create representative library from DANTE_LTR output*
- Input data
  - *GFF3 output from DANTE_LTR pipeline* - Use DANTE_LTR ouput from previous step
  - *Reference sequence matching DANTE_LTR output* - Use genome assembly in FASTA format

This tool will create a library of representative LTR-retrotransposons based on the DANTE_LTR output. This library is in FASTA format suitable for similarity based annotation using tools like RepeatMasker. This will allow to annotate LTR-RTs which are note complete (fragmented elements, non-autonomous elements, solo-LTR) and could not be identified in previous step.

## 2.4. Similarity based annotation of LTR-retrotransposons with RepeatMasker

- Tool: *Library Based Assembly Annotation*
- Input data
  - *Genome assembly to annotate*: uploaded genome assembly in FASTA format
  - Library of Repetitive Sequences*: library of representative LTR-retrotransposons created in previous step
- Parameters: use default parameters

This tool will annotate LTR-retrotransposons in genome assembly using RepeatMasker. Resulting annotation is provided in RepeatMasker format (.out) and as GFF3 format. Overlapping annotations and conflicts are resolved based on classification hierarchy.


# 3. Annotation of Tandem Repeats

## 3.1 Annotation of Tandem Repeats with TideCluster
- Tool: *TideCluster*
- Input data
  - *Reference fasta* - uploaded genome assembly in FASTA format
  - *Library* - optional library in FASTA format
- Parameters: use default parameters. With default setting TideCluster report satellites with monomer length between 40-3000nt. Consensus sequence is calculated for tandem repeats which occupies at leat 50000 nt in the assembly.

Output consist of several files:
- GFF3 file with annotation of tandem repeats
- HTML reports with detailed statistics, consensus, and graphical representation of tandem repeats
- FASTA file with consensus sequences of tandem repeats, this file can be used for similarity based annotation of tandem repeats in genome in next step
- Archive with all output files 
  
## 3.2 Annotation of Tandem Repeats in Genome Assembly using library based approach
- Tool: *TideCluster - Annotate Genome*
- Input data
  - *Genome assembly to annotate*: uploaded genome assembly in FASTA format
  - *Library of tandem repeats*: library of tandem repeats created in previous step

This is a tool for re-annotating tandem repeats using a similarity-based approach. This tool runs RepeatMasker using the tandem repeat library generated by TAREAN in TideCluster. Resulting RepeatMasker output is processed to retain only high-quality tandem repeat hits. Overlapping tandem repeat annotation are merged, and regions shorter than twice the monomer length are excluded from the output. Re-annotation of tandem repeats using similarity-based approach can fill in gaps in annotation provided by TideCluster

# 4. Using Annotated tandem repeats to improve LTR-RT annotation

This step is recommended as some tandem repeats in the genome share similarity with LTR-RTs and could be misannotated as LTR-RTs when library based approach is used. It is recommended to remove such regions from LTR-RT annotation by subtracking tandem repeat annotation track.

## 4.1. Merging two Tandem Repeat annotation tracks

- Tool: Concatenate datasets tail-to-head (cat)
- Input data
  - *First dataset*: TideCluster GFF3 file with tandem repeat annotation from step @sec:tidecluster
  - *Second dataset*: GFF3 file with similarity based tandem repeat annotation from step @sec:library_based

This step will create combined GFF3 with tandem repeats annotated but TideCluster and library based approach. 

## 4.2. Substracting tandem repeat annotation from LTR-RT annotation

- Tool: bedtools subtract
- Input data
  - *First dataset (-a)*: GFF3 file with LTR-RT annotation from step 3.1.
  - *Second dataset(-b)*: GFF3 file with merged tandem repeat annotation from step 4.1 
- Parameters: use default parameters (overlap in either strand)
- Output: GFF3 file with LTR-RT annotation without tandem repeats
