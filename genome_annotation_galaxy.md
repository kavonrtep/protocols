 <style>
     h1 {
         max-width: 700px;
     }

     p {
         max-width: 700px;
     }
     ul {
         max-width: 700px;
     }
     html, body {
         height: 100%;
     }
     html {
         display: table;
         margin: auto;
     }
     body {
         display: table-cell;
         vertical-align: middle;
     }
     .frame {

         width: 100%;
         border: 0px;
         white-space: nowrap;
         text-align: center; margin: 1em 0;
     }
     .helper {
         display: inline-block;
         height: 100%;
         vertical-align: middle;
     }
     img {
         background: #FFFFFF;
         vertical-align: middle;
     }

     table {
         border-collapse: collapse;
         width: 100%;
     }
     th.date {
         width: 20%
     }
     th.subject {
         width: 70%; /* Not necessary, since only 70% width remains */
     }

     tr {
         border-bottom: 1px solid #ccc;
     }

     th {
         text-align: left;    
     }
     th, td {
         padding: 5px;
     }
 </style>
# Genome annotation with DANTE_LTR and TideCluster on RepeatExplorer Galaxy server {.unnumbered}

This protocol describe step by step how to use DANTE, DANTE_LTR and TideCluster tools to annotated LTR-retrotransposons and tandem repeats in genome assemblies on the RepeatExplorer Galaxy server. DANTE_LTR was developed for annotation of LTR-RTs in Viridiplantae genomes. TideCluster can be used for annotation of tandem repeats inv any genome assembly.

# Input data
Upload your genome assembly in FASTA format to the Galaxy server. The genome assembly could be either plain FASTA file or compressed in GZIP format.

# Annotation of LTR-retrotransposons
## Annotation of transposable elements protein domain with DANTE tools {#dante}
- Tool: *Domain based ANnotation of Transposable Elements - DANTE*
- Input data
  - *Sequence in fasta format*: uploaded genome assembly in FASTA format
- Parameters
  - *Select REXdb database*: Viridiplantae_v3.0
  - Other parameters: default

This tool will annotate LTR-RT protein coding domains using similiarity search against REXdb database. The output include following files:
- GFF3 file with full DANTE output 
- Filtered GFF3 file with domains which passed the filtering criteria like minimum identity(35%), minimum similarity(45%), minimum alignment length(80% of domain length) and maximum number of interruptions (3 frameshifts or stop codons).
- FASTA file with protein sequences of annotated domains based on filtered GFF3 file.

## Annotation of complete LTR-retrotransposons with DANTE_LTR {#dante_ltr}
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

## Using complete LTR-retrotransposons GFF3 to create library of similarity based annotation {#library}
- Tool: *Create representative library from DANTE_LTR output*
- Input data
  - *GFF3 output from DANTE_LTR pipeline* - Use DANTE_LTR ouput from previous step
  - *Reference sequence matching DANTE_LTR output* - Use genome assembly in FASTA format

This tool will create a library of representative LTR-retrotransposons based on the DANTE_LTR output. This library is in FASTA format suitable for similarity based annotation using tools like RepeatMasker.

## Similiarity based annotation of LTR-retrotransposons with RepeatMasker {#repeatmasker}

- Tool: *Library Based Assembly Annotation*
- Input data
  - *Genome assembly to annotate*: uploaded genome assembly in FASTA format
  - Library of Repetitive Sequences*: library of representative LTR-retrotransposons created in previous step
- Parameters: use default parameters

This tool will annotate LTR-retrotransposons in genome assembly using RepeatMasker. Resulting annotation is provided in RepeatMasker format (.out) and ass GFF3 format. Overlaping annotations and conflicts are resolved based on classification hierarchy.


# Annotation of Tandem Repeats

## Annotation of Tandem Repeats with TideCluster
- Tool: *TideCluster*
- Input data
  - *Reference fasta* - uploaded genome assembly in FASTA format
  - *Library* - optional library in FASTA format
- Parameters: use default parameters. With default setting TideCluster report satellites with monomer length between 40-3000nt. Consensus sequence is calculated for tandem repeats which occupies at leat 50000 nt in the assemble
   