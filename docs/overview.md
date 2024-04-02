# Ensemblux framework overview
The Ensemblux workflow begins by demultiplexing pooled cells with each of its constituent tools

**Figure**
Figure 1. 

Upon demultiplexing pools with each individual constituent genetic demultiplexing tool, Ensemblux processes the outputs in a three-step pipeline:

- [Step 1: Accuracy-weighted probabilistic-weighted ensemble](#step-1-accuracy-weighted-probabilistic-ensemble)
- [Step 2: Graph-based doublet detection](#step-2-graph-based-doublet-detection)
- [Step 3: Ensemble-independent doublet detection](#step-3-ensemble-independent-doublet-detection)

**figure**
Figure 2. 

For demonstration purposes throughout this section, we leveraged simulated pools with known ground-truth sample labels that were generated with 80 independetly-sequenced induced pluripotent stem cell (iPSC) lines from individuals with Parkinson's disease and neurologically healthy controls that were differentiated towards a dopaminergic cell fate as part of the Foundational Data initiative for Parkinson's disease (FOUNDIN-PD; **CITE**)

 - - - -

### Step 1: Accuracy-weighted probabilistic ensemble
The accuracy-weighted probabilistic ensemble component of the Ensemblux framework utilizes an unsupervised weighting model to identify the most probable sample label for each cell. Ensemblux weighs each constituent tool’s assignment probability distribution by its estimated balanced accuracy for the dataset in a framework that was largely inspired by the work of Large et al. (**CITE**). The weighted assignment probabilities across all four constituent tools are then used to inform the most probable sample label for each cell.

**FIGURE**

To estimate the balanced accuracy of a particular constituent tool (e.g. Demuxalot) for real-word datasets lacking ground-truth labels, Ensemblux leverages the cells with a consensus assignment across the three remaining tools (e.g. Demuxlet, Souporcell, and Vireo-GT) as a proxy for ground-truth. To validate this approach for estimating the balanced accuracy of the constituent demultiplexing tools for the particular datset, we computed the Adjusted Rand Index between the sample labels after weighing the assignment probabilities by the estimated balanced accuracy and the true balanced accuracy across simulated pools ranging in size from four to 80 samples. 

**Figure**
 
 - - - -

### Step 2: Graph-based doublet detection
The graph-based doublet detection component of the Ensemblux framework was implemented to identify doublets that are incorrectly labeled as singlets by the accuracy-weighted probablistic ensemble component (Step 1). To demonstrate Step 2 of the Ensemblux framework we leveraged a simulated pool comprising 24 pooled samples, 17,384 cells, and a 15% doublet rate.

**FIGURE** diagrame

The graph-based doublet detection component beging by leveraging select variables returned from each constituent tool:

1. Demuxalot: doublet probability;
2. Demuxlet/Freemuxlet: singlet log likelihood – doublet log likelihood;
3. Demuxlet/Freemuxlet: number of single nucleotide polymorphisms (SNP) per cell;
4. Demuxlet/Freemuxlet: number of reads per cell;
5. Souporcell: doublet log probability;
6. Vireo: doublet probability;
7. Vireo: doublet log likelihood ratio 

**Figure**

Using these variables, Ensemblux then screens each pooled cell to identify the *n* most confident doublets (nCD) in the pool and performs a principal component analysis (PCA).

**FIGURE: Ground truth PCA, confident doublet PCA**

The PCA embedding is then converted into a Euclidean distance matrix and each cell is assigned a percentile rank based on their distance to each confident doublet. Ensemblux identifies the cells that exceed the designated percentile threshold (pT) for each confident doublet and computes the number of times each cell appears amongst the nearest neighbours (fNN) of a confident doublet; an fNN equal to nCD indicates that a cell was amongst the top nearest neighbours for each confident doublet. Ensemblux then plots the distribution of fNN values as a density plot. 

**fnn distribution plot**

 For real-world pools without ground-truth sample labels, Ensemblux performs an automated parameter sweep at varying combinations of nCD and pT values to optimize the graph-based doublet detection parameters. By default, nCD values range from 50 to 300, in increments of 50, while pT values depend on the expected doublet rate (exDR) and range from 1-  (exDR/6) to 1-exDR, in intervals of  (1-exDR)/6. For instance, if the expected doublet rate is 15%, the pT values tested in the parameter sweep will be 0.975, 0.950, 0.925, 0.900, 0.875, and 0.850, by default. 

 The density of fNN values for each combination of nCD and pT parameters are plotted and Pearson’s measure of kurtosis (k), or the tailedness of the distribution, is used to predict which combination of pT and nCD values optimizes the identification of true doublets while minimizing the rate of incorrectly labelled true singlets as doublets. Specifically, we screen for combinations of nCD and pT values that result in fNN distributions with high k as they signify that outlier cells — those with the highest fNN values — were identified as the nearest neighbours of many confident doublets. Thus, Ensemblux first identifies the pT that returns the highest k, on average, across nCD values.

 **FIGURE**

 Upon identifying the optimal pT value, Ensemblux plots the k corresponding to optimal pT across all nCD values tested in the parameter sweep. Importantly, the shape of the curve produced by plotting k across nCD values is dependent on the degree of separation between doublets and singlets in PCA space.  If singlets and doublets segregate strongly in PCA space, k will show a linear curve across nCD values; in this case, the choice of optimal nCD is inconsequential. In contrast, k will vary according to nCD for most pools due to singlets and doublets showing moderate segregation in PCA space; in this case, Ensemblux employs a two-step model for identifying optimal nCD. First, Ensemblux determines whether a point of inflection is identifiable due to a concave curve; in this case, Ensemblux identifies optimal nCD as the nCD value corresponding to the point of inflection on the curve, which, using pools with known ground-truth labels, we observed to represent the point of a negative return on doublet detection as an exceedingly high rate of true singlets are falsely identified as doublets as k decreases. Second, if an inflection point is still unidentifiable, Ensemblux identifies optimal nCD as the nCD value corresponding to the highest k. 

 **Figure**

 The cells that are identified as doublets using optimal pT and optimal nCD are then labelled as doublets by Ensemblux.

 **PCA FIGURES**

 - - - -

### Step 3: Ensemble-independent doublet detection
The ensemble-independent doublet detection component of the Ensemblux framework was implemented to further improve Ensemblux's ability to identify doublets. Benchmarking on simulated pools with known ground-truth sample labels revealed that certain genetic demultiplexing tools, namely Demuxalot and Vireo, showed high doublet detection specificity.

 **FIGURE**

 However, Steps 1 and 2 of the Ensemblux workflow failed to correctly label a subset of doublet calls by these tools. To mitigate this issue and maximize the rate of doublet identification, Ensemblux labels the cells that are identified as doublets by Vireo or Demuxalot as doublets, by default; however, users can nominate different tools for the ensemble-independent doublet detection component depending on the desired doublet detection stringency. 

 **FIGURE**
 - - - -




