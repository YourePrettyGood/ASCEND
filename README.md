# ASCEND
***ASCEND*** (**A**llele **S**haring **C**orrelation for the **E**stimation of **N**onequilibrium **D**emography) is a method to estimate the age and intensity of founder events/bottlenecks using population genotype data and a recombination map.

:arrow_right: **Current version:** 10.0

# :v: Tutorial

A series of examples/tutorials is present in the `docs` directory:

- for inferring founder events for present-day samples of a diploid species, click [here](https://github.com/sunyatin/ASCEND/blob/master/docs/present_DNA.pdf).
- for inferring founder events for ancient samples of a diploid species, considering only the 5 five chromosomes, click [here](https://github.com/sunyatin/ASCEND/blob/master/docs/ancient_DNA.pdf).

# :point_down: Installation

ASCEND is a Python3 script and does not require prior installation apart specific modules (numpy, scipy) that you can install using the command:

`pip3 install --user numpy`

`pip3 install --user pandas`

`pip3 install --user matplotlib`

`pip3 install --user scipy`

# :heavy_exclamation_mark: Requirements

For optimal use, ASCEND requires that your population is comprised of at least 5 samples. ASCEND is based on EIGENSTRAT-formatted input files and **requires genetic positions**, provided in the 3rd column of the .snp file (genetic positions can be given either in Morgans or centiMorgans). The method currently does not support packed formats like packedancestrymap, packedped, packedeigenstrat, etc. Please use `convertf` to convert the data to eigenstrat format before use.

Since ASCEND relies on the recombination map, make sure your SNPs have the most accurate genetic positions (see https://github.com/sunyatin/itara/blob/master/liftover.py to lift positions over a recombination map).

# :dna: Input

ASCEND requires that the input data is in EIGENSTRAT format. See [here](https://reich.hms.harvard.edu/software/InputFileFormats) for more details. **Currently, ASCEND does not accept _binary_ or _packed_ EIGENSTRAT files** (e.g. packedeigenstrat or packedancestrymap) (but we plan to handle this format soon). *If you have binary EIGENSTRAT *.geno file, please convert them to non-binary format using CONVERTF.*

The EIGENSTRAT format is comprised of three files:

- `*.ind` A 3-column file with each individual on one row, and columns are (1) individual name, (2) individual sex (note that sex is not used for the ASCEND analysis), (3) population label
- `*.snp` A 6-column file with each SNP on one row, and columns are (1) SNP name, (2) chromosome, (3) **genetic position (in Morgans or centiMorgans)**, (4) physical position (in base pairs), 5th and 6th columns are the ancestral/derived or reference/alternate alleles but these 2 columns are not taken into account for the ASCEND analysis
- `*.geno` The genotype matrix with no delimiter between genotypes, each row is one SNP and each column is one individual, genotypes are encoded as 0 (= 1/1), 1 (=0/1) or 2 (=0/0). Missing genotypes are encoded as 9.

You can convert your file into EIGENSTRAT using the CONVERTF program (see https://github.com/argriffing/eigensoft/tree/master/CONVERTF).

Note that although the .geno file must **not** be binary, but it can be gzip-compressed.

# :computer: Command line

To run an ASCEND analysis:

`python3 ASCEND.py -p [NameOfTheParameterFile].par`

Note that by default, ASCEND assumes that the **genetic positions are in Morgans** and that the **samples are diploid**.

For reliable estimation, use a minimum of 5 diploids in the target and outgroup populations.

For best performance, we advise to use ~15 individuals for the outgroup population.

# :wrench: Full list of parameters

Note that you can comment any line and option using "#" (the software will ignore those lines). Also, the options can be written in any order.

**Mandatory options**

- `genotypename: [STRING]` name of the input .geno file
- `snpname: [STRING]` name of the input .snp file
- `indivname: [STRING]` name of the input .ind file
- `outputprefix: [STRING]` prefix of the output file, ASCEND will automatically append the appropriate extensions
- `targetpop: [STRING]` name of the target population to analyze

**Optional options** (if not provided, ASCEND will take the default values)

- `outpop: [STRING]` (**recommended**) add this option to correct the within-population allele sharing correlation by a cross-population correlation to remove background LD; you have two options: (1) write `outpop: RANDOM` and add the option `outpopsize: n` to pick `n` random individuals (random sampling without replacement of all individuals not annotated with population the label `Ignore`) from the dataset that are not in your target population (recommended) or (2) the name of the specific outgroup population to use. If this option is not provided, ASCEND will not compute the cross-population correlation and will instead output `nan` in the corresponding column. Note that `outpop: RANDOM` will generate a file with extension `RandomOutpop.ind` where the individuals picked up to form the outgroup population are annotated with the population label "OUTGROUP". Note that if `outpop: RANDOM`, individuals with the population label `Ignore` will not be considered for outgroup sampling.

- `outpopsize: [INTEGER]` if you provided the option `outpop: RANDOM` then add this option with the size of the outgroup population to create randomly (recommended: 15)

*Related to genetic data*

- `chrom: [comma-separated list of integers]` add this option with a comma-separated list of chromosomes on which to restrict the analysis (for instance, `chrom: 1, 2, 3` to restrict the analysis to chromosomes 1, 2 and 3). **Check if your dataset contains sexual chromosomes, in such case, we recommend to restrict the analyses only to the autosomes using this option**
- `haploid: YES/NO` ASCEND assumes genotypes are diploid but if you set this option to YES it will interpret your genotypes as haploid (default: NO)
- `dopseudohaploid: YES/NO` set YES if your genotypes have to be pseudohaploidized (i.e. for heterozygous genotypes, one allele will be randomly picked and set in two copies) (default: NO). Note that even if the genotypes are *already* provided as pseudohaploid in the input .geno file, you still must set `dopseudohaploid: YES` so that ASCEND computes the allele sharing in an unbiased way.

*Related to SNP filtering*

- `maxpropsharingmissing: 1.0` maximum proportion of missing allele sharing allowed (above this threshold the SNP will be discarded) (default: 1.0)
- `minmaf: 0` minimum Minor Allele Frequency that is allowed for a SNP to be taken into account, i.e. if a SNP has `MAF<minmaf` or `MAF=minmaf`, it will be excluded (default: 0.0)

*Related to the decay curve parameters*

- `mindis: 0.001` minimum genetic distance in Morgans (default: 0.001 M)
- `maxdis: 0.3` maximum genetic distance in Morgans (default: 0.3 M)
- `binsize: 0.001` size of the genetic distance bin in Morgans (default: 0.001 M)
- `morgans: YES/NO` set NO if your input genetic distances are in centiMorgans (by default ASCEND assumes Morgans) (default: YES)

*Related to the algorithm*

- `usefft: YES/NO` whether to use the Mesh + Fast Fourier Transforms (FFT) algorithm which speeds up the calculation by up to 8,000 times with only marginal approximations. In general, we advise to use the FFT instead of the Naïve implementation for reasons of speed and accuracy, especially in case of high rates of missing genotypes (FFT handles missingness better than the Naïve implementation).
- `qbins: 100` number of mesh points within each bins of the decay curve to consider for the mesh-FFT approximation (a higher number increases the mesh resolution and hence the accuracy of the decay curve, but also slows down the computation - we found that 100 was a good compromise between speed and accuracy) (default: 100)
- `randomhet: YES/NO` by default, when two individuals are heterozygous at a site, we assume that they share only 1 allele (`randomhet: NO`) which is our way to handle phasing uncertainty; however if you set this option as `YES` then the number of alleles shared will be picked up randomly as either 0 (the reference allele is on different chromosomes between the two individuals) or 2 (the reference allele is on the same chromosome between the two individuals).
- `calculation_mode: auto/correlation/weighted_covariance` by default, ASCEND will automatically detect the format of your genotypes based on the `haploid` and `dopseudodiploid` options you provided, and run the (i) **allele sharing correlation** function for **diploid** or **haploid** data or (ii) run the **allele sharing weighted covariance** function for **pseudohaploid** data. This is to ensure that no bias is introduced when estimating the amplitude of the decay curve. Although we do not advise to do so, you can still force the use of a specific function using the values `correlation` or `weighted_covariance`, respectively. An error message will pop up if the nature of the input genotypes do not match the function you provided.

*Related to the fitting*

- `onlyfit: YES/NO` set YES if you want to do the estimation of the parameters directly, using the `.out` and `.perchr.out` files that have been already output by the script (using `onlyfit: YES` can be dangerous, if you have any doubt, we would advice to rerun the all analysis) (default: NO)
- `blocksizename: [file path]` add this option to indicate the name of a file containing the per-chromosome weights to use for the weighted jackknife analysis; the file must have two tab-separated columns: (i) the chromosome label (should be the same as in the .snp file) and (ii) the number of SNPs on the chromosome or the chromosome length in bp; if this option is not provided, ASCEND will automatically calculate the weight of each chromosome as the number of SNPs in the input .snp file.

*Misc*

- `seed: None` seed for the random number generator, if None, will generate a random seed (default: None)

# :tada: Output

Each call to ASCEND outputs a set of 9 files (or only 7 if the `blocksizename` option is provided) that we describe hereunder:

### `.log`

The log file.

### `.png`

ThA plot of the allele sharing correlation decay curve (blue points) along with the fitted exponential model (red line). In the top-right corner: the estimates of founder age (Tf) and intensity (If) with their associated 95% confidence intervals within brackets as well as the NRMSD.

### `.est`

A table containing, on each line: the estimate of the founder age (Tf, in generations before sampling) with standard error (SE), lower and upper boundaries of the confidence interval at 95%; intensity (If) with its SE and CI95 boundaries; the NRMSD.

### `.qweights`

If the `blocksizename` option was not provided, a file giving the weights for each chromosome (= number of SNPs).

### `.out`

The `.out` file contains the average allele sharing correlation averaged over all chromosomes:

- `bin.left.bound` the left boundary of the genetic distance bins (in cM)
- `mean.cor.pop` the within-population average allele sharing correlation values
- `mean.cor.bg` the cross-population average allele sharing correlation values
- `mean.cor.substracted` the within-population average allele sharing correlation subtracted by the cross-population
- `n.pairs` the number of well-defined SNP pairs * individual pairs used for the calculation of the statistics

Note that in case where you do not provide an outgroup population, the `cor.bg` and `cor.substracted` will have `nan` values.

### `.perchrom.outs`

The `.perchrom.out` file contains the allele sharing correlation for each chromosome:

- `chrom` the chromosome number
- `bin.left.bound` the left boundary of the genetic distance bins in cM
- `sum.cor.pop` the sum of within-population allele sharing correlation values over `n.pairs`
- `sum.cor.bg` the sum of cross-population allele sharing correlation values over `n.pairs`
- `sum.cor.substracted` the sum of within-population allele sharing correlation subtracted by the cross-population over `n.pairs`
- `n.pairs` the number of well-defined SNP pairs * individual pairs used for the calculation of the statistics

### `.perjk.outs`

The `.perjk.outs` file contains the average allele sharing correlation values calculated for each jackknife run:

- `run` the jackknife run
- `bin.left.bound` the left boundary of the genetic distance bins in cM
- `mean.cor.pop` the within-population average allele sharing correlation values
- `mean.cor.bg` the cross-population average allele sharing correlation values
- `mean.cor.substracted` the within-population average allele sharing correlation subtracted by the cross-population
- `n.pairs` the number of well-defined SNP pairs * individual pairs used for the calculation of the statistics

### `.fit`

The `.fit` file provides the estimates of the exponential model (that you can then use to estimate the founder age and the founder intensity) with their associated standard errors. To compute standard errors, the script performs a weighted block jackknife where blocks are the chromosomes and weights are their sizes or their number of SNPs. We fit an exponential function of the form `z(d) = A exp(-2dt) + c` where `z(d)` is the average allele sharing correlation at the genetic distance bin `d`, `A` is the amplitude and `t` is the rate of exponential decay.

The `.fit` file has 4 columns:
- `param` the name of the parameter
- `mean` the point estimate of the parameters using the decay curve averaged over all chromosomes
- `jk.mean` the point estimate of the parameters based on the jackknife procedure
- `jk.se` the standard error of the parameter estimates based on the jackknife procedure

There is a line for each parameter (A, t, c) + a line for the Normalized Root Mean Squared Deviation (NRMSD) calculated between the empirical decay curve and the theoretical decay curve.

### `.perjk.fits`

The file contains the estimated exponential parameters for each jackknife run:

- `run` the jackknife run
- `A` the amplitude of the exponential function
- `t` the rate of the exponential decay
- `c` the affine term
- `blockweights` the weight of the chromosome that was removed from the jackknife run

## `.RandomOutpop.ind`

If you used the option `outpop: RANDOM`, ASCEND will generate a file with the extension `.RandomOutpop.ind`. This file is a copy of your input `.ind` file but  the individuals picked up at random to create the outgroup population are annotated as "OUTGROUP". All the other individuals not in the target population nor in the outgroup population are set with population label "Ignore".

# :bulb: Full example

An example run is provided in the repository `example`. You can re-run it using the command:

`python3 ASCEND.py -p founder_event_50gBP_intensity10percent.par`

The example provided is a simulation with 3 chromosomes of a founder event occurring 50 generations ago with intensity 10% (20% of the genotypes were also replaced with missing genotypes) so your estimates of Tf and If in the output plot should overlap with these numbers.

# :skull: Typical parameter file for an aDNA analysis

If you want to analyze ancient DNA, we advise not subtracting background LD (by masking `outpop:` with a "#" symbol) and pseudohaploidizing the genotypes (`dopseudohaploid: YES`). A typical parameter file would therefore look like:

```
genotypename: FILE.geno
snpname: FILE.snp
indivname: FILE.ind
targetpop: TARGET_POPULATION
#outpop:
outputprefix: results/OUTPUT
binsize: 0.001
mindis: 0.001
maxdis: 0.3
maxpropsharingmissing: 1
minmaf: 0
chrom: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22
haploid: NO
dopseudohaploid: YES
morgans: CHECK_THIS_ON_YOUR_INPUT_FILE
onlyfit: NO
usefft: YES
qbins: 100
```

# :bug: Troubleshooting

### UnicodeDecodeError
If your input are in PACKED EIGENSTRAT format (i.e. the .geno file is binary), ASCEND will throw the error
`UnicodeDecodeError: 'utf-8' codec can't decode byte 0x86 in position 1936: invalid start byte`

To solve this problem, you will have to convert your input dataset to EIGENSTRAT using `convertf`, cf. https://github.com/DReichLab/AdmixTools/tree/master/convertf

# :copyright: License

This software is licensed for academic and non-profit use only.

# :space_invader: Support
Send queries to Rémi Tournebize (remi dot tournebize at gmail dot com) or Priya Moorjani (moorjani at berkeley dot edu).
