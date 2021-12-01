---
layout: page
title: ProkaryoteGeneAnnotation Tutorial
tagline: Quick gene annotations using NCBI reference material
description: Tutorial of how to use ProkaryoteGeneAnnotation to download and compile a reference dataset for predicting genes associated with 16S amplicon data.
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## INTRO

This package is meant to facilitate downloading and summarizing the annotations of completed GenBank and RefSeq prokaryote genome assemblies. It also contains functionality to add gene annotations to a phyloseq object's taxonomy table; or to subset a phyloseq object to only taxa that contain a set of genes. This tool is not meant to approximate or infer metagenomes, though it operates on many of the same principles as other tools that do. Also, this was built to work on a macOS platform. Windows and Linux are not supported. If you want, you may fork the repository and add that functionality.

To install this package:
```{r, eval=FALSE}
install.packages("devtools")
library(devtools)
devtools::install_github('Djeppschmidt/ProkaryoteGeneAnnotation')
```

There are three steps to this process:
1) get the reference data - feature tables from RefSeq and GenBank
2) summarize the tables by genus, species, and strain identifiers
3) subset, or modify phyloseq objects based on these identifiers

## Get reference data

The first step is to download the most up to date reference data. NCBI has a functionality to make a table that contains accession information for any data. We start by using their tool to filter to only prokaryote taxa, and remove any assemblies that are not complete. This can be done by following this link:

https://www.ncbi.nlm.nih.gov/genome/browse#!/prokaryotes/

Click the filters dropdown menu on the upper right hand side of the data table. Next to the Assembly level tab, check the box for "complete". Then click the download icon at the top left of the data table (below the filters).

Name this file whatever you want, and put it in whatever directory you want. In this example, it is named "prokaryote.csv"

Import this file into R:
```{r, eval=FALSE}
# import reference dataset:
# refdir from https://www.ncbi.nlm.nih.gov/genome/browse#!/prokaryotes/
# filter set to only bacteria and archaea
# and only whole genomes
refdir<-as.data.frame(as.matrix(read.csv("~/Documents/GitHub/SoilHealthDB/prokaryotes.csv", sep=",")))
```

Next we need to download the annotated feature tables. These tables are placed in an output directory of your choice, in a folder corresponding to the source of the annotations (GenBank or RefSeq). There are several inputs required for this function:

1) the reference data frame made in the first step (refdir)
2) an output directory path (outpath)
3) a directory path to the bin folder where wget is compiled (you may need to install this)

The bin folder where wget is compiled may not be in the same location bin folder that RStudio references. This function assumes wget is compiled in "/usr/local/bin/". If this is not the case on your machine, change the binPATH option.

Finally, make sure that your output directory has a large enough volume to accommodate at least 30 GB of data. I sometimes use an external hard drive for this type of task. 
```{r, eval=FALSE}
# run:
outpath<-"/Downloads"
binPath<-"/usr/local/bin/"
download.Feature.Tables(refdir, outpath, binPATH)
```

## Summarize the reference tables

This function takes a vector of gene names / symbols. An example is "nifH". You may include as many as you like in this vector. The output will be a table with Genus, Species, Strain, Accession number, and a column for each gene. Rows are unique genome assemblies. The value in the column will be zero if the assembly does not have the gene; and 1 if it does.

```{r, eval=FALSE}
# run:
# refdir = same reference table from above
# directory = same directory from the previous function (where the GenBank and RefSeq folders exist)
# genes = vector of genes
gene.tab<-get.genes(refdir, outpath, genes)
```


## Annotate or manipulate phyloseq objects using this reference data

This package offers the ability to subset phyloseq objects to isolate the taxa that likely have certain genes. This works by subsetting the phyloseq taxonomy based on matches to either the species or genus level. I strongly recommend using the species level cutoff.

```{r, eval=FALSE}
subset.phyloseq(phyloseq, genes, level)
```

And that's it.