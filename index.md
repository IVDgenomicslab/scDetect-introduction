## Summary

[scDetect](https://github.com/IVDgenomicslab/scDetect/) is a new cell type ensemble learning classification method for single-cell RNA sequencing across different data platforms, using a combination of gene expression rank-based method and majority vote ensemble machine-learning probability-based prediction method.

To further accurate predict the tumor cells in the single cell RNA-seq data, we developed scDetect-Cancer, a classification framework which incorporated the cell copy number information and epithelial origin information in the classification.

## Application of scDetect

First, we load the [rJava](https://cran.r-project.org/web/packages/rJava/index.html), [scDetect](https://github.com/IVDgenomicslab/scDetect/) package, and [Seurat](https://satijalab.org/seurat/install.html)

```markdown
library("rJava")
library("scDetect")
library("Seurat")
```

We will work with single cell data from two human pancreas dataset. "Muraro" dataset were generated from CEL-Seq2 platform, "Xin" dataset were generated from SMARTer platform.

Read the gene expression data and cell type lable.

```markdown
# Xin human pancreas dataset #
xin<-counts(xin_test)
xin_lable<-xin_test$label

# Muraro human pancreas dataset #
muraro<-counts(muraro_test)
muraro_lable<-muraro_test$label
```

#### Prediction

To make scDetect easy to use, all steps were integrated into one function -- scDetect. 

Here, we used Muraro pancreas dataset as the training dataset to predict the cell types in Xin pancreas dataset. Muraro and Xin pancreas dataset had different cell type numbers. There are 9 cell types in Muraro dataset and 4 cell types in Xin dataset. The cell types in Xin dataset were included in the Muraro dataset. In the tutorial, we used Muraro dataset as the training dataset to predict the cell types in the Xin dataset. We suggested that the training dataset should be contained more cell types than the dataset be predicted. So that there will be less unclassified and misclassified cells in the prediction dataset. 


```markdown
# Using Muraro dataset as the training dataset #
# Prediction #
prediction_results<-scDetect(vali_set_matrix = xin, train_set_matrix = muraro, train_set_lable = muraro_lable,p_value=0.2)
```

We can obtain a table showing the prediction results and detailed inforamtion.

The prediction results of scDetect included four columns:

predict_lable: Predicted cell type of the highest predict_score cell type;

predict_score: Highest predict score of the corresponding cell type;

pvalue: p value of the predict score based on the permutation analysis;

final_predict_lable: Predicted cell type based on the predict score and pvalue.


```markdown
prediction_results[1:20,]
##           predict_lable     predict_score pvalue final_predict_lable
## Sample_1           beta             0.475  0.263             Unknown
## Sample_2           beta             0.525  0.181                beta
## Sample_3           beta 0.666666666666667  0.045                beta
## Sample_4           beta 0.508333333333333    0.2                beta
## Sample_5           beta 0.591666666666667  0.095                beta
## Sample_6           beta              0.55   0.16                beta
## Sample_7           beta             0.625  0.079                beta
## Sample_8           beta 0.416666666666667  0.458             Unknown
## Sample_9           beta             0.625  0.079                beta
## Sample_10          beta 0.516666666666667  0.181                beta
## Sample_11          beta 0.666666666666667  0.045                beta
## Sample_12          beta 0.516666666666667  0.181                beta
## Sample_13          beta 0.633333333333333  0.072                beta
## Sample_14          beta             0.675  0.021                beta
## Sample_15          beta 0.716666666666667  0.021                beta
## Sample_16          beta 0.591666666666667  0.095                beta
## Sample_17          beta             0.625  0.079                beta
## Sample_18          beta 0.591666666666667  0.095                beta
## Sample_19          beta 0.633333333333333  0.072                beta
## Sample_20          beta              0.45  0.346             Unknown
```

#### Evaluation
Evaluate the prediction results.

```markdown
evaluate_results<-evaluate(xin_lable,prediction_results$final_predict_lable)
```

Accuracy of the cell type prediction results.

```markdown
evaluate_results$Acc
## 0.9758491
```

Confusion matrix of the cell type prediction results.

```markdown
evaluate_results$Conf
##          pred_lab
##  true_lab acinar alpha beta delta gamma Unknown
##   alpha      1   846    0     0     0      39
##   beta       1     7  352    15     0      97
##   delta      0     2    0    33     0      14
##   gamma      0     6    0     0    62      17
```


## Application of scDetect-Cancer

For the single cell RNA-seq data of the tumor samples. First, we load the [scDetect](https://github.com/IVDgenomicslab/scDetect/) package, and [Seurat](https://satijalab.org/seurat/install.html)

```markdown
library("scDetect")
library("Seurat")
```

We will work with single cell data from a test melanoma dataset. 

Read the gene expression data and cell type lable.

```markdown
# Melanoma reference dataset #
mela_ref<-counts(melanoma_ref)
mela_ref_lable<-melanoma_ref$label

# Melanoma test dataset #
mela_test<-counts(melanoma_test)
```

#### Prediction

To make scDetect-Cancer easy to use, all steps were integrated into one function -- scDetect-Cancer. 

Here, we used Melanoma reference dataset (without tumor cells) as the training dataset to predcit the cell types in a melanoma test dataset. 

The gene position file used for single cell copy number variation analysis and gene list file used for epithelial score analysis could be obtained [here](https://github.com/IVDgenomicslab/scDetect/tree/master/scDetect-Cancer-file).


Create temporary directory.

```markdown
output_dir<-tempdir()
```

```markdown
# Prediction #
scDetect_Cancer_results<-scDetect_Cancer(vali_set_matrix = mela_test, train_set_matrix = mela_ref, train_set_lable = mela_ref_lable, gene_position_file, gene_list, output_dir)
```

We can obtain a list included the prediction results and detailed inforamtion.

The prediction results:

```markdown
scDetect_Cancer_results$lable[1:20]
##   "Tumor"   "Bcell"   "Bcell"   "Unknown" "Tcell"   "Unknown" "Tumor"   "Unknown" "Tumor"   "Bcell"   "Tcell"   "Tcell"   "Tcell"   "Tumor"   "Unknown" "Tcell" "Tumor"   "Bcell"   "Tumor"   "Tcell"
```

The detailed inforamtion:


```markdown
scDetect_Cancer_results$detail_info[1:20,]
##                                        CNV_Class CNV_entropy_score anno_file Epithelial_score Epithelial_pvalue Epithelial_class   raw_lable final_lable
##cy81.Bulk.CD45.neg.B04.S112.comb             Tumor          8.072779     Other        0.2969218      2.881267e-51            Tumor  Fibroblast       Tumor
##cy94_cd45pos_4_C09_S33_comb                  Other          8.078587     Other        0.1443036      1.000000e+00            Other       Bcell       Bcell
##cy72.CD45.pos.D04.S904.comb                  Other          8.077199     Other        0.1684391      9.999999e-01            Other       Bcell       Bcell
##CY88CD45POS_2_D09_S429_comb                  Other          8.079058     Other        0.2982854      8.454728e-52            Tumor       Tcell     Unknown
##CY75_1_CD45_CD8_1__S29_comb                  Other          8.078603     Tcell        0.1341299      1.000000e+00            Other       Tcell       Tcell
##Cy80_II_CD45_F08_S932_comb                   Other          8.077468     Other        0.2424822      3.359626e-25            Tumor Endothelial     Unknown
##cy78.CD45.neg.2.C06.S606.comb                Tumor          8.073131     Other        0.3008013      9.071703e-53            Tumor Endothelial       Tumor
##CY94_CD45NEG_CD90POS_2_C02_S26_comb          Other          8.079323     Tcell        0.2200801      3.368614e-12            Tumor       Tcell     Unknown
##cy78.CD45.neg.2.A05.S581.comb                Tumor          8.069823     Other        0.3436408      4.530180e-67            Tumor  Fibroblast       Tumor
##cy79.p3.CD45.pos.PD1.neg.G01.S169.comb       Other          8.078866     Other        0.1801278      9.928248e-01            Other       Bcell       Bcell
##CY75_1_CD45_CD8_3__S106_comb                 Other          8.079064     Other        0.1324336      1.000000e+00            Other       Tcell       Tcell
##cy53.1.CD45.pos.2.E04.S1012.comb             Other          8.078719     Tcell        0.1793396      9.958538e-01            Other       Tcell       Tcell
##CY89A_CD45_POS_10_A04_S196_comb              Other          8.078433     Other        0.2101634      5.537329e-07            Other       Tcell       Tcell
##cy78.CD45.neg.3.A10.S682.comb                Tumor          8.074718     Other        0.2905377      1.049571e-48            Tumor  Fibroblast       Tumor
##cy80.Cd45.pos.PD1.pos.B01.S37.comb           Other          8.079013     Other        0.2456721      5.311377e-27            Tumor       Tcell     Unknown
##CY84_PRIM_POS_All_7_B11_S215_comb            Other          8.078439     Other        0.1528024      1.000000e+00            Other       Tcell       Tcell
##cy79.p4.CD45.neg.PDL1.neg.B09.S1077.comb     Tumor          8.074930     Other        0.2632536      2.483141e-36            Tumor       Bcell       Tumor
##Cy72_CD45_C03_S699_comb                      Other          8.078826     Other        0.1689189      9.999998e-01            Other       Bcell       Bcell
##cy79.p4.CD45.neg.PDL1.pos.C07.S415.comb      Tumor          8.072491     Other        0.2878176      1.405910e-47            Tumor  Fibroblast       Tumor
##CY75_1_CD45_CD8_8__S293_comb                 Other          8.079135     Other        0.1570361      1.000000e+00            Other       Tcell       Tcell
```


## SessionInfo
```markdown
sessionInfo()
#> R version 3.6.3 (2020-02-29)
#> Platform: x86_64-w64-mingw32/x64 (64-bit)
#> Running under: Windows 7 x64 (build 7600)
#> 
#> Matrix products: default
#> 
#> locale:
#> [1] LC_COLLATE=Chinese (Simplified)_China.936 
#> [2] LC_CTYPE=Chinese (Simplified)_China.936   
#> [3] LC_MONETARY=Chinese (Simplified)_China.936
#> [4] LC_NUMERIC=C                              
#> [5] LC_TIME=Chinese (Simplified)_China.936    
#> 
#> attached base packages:
#> [1] stats     graphics  grDevices utils     datasets  methods   base     
#> 
#> loaded via a namespace (and not attached):
#>  [1] compiler_3.6.3  magrittr_1.5    tools_3.6.3     htmltools_0.4.0
#>  [5] yaml_2.2.1      Rcpp_1.0.3      stringi_1.4.6   rmarkdown_2.1  
#>  [9] knitr_1.28      stringr_1.4.0   xfun_0.12       digest_0.6.25  
#> [13] rlang_0.4.5     evaluate_0.14
```

