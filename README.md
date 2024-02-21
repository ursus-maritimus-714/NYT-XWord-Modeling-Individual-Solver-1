# Predictive Modeling of Individual Solver 1 (IS1) Performance on the New York Times Crossword Puzzle

## Introduction

### Project Overview and Data Sources
This summary reports on the results of predictive modeling of my own (Individual Solver 1; IS1) performance on all puzzles issued during a 3+ year (Oct. 2020 - Feb. 2024) sample of the [New York Times (NYT) crossword puzzle](https://www.nytimes.com/crosswords). Previously, I conducted a [comprehensive exploratory data analysis (EDA) of IS1 performance](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Individual-Solver-1/blob/main/README.md) over this sample period. From this EDA, numerous features pertaining to both the puzzles themselves as well as to IS1 past performance were identified as candidate features for predictive modeling of IS1 solve times.    

Without access to two specific data sources this project would not have been possible. The first, [XWord Info: New York Times Crossword Answers and Insights](https://www.xwordinfo.com/), was my source for data on the puzzles themselves. This included a number of proprietary metrics pertaining to the grids, answers, clues and constructors. XWord Info has a contract with NYT for access to the raw data underlying these metrics, but I unfortunately do not. Therefore, I will not be able to share raw or processed data that I've acquired from their site. Nonetheless, [Jupyter notebooks](https://jupyter.org/) with all of my Python code for analysis and figure generation can be found [here](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/tree/main/notebooks). The second, [XWStats](xwstats.com), was my source for historical solve time data for both IS1 and for the "Global Median Solver" (GMS). XWStats (Matt) derives the GMS solve time as that at the 50th percentile out of ~1-2K individual solvers providing their solve times per puzzle. In the context of IS1 modeling, historical GMS solve times were used to derive a "Strength of Schedule" adjustment to features capturing IS1 past performance prior to a given solve time to be predicted (see Methods section for details). Additionally, please also see the [EDA](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Global-Median-Solver?tab=readme-ov-file#readme) and [predictive modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Global-Median-Solver/blob/main/README.md) of GMS solve time that I have previously completed. 

Please visit, explore and strongly consider financially supporting both of these wonderful sites; XWord Info via [membership purchase at one of several levels](https://www.xwordinfo.com/Pay) and XWStats via [BuyMeACoffee](https://www.buymeacoffee.com/xwstats). 

### Overview of NYT Crossword and IS1 Characteristics
The NYT crossword has been published since 1942, and many consider the "modern era" to have started with the arrival of Will Shortz as (only) its 4th editor 30 years ago. A new puzzle for each day is published online at either 6 PM (Sunday and Monday puzzles) or 10 PM (Tuesday-Saturday puzzles) ET the prior evening. Difficulty for the 15x15 grids (Monday-Saturday) is intended to increase gradually across the week, with Thursday generally including a gimmick or trick of some sort (e.g., "rebuses" where the solver must enter more than one character into one or more squares). Additionally, nearly all Sunday through Thursday puzzles have themes, some of which are revealed via letters placed in circled or shaded squares. Friday and Saturday are almost always themeless puzzles, and tend to have considerably more open constructions and longer (often multiword) answers than the early week puzzles. The clue sets tend to be more wordplay heavy/punny as the week goes on, and the answers become less common in the aggregate as well. Sunday puzzles have larger grids (21x21), and almost always feature a wordplay-intensive theme to which the longest answers in the puzzle pertain. The intended difficulty of the Sunday puzzle is approximately somewhere between a tough Wednesday and an easy Thursday. 

**Figure 1** shows dimensionality reduction via Principal Component Analysis (PCA) of 23 grid, clue and answer-related features obtained from XWord Info. This analysis demonstrates that, while puzzles from a given puzzle day do indeed aggregate with each other in n-dimensional "puzzle property space", the puzzle days themselves nonetheless exist along a continuum. Sunday is well-separated from the other puzzle days in this analysis by PCA1, which undoubtedly incorporates one or more grid size-contingent features.   

**<h4>Figure 1. PCA of Select Puzzle Grid, Clue and Answer Features**                                                                  

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/5fa4537f-1f50-4383-ac3a-833620347070)
*<h5>The first 3 principal components accounted for 47.6% of total variance. All puzzles issued from Jan. 1, 2018- Feb. 19, 2024 were included in this analysis (N=2,241).*
###

The overlapping distributions of per puzzle day IS1 solve times across the entire sample period (**Figure 2**) show a parallel performance phenomenon to the continuum of puzzle properties seen in **Fig. 1**. While solve difficulty increased as the week progressed, puzzle days of adjacent difficulty still had substantially overlapping IS1 solve time distributions. Other than for the "easy" days (Monday and Tuesday), distributions of IS1 solve times were quite broad. It should be noted, however, that the broadness of each puzzle day-specific IS1 solve time distribution was also increased by the fairly dramatic improvement in IS1 performance over the full sample period. The temporal dynamics of this improvement will be highly evident in the next section's figures.     

**<h4>Figure 2. Distributions of IS1 Solve Times by Puzzle Day for Full Sample Period**                   

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/120f724e-3782-47f2-948b-cbaa2f17911e)
*<h5>All puzzles completed by IS1 in the sample period were included in this analysis (N=1,197).* 

### Key Outcomes from the IS1 EDA

One of the most important findings from the EDA as far as implications for predictive modeling was that, despite several periods of volatility across puzzle days (e.g., April-May 2023), IS1 demonstrated continual, marked improvement over the course of the sample period across all puzzle days (**Figure 3**). Coupled to the fact that puzzle day-specific recent past performance was highly positively correlated to performance on the next puzzle both overall (r=.74) and across puzzle days (**Figure 4**), this created an imperative to explore and potentially include different variants of this feature type in the predictive modeling stage.    

**<h4>Figure 3. IS1 Solve Time Overview by Puzzle Day: 10-Puzzle Moving Averages and Distributions of Raw Values**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/3366123c-350a-42f8-908d-9a2a79debd97)
*<h5>All puzzles completed by IS1 in the sample period were included in this analysis (N=1,197).* 

