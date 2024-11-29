# gwas-tutorial
This repository follows a tutorial on GWAS in Plink by math et al on Youtube
Author: Paul K. Yu

Link to Youtube tutorial: [https://www.youtube.com/watch?v=7QMSZx3io-Q](https://www.youtube.com/watch?v=7QMSZx3io-Q)

To run gplink GUI: Download here: [https://zzz.bwh.harvard.edu/plink/gplink.shtml](https://zzz.bwh.harvard.edu/plink/gplink.shtml)
```
java -jar gPLINK2.jar
```

Plink commands: (create output folder ahead of time)

1. This PLINK command incorporates covariates from `ajhg/samples.covar`, processes genetic data from the `ajhg/adgwas` dataset, prepares output compatible with GPLink, and recodes the data into PED/MAP format, saving the results in the `test` directory with the prefix `input_test`.
```
plink --covar ajhg/samples.covar --file ajhg/adgwas --gplink --out test/input_test --recode
```

2. This PLINK command filters genetic data from `test/input_test` for Hardy-Weinberg equilibrium with a p-value threshold of 0.05, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `hwe` directory with the prefix `hwe_all_test`.
```
plink --file test/input_test --gplink --hwe .05 --out hwe/hwe_all_test --recode
```

3. This PLINK command filters genetic data from `hwe/hwe_all_test` to include only variants with a minor allele frequency (MAF) of at least 0.01, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `maf` directory with the prefix `maf_rmvd`.
```
plink --file hwe/hwe_all_test --gplink --maf 0.01 --out maf/maf_rmvd --recode
```

4. This PLINK command filters genetic data from `maf/maf_rmvd` to exclude variants with more than 10% missing genotypes, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `geno` directory with the prefix `geno_rmvd`.
```
plink --file maf/maf_rmvd --gplink --geno .10 --out geno/geno_rmvd --recode
```

5. This PLINK command filters genetic data from `geno/geno_rmvd` to exclude individuals with more than 10% missing genotype data, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `mind` directory with the prefix `mind10`.
```
plink --file geno/geno_rmvd --gplink --mind .10 --out mind/mind10 --recode
```

6. This PLINK command performs a sex check on the genetic data from `ajhg/adgwas`, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `sex` directory with the prefix `check_sex`.
```
plink --check-sex --file ajhg/adgwas --gplink --out sex/check_sex --recode
```

7. This PLINK command processes genetic data from `mind/mind10` for chromosomes 1 through 22, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results for chromosome 22 in the `chr` directory with the prefix `chr22`.
```
plink --chr 1-22 --file mind/mind10 --gplink --out chr/chr22 --recode
```

8. This PLINK command calculates pairwise genetic relationships using the data from `chr/chr22`, filters out pairs with a genotypic relationship below 20%, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `cryptic` directory with the prefix `cryptic`. This is one way of accounting for population structure to see if it is affecting GWAS.
```
plink --file chr/chr22 --genome --gplink --min .20 --out cryptic/cryptic --recode
```

9. This PLINK command performs a Principal Component Analysis (PCA) on the data from `cryptic/cryptic` using variance-weighted (var-wts) components, prepares GPLink-compatible outputs, recodes the data into PED/MAP format, and saves the results in the `pca` directory with the prefix `pca`. This is to see what PC is important for accounting for number of variants in model and include this as covariates in the association test.
```
plink --file cryptic/cryptic --gplink --out pca/pca --pca var-wts --recode
```

10. This PLINK command performs a logistic regression association test using the first five principal components from `pca/pca.eigenvec` as covariates, adjusts for multiple testing, processes genotype data from `pca/pca`, and saves the results (along with recoded data) in the `ass_test` directory.
```
plink --adjust --allow-no-sex --covar pca/pca.eigenvec --covar-number 1-5 --file pca/pca --gplink --logistic --out ass_test/ass_test --recode
```

11. This PLINK command performs a logistic regression association test with a 95% confidence interval using the first five principal components from `pca/pca.eigenvec` as covariates, adjusts for multiple testing, processes genotype data from `pca/pca`, and saves the results (along with recoded data) in the `assoc_ci` directory.
```
plink --adjust --allow-no-sex --ci .95 --covar pca/pca.eigenvec --covar-number 1-5 --file pca/pca --gplink --logistic --out assoc_ci/assoc_ci --recode
```

12. This PLINK command performs a logistic regression association test with a 95% confidence interval, processes genotype data from `pca/pca` without including principal components as covariates, adjusts for multiple testing, and saves the results (along with recoded data) in the `assoc_nopca` directory.
```
plink --adjust --allow-no-sex --ci .95 --file pca/pca --gplink --logistic --out assoc_nopca/assoc_nopca --recode
```

13. This PLINK command performs a logistic regression association test with a 95% confidence interval, using the first principal component from `pca/pca.eigenvec` as a covariate, adjusts for multiple testing, processes genotype data from `pca/pca`, and saves the results (along with recoded data) in the `pca1` directory.
```
plink --adjust --allow-no-sex --ci .95 --covar pca/pca.eigenvec --covar-number 1 --file pca/pca --gplink --logistic --out pca1/pca1 --recode
```

Test genomic inflation factor (if above 1, there is population structure in data), add a certain number of PCs as covariates to the association test.
```
gwas-tutorial.Rmd
```
