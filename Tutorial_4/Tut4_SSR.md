CBC Phylogenetics Tutorial 4: Population Genetics using SSRs
================
Clarke van Steenderen
Last updated 03/09/2020

## AIMS OF THIS TUTORIAL :white\_check\_mark:

1.  Analyse microsatellite (SSR) data in R using a range of different R
    packages
2.  Reproduce the results reported by [Hopper *et. al.*
    (2018)](https://onlinelibrary.wiley.com/doi/full/10.1111/eva.12755)
    for *Neochetina bruchi* SSR data, with a few additional neat
    functions
3.  Convert SSR data into different formats for use across R packages

Many of the functions used here were gleaned from [this great population
genetics
tutorial](http://grunwaldlab.github.io/Population_Genetics_in_R/Introduction.html)
by NJ Grünwald, ZN Kamvar and SE Everhart.

## DOCUMENT CONTENTS

1.  [Overview of the SSR data](#overview)
2.  [DAPC analysis](#dapc)
3.  [STRUCTURE-like “compoplots”](#compoplots)
4.  [Linkage Disequilibrium (poppr)](#LDpoppr)
5.  [Minimum spanning networks](#msn)
6.  [Private Alleles (Ap)](#Ap)
7.  [Convert from genclone/genind](#convert)
8.  [Ar, He, Ho, HWE, and Fis](#diveRsity)
9.  [Inbreeding coefficients using inbreedR](#inbreedR)
10. [FST and Jost’s D values](#fst)
11. [Linkage Disequilibrium (genepop)](#LDgenepop)
12. [Linear Mixed Models for genetic diversity](#lmm)
13. [STRUCTURE for SSR data](#structure_ssr)

## What are SSRs?

SSR (simple-sequence repeats), or “microsatellites” are are repetitive
regions (= motifs) of one to ten nucleotides that occur throughout the
genomes of eukaryotes and prokaryotes (mostly in non-coding regions),
and mutate rapidly. A motif could, for example, be a dinucleotide repeat
unit such as \[AG\]n, or a pentanucleotide unit such as \[AGTTA\]n.  The
diagram below illustrates how SSR regions differ in length (due to
differing numbers of motifs) on either chromosome (as SSRs are
co-dominant), and how these might show up on a gel.

<br/>

![](ssr_gel.png)

<br/>

We will use the data for *Neochetina bruchi* provided in the
supplementary section of the [Hopper *et. al.* 2018 paper](https://onlinelibrary.wiley.com/doi/full/10.1111/eva.12755). This is in the
**Tutorial 4/Data** folder
([**hopper\_neochetina\_b.csv**](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Neochetina_SSR/hopper_neochetina_b.csv)).
The paper reports that raw SSR data were read into Geneious first, and
then the output was opened in Excel and saved in GenAlex format
(Microsoft Excel plugin).

## Overview of the SSR data <a name = "overview"></a>

``` r
if (!require("pacman")) install.packages("pacman") # pacman is a package that installs other required packages
```

    ## Loading required package: pacman

``` r
pacman::p_load(adegenet, car, diveRsity, genepop, ggplot2, inbreedR, lattice, lme4, lmerTest, magrittr, nlme, pegas, PopGenReport, poppr, RColorBrewer, tidyr, vegan)

neochetina_b = poppr::read.genalex("https://raw.githubusercontent.com/CJMvS/CBC_Tutorials/master/Tutorial_4/Data/hopper_neochetina_b.csv")

# create a genotype accumulation curve
gac_neochetina_b = poppr::genotype_curve(neochetina_b, sample = 1000, quiet = TRUE)
```

![](FigsTut4/unnamed-chunk-1-1.png)<!-- -->

``` r
# if you want to change the theme:
p = last_plot() + theme_bw()
p
```

![](FigsTut4/unnamed-chunk-1-2.png)<!-- -->

``` r
# create a locus summary table

locus_tab_neo_b = locus_table(neochetina_b)
```

    ## 
    ## allele = Number of observed alleles

    ## 
    ## 1-D = Simpson index
    ## Hexp = Nei's 1978 gene diversity
    ## ------------------------------------------

``` r
locus_tab_neo_b
```

    ##       summary
    ## locus  allele   1-D  Hexp Evenness
    ##   NB40   7.00  0.59  0.60     0.67
    ##   NB46   7.00  0.40  0.40     0.53
    ##   NB27   2.00  0.49  0.49     0.98
    ##   NB5    8.00  0.38  0.38     0.44
    ##   NB43   4.00  0.14  0.14     0.45
    ##   NB8   11.00  0.78  0.78     0.71
    ##   NB26   6.00  0.64  0.64     0.77
    ##   NB13   7.00  0.47  0.47     0.53
    ##   mean   6.50  0.49  0.49     0.63

``` r
# genotypic diversity:

# MLG = Number of multilocus genotypes (MLG) observed
# eMLG = The number of expected MLG at the smallest sample size ≥ 10 based on rarefaction
# H = Shannon-Wiener Index of MLG diversity
# G = Stoddart and Taylor’s Index of MLG diversity
# lambda = Simpson’s Index
# E.5 = Evenness
# Hexp = Nei’s unbiased gene diversity
# these last two are for dealing with linkage disequlibrium, which we'll look at later:
# Ia = The index of association
# rbarD = The standardized index of association

summary_stats = poppr(neochetina_b)
summary_stats
```

    ##         Pop   N MLG  eMLG       SE    H     G lambda   E.5  Hexp       Ia
    ## 1 AUSTRALIA  21  20  9.79 4.10e-01 2.98  19.2  0.948 0.974 0.401  0.03289
    ## 2        CA  25  25 10.00 0.00e+00 3.22  25.0  0.960 1.000 0.464  0.05357
    ## 3        FL  21  21 10.00 0.00e+00 3.04  21.0  0.952 1.000 0.433  0.00688
    ## 4        TX  25  25 10.00 0.00e+00 3.22  25.0  0.960 1.000 0.429  0.20575
    ## 5       SAB   6   6  6.00 0.00e+00 1.79   6.0  0.833 1.000 0.547 -0.26531
    ## 6       SAE  18  18 10.00 5.43e-07 2.89  18.0  0.944 1.000 0.550  0.14677
    ## 7    UGANDA  26  25  9.86 3.45e-01 3.20  24.1  0.959 0.979 0.432  0.40249
    ## 8   URUGUAY  29  29 10.00 1.03e-06 3.37  29.0  0.966 1.000 0.426  0.37781
    ## 9     Total 171 169  9.99 7.85e-02 5.13 167.1  0.994 0.993 0.488  0.19941
    ##      rbarD         File
    ## 1  0.00511 neochetina_b
    ## 2  0.00793 neochetina_b
    ## 3  0.00101 neochetina_b
    ## 4  0.03645 neochetina_b
    ## 5 -0.03915 neochetina_b
    ## 6  0.02167 neochetina_b
    ## 7  0.06870 neochetina_b
    ## 8  0.05538 neochetina_b
    ## 9  0.02936 neochetina_b

``` r
# multilocus genotype table. High evenness (E.5 value close to, or equal to 1) will correspond with consistent genotype numbers in graphs (i.e a more equal abundance of MLGs, where there isn't one, or a group of genotypes that predominate)
n_b_tab = mlg.table(neochetina_b)
```

![](FigsTut4/unnamed-chunk-1-3.png)<!-- -->

``` r
# rarefaction curve to see how sample size affects genotypic richness results. A group might initially appear to have more genotypes than another, but that's only because it has a larger sample size.
# Have a look at how the eMLG (expected MLG values, which are based on a refraction curve) differ from the observed MLG values

min_sample = min(rowSums(n_b_tab))
vegan::rarecurve(n_b_tab, sample = min_sample, xlab = "Sample Size", ylab = "Expected MLGs",
                 main = "Rarefaction of Neochetina bruchi populations")
```

![](FigsTut4/unnamed-chunk-1-4.png)<!-- -->

This rarefaction graph is not as drastic as the one presented
[here](http://grunwaldlab.github.io/Population_Genetics_in_R/Genotypic_EvenRichDiv.html).
Now we can check whether there are any deviations from Hardy-Weinberg
Equilibrium (HWE) across and within each population. See page 75 of the
**Introduction to Conservation Genetics** by Frankham *et. al* 2002 for
some really great definitions and explanations of various population
genetics concepts (in the **Helpful Books** folder). See also [this definition](https://www.nature.com/scitable/definition/hardy-weinberg-equilibrium-122/).

``` r
# HW equilibrium (for each marker across all populations)
neo_b.hwe = pegas::hw.test(neochetina_b, B = 1000) # performs 1000 permutations
neo_b.hwe 
```

    ##          chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40 241.62465 21 0.000000e+00    0.000
    ## NB46 125.15612 21 1.110223e-16    0.000
    ## NB27   7.05961  1 7.884115e-03    0.008
    ## NB5   60.28414 28 3.745816e-04    0.071
    ## NB43  42.18962  6 1.686862e-07    0.027
    ## NB8  256.53829 55 0.000000e+00    0.000
    ## NB26  28.20725 15 2.030713e-02    0.009
    ## NB13  16.58297 21 7.360548e-01    0.305

``` r
# HWE for each marker within each population:
neo_b.hwe.pop = seppop(neochetina_b) %>% lapply(hw.test, B = 1000)
neo_b.hwe.pop
```

    ## $AUSTRALIA
    ##            chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40  0.41200783  3 9.377509e-01    0.887
    ## NB46 21.84000000  3 7.042735e-05    0.000
    ## NB27  0.04338843  1 8.349956e-01    1.000
    ## NB5   0.38349160  3 9.436312e-01    1.000
    ## NB43  0.01249256  1 9.110057e-01    1.000
    ## NB8  13.72000000 10 1.861506e-01    0.118
    ## NB26  2.40304311  6 8.791570e-01    0.698
    ## NB13  1.56198347  3 6.680402e-01    1.000
    ## 
    ## $CA
    ##           chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40  5.5074611  6 4.805554e-01    0.193
    ## NB46 19.6111111  3 2.043380e-04    0.001
    ## NB27  3.0738742  1 7.955966e-02    0.099
    ## NB5   0.9070295  3 8.237311e-01    1.000
    ## NB43 25.0472590  3 1.509315e-05    0.046
    ## NB8  27.1648586 21 1.654628e-01    0.026
    ## NB26  5.0302457  6 5.399398e-01    0.291
    ## NB13  1.9888231  6 9.207237e-01    1.000
    ## 
    ## $FL
    ##           chi^2 df Pr(chi^2 >) Pr.exact
    ## NB40  3.9002836  6 0.690168739    0.424
    ## NB46 21.0581717  6 0.001790981    0.042
    ## NB27  0.2568249  1 0.612309921    0.663
    ## NB5   0.3834916  6 0.998981706    1.000
    ## NB43  0.1242604  3 0.988775019    1.000
    ## NB8  21.5933333 21 0.423259647    0.029
    ## NB26  0.1620370  3 0.983471757    1.000
    ## NB13  5.4791439  6 0.483981027    0.348
    ## 
    ## $TX
    ##           chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40  0.0000000  0 1.000000e+00    1.000
    ## NB46 41.7355372  6 2.073621e-07    0.000
    ## NB27  0.9872206  1 3.204226e-01    0.432
    ## NB5   0.6625203  3 8.819821e-01    1.000
    ## NB43  0.1890359  3 9.793396e-01    1.000
    ## NB8  15.9743487 10 1.003691e-01    0.198
    ## NB26  2.4963018  3 4.759598e-01    0.343
    ## NB13 12.0867347  6 6.006160e-02    0.058
    ## 
    ## $SAB
    ##          chi^2 df Pr(chi^2 >) Pr.exact
    ## NB40 6.3750000  6   0.3825186    0.193
    ## NB46 0.2400000  3   0.9708874    1.000
    ## NB27 0.3750000  1   0.5402914    1.000
    ## NB5  3.0612245  3   0.3822816    0.657
    ## NB43 0.6666667  1   0.4142162    1.000
    ## NB8  4.3600000  6   0.6280814    0.864
    ## NB26 6.2400000  3   0.1005000    0.147
    ## NB13 5.6250000  6   0.4664793    0.506
    ## 
    ## $SAE
    ##          chi^2 df Pr(chi^2 >) Pr.exact
    ## NB40 17.431953 10  0.06533589    0.028
    ## NB46 22.888889 15  0.08652761    0.000
    ## NB27  3.306122  1  0.06902218    0.121
    ## NB5  15.245000 10  0.12338162    0.057
    ## NB43  0.281250  1  0.59588309    1.000
    ## NB8  29.039715 21  0.11305127    0.389
    ## NB26  2.450617  3  0.48428179    0.360
    ## NB13  1.469388  1  0.22544232    0.537
    ## 
    ## $UGANDA
    ##          chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40  7.940877  6 2.424669e-01    0.034
    ## NB46 32.587350  6 1.258755e-05    0.001
    ## NB27  4.012755  1 4.515730e-02    0.102
    ## NB5   1.235000  6 9.751207e-01    1.000
    ## NB43  0.000000  0 1.000000e+00    1.000
    ## NB8   2.081327 10 9.956780e-01    0.900
    ## NB26  5.747018 10 8.360509e-01    0.737
    ## NB13  5.155834  3 1.607354e-01    0.125
    ## 
    ## $URUGUAY
    ##          chi^2 df  Pr(chi^2 >) Pr.exact
    ## NB40 90.592301 21 1.275255e-10    0.000
    ## NB46 21.086029  6 1.770446e-03    0.005
    ## NB27  3.399967  1 6.519772e-02    0.107
    ## NB5  31.752870 21 6.200875e-02    0.133
    ## NB43  3.473136  3 3.242630e-01    0.258
    ## NB8  23.559927 36 9.450839e-01    0.744
    ## NB26 14.284145 10 1.604209e-01    0.073
    ## NB13  3.074968 15 9.995314e-01    0.950

``` r
neo_b.hwe.mat = sapply(neo_b.hwe.pop, "[", i = TRUE, j = 3) # Take the third column with all rows (chi square p-value)

# produce a heat map to show deviations from HWE across populations and markers
alpha  = 0.05 # set significance at p <= 0.05
newmat_Neo_b = neo_b.hwe.mat
newmat_Neo_b[newmat_Neo_b > alpha] = 1
colrs = colorRampPalette(brewer.pal(8, "Blues"))(25) # could choose Blues, Greens, Reds, BuPu, etc.
lattice::levelplot(t(newmat_Neo_b), col.regions = colrs, ylab = "SSR Marker", xlab = "Population", main = "Deviations from HWE",
                   scales=list(x=list(rot=90))) 
```

![](FigsTut4/unnamed-chunk-2-1.png)<!-- -->

``` r
# the 't' before the newmat_Neo_b just swaps rows and columns around so that population is on the x and SSR marker on the y
# the scales argument changes the text direction. Here it's changing the x labels to 90 degrees
```

## DAPC analysis <a name = "dapc"></a>

Let’s have a look at genetic cluster groups; similar to a STRUCTURE
analysis. The Hopper *et. al.* (2018) paper mentions that they didn’t
present STRUCTURE results in their main document, because the program
assumes HWE and LE (linkage equilibrium). It seems advisable to first
run the tests for HWE and LE assumptions, and then decide which genetic
clustering method to use.

> Using the **adegenet** package, we will create a discriminant analysis
> of principal components (DAPC). It is very important to first obtain
> an optimal number of PCs to use in the analysis. Too many leads to
> overestimates of accuracy (over-fitting), but too few loses power to
> discriminate between groups.

``` r
set.seed(123) # this makes the results reproducible if you run it again at another time

best_a_score = adegenet::optim.a.score(dapc(neochetina_b, n.pca = 50, n.da = nPop(neochetina_b))) # shows that the best no. of PCs is 21
```

![](FigsTut4/unnamed-chunk-3-1.png)<!-- -->

``` r
# another validation check for the number of PCs:
x = neochetina_b 
mat = tab(x, NA.method = "mean") # the tab function accesses allele counts for each sample at each locus. The NA.method = "mean" replaces any NA values with the mean, if applicable
gr = pop(x) # access group names. Same as using as.factor(neochetina_b$pop)
xval = xvalDapc(mat, gr, n.pca.max = 300, training.set = 0.9, result = "groupMean", center = T, scale = F, n.pca = NULL, n.rep = 50, xval.plot = T) # a training set of 90% of the data is sub-setted, and used to test the remaining 10%
```

![](FigsTut4/unnamed-chunk-3-2.png)<!-- -->

``` r
xval[2:6] # this indicates that the best no. of PCs is 25
```

    ## $`Median and Confidence Interval for Random Chance`
    ##       2.5%        50%      97.5% 
    ## 0.08355022 0.12388505 0.17917426 
    ## 
    ## $`Mean Successful Assignment by Number of PCs of PCA`
    ##         5        10        15        20        25        30        35        40 
    ## 0.3432500 0.4125417 0.3915833 0.4359167 0.4551667 0.4149167 0.3995833 0.4275000 
    ## 
    ## $`Number of PCs Achieving Highest Mean Success`
    ## [1] "25"
    ## 
    ## $`Root Mean Squared Error by Number of PCs of PCA`
    ##         5        10        15        20        25        30        35        40 
    ## 0.6609563 0.5947203 0.6127145 0.5713401 0.5510788 0.5907444 0.6072273 0.5773408 
    ## 
    ## $`Number of PCs Achieving Lowest MSE`
    ## [1] "25"

``` r
# select the lowest of the two PCs from the two methods

# plot the DAPC
dapc.neo_b = dapc(neochetina_b, var.contrib = TRUE, scale = FALSE, n.pca = 21, n.da = nPop(neochetina_b)) # set the n.pca to 21, as predicted by the optim.a.score() function above
levels(pop(neochetina_b)) # confirm the number of populations in the data, so that you can assign colours:
```

    ## [1] "AUSTRALIA" "CA"        "FL"        "TX"        "SAB"       "SAE"      
    ## [7] "UGANDA"    "URUGUAY"

``` r
colrs = c("blue", "black", "deeppink", "grey", "gold", "orange", "green", "red")
scatter(dapc.neo_b, clabel = F, cstar = 1, cell = 1, col = colrs, cex = 1, solid = 0.8, legend = T, scree.pca = T) 
```

![](FigsTut4/unnamed-chunk-3-3.png)<!-- -->

``` r
# mstree = TRUE adds a minimum spanning tree
# cstar adds in lines joining points
# cell = 1 adds ellipses around groups

# if you want to automatically assign colours, you can use this:
jBrewColors = brewer.pal(n = 8, name = "Set1") # see this site for more colour options! https://www.datanovia.com/en/blog/the-a-z-of-rcolorbrewer-palette/#:~:text=Display%20all%20brewer%20palettes,-To%20display%20all&text=Sequential%20palettes%20(first%20list%20of,YlGn%2C%20YlGnBu%20YlOrBr%2C%20YlOrRd. 
scatter(dapc.neo_b, clabel = F, cstar = 1, cell = 1, col = jBrewColors, cex = 1, solid = 0.8, legend = T, scree.pca = T) 
```

![](FigsTut4/unnamed-chunk-3-4.png)<!-- -->

``` r
# if you want to view discriminant functions as distribution graphs instead
# discriminant function 1:
scatter(dapc.neo_b, 1, 1, col = colrs, solid = 0.4, legend = T)
```

![](FigsTut4/unnamed-chunk-3-5.png)<!-- -->

``` r
# discriminant function 2:
scatter(dapc.neo_b, 2, 2, col = colrs, solid = 0.4, legend = T)
```

![](FigsTut4/unnamed-chunk-3-6.png)<!-- -->

``` r
# have a look at the loading plot to see which marker produced the most variation
# axis refers to PC1 or PC2
# setting the thresh allows for all markers with a loading above that to be labelled on the graph
contrib = loadingplot(dapc.neo_b$var.contr, axis = 2, thresh = 0.1, lab.jitter = 1)
```

![](FigsTut4/unnamed-chunk-3-7.png)<!-- -->

Hopper *et. al* report an additional two DAPC plots in Figure 2, where
they have removed the SAE and SAB populations. Let’s do that here, using
the **popsub()** function in the poppr package.

``` r
# removing SAB
neo_sub_SAB = popsub(neochetina_b, blacklist = "SAB")
# get the new optimal PCA number
best_a_score = optim.a.score(dapc(neo_sub_SAB, n.pca = 50, n.da = nPop(neo_sub_SAB)))
```

![](FigsTut4/unnamed-chunk-4-1.png)<!-- -->

``` r
# create the DAPC
dapc.neo_b2 = dapc(neo_sub_SAB, var.contrib = TRUE, scale = FALSE, n.pca = 20, n.da = nPop(neo_sub_SAB))
levels(pop(neo_sub_SAB)) # confirm the number of populations in the data, so that you can assign colours:
```

    ## [1] "AUSTRALIA" "CA"        "FL"        "TX"        "SAE"       "UGANDA"   
    ## [7] "URUGUAY"

``` r
colrs = c("blue", "black", "deeppink", "grey", "orange", "green", "red")
# plot the DAPC
scatter(dapc.neo_b2, clabel = F, cstar = 1, cell = 1, col = colrs, cex = 1, solid = 0.8, legend = T, scree.pca = T) #mstree = TRUE adds a minimum spanning tree
```

![](FigsTut4/unnamed-chunk-4-2.png)<!-- -->

``` r
# remove both SAB and SAE
neo_sub_SAB.SAE = popsub(neochetina_b, blacklist = c("SAB", "SAE"))
best_a_score = optim.a.score(dapc(neo_sub_SAB.SAE, n.pca = 50, n.da = nPop(neo_sub_SAB.SAE)))
```

![](FigsTut4/unnamed-chunk-4-3.png)<!-- -->

``` r
dapc.neo_b3 = dapc(neo_sub_SAB.SAE, var.contrib = TRUE, scale = FALSE, n.pca = 27, n.da = nPop(neo_sub_SAB.SAE))
levels(pop(neo_sub_SAB.SAE)) # confirm the number of populations in the data, so that you can assign colours:
```

    ## [1] "AUSTRALIA" "CA"        "FL"        "TX"        "UGANDA"    "URUGUAY"

``` r
colrs = c("blue", "black", "deeppink", "grey","green", "red")
scatter(dapc.neo_b3, clabel = F, cstar = 1, cell = 1, col = colrs, cex = 1, solid = 0.8, legend = T, scree.pca = T) #mstree = TRUE adds a minimum spanning tree
```

![](FigsTut4/unnamed-chunk-4-4.png)<!-- -->

Now we can start having a look at how accurate our predefined groups are
based on the DAPC analysis.

``` r
# get a summary of the DAPC on all populations
s = summary(dapc.neo_b)
s$assign.prop # the DAPC analysis had an overall accuracy of 66% (i.e. overall successful assignment of samples to their predefined cluster groups)
```

    ## [1] 0.6608187

``` r
s$assign.per.pop # accuracy of sample assignment to each predefined group. The least accurate was the Australia group (48%), and the highest Texas (TX) (84%)
```

    ## AUSTRALIA        CA        FL        TX       SAB       SAE    UGANDA   URUGUAY 
    ## 0.4761905 0.5600000 0.6190476 0.8400000 0.6666667 0.7222222 0.5769231 0.7931034

``` r
# we can plot a bar chart showing these accuracy figures:
par(mar=c(6, 5, 2, 1))
success_vals = summary(dapc.neo_b)$assign.per.pop*100
barplot_success_props = barplot(success_vals, ylab = "% reassignment to actual cluster group", xlab = "Cluster Group", ylim = c(0,100), las = 2, main = "Successful reassignments to cluster groups for N. bruchi", col = "white") # las changes the orientation of x labels
text(barplot_success_props, success_vals, labels = round(success_vals, 0), pos = 3, col = "black", cex = 0.8)
```

![](FigsTut4/unnamed-chunk-5-1.png)<!-- -->

``` r
# you can access the posterior probability values for each sample to a group with this line:
# dapc.neo_b$posterior

# graphical representation of posterior probabilities of each sample to each possible group (i.e. assignment accuracy):
# heat colours represent cluster group probability, and blue dots are the predefined groups for individuals
# really nice way to see what the probability is of an individual falling in a specific cluster group compared to what was expected/predefined
assignplot(dapc.neo_b, cex = 0.6, pch=16)
```

![](FigsTut4/unnamed-chunk-5-2.png)<!-- -->

> Having a look at the assignment plot for all populations, one can
> visually see that the Australian group had quite a low accuracy rate
> (48%) (darker red = higher probability of assignment).

Zoom in on just the Australian samples :mag:

``` r
assignplot(dapc.neo_b, cex.lab = 0.65, only.grp = "AUSTRALIA", pch=16)
```

![](FigsTut4/unnamed-chunk-6-1.png)<!-- -->

Look at sample **NBAU2**, for example. It was predefined as an
Australian sample (blue dot), but it had a very high posterior
probability of belonging to the SAE population. If we have a look again
at the posterior probability values (using **dapc.neo\_b$posterior**),
you will see that this sample was assigned to the Australian group with
a probability score of 0.0069, and to the SAE group with a value of
0.973:

``` r
dapc.neo_b$posterior[2,] # NBAU2 is the second sample. Use index numbers to access particular posterior probability values, or the name of the sample:
```

    ##    AUSTRALIA           CA           FL           TX          SAB          SAE 
    ## 6.903191e-03 1.253965e-02 2.112057e-03 1.253633e-05 1.385026e-03 9.726540e-01 
    ##       UGANDA      URUGUAY 
    ## 8.001135e-04 3.593447e-03

``` r
# dapc.neo_b$posterior["NBAU2",]
```

Now zoom in on the Texas population :mag:

``` r
assignplot(dapc.neo_b, cex.lab = 0.8, only.grp = "TX", pch=16)
```

![](FigsTut4/unnamed-chunk-8-1.png)<!-- -->

We can see that most assignments were accurate (84%), but some had
higher probabilities of belonging elsewhere. For example, sample
**NBTX2** had a higher probability of belonging to the Uruguay group

We can also plot a contingency table (Fig. 2(d) in the Hopper paper) to
get an overall view of how well samples are reassigned to their
predefined groups:

``` r
table.value(table(dapc.neo_b$assign, pop(neochetina_b)), col.lab = levels(pop(neochetina_b)))
```

![](FigsTut4/unnamed-chunk-9-1.png)<!-- -->

In this table, columns represent the predefined population groups that you assigned to samples, and the rows represent the cluster that the DAPC algorithm placed the samples in.

## STRUCTURE-like “compoplots” <a name = "compoplots"></a>

Similar to the output of STRUCTURE, we can create a graphic showing the
membership posterior probabilities of each sample to our predefined
groups.

``` r
# we will assign the same colours here as we did in the DAPC plot
mycol = c("blue", "black", "deeppink", "grey", "gold", "orange", "green", "red")
adegenet::compoplot(dapc.neo_b, show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-1.png)<!-- -->

``` r
# otherwise, you can just pick random colours, using this line, for example:
# mycol = rainbow(8)

# subset data to look at particular populations: 
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="AUSTRALIA"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-2.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="CA"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-3.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="FL"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-4.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="TX"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-5.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="SAB"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-6.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="SAE"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-7.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="UGANDA"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-8.png)<!-- -->

``` r
adegenet::compoplot(dapc.neo_b, subset = (dapc.neo_b$grp=="URUGUAY"), show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-9.png)<!-- -->

``` r
# create a subset of samples with a proportion of successful reassignments less than a certain threshold
# for example, less than a 40% probability of belonging to a specific cluster group (an indication of high admixture):
temp = which(apply(dapc.neo_b$posterior, 1, function(e) all(e < 0.4)))
compoplot(dapc.neo_b, subset = temp, show.lab = T, posi = "topright", col.pal = mycol)
```

![](FigsTut4/unnamed-chunk-10-10.png)<!-- -->

## Linkage Disequilibrium (poppr) <a name = "LDpoppr"></a>

The null hypothesis for LD is that there is no linkage between markers
(i.e. the presence of one locus doesn’t determine the presence of
another; all loci are independent. In such a case, there is not
significant LD, or rather, there is a state of linkage equilibrium).
[This YouTube
video](https://www.youtube.com/watch?v=OnAk6cEbRyU&ab_channel=BigBio)
explains LD very well, and helped me to better understand the concept.
Clonal populations (or those with high levels of interbreeding) will
typically have significant linkage between loci, where sexual ones
won’t.

> LD is expressed using the Index of Association (IA and rd values),
> with an associated p-value. High index values indicate linkage
> disequilibrium (i.e. alleles are linked, and not independent) We will
> first have a look at assessing LD using the poppr package. Hopper *et.
> al.* used the genepop package, which we’ll get to later. In the figure below, heatmap colours represent the index value, and the values printed on each block are p-values.

``` r
# LD across marker pairs over all populations:
LD_pairs = pair.ia(neochetina_b, sample = 999, index = "rbarD") 
```

![](FigsTut4/unnamed-chunk-11-1.png)<!-- -->

``` r
# extract just the p-values for rD
rD_p_vals = LD_pairs[,4]
# find how many are significant (i.e. are in significant linkage disequilibrium)
length(which(rD_p_vals < 0.05))
```

    ## [1] 4

``` r
# if you want to look at individual populations, you can first subset them:
LD.Aus = popsub(neochetina_b, "AUSTRALIA")
LD.CA = popsub(neochetina_b, "CA")
LD.FL = popsub(neochetina_b, "FL")
LD.TX = popsub(neochetina_b, "TX")
LD.SAB = popsub(neochetina_b, "SAB")
LD.SAE = popsub(neochetina_b, "SAE")
LD.UGANDA = popsub(neochetina_b, "UGANDA")
LD.URUGUAY = popsub(neochetina_b, "URUGUAY")

# and then have a look at each one:
# Australia:
LD.Aus # 21 diploid individuals, with 20 original MLGs. This means that one sample was identical to another 
```

    ## 
    ## This is a genclone object
    ## -------------------------
    ## Genotype information:
    ## 
    ##    20 original multilocus genotypes 
    ##    21 diploid individuals
    ##     8 codominant loci
    ## 
    ## Population information:
    ## 
    ##     1 stratum - Pop
    ##     1 populations defined - AUSTRALIA

``` r
rD.Aus = pair.ia(LD.Aus, sample = 999)
```

![](FigsTut4/unnamed-chunk-11-2.png)<!-- -->

``` r
# and the number of marker pairs that showed a significant p-value:
length(which(rD.Aus[,4] < 0.05))
```

    ## [1] 2

``` r
# do the same for the others, if required.
```

## Minimum spanning networks <a name = "msn"></a>

Let’s have a quick look at the markers used in the Hopper study:

| Locus  | Primer Sequences 5’-3’                      | Dye | bp      | Motif   | (Q)/Ta° |
| ------ | ------------------------------------------- | --- | ------- | ------- | ------- |
| Nb\_40 | F: GCCTCCCTCGCGCCAAGCCACTCGTGCTAGACTTC      | FAM | 168-212 | AAT(10) | 60      |
|        | R: GTTTATCAGCAGCCTCAATAACCTC                |     |         |         |         |
| Nb\_46 | F: GCCTTGCCAGCCCGCTTGGTCAGGGTTGTGAGAAG      | VIC | 217-244 | AGG(8)  |         |
|        | R: GTTTCGAACACCGTGACAGTTAAAG                |     |         |         |         |
| Nb\_5  | F: GCCTTGCCAGCCCGCGTTTCCGGTTCAGGGTTGTG      | VIC | 230-257 | AGG(9)  | (Q) 64  |
|        | R: GTTTACCACCTACGCATAATCCC                  |     |         |         |         |
| Nb\_27 | F: GCCTCCCTCGCGCCAGCAGACTTATCCGATCTCAAGG    | FAM | 201-207 | AGG(8)  |         |
|        | R: GTTTATTCTGTCAGGGTTGTGAGAC                |     |         |         |         |
| Nb\_43 | F: CAGGACCAGGCTACCGTGTTTCGAACCGCACAAGATCC   | NED | 199-211 | AGG(8)  |         |
|        | R: GTTTAGAACCTCCTCCTCTTGTCAG                |     |         |         |         |
| Nb\_8  | F: GCCTTGCCAGCCCGCTTGAGTTAGCTAGACTTCGCC     | VIC | 263-290 | AAT(13) | (Q) 63  |
|        | R: GTTTCTCACAACCCTGACAAGAGG                 |     |         |         |         |
| Nb\_13 | F: CGGAGAGCCGAGAGGTGTTTGTGAGGTCGCTGGTTAC    | PET | 224-245 | AGG(10) |         |
|        | R: GTTTACAAGGAAATTCTGCCAGGG                 |     |         |         |         |
| Nb\_26 | F: CAGGACCAGGCTACCGTGAGATGGGAAATGATGTTGCTTG | NED | 208-223 | AGG(8)  |         |
|        | R: GTTTCTGCATGGAAATTCTGTCAGG                |     |         |         |         |

``` r
# specify the nucleotide repeats for each motif for each marker
reps = c(NB40 = 3, NB46 = 3,  NB27 = 3, NB5 = 3, NB43 = 3,  NB8 = 3, NB26 = 3, NB13 = 3)
msn_grps = c("AUSTRALIA", "CA", "FL", "SAB", "SAE", "TX", "UGANDA", "URUGUAY")
msn_grps2 = c("TX", "UGANDA") # make a list of populations you want to include
# plot the network:
msn_neo_b = bruvo.msn(neochetina_b, replen = reps, sublist = msn_grps2, showplot = F)
# or, use the plot_poppr_msn function:
set.seed(120) # makes sure that you get the same output if this is run again (as opposed to a ranmdom start)
# inds = "none" removes labels, nodebase scales the size of the nodes to log(1.25)
# could use palette = cm.colors, heat.colors, terrain.colors, or use the RCOlor Brewer to choose other palettes
cols_msn <- colorRampPalette(brewer.pal(8, "Paired")) #Blues, Greens, Reds, BuPu
plot_poppr_msn(neochetina_b, msn_neo_b, inds = "none", palette = cols_msn)
```

![](FigsTut4/unnamed-chunk-12-1.png)<!-- -->

We will get the values reported in Table 2 of the Hopper paper in the
sections that follow:

## Private Alleles (Ap) <a name = "Ap"></a>

``` r
# private alleles (Ap). These are alleles that occur in only one population
Ap = poppr::private_alleles(neochetina_b, count.alleles = F)
rowSums(Ap)
```

    ## AUSTRALIA        CA        FL        TX       SAB       SAE    UGANDA   URUGUAY 
    ##         0         1         0         0         0         3         1         8

## Convert from genclone/genind –\> genepop format <a name = "convert"></a>

> To start working in other R packages, such as diveRsity, inbreedR,
> PopGenReport, and genepop, we need to convert the genalex file we
> initially read from a **genclone/genind** object into a **genepop**
> object. The following function does this, and was taken from
> [this](https://github.com/romunov/zvau/blob/master/R/writeGenPop.R)
> GitHub repository.

``` r
writeGenPop <- function(gi, file.name, comment) {
  
  if (is.list(gi)) {
    # do all genind objects have the same number of loci?
    if (length(unique(sapply(gi, nLoc))) != 1) stop("Number of loci per individual genind object in a list is not equal for all.")
    gi.char <- gi[[1]]
    loc.names <- locNames(gi[[1]])
  } else {
    gi.char <- gi
    loc.names <- locNames(gi)
  }
  
  # Calculate the length of two alleles.
  lng <- as.character(na.omit(genind2df(gi.char)[, locNames(gi.char)[1]]))
  lng <- unique(nchar(lng))
  
  stopifnot(length(lng) == 1)
  
  cat(paste(comment, "\n"), file = file.name)
  cat(paste(paste(loc.names, collapse = ", "), "\n"), file = file.name, append = TRUE)
  
  if (is.list(gi)) {
    pop.names <- seq_len(length(gi))
  } else {
    pop.names <- popNames(gi)
  }
  
  for (i in pop.names) {
    cat("pop\n", file = file.name, append = TRUE)
    if (is.list(gi)) {
      intm <- gi[[i]]
      loc.names <- locNames(gi[[i]])
    } else {
      intm <- gi[pop(gi) == i, drop = FALSE]
    }
    ind.names <- indNames(intm)
    intm <- genind2df(intm, sep = "")
    intm[is.na(intm)] <- paste(rep("0", lng), collapse = "")
    out <- cbind(names = paste(ind.names, ",", sep = ""), intm[, loc.names])
    write.table(out, file = file.name, row.names = FALSE, col.names = FALSE, append = TRUE, quote = FALSE)
  }
  
  return(NULL)
}
```

Let’s use this to convert the **neochetina\_b** object into genepop
format, and then run some basic stats (rare alleles, expected and
observed heterozygosities, HWE, and inbreeding coefficients) using the
diveRsity package.

## Ar, He, Ho, HWE, and Fis <a name = "diveRsity"></a>

Ar = rare alleles  
He = expected heterozygosity  
Ho = observed heterozygosity  
HWE = Hardy-Weinberg Equilibrium  
Fis = fixation index/inbreeding coefficient

``` r
# specify where the output file must be written to, and save as a .txt file
writeGenPop(neochetina_b, "~/CBC_tuts/Tutorial_4/Data/Neochetina_b_writeGenPop.txt", comment = "Neochetina_b object converted using the writeGenPop function") 
```

    ## NULL

``` r
# read in the converted file, and then specify a name for the output results folder 
# we'll run 1000 bootstraps for these tests, and we'll set the Hardy-Weinberg exact test to 10 000 monte carlo repetitions
div_basic_stats = diveRsity::divBasic("~/CBC_tuts/Tutorial_4/Data/Neochetina_b_writeGenPop.txt", outfile = "diveRsity_Results", gp = 3, bootstraps = 1000, HWEexact = T, mcRep = 10000)
```

The converted genepop-format file is
[here](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Data/Neochetina_b_writeGenPop.txt).

Have a look at the “overall” column in the [\[divBasic\] output Excel
file](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/diveRsity_Results-%5BdiveRsity%5D/%5BdivBasic%5D.xlsx),
and compare the values to those in Hopper’s Table 2.

The significant FIS values are marked in bold in the table. This was
ascertained through the upper and lower confidence intervals. If the
range of the upper and lower CI values do not cross zero, then the value
is significant:

``` r
# First we want to extract the relevant information from our results
ciTable <- lapply(div_basic_stats$fis, function(x){
  return(c(x["overall", "fis"], x["overall", "BC_lower_CI"],
           x["overall", "BC_upper_CI"]))
})
# convert this into a dataframe
ciTable <- as.data.frame(do.call("rbind", ciTable))
dimnames(ciTable) <- list(paste("pop_", 1:nrow(ciTable), sep = ""),
                          c("Fis", "lower", "upper"))
# inspect the table
ciTable
```

    ##           Fis   lower  upper
    ## pop_1  0.1022 -0.0470 0.2543
    ## pop_2  0.1535  0.0098 0.3036
    ## pop_3  0.1408  0.0007 0.3026
    ## pop_4  0.0723 -0.0433 0.1863
    ## pop_5 -0.0381 -0.3338 0.1652
    ## pop_6  0.2084  0.0688 0.3420
    ## pop_7  0.0463 -0.1037 0.1754
    ## pop_8  0.2173  0.0743 0.3484

``` r
# plot the CI values
ggplot(ciTable, aes(x = levels(pop(neochetina_b)), y = Fis)) +
  geom_point() + 
  geom_errorbar(aes(ymin = lower, ymax = upper), width = 0.3) +
  theme(axis.text.x=element_text(angle = 90))
```

![](FigsTut4/unnamed-chunk-16-1.png)<!-- -->

> From this graph, CA, SAE, and URUGUAY are significant. You can tell this by looking at whether the error bars span over the x-axis. Those that don't are significant.

## Inbreeding coefficients using inbreedR <a name = "inbreedR"></a>

Let’s see how the g2 and pg2 values (inbreeding coefficients) were
obtained:

``` r
setwd("~/CBC_tuts/Tutorial_4/Data")
neochetina_inbreedR = read.csv("https://raw.githubusercontent.com/CJMvS/CBC_Tutorials/master/Tutorial_4/Data/hopper_neochetina_b.csv", row.names = 1) # read in the same original genalex data, but as a plain .csv file now instead of read.genalex()
neochetina_inbreedR = neochetina_inbreedR[-c(1,2),] # remove rows 1 and 2 
colnames(neochetina_inbreedR)[1]="pop" # reassign this column name to make things easier down the line

# create an empty data frame to store results
g2.data = matrix(ncol=3, nrow=8)
colnames(g2.data) = c("Population", "g^2", "p")
g2.data[,1] = levels(as.factor(neochetina_inbreedR$pop))

# australia
aus_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="AUSTRALIA",-1]) # convert the dataframe to the format that the inbreedR package requires
g.aus = inbreedR::g2_microsats(aus_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[1,2]=round(g.aus$g2,2)
g2.data[1,3]=round(g.aus$p_val,2)

# california
CA_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="CA",-1])
g.CA = inbreedR::g2_microsats(CA_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[2,2]=round(g.CA$g2,2)
g2.data[2,3]=round(g.CA$p_val,2)

# florida
FL_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="FL",-1])
g.FL = inbreedR::g2_microsats(FL_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[3,2]=round(g.FL$g2,2)
g2.data[3,3]=round(g.FL$p_val,2)

# SAB
SAB_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="SAB",-1])
g.SAB = inbreedR::g2_microsats(SAB_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[4,2]=round(g.SAB$g2,2)
g2.data[4,3]=round(g.SAB$p_val,2)

# SAE
SAE_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="SAE",-1])
g.SAE = inbreedR::g2_microsats(SAE_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[5,2]=round(g.SAE$g2,2)
g2.data[5,3]=round(g.SAE$p_val,2)

# texas
TX_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="TX",-1])
g.TX = inbreedR::g2_microsats(TX_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[6,2]=round(g.TX$g2,2)
g2.data[6,3]=round(g.TX$p_val,2)

# uganda
UGANDA_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="UGANDA",-1])
g.UGANDA = inbreedR::g2_microsats(UGANDA_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[7,2]=round(g.UGANDA$g2,2)
g2.data[7,3]=round(g.UGANDA$p_val,2)

#uruguay
URU_inbreedR = inbreedR::convert_raw(neochetina_inbreedR[neochetina_inbreedR$pop=="URUGUAY",-1])
g.URUGUAY = inbreedR::g2_microsats(URU_inbreedR, nperm = 1000, nboot = 1000, verbose = F)
g2.data[8,2]=round(g.URUGUAY$g2,2)
g2.data[8,3]=round(g.URUGUAY$p_val,2)

g2.data = as.data.frame(g2.data) ;g2.data
```

    ##   Population   g^2    p
    ## 1  AUSTRALIA  0.06  0.2
    ## 2         CA  0.11 0.04
    ## 3         FL   0.1 0.11
    ## 4        SAB  0.01  0.5
    ## 5        SAE  0.02 0.37
    ## 6         TX -0.01  0.6
    ## 7     UGANDA  0.01 0.45
    ## 8    URUGUAY -0.05 0.82

## FST and Jost’s D values <a name = "fst"></a>

Let’s get the pairwise FST and Jost’s D values, as reported in Table 3
of the Hopper paper. We’ll use the PopGenReport package.
The pairwise FST and Jost's D values measure the fraction/proportion of allelic variation/differention among populations. The higher the value, the more genetically distant they are from each other. 

``` r
# First, we need to convert the original neochetina_b data into a genind object using the genclone2genind() function:
convert_neo = poppr::genclone2genind(neochetina_b)
# pairwise FST values:
fst_vals = pairwise.fstb(convert_neo)
# round them off to two decimal places
fst_vals = round(fst_vals,2) ;fst_vals
```

    ##           AUSTRALIA   CA   FL   TX  SAB  SAE UGANDA URUGUAY
    ## AUSTRALIA      0.00 0.03 0.02 0.10 0.04 0.07   0.01    0.06
    ## CA             0.03 0.00 0.04 0.05 0.04 0.05   0.03    0.04
    ## FL             0.02 0.04 0.00 0.09 0.04 0.08   0.03    0.08
    ## TX             0.10 0.05 0.09 0.00 0.10 0.08   0.09    0.07
    ## SAB            0.04 0.04 0.04 0.10 0.00 0.04   0.04    0.05
    ## SAE            0.07 0.05 0.08 0.08 0.04 0.00   0.07    0.09
    ## UGANDA         0.01 0.03 0.03 0.09 0.04 0.07   0.00    0.06
    ## URUGUAY        0.06 0.04 0.08 0.07 0.05 0.09   0.06    0.00

``` r
# Jost's D:
# this creates a folder in the location specified by path.pgr, with all the results, saved in a folder called "results"
jostsD = popgenreport(convert_neo, mk.counts = F, mk.differ.stats = T, mk.pdf = F, path.pgr="~/CBC_tuts/Tutorial_4")
```

    ## Warning in dir.create(file.path(path.pgr, foldername)): 'C:
    ## \Users\s1000334\Documents\CBC_tuts\Tutorial_4\results' already exists

    ## There is no  results  folder. I am trying to create it; 
    ## otherwise please create the folder manually. 
    ## Compiling report...
    ##  - No valid coordinates were provided. 
    ##    Be aware you need to provide a coordinate (or NA) for each individual
    ##    and the coordinate heading in slot @other has to be 'latlong' or 'xy'.
    ##    Some of the analyses require coordinates and will be skipped!
    ## - Pairwise differentiations ...
    ## Analysing data ...
    ## All files are available in the folder: 
    ## ~/CBC_tuts/Tutorial_4/results

These results are
[here](https://github.com/CJMvS/CBC_Tutorials/tree/master/Tutorial_4/results).

## Linkage Disequilibrium (genepop) <a name = "LDgenepop"></a>

Now let’s have a look at Linkage Disequilibrium again, using the genepop
package. The [**LD.txt**
file](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/LD_genepop.txt)
produced here is in the Tutorial 4 folder.

``` r
# we'll read in the file that was converted using the writeGenPop function again
# this writes out a results .txt file in the same folder
test_LD("C:/Users/s1000334/Documents/CBC_tuts/Tutorial_4/Data/Neochetina_b_writeGenPop.txt", outputFile = "LD_genepop.txt", settingsFile = "", dememorization = 10000, batches = 100, iterations = 5000,  verbose = interactive())
```

These results correspond with the supplementary data (Appendix 1, Table
S3) in the Hopper paper, where only one pairwise difference was
significant for the *N. bruchi* markers (Nb27 and Nb26; Chi-square
\>91.293022, df = 16, p \<1.44e-12).

## Linear Mixed Models for genetic diversity <a name = "lmm"></a>

The final part of this tut will look at the comparisons of allelic
richness and expected heterozygosity across populations using linear
mixed models. Page 779 of the Hopper paper states:

> “To compare genetic diversity among the introduced and native
> populations, we tested for the effects of population (collection site)
> on genetic diversity by fitting linear mixed models (LMM) with the
> lmer function in the lme4 package (Bates, Maechler, Bolker, & Walker,
> 2015). Implementing an LMM accounts for the variability of the
> microsatellite loci by modeling locus as a random effect, and
> collection site \[population\] as a fixed effect with allelic richness
> or expected heterozygosity as the response variables in separate
> models.”

Let’s reproduce that by extracting the allelic richness and expected
heterozygosity results from the **div\_basic\_stats** object we created
earlier using the divBasic() function in the diveRsity package. We’ll
use the **nlme** package, which does exactly the same thing as lme4; it
just has a slighlty different format for arguments.

``` r
# extract allele richness data
allele_richness_neo_b = div_basic_stats$Allelic_richness
allele_richness_neo_b = allele_richness_neo_b[-9,] # remove the "overall" row
allele_richness_neo_b = cbind(marker = rownames(allele_richness_neo_b), allele_richness_neo_b) # make a column for marker names
allele_richness_neo_b = as.data.frame(allele_richness_neo_b) # make sure it's a data frame object
allele_richness_neo_b = tidyr::gather(allele_richness_neo_b, key = "Pop", value = "Allelic_richness", -marker) # use the gather function in the tidyr package to reorganise data so that you can model it
str(allele_richness_neo_b) # allelic richness is a character class at the moment
```

    ## 'data.frame':    64 obs. of  3 variables:
    ##  $ marker          : chr  "NB40" "NB46" "NB27" "NB5" ...
    ##  $ Pop             : chr  "NBAU1," "NBAU1," "NBAU1," "NBAU1," ...
    ##  $ Allelic_richness: chr  "2.96" "1.87" "2" "2.05" ...

``` r
allele_richness_neo_b$Allelic_richness = as.numeric(allele_richness_neo_b$Allelic_richness) # change it to numeric
# create factors for marker and population:
ssr.marker = as.factor(allele_richness_neo_b$marker)
ssr.pop = as.factor(allele_richness_neo_b$Pop)

# Create Linear Mixed Models:
#----------------------------

# model first without any variance structure for the response variable:
# allelic richness as the response and population as the fixed variables, and marker as the random effect:
mod1 = nlme::lme(Allelic_richness~ssr.pop, random = ~1|ssr.marker, data = allele_richness_neo_b)
plot(mod1, pch=16, main="Allelic richness ~ Population, \nwith no variance structure.") # the residuals seem to be increasing as the fitted values increase, and so we'll create another model where we apply a different variance structure to each population.
```

![](FigsTut4/unnamed-chunk-20-1.png)<!-- -->

``` r
Anova(mod1)
```

    ## Analysis of Deviance Table (Type II tests)
    ## 
    ## Response: Allelic_richness
    ##         Chisq Df Pr(>Chisq)
    ## ssr.pop  8.68  7     0.2765

``` r
# give a variance structure to each population using the varIdent() function:
varstruc <- varIdent(form= ~ 1|ssr.pop)
mod2 = lme(Allelic_richness ~ ssr.pop, random = ~1 | ssr.marker, weights = varstruc, data = allele_richness_neo_b)
plot(mod2, pch=16, main="Allelic richness ~ Population, with a unique \nvariance structure for each population") # these residuals look better
```

![](FigsTut4/unnamed-chunk-20-2.png)<!-- -->

``` r
Anova(mod2) # Hopper reported Chi^2 of 11.03, df = 7, and p = 0.14
```

    ## Analysis of Deviance Table (Type II tests)
    ## 
    ## Response: Allelic_richness
    ##          Chisq Df Pr(>Chisq)
    ## ssr.pop 11.087  7     0.1349

``` r
anova(mod1, mod2) # comparing the two models, the second one with the variance structure is significantly better (smaller AIC and larger log likelihood)
```

    ##      Model df      AIC      BIC    logLik   Test  L.Ratio p-value
    ## mod1     1 10 153.2396 173.4931 -66.61981                        
    ## mod2     2 17 149.2844 183.7154 -57.64223 1 vs 2 17.95516  0.0122

Let’s do the same for expected heterozygosities:

``` r
# extract expected heterozygosity data
heterozyg_neo_b = div_basic_stats$He
heterozyg_neo_b = heterozyg_neo_b[-9,] # remove the "overall" row
heterozyg_neo_b = cbind(marker = rownames(heterozyg_neo_b), heterozyg_neo_b) # make a column for marker names
heterozyg_neo_b = as.data.frame(heterozyg_neo_b) # make sure it's a data frame object
heterozyg_neo_b = tidyr::gather(heterozyg_neo_b, key = "Pop", value = "Allelic_richness", -marker) # use the gather function in the tidyr package to reorganise data so that you can model it
str(heterozyg_neo_b) # currently a character class
```

    ## 'data.frame':    64 obs. of  3 variables:
    ##  $ marker          : chr  "NB40" "NB46" "NB27" "NB5" ...
    ##  $ Pop             : chr  "NBAU1," "NBAU1," "NBAU1," "NBAU1," ...
    ##  $ Allelic_richness: chr  "0.65" "0.25" "0.5" "0.22" ...

``` r
heterozyg_neo_b$He = as.numeric(heterozyg_neo_b$Allelic_richness) # change it to numeric
# create factors for marker and population:
ssr.marker = as.factor(heterozyg_neo_b$marker)
ssr.pop = as.factor(heterozyg_neo_b$Pop)

# Create Linear Mixed Models:
#----------------------------

# model first without any variance structure
mod3 = lme(He ~ ssr.pop, random = ~1 | ssr.marker, data = heterozyg_neo_b)
plot(mod3, pch=16, main="Heterozygosity ~ Population")
```

![](FigsTut4/unnamed-chunk-21-1.png)<!-- -->

``` r
Anova(mod3) # Hopper reported Chi^2 of 6.89, df = 7, p = 0.44
```

    ## Analysis of Deviance Table (Type II tests)
    ## 
    ## Response: He
    ##          Chisq Df Pr(>Chisq)
    ## ssr.pop 6.4167  7      0.492

``` r
# experiment with a variance structure to each population:
varstruc1 <- varIdent(form= ~ 1|ssr.pop)
mod4 = lme(He ~ ssr.pop, random = ~1 | ssr.marker, weights = varstruc1, data = heterozyg_neo_b)
plot(mod4, pch=16, main="Heterozygosity ~ Population with a unique \nvariance structure for each population")
```

![](FigsTut4/unnamed-chunk-21-2.png)<!-- -->

``` r
Anova(mod4)
```

    ## Analysis of Deviance Table (Type II tests)
    ## 
    ## Response: He
    ##          Chisq Df Pr(>Chisq)
    ## ssr.pop 7.1361  7     0.4149

``` r
anova(mod3, mod4) # not a significant difference between models: variance structure not important here
```

    ##      Model df       AIC      BIC   logLik   Test L.Ratio p-value
    ## mod3     1 10 -5.327086 14.92643 12.66354                       
    ## mod4     2 17 -4.857284 29.57370 19.42864 1 vs 2 13.5302  0.0602

## STRUCTURE for SSR data (co-dominant) <a name = "structure_ssr"></a>

The table below shows the template for co-dominant data for STRUCTURE:

|          |   | Locus 1 |     | Locus 2 |     |
| -------- | - | ------- | --- | ------- | --- |
| Sample A | 1 | 260     | 260 | 240     | 241 |
| Sample B | 1 | 260     | 262 | 240     | 242 |
| Sample C | 2 | 250     | \-9 | 235     | 230 |
| Sample D | 3 | 255     | 255 | 270     | 270 |
| Sample E | 3 | 255     | 250 | 275     | \-9 |

Edit the
[**hopper\_neochetina\_b.csv**](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Neochetina_SSR/hopper_neochetina_b.csv)
file such it looks like the [text
file](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Neochetina_SSR/hopper_neochetina_b.txt)
with the same name (in the
[**Tutorial 4/Neochetina\_SSR**](https://github.com/CJMvS/CBC_Tutorials/tree/master/Tutorial_4/Neochetina_SSR)
folder). Replace population names with integers to signify grouping.

> :bulb: SSR data input is the same as for ISSRs, but because this is
> co-dominant (i.e. you can tell whether alleles are heterozygous or
> not), there are two columns for each locus. The *Neochetina* data
> contains 171 individuals and 8 loci. All the input parameters are the
> same as before, except that **ploidy** must be set to 2, and the
> **““special format data: file stores data for individuals in a
> single line”** check box must be selected in the project wizard
> settings.

In the Hopper *et al.* (2018) data, the groups are:

  - 1 = Australia
  - 2 = California
  - 3 = Florida
  - 4 = Texas
  - 5 = SA wolseley
  - 6 = SA Enseleni
  - 7 = Uganda
  - 8 = Uruguay

The [.txt file for
colours](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Neochetina_SSR/colours.txt),
and the [.txt file for group
names](https://github.com/CJMvS/CBC_Tutorials/blob/master/Tutorial_4/Neochetina_SSR/names.txt)
is in the **Tutorial 4/Neochetina\_SSR** folder.

In supplementary data file appendix 5 of the Hopper paper, STRUCTURE was
run with 1 million mcmc reps and 100 000 burnin; with K set from 1 to
10, with 10 repeats for each K value. Just as a demonstration, we’ll run
this with 10 000 burnin and 5000 mcmc reps, with K set from 1 to 10, and
8 repeats for each value of K. We’ll also set the threshold from 0.5 to
0.8.

From this short run, Structure Selector found support for **K = 2** and
**K = 3**. The CLUMPAK output for these are:

## ![](k=2_neochetina_unsupervised.png)

<br/> <br/>

![](k=3_neochetina_unsupervised.png)

For K = 3, half of the 8 repeats produced the first plot, and the other
half produced the second.

:bulb: Let’s re-run this analysis with the same parameters as before,
but under the **Ancestry Model** settings, select **“Use sampling
locations as prior (LOCPRIOR)”**.

Zip the **Results** folder, and upload to Structure Selector again. This
analysis supports K = 2, K = 3, and K = 4. Comparing these to the
unsupervised CLUMPAK results:

![](clumpak_compare.png)

## Supervised vs unsupervised STRUCTURE analysis <a name = "compare_structure"></a>

To compare the supervised and unsupervised runs for a particular K
(we’ll just do this for the major cluster groups for K = 2), after
Structure Selector has run for each, select the desired K value to
produce the graphical presentation. Click on **Download CLUMPAK
results** (both of these folders are in the
**Tutorial\_4/Neochetina\_SSR** folder, saved as
[**CLUMPAK\_supervisedK=2**](https://github.com/CJMvS/CBC_Tutorials/tree/master/Tutorial_4/Neochetina_SSR/CLUMPAK_supervisedK%3D2/1600179573)
and
[**CLUMPAK\_unsupervisedK=2**](https://github.com/CJMvS/CBC_Tutorials/tree/master/Tutorial_4/Neochetina_SSR/CLUMPAK_unsupervisedK%3D2/1600181772)).
Open each nested folder until you get to one called **MajorCluster**.
Open the **CLUMPP.files** folder. Create a new folder (here I named them
**supervised\_clumppIndFile.output** and
**unsupervised\_clumppIndFile.output**), and paste the
**ClumppIndFile.output** into it. Zip the folder. (Do this for both the
supervised and unsupervised runs). Open the
[CLUMPAK](http://clumpak.tau.ac.il/) server, and go to the **Compare**
tab. Upload both zipped folders you created, each containing the
ClumppIndFile.output. The similarity score for the supervised and
unsupervised run here is 0.82 (82%). This is the output:

<br/> <br/>

![](supervised_vs_unsupervised.png)