**<h4>Figure 4. Puzzle Day-Specific, Recent Performance (RPB) Correlation to IS1 Performance on the Next Puzzle**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/1a94caba-53dc-4567-9d1c-ab646bbe00ac)
*<h5> Puzzle-day specific, recent past performance (x-axis) was calculated over the 8 day-specific puzzles previous to the next solve (y-axis). All puzzles completed by IS1 from Jan. 1, 2022- Feb. 19, 2024 were included in this analysis (N=967).* 

###
Along with the recent performance baseline (RPB) discussed above, multiple features pertaining to the puzzles themselves demonstrated moderately strong or strong correlations with IS1 performance on individual puzzles (**Figure 5**). Two that stood out in particular for their correlational strength with IS1 performance were 'Average Answer Length' and 'Freshness Factor', the latter of which is a proprietary XWord Info measure of the rareness of a given answer in the NYT puzzle. The strengths of these positive correlations with IS1 solve times (for all 15x15 puzzles: r=.56 for both features) can be seen both in the correlation heatmap (A; top row - 5th and 11th columns) as well as in the overall (black) and per-puzzle day (colors) feature correlation scatterplots in panel B. The feature density plots (C) show that the distributions of these features were well-separated across puzzle days. This is an important property for candidate predictive features to have since, as is shown in **Fig. 2**, distribution peaks of solve times for individual puzzle days were themselves well-separated.   

**<h4>Figure 5. Correlations of Puzzle-Related and Past Performance Features to IS1 Solve Times**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/1aeb9bf0-1ce4-454f-83e4-2626185080ce)
*<h5> All puzzles completed by IS1 from Jan. 1, 2022- Feb. 19, 2024 were included in this analysis (N=967).*

## Methods
### Predictive Feature Generation

For predictive feature generation, all puzzles completed by IS1 from Aug. 8, 2021-Feb. 17, 2024 (N=1,196) were included. This full sample included puzzles issued by NYT as early as August, 2021. The right panel of **Figure 6** summarizes predictive features included in the modeling stage (N=40) by broad class. A few key example features from each class are mentioned below. **Supplementary Table 1** comprehensively lists out, classifies and describes all included features.  

* 'Solver Past Performance Features' (n=6) included 'IS_RPB_l8'. This feature captured puzzle day-specific recent performance baseline (RPB) over the 8 puzzles immediately prior to a puzzle with a time being predicted. A number of temporal integration windows and time-decay weighting curves for this feature were tested in [preliminary univariate linear regression modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/tree/main/notebooks/learning%20rate%20modeling), and an 8 puzzle window (l8) with *no* time-decay weighting yielded the lowest root mean square error (RMSE) mean training error. Furthermore, it was found that predictions with these parameters further improved with adjustment of this past performance feature by the performance of the GMS over the same set of puzzles relative to the *GMS' own* recent performance prior to those puzzles. This is referred to as 'Strength of Schedule' (SOS) adjustment from here forward, and the left panel of **Fig. 6** depicts creation of this feature. Also note that another feature in this class, which captured normalized past performance against the constructor(s) of a puzzle being predicted ('IS Past Perf vs Constr'), used this SOS-adjustment in calculation of the baseline solve time expectation component (see **Supp. Table 1**).   
 
* 'Puzzle: Clue or Answer Features' (n=19) included 'Freshness Factor', which measured the aggregate rarity of answers in a given puzzle across all NYT crossword puzzles from before or since the issue date. This class also included other measures of answer rarity, including the number of entirely unique answers in a puzzle ('Unique Answer #') and the 'Scrabble Score', which assigns corresponding Scrabble tile values to each letter in an answer (rarer letters = higher tile values, hence a different angle at assessing answer rarity). On the clue side of the ledger, this class also included a count of the frequency of wordplay in clues for a given puzzle ('Wordplay #'). Later week puzzles contained more such clues, and early week puzzles often contained very, very few. 

