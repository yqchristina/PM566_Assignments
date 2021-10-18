PM566_Midterm
================
Christina Lin
10/17/2021

# Exploring Cytochrome P450 Enzymes Found in the Human Brain

## Introduction

This project is based on my PhD thesis exploring the enzymes in the
brain that can produce the neurosteroid pregnenolone from cholesterol.
In classical steroid-producing organs such as the adrenals, pregnenolone
is metabolized from cholesterol by the cytochrome P450 (CYP450) enzyme
CYP11A1. However, CYP11A1 protein is difficult to detect in the brain
and preliminary experiments have revealed that a potential alternate
pathway not involving CYP11A1 is used by human brain cells to produce
pregnenolone. Therefore, this project will analyze known CYP450s in the
UniProt database to answer 3 main questions: 1) Which CYP450 enzymes are
expressed in the brain? 2) Which CYP450 enzymes are involved in
cholesterol/steroid metabolism? 3) Which CYP450 enzyme is similar to
CYP11A1?

## Methods

List of CYP450s were obtained from the UniProt database by searching
“cytochrome P450”. Additional filters were applied: “Homo
sapiens(human)” for species and “Reviewed” results to extract
information only from manually annotated records from literature and
curator-evaluated computational analysis. The columns of interest are
protein name, gene name, length (of protein), mass, tissue specificity,
cofactor, function, subcellular location, pathway, and sequence. The
[results](https://www.uniprot.org/uniprot/?query=cytochrome%20p450&fil=organism%3A%22Homo%20sapiens%20(Human)%20%5B9606%5D%22%20AND%20reviewed%3Ayes&columns=id%2Centry%20name%2Cprotein%20names%2Cgenes%2Corganism%2Clength%2Cmass%2Ccomment(TISSUE%20SPECIFICITY)%2Ccomment(COFACTOR)%2Ccomment(FUNCTION)%2Ccomment(SUBCELLULAR%20LOCATION)%2Ccomment(PATHWAY)%2Csequence&sort=score)
were downloaded as a CSV file.

### Data Wrangling

``` r
cyp450 <- fread("UniProt_hCYP450s.csv")
setnames(cyp450, "Gene names", "Gene_name")
setnames(cyp450, "Protein names", "Protein_name")
setnames(cyp450, "Function [CC]", "Function")
setnames(cyp450, "Subcellular location [CC]", "Subcellular_location")
setnames(cyp450, "Tissue specificity", "Tissue_expression")
```

Some enzymes have multiple gene names. For this analysis, only the first
gene name containing “CYP” will be used. Rows that do not have a gene
name starting with “CYP” are removed.

``` r
cyp450$Gene_name <- stringr::str_extract(cyp450$Gene_name, "CYP[[:alnum:]]+")
start_rows <- nrow(cyp450)
cyp450 <- cyp450[!is.na(Gene_name),]
end_rows <- nrow(cyp450)
```

The initial data table started with 82 proteins. After simplifying the
gene names and removing entries that do not have “CYP” in the gene name,
there are 62 proteins left.

Next, the “Mass” column will be converted to a numeric variable by
removing the “,” character and converting the values to integers.

``` r
cyp450$Mass <- stringr::str_remove_all(cyp450$Mass, ",")
cyp450$Mass <- as.integer(cyp450$Mass)
summary(cyp450$Mass)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   12413   56315   57501   56421   59206   76690

``` r
summary(cyp450$Length)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##     118     494     503     495     519     677

When checking for the masses of the proteins, we see that the lowest
mass protein is 12413 Dalton and the shortest length protein is 118
amino acids. Since CYP450s are enzymes involved in complex metabolic
pathways and typically have multiple functional domains, these small
proteins are likely not CYP450 with cholesterol-metabolizing potential.
Therefore, proteins that are less than 35 000 Dalton in mass will be
removed.

``` r
cyp450 <- cyp450[Mass >= 35000,]
smallest <- cyp450[which.min(Mass),]
largest <- cyp450[which.max(Mass),]
```

## Preliminary Results

The average mass of cyptochrome P450 enzymes found in humans is 57816.35
Dalton and the average length is 507.2 amino acids. The smallest CYP450
is CYP4Z2P, which is 40159 Dalton in mass and 340 amino acids in length.
The largest CYP450 is CYPOR, which is 76690 Dalton in mass and 677 amino
acids in length.

``` r
ggplot(cyp450, mapping=aes(x = Mass, y = Length, color = Gene_name)) +
  geom_point() +
  xlab("Mass (Dalton)") +
  ylab("Length (amino acids)") +
  ggtitle("Mass and length of human CYP450s")
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## Conclusion
