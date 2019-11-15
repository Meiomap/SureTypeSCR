# Prerequisites
```
*SureTypeSC (https://github.com/Meiomap/SureTypeSC_2) [python package]

*reticulate (https://rstudio.github.io/reticulate/) [r package]

*knitr(https://github.com/yihui/knitr) [r package]

*BiocStyle(https://bioconductor.org/packages/release/bioc/html/BiocStyle.html) [r package]
```

# Installation

After downloading and unpacking the package and rename the fold to be SureTypeSCR, run the command line below to install the R package SureTypeSCR

```
R CMD build SureTypeSCR

R CMD INSTALL SureTypeSCR_0.99.0.tar.gz
```

or the other way is possible (only if this repository is public)
```
devtools::install_github('Meiomap/SureTypeSCR', build_vignettes = TRUE)
```


# Introduction

A key motivation of SureTypeSCR is that accurate genotyping of DNA from a single cell is required for application such as de novo mutation detection and linkage analysis, but achieving high precision genotyping in the single cell environment is challenging due to the errors caused by whole genome amplification.

SureTypeSCR, based on python, is a two-stage machine learning algorithm that filters a substantial part of the noise, thereby retaining of the majority of
the high quality SNPs. SureTypeSCR consists of two layers, Random Forest (RF) and Gaussian Discriminant Analysis (GDA).



## Module references

The package includes a list of references to python
modules including numpy, pandas and SureTypeSC.

```{r loadup}
library(SureTypeSCR)
```



## Importing data for direct handling by python functions

The reticulate package is designed to limit the amount
of effort required to convert data from R to python
for natural use in each language. SureTypeSCR package includes test data. Users can play with the test data.

```{r doimp}
cluster_path = system.file('files/HumanKaryomap-12v1_A.egt',package='SureTypeSCR')
manifest_path = system.file('files/HumanKaryomap-12v1_A.bpm',package='SureTypeSCR')
samplesheet = system.file('files/Samplesheetr.csv',package='SureTypeSCR')
clf_rf_path = system.file('files/clf_30trees_7228_ratio1_lightweight.clf',package='SureTypeSCR')
clf_gda_path = system.file('files/clf_GDA_7228_ratio1_58cells.clf',package='SureTypeSCR')
```

## To process the raw gtc files without genomesutdio, we can use scbasic.

```{r dota}
df <- scbasic(manifest_path,cluster_path,samplesheet)
```

## GTC data quality check
### call rate over all samples

```{r call}
df <- scbasic(manifest_path,cluster_path,samplesheet)

callrate_allsamples <- callrate(df,th=0.3)

callrate_onechr <- callrate_chr(df,'X', th=0.15)

geno_freq <- allele_freq(df,th=0.5)

```

### M ans A features calculate of one locus

```{r locus}
df <- scbasic(manifest_path,cluster_path,samplesheet)

locus <- locus_ma(df,'rs3128117')

am <- sample_ma(df,'Kit4_4mos_SC21','1')

```



### PCA

```{r PCA}

df <- scbasic(manifest_path,cluster_path,samplesheet)

pca_all <- pca_samples(df,th=0.1)

pca_onechr <- pca_chr(df,'X',th=0.1)
```


## To convert pandas dataframe to Data object and rearrange the index to multi-index level,
we use create_from_frame. And users can check the valuse by specifiy the 'df' attribution.
```{r dotinde}
dfs <- create_from_frame(df)

values <- dfs$df

values$shape

values$columns

values$dtypes

values['sc21']['score'][900:1200]


```

## To select certain chromosomes, apply the threshold on the Gencall score and calculate
m and a features (training data preparation)

```{r dotx}
dfs <- restrict_chrom(dfs,c('1','2'))
dfs <- apply_thresh(dfs,0.01)
dfs <- calculate_ma(dfs) 

```

# Train and Predict

## To load classifiers (rf: Random Forest; gda: gaussian discriminat analysis) and predict

```{r dorpart}
clf_rf <- scload(clf_rf_path)
clf_gda <- scload(clf_gda_path)

result_rf <- scpredict(clf_rf, dfs, clftype='rf')
result_gda <- scpredict(clf_gda,result_rf,clftype='gda')

```

## To train the cascade of Random Forest and Guassian Discriminant Analysis:

```{r dopt}
trainer <- scTrain(result_rf,clfname='gda')

result_end <- scpredict(trainer,result_gda,clftype='rf-gda') 

```


# Save the results

After prediction, we can save the results

```{r doincr}

scsave(result_end,'recall.txt',clftype='rf',threshold=0.15,all=FALSE)

```





# Conclusions

We need more applications and profiling.