* 'Puzzle: Grid Features' (n=11) included both the number of answers ('Answer #') and 'Average Answer Length' in a given puzzle. As puzzles got more difficult across the week, the former tended to decrease and the latter tended to increase. This class also included 'Open Squares #', which is a proprietary measure of XWord info capturing white squares not bordered by black squares (tended to increase as puzzles increased in difficulty across the week). This category also included features capturing other design principles of puzzles, including 'Unusual Symmetry'. This feature captured puzzles deviating from standard rotational symmetry (e.g., those with left-right mirror or diagonal symmetry), that could have had implications for their difficulty.

* 'Circadian Features' (n=3) included 'Solve Day Phase', which broke puzzles completion timestamps (obtained per solve via XW Stats) into four 6-hour time slots. Per puzzle being predicted, 'IS_per_sdp_avg_past_diff_from_RPB' was a feature that measured how recent puzzle day-specific performance in the pertinent ‘Solve Day Phase’ compared to RPB across all solve phases. Calculation of this feature used SOS-adjustment (see 'Solver Past Performance Features') in deriving RPB.

* 'Puzzle Day' (n=1) was a class of one, simply assigning a number to the puzzle day of week for a given solve.

**<h4>Figure 6. Overview of Solver Past Performance Features Development, and Predictive Features By Class**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/fb48ee5a-3ee3-42af-9a16-59d72bf74ac6)

### Machine Learning Regression Modeling 
For the modeling phase, puzzles completed in the first solve year (2021) were removed to minimize the potential negative effects of high baseline performance volatility (see EDA linked in Intro and **Fig. 3**). This reduced the overall sample size to N=965. Importantly, as they were generated prior to this filtering, 'Solver Past Performance Features' included in modeling accrued from the beginning of the solve period (August 2021). Additionally, for the main model 21x21 puzzles (Sun) were also removed from the sample. This resulted in a final modeling 15x15 puzzle N=828. The 21x21 puzzles (N=137) were, however, included in by-puzzle-day modeling (see **Figure 8**). 

After predictive features were generated for each puzzle, the best regression model for prediction of the TF (raw GMS solve time, in minutes) was found ('Best Model'). To find 'Best Model', 4 different regression models were explored using [scikitlearn (scikit-learn 1.1.1)](https://scikit-learn.org/stable/auto_examples/release_highlights/plot_release_highlights_1_1_0.html): Linear, Random Forest, Gradient Boosting, and HistGradient Boosting. For evaluation of models including all 15x15 puzzles, an 80/20 training/test split (662/166 puzzles) and 5-fold training set cross-validation were used. Additionally, hyperparameter grid search optimization was used per model as warranted (for ex., for Gradient Boosting the grid search was conducted for imputation type, scaler type, learning rate, maximum depth, maximum features, and subsample proportion used for fitting individual base learners). 'Best Model' (ie, lowest RMSE training error when hyperparameter-optimized) was a Linear Regression model. See the 'Model Metrics' csv files in the 'Reporting' folder for details, including how this model performed relative to the other models). Also, see **Supplementary Figure 1** for some details on the performance of 'Best Model' (ie, Data Quality Assessment, K-best features selection, and Feature Importances).

## Key Modeling Results

**1)** 'Best Model' predicted the TF (raw IS1 solve time, in minutes) more accurately than did a univariate linear model with puzzle day-specific (PDS), mean IS1 solve time *across the entire sample period* as the sole predictive input ('Mean PDS IST'). 'Best Model' also outperformed a variant that simply guessed the mean of the training set TF, *across all 15x15 puzzle days*, for each individual puzzle ('Dummy')(**Figure 7**). The 'Full Model' mean training error of 3.66 minutes, which corresponded to a 33.4% difference from the training set mean across all 15x15 puzzle days. In contrast, the 'Mean PDS IST' and 'Dummy' benchmark models had mean training errors of 3.93 and 5.66 minutes, respectively (corresponding to 35.9% and 51.8% differences from the training set mean). 

**Figure 7. Best Model Prediction Quality vs Benchmark Models**


*<h5> 'Best Model' was a Linear Regression Model (K-best features = 23).* 

###




## Data Supplement

**<h4>Table S1. Features Included in Predictive Modeling**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/e17a47b3-cc97-4fd6-9051-6c2cf17e9ef6)

**<h4>Figure S1. Best Model Metrics**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/2096dfac-7c54-405c-9227-89c39f987b8c)
*<h5> The Linear Regression Model yielded 'Best Model', out of the 4 tested. See the Model Metrics file in Reporting folder for full details. Panel A shows K best features selection for 'Best Model' based on mean CV score (k=7). Panel B shows the feature importances for the k best features in 'Best Model'. See Table S1 above for descriptions of these features, and also for those not selected. Panel C shows a Data Quality Assessment for best model. It appears that model quality may have been continuing to gradually improve at the number of samples used in the training set (n=~600).*

