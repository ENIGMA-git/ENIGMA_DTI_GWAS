---
  title: ENIGMA-DTI Quality Control (QC) Protocol - Pre Imputation
  author: [Barbara Molz, Gabriëlla A. M. Blokland, Nina Roth Mota]
  date: April 2024
---

---

# ENIGMA QC - Pre Imputation Protocol

_Version 1.1 - July 25, 2024_

**Written by: Barbara Molz, Gabriëlla A. M. Blokland, Nina Roth Mota**

Please note this protocol was developed for the ENIGMA consortium. If you are not part of the ENIGMA consortium and wish to use this protocol please register on the ENIGMA mailing list so that we can contact you regarding any updates or issues that arise relating to this protocol. You can register at <http://enigma.ini.usc.edu/members/join/>

If you use data generated by this protocol for non-ENIGMA projects please include the following citation for the protocol in your work:

> ENIGMA Genetics Support Team. ENIGMA 1KGP_p3v5 Cookbook [Online].The Enhancing Neuroimaging Genetics through Meta-Analysis (ENIGMA) Consortium. <http://enigma.ini.usc.edu/genetics-protocols/>

## **IMPORTANT** Read first!

This document contains ENIGMA-DTI-GWAS project instructions for different levels of prior genotype processing. If you have previously contributed to an ENIGMA GWAS (following their QC protocol and imputation using 1000 Genomes Phase 3 version 5 **All-ethnicity reference panel**) reimputation might not be required for your data set or only partially (i.e. Chr X).

Please make sure to preprocess and run your diffusion MRI data through the <a href="https://github.com/ENIGMA-git#enigma-dti-imagingENIGMA">TBSS pipeline</a> first and only use data for the subsequent preImputationQC from individuals who passed this pipeline.

If you haven't done so yet, please also fill out the preQC form for each of your cohorts: <a href="https://docs.google.com/forms/d/1g-dnBJMQjpcNZRjrnZh9l9cPQCVv9z7cxhMjzoklIdI/edit?usp=sharing"> preQCSummaryData</a> .

**Before you begin, please check carefully which case outlined below applies to you!**

<u>**Case 1**</u>

