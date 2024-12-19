# Tips for using cellranger with Macaca mulatta samples

## Intro
If interested in doing Rhesus TCR sequencing with 10x single-cell technology, it's possible to use the human enrichment primers. This is convenient but means we have to do some extra steps in preparing the reference for gene expression counting and TCR assembly, with either `cellranger multi` or a combination of `cellranger count` and `cellranger vdj`. The steps are as follows:
  1) If necessary, download `cellranger`
  2) Pull TR and IG sequences from IMGT with `fetch-imgt`
  3) Make the VDJ reference
  4) Filter ensembl GTF for relevant gene types
  5) Make STAR reference for gene expression
  6) Quantify gene counts and assemble T/B receptor genes.



## `cellranger`
Our system (MSI) has loadable modules for `cellranger` v7.2.0, however, making a VDJ-compatible reference requires modification of a helper script `fetch-imgt` included with the software. As such, I downloaded the most recent version from 10x (v8.0.0). ~NB~: unpacking `cellranger-8.0.0.tar.gz` took ~4hrs walltime. This is due to the numerous python libraries.

## `fetch-imgt`
We used the TCR sequences available at IMGT to build our VDJ reference. 10x has a [small vignette](https://kb.10xgenomics.com/hc/en-us/articles/6206590727821-Building-a-custom-reference-for-V-D-J-using-IMGT-tool) for doing this and provides a helpful hack for getting C-Region genes when particular IMGT versions don't include them. However, this particular hack only works for IG kappa and lambda C-regions (i.e., BCR). The edit below will get the right IMGT version for TR and IG heavy C-Regions in _Macaca mulatta_. 

```
    c_genes = ["TRAC", "TRBC", "TRDC", "TRGC", "IGHC"]
    c_label = None
    #c_query = "14.1"
    c_query = "7.2"
```

## Inner enrichment primers for `cellranger multi` or `cellranger vdj`
Custom VDJ refernces require specification of the inner enrichment primers used for the VDJ C-regions. Recall that we simply used the Human primers supplied by 10x, [which we can find here](https://kb.10xgenomics.com/hc/en-us/articles/360047454291-Which-genes-does-each-V-D-J-specific-primer-map-to). If we use the inner TR[A-B] sequences as they are, `cellranger multi` fails preflight checks with the error

```
None of the C-REGIONs in the reference (/scratch.global/aherman/rm_tcr/Mmul_IMGT_VDJ) is targeted by the following inner enrichment primer(s): AGTCTCTCAGCTGGTACACG
```

This is apparently because the software is looking for exact primer matches to the genome, which we might not expect between Human and Rhesus. We therefore made a `blast` database of our TCR reference and blasted the enrichment primer above. This showed that there is a single SNP difference between the primer and Rhesus C-Region. So we just need to use the "Sbjct" sequence from the `blast` result below as the other primer sequence.

```
>1299_TRAC*02 IMGT000076|TRAC|C-REGION|TR|TRA|None|02
Length=272

 Score = 32.8 bits (35),  Expect = 5e-04
 Identities = 19/20 (95%), Gaps = 0/20 (0%)
 Strand=Plus/Minus

Query  1   AGTCTCTCAGCTGGTACACG  20
           || |||||||||||||||||
Sbjct  42  AGCCTCTCAGCTGGTACACG  23
```
