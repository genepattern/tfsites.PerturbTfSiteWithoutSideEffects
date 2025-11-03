# tfsites.PerturbTfSiteWithoutSideEffects v1

**Author(s):** Joe Solvason  

**Contact:** Joe Solvason (solvason@ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transciption factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00480


## Introduction

tfsites.PerturbTfSiteWithoutSideEffects ...

## Methodology

For every nucleotide in the sequence, all possible SNVs are made. For each SNV, we determine its effect, if any, on any binding sites that exist in the sequence. These are the possible effects of a SNV on a binding site: 
- `inc`
    - The affinity/score of the binding site increases
    - The affinity/score fold change from the reference binding site to the alternate binding site is greater than 1
- `dec`
    - The affinity/score of the binding site decreases
    - The affinity/score fold change from the reference binding site to the alternate binding site is less than 1
- `denovo`
    - A binding site is created
    - The reference k-mer is not a predicted binding site (it either didn't follow the binding site definition or didn't meet the PWM minimum score threshold), but the alternate k-mer is a predicted binding site
- `del`
    - A binding site is deleted
    - The reference k-mer is a predicted binding site (it either followed the binding site definition or met the PWM minimum score threshold), but the alternate k-mer is not a predicted binding site
  
If an affinity optimization threshold is provided by the user, then we report only the binding sites that have an increased affinity/score with a fold change greater than or equal to the threshold. Similarly, if an affinity reduction threshold is provided, then we report only the binding sites that have a decreased affinity/score with a fold change less than or equal to the threshold. 

Using the list of all identified SNV effects, an image of the sequence is generated and it displays all possible alternate nucleotides. The background of each nucleotide is colored according to the mutation type of the SNV. If the SNV has no effect, then its background is blank. If a SNV has multiple effects, then its background will be split into multiple colors. The intensity of the background color is determined by the following options: (1) magnitude of the affinity/score fold change, if the SNV effect is `inc` or `dec`, (2) magnitude of the alternate k-mer's affinity/score, if the SNV effect is `denovo`, or (3) full intensity, if the SNV effect is `del`. 

To find putative binding sites, we iterate across every k-mer in the DNA sequence. If using PBM data, we identify the k-mers that conform to the binding site definition for each transcription factor. If using PWM data, we can also use a binding site definition but it is not required. If a site definition is not provided for PWM data, we use the PWM minimum score to define a predicted binding site. The user can also choose to plot all denovo binding sites created from SNVs, in addition to existing putative binding sites. 

If the user wishes to analyze only a portion of the sequence, then a zoom range can be specified. If the sequence is greater than 500 nucleotides in length, the sequence will automatically be separated into 500-bp windows and outputted as separate files. In addition, the individual files will be appended together to create a single output file with the entire sequence. The user can also choose to output the files in `.svg` format in addition to `.png`.

## Parameters

<span style="color: red;">*</span> indicates required parameter

### Inputs and Outputs
- <span style="color: red;">*</span>**DNA sequence(s) to annotate (.tsv)**
    - File containing one or more DNA sequences to be annotated. 
- <span style="color: red;">*</span>**TF name (string)**
    - Name of the transcription factor to use for SNV analysis.
- **core binding site definition (string)**
    - `Default = None`
    - IUPAC definition of core TF binding site (see [here](https://www.bioinformatics.org/sms/iupac.html)). Only optional if using PWM data but required if using affinity data.
- **affinity reference data (.tsv)**
    - `Default = None`
    - PBM affinity dataset used to assign a value to each binding site.
- **PWM data (.txt)**
    - `Default = None`
    - PWM dataset used to assign a value to each binding site.
- **PWM minimum score (float)**
    - `Default = 0.7`
    - PWM score required to predict a binding site.
- <span style="color: red;">*</span>**output filename (string)**
    - Base name of the output files.

### Other Parameters
- **output image as svg (boolean)**
    - `Default = False`
    - Option to output images as `.svg` in addition to `.png`. For manuscript preparation, `.svg` format is preferable.
- **SNV effect type to report (string)**
    - `Default = all`
    - Specify one or more mutation types to analyze. SNV mutations can either increase (optimize) or decrease (sub-optimize) the affinity/score, delete a binding site, or create a binding site. Therefore, the possible mutation types are `inc`, `dec`, `denovo`, and `del`. This option also takes the value `all` if the user would like to analyze all of the listed mutation types.
- **affinity optimization threshold (float)**
    - `Default = 1`
    - Fold change threshold for mutations that increase the affinity/score. Only SNVs with fold change above this threshold will be reported. By default, all SNVs will be reported.
- **affinity reduction threshold (float)**
    - `Default = 1`
    - Fold change threshold for mutations that decrease the affinity/score. Only SNVs with fold change below this threshold will be reported. By default, all SNVs will be reported.
- **plot resolution (integer)**
    - `Default = 150`
    - Resolution of the plot, in dots (pixels) per inch. Manuscripts require 300 DPI. The DPI does not affect the resolution of `.svg` files.
- **zoom range (dash-separated string)**
    - `Default = None`
    - Given a start position and an end position, zoom into a portion of the sequence. The numbers in the range are inclusive and 1-indexed. For example, the first 200 nucleotides of the sequence would be specified as: 1-200.

## Input Files

1.  DNA sequence(s) to annotate (.tsv)
- Columns:
    - `Sequence Name:` name of the DNA sequence
    - `Sequence:` the sequence
 
```
Sequence Name	    Sequence
ZRS                 AACTTTAATGCCTATGTTTGATTTGAAGTCATAGCATAAAAGGTAACATAAGCAACATCCTGACCAATTATCCAAACCATCCAGACATCCCTGAATGGC...
```
    
2. affinity reference data (.tsv)

ETS
```
PBM Kmer     PBM Relative Affinity
AAAAAAAA     0.15
AAAAAAAC     0.11
AAAAAAAG     0.13
AAAAAAAT     0.13
AAAAAACA     0.12
```


## Output Files
1.  SNV effects output table (.tsv)
- Columns:
    - `Sequence Name:` name of the sequence being analyzed
    - `Kmer ID:` unique ID given to binding site
    - `SNV Position (0-indexed):` position of the SNV
    - `Reference Nucleotide:` reference nucleotide
    - `Alternate Nucleotide:` alternate nucleotide
    - `Start Position (1-indexed):` position at which the k-mer starts, where counting begins at one
    - `End Position (1-indexed):` position at which the k-mer ends, where counting begins at one
    - `Reference Kmer:` reference k-mer
    - `Alternate Kmer:` alternate k-mer
    - `Site Direction:` direction of the binding site 
    - `Reference Value:` the affinity/score of the reference binding site
    - `Alternate Value:` the affinity/score of the alternate binding site
    - `Fold Change:` the ratio between `Reference Value` and `Alternate Value`
    - `SNV Effect:` the type of SNV effect
 
```
Sequence Name    TF Name     Kmer ID      Kmer                Start Position (1-indexed)    End Position (1-indexed)  Ref Data Type   Value    Site Direction   Duplicate Kmer IDs
ZRS              ETS         ETS:1        CTATCCTG            335                           328                       Affinity        0.15     -
ZRS              ETS         ETS:2        TTTTCCCC            432                           425                       Affinity        0.14     -                ETS:1,ETS:20
ZRS              HOX         HOX:1        TTTAATAT            323                           316                       Affinity        0.75     -	
ZRS              HOX         HOX:2        TTTATGAC            415                           408                       Affinity        0.84     -
ZRS              HAND        HAND:1       CAGATG              416                           421
```


2.  SNV effects image(s) (.png)

<img src="./04-output_visualizeInSilicoSnvs-image_seq=ZRS_tf=ETS_zoom=320-490.png"/>

  
## Example Data

Example input data is available [here](https://github.com/genepattern/tfsites.AnnotateAndVisualizeInSilicoSNVAnalysis/tree/develop/data).    
    
## Version Comments

- **1.0.4** (2024-11-21): Updated for tfsites website.
- **1.0.3** (2024-10-17): Third draft completed.
- **1.0.2** (2024-09-12): Second draft completed. Updated parameters and visualizations.
- **1.0.1** (2024-02-02): Draft completed.
- **1.0.0** (2023-01-12): Initial draft of document scaffold.
