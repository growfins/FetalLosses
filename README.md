## Comparison of missing data imputation methods for evaluation reference values of morphometric parameters of human fetus in the perinatal period

Authors:
- [Natalya Sheiko](https://github.com/growfins)
- [Evgeniia Molokova](https://github.com/EvgeniiaMolokova)
- [Ekateriva Ivanova](https://github.com/caterinagf)
- [Anna Antonova](https://github.com/AnnaAntV)

### Introduction

Fetal loss and neonatal mortality is a serious and urgent problem contributing to epidemiology and demography all over the world. According to Sustainable Development Goals (SDGs) most countries of the world [has agreed](https://data.unicef.org/topic/child-survival/child-survival-sdgs/) to reduce neonatal mortality to at least as low as 12 deaths per 1,000 live births and under-5 mortality to at least as low as 25 deaths per 1,000 live births by 2030. 

To reach these purposes, it is essential to understand the causes of child death, including the reasons for fetal losses and its underlying factors.
The perinatal post-mortem examination provides necessary information about fetal maturation and development which is important to determine the actual cause of death. Elaboration of accurate morphometric parameters reference values is essential for comprehensive interpretation of post-mortem evaluation and subsequent clinical management of fetal loss. 

However, the analysis of lethal outcomes in perinatal period is complicated due to complexity and specificity of obtained data.
The following concerns were mentioned: *missing values, insufficiently standardized structure of information and absence of common criteria for assessing fetal growth between different specialists*. In literature references There are numerous studies presenting reference ranges for fetal organ weights and body measurements in literature references, though many of them have significant limitations, for example lack of relationship with gestational age.

### Aim, tasks and data
The **aim** of this project was to develop reference values of morphometric parameters obtained in perinatal post-mortem examinations in relation with gestational age after processing missing data by different imputation methods.

The following **tasks** were set in order to achive the goal:
1. Processing and filtration of the obtained data with grouping on gestational age; 
2. Working out missing values in numeric parameters by different imputation methods; 
3. Performing descriptive statistics of morphometric values; 
4. Comprehensive analysis between results of different methods of data imputation; 
5. Formulation of reference values.

In our project we were provided with a **dataset** based on the protocols of fetal and neonatal post-mortem studies from one medical center. This database comprises **10,639 cases with 21 clinical and morphometric parameters**, that were collected from 2005 to 2014.

### Workflow
The workflow of the project presented at the following scheme. Some part of scheme will be discussed below.

![Диаграмма без названия drawio (3)](https://user-images.githubusercontent.com/102663823/215112950-505c069f-69aa-4b8d-86e1-a97fa7c8deaf.svg)

#### Work with raw data
The raw data were organized in 10 Excel files. At this stage one large .csv file with more that 10000 rows was combined.

#### Data cleansing
The dataset was equally structured, and processed for possible omissions and typos. Binary features notation ambiguity were corrected, after which the numeric and categorical parameters were brought to the desired type and correct units of measurement. 

The key feature of data cleansing was the live birth feature. Data were collected basing on the mortality reports, so the live birth and time of living became separate columns of data. During this stage, values confused between live birth and time of living were joined to one age column, after which the column was recalculated to days. 

#### Filtration
A more detailed description of exclusion criteria are the following (order is important):
1. report number or gender is missing;
2. live birth more than 24h;
3. gestational age less 20w and more 42w;
4. the presence of maceration;
5. chosen congenital disorders;
6. multiple pregnancy;
7. fetal growth restrictions.

The filtered database consisted of 3,148 cases, which morphometric parameters were grouped according to the gestational age.

#### Missing values imputation

Four imputation methods were performed.
- In case of NA deletion the observation (the row of dataset) with at least one missed variable was eliminated. 
- The single-imputation with mean/median/mode was processed according to each gestational week. 
- For the K-nearest neighbors algorithm the optimal k-value was chosen 5. 
- The Multiple imputation with chained equations (MICE) was performed via standard algorithm with number of implementations = 5 . 

As a result there were generated 6 new datasets corresponding to each method, and their descriptive statistics were produced.

#### Imputation methods comparison

The following descriptive statistic were selected:
 - mean ± standard deviaton;
 - 5th, 25th, 50th, 75th, 95th percentile.

**The comparative analysis** between primary and new datasets (grouped according to the gestational age) was performed with the Friedman test as a non-parametric alternative to the one-way ANOVA with repeated measures (most of data did not have normal distribution). 

**(Graphical example of non-normally distributed data. Feature - Brain mass)**

![Rplot](https://user-images.githubusercontent.com/102663823/215095427-4193e47d-70e5-47a9-ab58-30a259d811b0.svg)

At the same time, we exclude the “deletion of NA” dataset from comparative analysis because of its unformativeness. Such data will evidently have strong differs from the others due to decreasing number of observations.

**Results were visualized** using [violin plots (ggpubr)](http://rpkgs.datanovia.com/ggpubr/reference/ggviolin.html). The “deletion of NA” dataset was also included to compare the differences visually.

**(Example of graphical visualisation)**

![Rplot2](https://user-images.githubusercontent.com/102663823/215253630-8dd4220d-caf2-46ab-9cff-482633484451.svg)

#### Reference values assessment

According to the results of the work, the statistical report was formed.

**(An extract from the statistical report on the comparative analysis result)**

<img width="1463" alt="image" src="https://user-images.githubusercontent.com/102663823/215242979-f0578757-7ee5-464f-bc51-fdb68be809ac.png">

### Results

The significant differences between primary dataset and its imputated variants in appliance with gestational age were identified for spleen at 22, 23, 24, 26, 27 weeks of gestation with p-values <0.0001, <0.0001, <0.0001, <0.0001, 0.0012, respectively.
However, after the pairwise post-hoc test for multiple comparisons of mean rank sums for unreplicated blocked data (Nemenyi-Test) no pairwise differences were found.

### Conclusion and further plans

We did not observe any significant pairwise differences in variables grouping by gestational age between the primary dataframe and its imputation variants. Thus, we suggest that the application of different imputation methods can be considered as an alternative to complete exclusion an object from the dataset.

In further work we are going to compare our resuls with data from various literary sources and built a ML-model for predicting fetus growth and/or vitality rate.

### Literature and references
1. R Core Team (2022). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria.  URL https://www.R-project.org/.
2. Wickham H, Averick M, Bryan J, Chang W, McGowan L, François R, et al. Welcome to the Tidyverse. J Open Source Softw. 2019 Nov 21;4(43):1686.
3. ggplot2 Based Publication Ready Plots • ggpubr [Internet]. [cited 2023 Jan 27]. Available from: https://rpkgs.datanovia.com/ggpubr/
4. Pipe-Friendly Framework for Basic Statistical Tests • rstatix [Internet]. [cited 2023 Jan 27]. Available from: https://rpkgs.datanovia.com/rstatix/
5. Resources | Welcome to amices [Internet]. [cited 2023 Jan 27]. Available from: https://amices.org/
6. Phillips JB, Billson VR, Forbes AB. Autopsy standards for fetal lengths and organ weights of an Australian perinatal population. Pathology. 2009;41(6):515-26. doi: 10.1080/00313020903041093. PMID: 19588281.


### Links

All statistical analysis was performed using [R Statistical Software (version 4.2.1)](https://www.r-project.org/) and following libraries:  

[dplyr](https://dplyr.tidyverse.org/) 

[tibble](https://tibble.tidyverse.org/)

[stringr](https://stringr.tidyverse.org/)

[readr](https://readr.tidyverse.org/)

[readxl](https://readxl.tidyverse.org/)

[data.table](https://cran.r-project.org/web/packages/data.table/index.html)

[flextable](https://ardata-fr.github.io/flextable-book/)

[mice](https://cran.r-project.org/web/packages/mice/mice.pdf)

[ggmice](https://amices.org/ggmice/articles/ggmice.html)

[ggplot2](https://ggplot2.tidyverse.org/)

[Rmisc](https://cran.r-project.org/web/packages/Rmisc/index.html)

[tidyr](https://tidyr.tidyverse.org/)

[plyr](https://cran.r-project.org/web/packages/plyr/index.html)

[ggpubr](https://cran.r-project.org/web/packages/ggpubr/index.html)

[rstatix](https://cran.r-project.org/web/packages/rstatix/index.html)

[tydyverse](https://www.tidyverse.org/)

[reshape2](https://cran.r-project.org/web/packages/reshape2/index.html)

[caret](https://topepo.github.io/caret/)

[RANN](https://cran.r-project.org/web/packages/RANN/index.html)

[openxlsx](https://cran.r-project.org/web/packages/openxlsx/index.html)

[stats](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/00Index.html)

[PMCMRplus](https://cran.r-project.org/web/packages/PMCMRplus/index.html)