If you have not run your QC and imputation according to the 2017 Enigma protocol (<https://enigma.ini.usc.edu/wp-content/uploads/2020/02/ENIGMA-1KGP_p3v5-Cookbook_20170713.pdf> found on <https://enigma.ini.usc.edu/protocols/genetics-protocols/>), please run the **current ENIGMA-DTI QC**, according to the instructions below as well as the latest imputation protocol (which you will receive after submitting the QC-outputs described below).

<u>**Case 2**</u>

If you have run your QC and imputation according to the 2017 Enigma protocol (see Case 1 for links) with the **X-Chromosome included** in the imputation, your sample does not require any reimputing.
Please do however still **run the current ENIGMA-DTI QC protocol below** (~1-2h) as it outputs important basic statistics needed for consortium-level QC and return the details as described. This can even be done with already QCed data as input, where both input and output should be fairly similar, thus can be used as a sanity check.

<u>**Case 3**</u>

If you have run your QC and imputation according to the 2017 Enigma protocol (see Case 1 for links), **excluding the X-Chromosome imputation**, you might only want to rerun the X-Chromosome imputation.

Please do still **run the current ENIGMA-DTI QC protocol below** (~1-2h) as it outputs important basic statistics needed for consortium-level QC and return the details as described. This can even be done with already QCed data as input, where both input and output should be fairly similar, thus can be used as a sanity check.

Further, **continue with the ENIGMA-DTI imputation protocol only for the X chromosome** (provided by email upon request after sending the QC information), which will contain information on which steps can be skipped/adapted.

<br/>

> **NOTE:** If none of the cases above apply to you or if in doubt, please contact us at **enigma.DTIgenetics@gmail.com** for advice!

<br/>

## Programs required for this protocol

It is assumed that you have access to a server or computer running Linux/Unix. If this is not the case please contact us at enigma.DTIgenetics@gmail.com. Before we start, you need to download and install some required programs (which you may already have). The required programs are:

- Plink1.9 can be downloaded here: <https://www.cog-genomics.org/plink2>
- R can be downloaded here: <http://cran.stat.ucla.edu/>

Once you have installed these software tools, ensure executable program files are on your search path:

```bash
export PATH=/path/to/plink:/path/to/R:$PATH
```

Please let us know if you run into problems along the way. Please address any questions to: <enigma.DTIgenetics@gmail.com>

## Important Notes

Before starting, please ensure your genetic data aligns with the GRCh37/b37/hg19 genome builds.

You should start this protocol using a dataset in a binary file format for PLINK1.9, that means, in a .bed/.bim/.fam format. For more information, visit: <https://www.cog-genomics.org/plink/1.9/input#bed>

If your cohort has a **case-control** design, please assign phenotype (i.e. case-control status) in the Plink (.fam) file – coding controls or non-affected individuals as 1 and cases as 2. This information will be incorporated in the plink log files and will help with QC and troubleshooting. If in doubt, please follow the population-based instructions below.

If your cohort is **population-based** please code all individuals as if they were controls (1).
This could be changed by e.g. changing the respective column in the .fam file with

```bash
awk '$6=1' < yourinput.fam > output.fam
```

Please ensure that sex is also encoded in the .fam file (male=1 and female=2).

**Very important** Please remove individuals who do not have DTI data available before running this protocol. You can do this by using the function --keep on plink1.9 and a list\* of individuals, so that your genetic data is restricted to only those with also available DTI data.

> \*The list of individuals is a text file with family IDs (FID) in the first column and within-family IDs (IID) in the second column, one line per individual, which will remove all unlisted samples from the current dataset.
> Example line of Plink code to exclude non-DTI imaging individuals:

```bash
plink --bfile all_genotyped_filename --keep dti_fid-iid_list.txt --make-bed --out genotyped_with_dti_filename
```

## File naming convention for your bed/bim/fam files

COHORT_ANCESTRY_ANALYST_DATE.bed/bim/fam

- Cohort = cohort abbreviation in upper case
- Ancestry = EUR (or AFR, AMR or EAS,...)
- Analyst= Initials of analyst’s name
- Date on which file was prepared, Format: YYYYMMDD

If your cohort was included in the Grasby et al (2020) ENIGMA paper, please use the appropriate abbreviation according to Supplementary table 2 (<https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7295264/bin/NIHMS1586184-supplement-S1-S20.xlsx>); if it was not, please provide a short cohort abbreviation to be used for file labeling throughout the project.

We are working with the assumption that the ideal minimal final sample size of each participating cohort is approximately 100 individuals. If you know/suspect that this might not be the case for your cohort, please contact us at <enigma.DTIgenetics@gmail.com>

\*\*\*

# PRE IMPUTATION PIPELINE

Paste the code lines below into a terminal window or shell script, substitute the correct values where text is highlighted in
<span style="color: red;">red</span>. (Rendering in red is only available in the HTML version, if you are unsure what to change, please use this version and not the GitHub readMe). 

For this protocol, we will assume that your working directory is:

```bash
$HOME/enigma/DTIgenetics/
mkdir -p $HOME/enigma/DTIgenetics/
```

Replace **yourrawdata** with the file name according to the convention mentioned before (no extensions needed)

Place <span style="color: red;">yourrawdata</span>.bim/.bed/.fam in this directory

```bash
cd $HOME/enigma/DTIgenetics/
```

## Sex-check

Let’s start by checking for sex-mismatch from the phenotypic and the genetic data.
First, ensure sex (as entered into the phenotype database) is encoded in the .fam file (male=1 and female=2).
From the directory where your dataset (i.e. your binary plink files = <span style="font-family:'consolas';color: red;">yourrawdata</span>.bim/.bed/.fam) is located, type the following commands to perform a series of steps:

`export datafile=`<span style="font-family:'consolas';color: red;background-color:rgb(220,220,220)">yourrawdata</span>

### Perform SNP filtering and pruning for a more accurate sex-check analysis:

```bash
plink --bfile ${datafile} --mind 1 --hwe 1e-06 --geno 0.01 --maf 0.05 --make-bed --out ${datafile}_filtered
plink --bfile ${datafile}_filtered --indep-pairphase 20000 2000 0.5 --out ${datafile}_pruned
plink --bfile ${datafile}_filtered --extract ${datafile}_pruned.prune.in --make-bed --out ${datafile}_pruned
```

### Split-X to deal with pseudo-autosomal regions

First, check if your data is already split, i.e. whether it contains an XY region.

You can check if this is the case with the following code line:

```bash
awk '{if ($1==25) print $1}' ${datafile}_pruned.bim | wc -l
```

If this number is >0 there is already an XY region. In that case, skip the next plink command below

```bash
plink --bfile ${datafile}_pruned --split-x b37 no-fail --make-bed --out ${datafile}_splitX
```

<br/><br/>

> **Note:** If this fails you can also try to change the flag to `--split-x hg19` instead of `b37` or try `no-fail` with or without single **''** !

As a rule of thumb, F estimates < 0.2 yield female calls, and F>0.8 yield male calls. Unless you have experience and reason enough to bend these thresholds slightly, we will use them as default:

<br/>

> **Note:** If you skipped the command above please continue with `${datafile}_pruned` file instead of the `${datafile}_splitX`.

<br/>

```bash
plink --bfile ${datafile}_splitX --check-sex 0.2 0.8 --out ${datafile}_sexcheck
```

Check the log file and the **${datafile}\_sexcheck** file to identify if problems were detected. You can use the following command to select individuals with sex mismatch:

```bash
grep PROBLEM ${datafile}_sexcheck.sexcheck > sex.drop
```

The next command just extracts some important information to be sent to us as a general sanity check:

```bash
wc -l sex.drop > ${datafile}_sexcheck_PROBLEM.txt
```

We now remove samples with sex inconsistencies between PEDSEX and SNPSEX information from your initial raw dataset:

```bash
plink --bfile ${datafile} --remove sex.drop --make-bed --out ${datafile}_QC1
```

<br/>

> **Note:** Please perform the step above even if no sex mismatch was detected in order to keep file name consistency.

<br/>

For the purpose of reporting, we also count the number of SNPs on the X chromosome

```bash
awk '{if ($1=="23") print $0}' ${datafile}_QC1.bim | wc -l > snp_count_X.txt
echo "The number of SNPs on the X chromosome is:"
cat snp_count_X.txt
```

## Check for related and duplicated samples

Starting from the dataset already cleared for sex mismatch we will check for relatedness among samples and also identify possible duplicated samples. For this analysis, we will also perform SNP filtering and pruning first:

```bash
plink --bfile ${datafile}_QC1 --mind 1 --geno 0.01 --maf 0.05 --make-bed --out ${datafile}_QC1tmp
plink --bfile ${datafile}_QC1tmp --indep-pairwise 100 5 0.2 --out ${datafile}_QC1pruned
plink --bfile ${datafile}_QC1tmp --extract ${datafile}_QC1pruned.prune.in --make-bed --out ${datafile}_QC1pruned
```

The following command creates a list with possible duplicates called `pihat_duplicates.genome` (PI_HAT > 0.9)

`plink --bfile ${datafile}_QC1pruned --genome --min 0.9 --out pihat_duplicates`

<br/>

> **Note:**

- If this file contains duplicates (but not monozygotic twins), remove one of each pair and save it again;
- If the file has both duplicates and monozygotic twin, please use other filtering commands from PLINK to remove ONLY the duplicates;
- Please perform the step above even if no duplicated sample was detected or rename your file (i.e. `${datafile}_QC1` to `${datafile}_QC2`) in order to keep file name consistency.

<br/>

Now we exclude one of each pair of duplicated samples. Open the `pihat_duplicates.genome` file and create a secondary pihat_duplicates.txt file yourself containing only FID and IID columns of _one_ of each pair. This can be done with e.g. using `awk` and extracting the first two columns of `pihat_duplicates.genome` file and continue with the following command:

```bash
plink --bfile ${datafile}_QC1 --remove pihat_duplicates.txt --make-bed --out ${datafile}_QC2
```

The next command again just extracts some information to be sent to us as a sanity check.

```bash
wc -l pihat_duplicates.txt > ${datafile}_QC1pruned_duplicates_count.txt
```

We will repeat this to get a rough estimate about relatedness in your sample. While not excluded this will allow us to get a better estimate of unreleated individuals for later steps.

`plink --bfile ${datafile}_QC1pruned --genome --min 0.25 --max 0.9 --out pihat_relatedness`

The next command again just extracts the information to be sent to us as a sanity check.

```bash
wc -l pihat_relatedness.genome > ${datafile}_QC1pruned_relatedness_count.txt
```

## Multi-Dimensional Scaling (MDS) Protocol

Please note if you ran the MDS Protocol for ENIGMA1, 2 or 3 and you have not added any new individuals or new genotyping you do **not** need to re-run the MDS analysis!

<br/>

> **Note:** The MDS protocol assumes you are using a bash shell. To check which shell you are using on your server type:

<br/>

`echo $SHELL`

If you are not using bash please change to the bash shell to run the protocol by typing:

`bash`

when you are finished, go back to your usual shell (if you weren’t already in bash) type:

`exit`

We will download the customized reference data from the following webpages to your working directory which we called `$HOME/enigma/DTIgenetics/` where all files from the previous steps are located.

`cd $HOME/enigma/DTIgenetics/`

Download the following 3 files from <https://enigma.ini.usc.edu/website_downloads/ENIGMA_DTI_downloads/HapMap3/> and put them in the directory created above.

```bash
wget https://enigma.ini.usc.edu/website_downloads/ENIGMA_DTI_downloads/HapMap3/HM3_b37.bed.gz
wget https://enigma.ini.usc.edu/website_downloads/ENIGMA_DTI_downloads/HapMap3/HM3_b37.bim.gz
wget https://enigma.ini.usc.edu/website_downloads/ENIGMA_DTI_downloads/HapMap3/HM3_b37.fam.gz
```

> **Note:** If the wget command errors out or downloads an html file, please download everything directly from the website by pointing your browser to https://enigma.ini.usc.edu/website_downloads/ENIGMA_DTI_downloads/HapMap3/
> If you do not have `wget` installed, you can install it using `apt-get install wget` or `yum install wget` on Linux, or `brew install wget` on Mac (assuming you have package manager Homebrew installed).

Starting from the dataset generated in the previous QC step (i.e. **\${datafile}\_QC2**), filter SNPs out from your dataset which do not meet Quality Control criteria for the MDS analysis (Minor Allele Frequency < 0.01; Genotype Call Rate < 95%; Hardy-Weinberg Equilibrium < 1x10<sup>-6</sup>).

```bash
plink --bfile ${datafile}_QC2 --mind 1 --hwe 1e-6 --geno 0.05 --maf 0.01 --make-bed --out ${datafile}_QC2_filtered
```

Unzip the **HM3_b37 genotypes**. Prepare the HM3_b37 and the raw genotype data by extracting only SNPs that are in common between the two genotype data sets - this avoids exhausting the system memory. We are also removing the strand ambiguous SNPs from the genotyped data set to avoid strand mismatch among these SNPs. Your genotype files should be filtered to remove markers that do not satisfy the quality control criteria above.

```bash
gunzip HM3_b37.*
awk '{print $2}' HM3_b37.bim > HM3_b37.snplist.txt
plink --bfile ${datafile}_QC2_filtered --extract HM3_b37.snplist.txt --make-bed --out ${datafile}_QC2local
awk '{ if (($5=="T" && $6=="A")||($5=="A" && $6=="T")||($5=="C" && $6=="G")||($5=="G" && $6=="C")) print $2, "ambig"; else print $2 ;}' ${datafile}_QC2_filtered.bim | grep -v ambig > local.snplist.txt
plink --bfile HM3_b37 --extract local.snplist.txt --make-bed --out HM3_b37_external
```

Merge the two sets of plink files – In merging the two files plink will check for strand differences. If any strand differences or multiallelic SNPs are found, plink will crash. Ignore warnings regarding different physical positions.

```bash
plink --bfile ${datafile}_QC2local --bmerge HM3_b37_external.bed HM3_b37_external.bim HM3_b37_external.fam --make-bed --out ${datafile}_QC2local_HM3b37merge
```

If plink crashed with a strand error (e.g. ERROR: Stopping due to mis-matching SNPs -- check +/- strand?) and or multiallelic error (e.g. Error: 4 variants with 3+ alleles present), run the following two lines of alternate code:

```bash
plink --bfile ${datafile}_QC2local --flip ${datafile}_QC2local_HM3b37merge-merge.missnp --make-bed --out ${datafile}_QC2local_flipped
plink --bfile ${datafile}_QC2local_flipped --bmerge HM3_b37_external.bed HM3_b37_external.bim HM3_b37_external.fam --make-bed --out ${datafile}_QC2local_HM3b37merge
```

If after the alternate step above plink still crashed with a multiallelic error (e.g. Error: 4 variants with 3+ alleles present), examine the HM3_b37merg-merge.missnp file and remove the problematic multiallelic SNPs by running the alternative code:

```bash
plink --bfile ${datafile}_QC2local_flipped --exclude  ${datafile}_QC2local_HM3b37merge-merge.missnp --make-bed --out ${datafile}_QC2local_flipped2
plink --bfile ${datafile}_QC2local_flipped2 --bmerge HM3_b37_external.bed HM3_b37_external.bim HM3_b37_external.fam --make-bed --out ${datafile}_QC2local_HM3b37merge
```

<br/>

> **Note:** If Plink fails with either error message, try flipping the SNPs first (the first alternate code above) and only exclude the multiallelic SNPs, as a second step.

<br/>

### Run the MDS analysis

```bash
plink --bfile ${datafile}_QC2local_HM3b37merge --cluster --mind .05 --mds-plot 10 --extract local.snplist.txt --out ${datafile}_QC2_HM3b37mds
```

### Plot the MDS results using R

Format the plink output into a tab delimited and an R compatible format.

```bash
cat ${datafile}_QC2_HM3b37mds.mds | tr -s ' ' '\t' > ${datafile}_QC2_HM3b37.mds.tsv
awk 'BEGIN{OFS=","};{print $1, $2, $3, $4, $5, $6, $7}' >> ${datafile}_QC2_HM3b37mds2R.mds.csv ${datafile}_QC2_HM3b37mds.mds
```

From this point until the end of the section, you are working in the R statistical package. Type `R` to start R in unix and `q()` followed by `n` to close the R session after the plot has been created.

Please change the **DATASETNAME** below to match cohort-specific abbreviation of your sample (e.g. BIG) and **DATAFILE** with the full root name used before in the shell scripts
You can find out the actual name via `echo $datafile` in the shell and ensure you are running R from the same working directory as before (i.e. where the .csv file is located --> `$HOME/enigma/DTIgenetics/${datafile}_QC2_HM3b37mds2R.mds.csv`

Now we start R by typing `R`

```r
#In R
library(calibrate)
#If you don’t have the calibrate package, install it using
#install.packages("calibrate")
mds.cluster = read.csv("DATAFILE_QC2_HM3b37mds2R.mds.csv", header=T);
#first we create an additional column holding the respective ancestry labels
mds.cluster$POP=rep("DATASETNAME",length(mds.cluster$C1));
mds.cluster$POP[which(mds.cluster$FID == "CEU")] <- "CEU";
mds.cluster$POP[which(mds.cluster$FID == "CHB")] <- "CHB";
mds.cluster$POP[which(mds.cluster$FID == "YRI")] <- "YRI";
mds.cluster$POP[which(mds.cluster$FID == "TSI")] <- "TSI";
mds.cluster$POP[which(mds.cluster$FID == "JPT")] <- "JPT";
mds.cluster$POP[which(mds.cluster$FID == "CHD")] <- "CHD";
mds.cluster$POP[which(mds.cluster$FID == "MEX")] <- "MEX";
mds.cluster$POP[which(mds.cluster$FID == "GIH")] <- "GIH";
mds.cluster$POP[which(mds.cluster$FID == "ASW")] <- "ASW";
mds.cluster$POP[which(mds.cluster$FID == "LWK")] <- "LWK";
mds.cluster$POP[which(mds.cluster$FID == "MKK")] <- "MKK";

#get the colors for plotting
colors=rep("red",length(mds.cluster$C1));
colors[which(mds.cluster$POP == "CEU")] <- "lightblue";
colors[which(mds.cluster$POP == "CHB")] <- "brown";
colors[which(mds.cluster$POP == "YRI")] <- "yellow";
colors[which(mds.cluster$POP == "TSI")] <- "green";
colors[which(mds.cluster$POP == "JPT")] <- "purple";
colors[which(mds.cluster$POP == "CHD")] <- "orange";
colors[which(mds.cluster$POP == "MEX")] <- "grey50";
colors[which(mds.cluster$POP == "GIH")] <- "black";
colors[which(mds.cluster$POP == "ASW")] <- "darkolivegreen";
colors[which(mds.cluster$POP == "LWK")] <- "magenta";
colors[which(mds.cluster$POP == "MKK")] <- "darkblue";

pdf(file="mdsplot_DATASETNAME_QC2_outliersincluded.pdf",width=7,height=7)

#Please note - the plot will directly be saved as a PDF in your current working directory under the above specified name
#and does not appear in the PLOT window
plot(rev(mds.cluster$C2), rev(mds.cluster$C1), col=rev(colors), ylab="Dimension 1", xlab="Dimension 2",pch=20)
minor.tick(nx = 5, ny = 5,   # Ticks density
         tick.ratio = 0.5) # Ticks size
legend("bottomleft", c("DATASETNAME", "CEU", "CHB", "YRI", "TSI", "JPT", "CHD", "MEX", "GIH", "ASW","LWK", "MKK"), fill=c("red", "lightblue", "brown", "yellow", "green", "purple", "orange", "grey50", "black", "darkolivegreen", "magenta", "darkblue"))
dev.off();
#If you want to know the subject ID label of each sample on the graph, you can label your sample points by uncommenting the commands below.
#This is optional and you can choose not to do this if you are worried about patient information being sent; when you send us your MDS plot please make sure the subject ID labels are NOT on the graph.
#POPlabels <- c("CEU", "CHB", "YRI", "TSI", "JPT", "CHD", "MEX", "GIH", "ASW", "LWK", "MKK");
#textxy(mds.cluster[which(!(mds.cluster$POP %in% POPlabels)), "C2"], mds.cluster[which(!(mds.cluster$POP %in% POPlabels)), "C1"], mds.cluster[which(!(mds.cluster$POP %in% POPlabels)), "IID"])
```

</span>

Your output will look something like this when viewed as a PDF file:

> **Note:** The dotted lines indicate the cut-offs chosen for this specific data set, the dotted circle indicates the EU cluster. These details will not be part of your plot and are just included here to help you decide on an appropriate threshold in your sample.

![MDS PLOT Outliers included](https://github.com/ENIGMA-git/ENIGMA_DTI_GWAS/blob/main/Pre%20Imputation%20Quality%20Control%20Protocol/mdsplot_g1000mix_QC2_outliersincluded.jpg?raw=true)

Or in case of a more diverse data set the PDF might look like this:
![MDS PLOT Outliers included](https://github.com/ENIGMA-git/ENIGMA_DTI_GWAS/blob/main/Pre%20Imputation%20Quality%20Control%20Protocol/mdsplot_ABCD_EUmix_QC2_outliersincluded.jpg?raw=true)

### Identify Ancestry Outliers

For predominantly European ancestry cohorts, we now exclude individuals below cut-off thresholds. These thresholds are not fixed but will need to be determined based on the visualization/plot of the two dimensions you just created.
Please have a good look at the two examples above - depending on your sample please change the cut-off threshold in the code below.

For predominantly other ancestry cohorts, the threshold should be determined according to the respective cohort cluster.

```r
# First we grab your specific sample and create a column where you can mark outliers
#--> Change “DATASETNAME” to your sample-specific cohort abbreviation
MDS_mySample <- mds.cluster[which(mds.cluster$POP %in% c("`DATASETNAME")),]
MDS_mySample$outlier  <- 0
# calculate outliers - first a threshold example for the g1000mix, a quite homogenous sample
# NOTE: change the cut-off below depending on YOUR sample 
MDS_mySample$outlier[MDS_mySample$C1 < -0.01 | MDS_mySample$C2 > -0.05] <- 1
```
> For the more diverse data set ABCD EU, as seen in the 2nd graph above, the threshold looks quite different, and will also result in a change of the '<' sign for the second PC! Please note, the line below does not have to be executed and is just included for illustrative purposes: `MDS_mySample$outlier[MDS_mySample$C1 < 0.04| MDS_mySample$C2 < 0.06] <- 1

```r
# sort sample specific outliers /Europeans, just grab the FID and IID column for later use
MDS_mySample_outliers<- MDS_mySample[which(MDS_mySample$outlier == 1),1:2]
MDS_mySample_european<- MDS_mySample[which(MDS_mySample$outlier == 0 ),1:2]
#For further plotting we also grab the full reference details again and combine them with the full EU information in your sample
#First let's get the reference - we get everything not part of your dataset
MDS_ref_eur <- mds.cluster[which "(!(mds.cluster$POP %in% "DATASETNAME")),]
#combine this with all Europeans (or respective other ancestry cohort)
MDS_mySample_eur_plus_ref <- rbind(MDS_ref_eur, mds.cluster[which (mds.cluster$IID %in% MDS_mySample_european$IID ),])
# repeat the colours again for the updated dataframe and then plot - note again: plot is directly saved
colors=rep("red",length(MDS_mySample_eur_plus_ref$C1));
colors[which(MDS_mySample_eur_plus_ref$POP == "CEU")] <- "lightblue";
colors[which(MDS_mySample_eur_plus_ref$POP == "CHB")] <- "brown";
colors[which(MDS_mySample_eur_plus_ref$POP == "YRI")] <- "yellow";
colors[which(MDS_mySample_eur_plus_ref$POP == "TSI")] <- "green";
colors[which(MDS_mySample_eur_plus_ref$POP == "JPT")] <- "purple";
colors[which(MDS_mySample_eur_plus_ref$POP == "CHD")] <- "orange";
colors[which(MDS_mySample_eur_plus_ref$POP == "MEX")] <- "grey50";
colors[which(MDS_mySample_eur_plus_ref$POP == "GIH")] <- "black";
colors[which(MDS_mySample_eur_plus_ref$POP == "ASW")] <- "darkolivegreen";
colors[which(MDS_mySample_eur_plus_ref$POP == "LWK")] <- "magenta";
colors[which(MDS_mySample_eur_plus_ref$POP == "MKK")] <- "darkblue";

pdf(file="mdsplot_DATASETNAME_QC2_outliersExcluded.pdf", width=7, height=7)
plot(rev(MDS_mySample_eur_plus_ref$C2), rev(MDS_mySample_eur_plus_ref$C1), col=rev(colors), ylab="Dimension 1", xlab="Dimension 2", pch=20)
legend("bottomleft", c("DATASETNAME", "CEU", "CHB", "YRI", "TSI", "JPT", "CHD", "MEX", "GIH", "ASW","LWK", "MKK"), fill=c("red", "lightblue", "brown", "yellow", "green", "purple", "orange", "grey50", "black", "darkolivegreen", "magenta",` `"darkblue"))
# save files - don't forget to replace DATASETNAME with your cohort-specific abbreviation
#let's get the outliers in your sample
write.table(MDS_mySample_outliers, "DATASETNAME_pop_strat_mds.outlier.txt", sep="\t", quote=FALSE, row.names=FALSE)
#let's get the europeans in your sample
write.table(MDS_mySample_european, "DATASETNAME_pop_strat_mds.eur.txt", sep="\t", quote=FALSE, row.names=FALSE)
# quit R
q()
```

Your output will now have outliers removed that are not part of the European (or respective ancestry of your cohort) cluster:
In case of the first included graph the output contains only the individuals from your sample within the dotted circle.

![MDS PLOT Outliers excluded](https://github.com/ENIGMA-git/ENIGMA_DTI_GWAS/blob/main/Pre%20Imputation%20Quality%20Control%20Protocol/mdsplot_g1000mix_QC2_outliersexcluded.jpg?raw=true)

Back in the shell, we exclude outliers from your actual bfile and use this QC3 file for subsequent steps:

```bash
plink --bfile ${datafile}_QC2_filtered  --keep ${datafile}_pop_strat_mds.eur.txt --make-bed --out ${datafile}_QC3
```

## Get genetic principal components in ancestry homogenous group

> **Note:** These newly generated genetic Principal Components will be used as covariates in the association protocol (after imputation). So for now, only save the following data (i.e. no need to further split it again, nor send them to us.)

```bash
plink --bfile ${datafile}_QC3 --pca --extract local.snplist.txt --out ${datafile}_PCACovariates
```

This extracts by default 20 PCs, which can be found in the **.eigenvec** file. More info on how many PCs to include as covariates will follow in the association protocol but to get a feeling you can create a scree plot and see how many PC's are needed to explain most of the variance.

This can be done in R, so let's start this by typing `R`

```r
library(data.table)
library(ggplot2)

#read in the eigenvalue file
eigv <- fread("${datafile}_PCACovariates.eigenval")
#calculate variance explained
var_explained <- (eigv/(sum(eigv))*100
#plot the scree plot
qplot(c(1:20), var_explained$V1) +
geom_line() +
xlab("Principal Component") +
ylab("Variance Explained") +
ggtitle("Scree Plot") +
ylim(0, 100)
```

---

## Cohort QC summary data

Using the **\${datafile}**, **\${datafile}\_QC2_filtered** and **\${datafile}\_QC3** file, respectively, please run the bash codes below to obtain an overview of the cohort stats pre- and post-QC:

### Pre-QC subject numbers

```bash
echo "COHORTNAME N_CONTROLS N_CASES N_CONTROLS_M N_CONTROLS_F N_CASES_M N_CASES_F PROP_CONTROLS_F PROP_CONTROLS_M PROP_CASES_F PROP_CASES_M" > ${datafile}_basic_stats_preQC.txt
echo -n "${datafile} " >> ${datafile}_basic_stats_preQC.txt
#How many controls/unaffected are in the cohort?
N_CONTROLS=`awk '{if ($6==1) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CONTROLS " >> ${datafile}_basic_stats_preQC.txt
#How many patients are in the cohort?
N_CASES=`awk '{if ($6==2) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CASES " >> ${datafile}_basic_stats_preQC.txt
#What is the absolute number of males among controls/unaffected?
N_CONTROLS_M=`awk '{if ($6==1 && $5==1) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CONTROLS_M " >> ${datafile}_basic_stats_preQC.txt
#What is the absolute number of females among controls/unaffected?
N_CONTROLS_F=`awk '{if ($6==1 && $5==2) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CONTROLS_F " >> ${datafile}_basic_stats_preQC.txt
#What is the absolute number of males among patients?
N_CASES_M=`awk '{if ($6==2 && $5==1) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CASES_M " >> ${datafile}_basic_stats_preQC.txt
#What is the absolute number of females among patients?
N_CASES_F=`awk '{if ($6==2 && $5==2) print $0}' ${datafile}.fam | wc -l`
echo -n "$N_CASES_F " >> ${datafile}_basic_stats_preQC.txt
#What is the proportion of females among controls/unaffected?
PROP_CONTROLS_F=`echo "$N_CONTROLS_F/$N_CONTROLS" | bc -l`
#What is the proportion of males among controls/unaffected?
PROP_CONTROLS_M=`echo "$N_CONTROLS_M/$N_CONTROLS" | bc -l`
#What is the proportion of females among patients?
PROP_CASES_F=`if (($N_CASES == 0)); then echo 0; else echo "$N_CASES_F/($N_CASES_F+$N_CASES_M)" | bc -l; fi`
#What is the proportion of males among patients?
PROP_CASES_M=`if (($N_CASES == 0)); then echo 0; else echo "$N_CASES_M/($N_CASES_F+$N_CASES_M)" | bc -l; fi`
echo -n "$PROP_CONTROLS_F $PROP_CONTROLS_M $PROP_CASES_F $PROP_CASES_M " >> ${datafile}_basic_stats_preQC.txt
```

### SNP level QC

```bash
echo "COHORTNAME N_SNPs_preQC N_samples_preQC SNPs_removed_>missingnessT Samples_removed_>missingnessT SNPs_removed_<MAFt SNPs_removed_HWE Samples_removed_MDS_outliers N_SNPs_postQC N_samples_postQC" > ${datafile}_qc_summary.txt
echo -n "${datafile} " >> ${datafile}_qc_summary.txt
NLINES=`wc -l ${datafile}.bim | awk '{print $1}'`
echo -n $NLINES" " >> ${datafile}_qc_summary.txt
NLINES=`wc -l ${datafile}.fam | awk '{print $1}'`
echo -n $NLINES" " >> ${datafile}_qc_summary.txt
qc_VAR=$(grep "variants removed due to missing genotype data (--geno)" ${datafile}_QC2_filtered.log | awk '{printf $1}')
if [ -z "$qc_VAR" ];then echo -n "NaN"" " >> ${datafile}_qc_summary.txt; else echo -n "$qc_VAR"" " >> ${datafile}_qc_summary.txt;fi
qc_VAR=$(grep "people removed due to missing genotype data (--mind)" ${datafile}_QC2_filtered.log | awk '{printf $1}')
if [ -z "$qc_VAR" ];then echo -n "NaN"" " >> ${datafile}_qc_summary.txt; else echo -n "$qc_VAR"" " >> ${datafile}_qc_summary.txt;fi
qc_VAR=$(grep "variants removed due to minor allele threshold(s)" ${datafile}_QC2_filtered.log | awk '{printf $1}')
if [ -z "$qc_VAR" ];then echo -n "NaN"" " >> ${datafile}_qc_summary.txt; else echo -n "$qc_VAR"" " >> ${datafile}_qc_summary.txt;fi
qc_VAR=$(grep "variants removed due to Hardy-Weinberg exact test" ${datafile}_QC2_filtered.log | awk '{printf $2}')
if [ -z "$qc_VAR" ];then echo -n "NaN"" " >> ${datafile}_qc_summary.txt; else echo -n "$qc_VAR"" " >> ${datafile}_qc_summary.txt;fi
NLINES2=`wc -l ${datafile}_QC2.bim | awk '{print $1}'`
NLINES3=`wc -l ${datafile}_QC3.bim | awk '{print $1}'`
N_excluded_MDS_outliers=`echo "$NLINES2-$NLINES3" | bc -l`
echo -n $N_excluded_MDS_outliers" " >> ${datafile}_qc_summary.txt
NLINES=`wc -l ${datafile}_QC3.bim | awk '{print $1}'`
echo -n $NLINES" " >> ${datafile}_qc_summary.txt
NLINES=`wc -l ${datafile}_QC3.fam | awk '{print $1}'`
echo -n $NLINES" " >> ${datafile}_qc_summary.txt
```

### Post-QC subject numbers

```bash
echo "COHORTNAME N_CONTROLS N_CASES N_CONTROLS_M N_CONTROLS_F N_CASES_M N_CASES_F PROP_CONTROLS_F PROP_CONTROLS_M PROP_CASES_F PROP_CASES_M" > ${datafile}_basic_stats_postQC.txt
echo -n "${datafile} " >> ${datafile}_basic_stats_postQC.txt
#How many controls/unaffected are in the cohort?
N_CONTROLS=`awk '{if ($6==1) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CONTROLS " >> ${datafile}_basic_stats_postQC.txt
#How many patients are in the cohort?
N_CASES=`awk '{if ($6==2) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CASES " >> ${datafile}_basic_stats_postQC.txt
#What is the absolute number of males among controls/unaffected?
N_CONTROLS_M=`awk '{if ($6==1 && $5==1) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CONTROLS_M " >> ${datafile}_basic_stats_postQC.txt
#What is the absolute number of females among controls/unaffected?
N_CONTROLS_F=`awk '{if ($6==1 && $5==2) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CONTROLS_F " >> ${datafile}_basic_stats_postQC.txt
#What is the absolute number of males among patients?
N_CASES_M=`awk '{if ($6==2 && $5==1) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CASES_M " >> ${datafile}_basic_stats_postQC.txt
#What is the absolute number of females among patients?
N_CASES_F=`awk '{if ($6==2 && $5==2) print $0}' ${datafile}_QC3.fam | wc -l`
echo -n "$N_CASES_F " >> ${datafile}_basic_stats_postQC.txt
#What is the proportion of females among controls/unaffected?
PROP_CONTROLS_F=`echo "$N_CONTROLS_F/$N_CONTROLS" | bc -l`
#What is the proportion of males among controls/unaffected?
PROP_CONTROLS_M=`echo "$N_CONTROLS_M/$N_CONTROLS" | bc -l`
#What is the proportion of females among patients?
PROP_CASES_F=`if (($N_CASES == 0)); then echo 0; else echo "$N_CASES_F/($N_CASES_F+$N_CASES_M)" | bc -l; fi`
#What is the proportion of males among patients?
PROP_CASES_M=`if (($N_CASES == 0)); then echo 0; else echo "$N_CASES_M/($N_CASES_F+$N_CASES_M)" |  bc -l; fi`
echo -n "$PROP_CONTROLS_F $PROP_CONTROLS_M $PROP_CASES_F $PROP_CASES_M " >> ${datafile}_basic_stats_postQC.txt
```

---

# Information about data transfer

Please send the following information and files to The ENIGMA-DTI Genetics Support Team <enigma.DTIgenetics@gmail.com> for checking before running the imputation protocol and fill out the postQC form: <a href="https://docs.google.com/forms/d/1-UeFnUPRQuJXu6aiYkTHtbziubHeYjB3vBZ5VqN-WMc/edit?usp=sharing"> postQCSummaryData</a>.

We will get back to you shortly with any questions or to give the go-ahead for the imputation.

- All **Plink log** files generated during QC and MDS steps
- From duplicated/relatedness we need  created .txt-files:
  - `${datafile}_QC1pruned_duplicates_count.txt`
  - `${datafile}_QC1pruned_relatedness_count.txt`
- The `${datafile}_sexcheck_PROBLEM.txt` file
- Number of SNPs on the X chromosome: `snp_count_X.txt`
- From MDS part
  - `mdsplot_${datafile}_QC2_outliersincluded.pdf`
  - `mdsplot_${datafile}_QC2_outliersexcluded.pdf`
- From Cohort QC summary
  - `${datafile}_basic_stats_preQC.txt`
  - `${datafile}_qc_summary.txt`
  - `${datafile}_basic_stats_postQC.txt`

The following lines of bash code gather and compress the requested files into a single zip archive:

```bash
mkdir output_final
cp *.pdf *.log ${datafile}_QC1pruned_duplicates_count.txt ${datafile}_QC1pruned_relatedness_count.txt ${datafile}_sexcheck_PROBLEM.txt ${datafile}_basic_stats_postQC.txt ${datafile}_qc_summary.txt ${datafile}_basic_stats_preQC.txt snp_count_X.txt output_final/
```

The line below checks if the expected amount of files have been moved.
If you ran the whole pipeline there should be **27 files** - **18 PLINK log files** and the **9 files** specified above.

> **Note:** In case you skipped some steps, the number of log files will be less, depending on how many plink commands you skipped - change the number **(==27)** accordingly.
> As a rule of thumb the number won’t be lower than 20 - if you have less files you are missing crucial files or skipped necessary steps.

```bash
if ((`ls output_final | grep -v ^d | wc -l` == 27)); then echo 'Correct amount of files moved; ZIP folder now' ;
else echo 'File number not correct; Please check again';
fi
zip -r ${datafile}_ENIGMA-DTI_FilesToSend.zip output_final/
```

<center>Many thanks for participating</center>
<center>**The ENIGMA-DTI Genetics Support Team**</center>

---
