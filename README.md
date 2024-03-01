# Predictive Modeling of Individual Solver 1 (IS1) Performance on the New York Times Crossword Puzzle

## Introduction

### Project Overview and Data Sources
This summary reports on the results of predictive modeling of my own (Individual Solver 1; IS1) performance on all puzzles issued during a 3+ year (Oct. 2020 - Feb. 2024) sample of the [New York Times (NYT) crossword puzzle](https://www.nytimes.com/crosswords). Previously, I conducted a [comprehensive exploratory data analysis (EDA) of IS1 performance](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Individual-Solver-1/blob/main/README.md) over this sample period. From this EDA, numerous features pertaining to both the puzzles themselves as well as to IS1 past performance were identified as candidate features for predictive modeling of IS1 solve times.    

Without access to two specific data sources this project would not have been possible. The first, [XWord Info: New York Times Crossword Answers and Insights](https://www.xwordinfo.com/), was my source for data on the puzzles themselves. This included a number of proprietary metrics pertaining to the grids, answers, clues and constructors. XWord Info has a contract with NYT for access to the raw data underlying these metrics, but I unfortunately do not. Therefore, I will not be able to share raw or processed data that I've acquired from their site. Nonetheless, [Jupyter notebooks](https://jupyter.org/) with all of my Python code for analysis and figure generation can be found [here](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/tree/main/notebooks). The second, [XWStats](xwstats.com), was my source for historical solve time data for both IS1 and for the "Global Median Solver" (GMS). XWStats (Matt) derives the GMS solve time as that at the 50th percentile out of ~1-2K individual solvers providing their solve times per puzzle. In the context of IS1 modeling, historical GMS solve times were used to derive a "Strength of Schedule" adjustment to features capturing IS1 past performance prior to a given solve time to be predicted (see Methods section for details). I have previously completed both [EDA](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Global-Median-Solver?tab=readme-ov-file#readme) and [predictive modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Global-Median-Solver/blob/main/README.md) for the GMS. In addition, I have also now completed [EDA](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Individual-Solver-2/blob/main/README.md) and [predictive modeling]() for a second individual solver (IS2) as well. 

Please visit, explore and strongly consider financially supporting both of these wonderful sites; XWord Info via [membership purchase at one of several levels](https://www.xwordinfo.com/Pay) and XWStats via [BuyMeACoffee](https://www.buymeacoffee.com/xwstats). 

### Overview of NYT Crossword and IS1 Characteristics
The NYT crossword has been published since 1942, and many consider the "modern era" to have started with the arrival of Will Shortz as (only) its 4th editor 30 years ago. A new puzzle for each day is published online at either 6 PM (Sunday and Monday puzzles) or 10 PM (Tuesday-Saturday puzzles) ET the prior evening. Difficulty for the 15x15 grids (Monday-Saturday) is intended to increase gradually across the week, with Thursday generally including a gimmick or trick of some sort (e.g., "rebuses" where the solver must enter more than one character into one or more squares). Additionally, nearly all Sunday through Thursday puzzles have themes, some of which are revealed via letters placed in circled or shaded squares. Friday and Saturday are almost always themeless puzzles, and tend to have considerably more open constructions and longer (often multiword) answers than the early week puzzles. The clue sets tend to be more wordplay heavy/punny as the week goes on, and the answers become less common in the aggregate as well. Sunday puzzles have larger grids (21x21), and almost always feature a wordplay-intensive theme to which the longest answers in the puzzle pertain. The intended difficulty of the Sunday puzzle is approximately somewhere between a tough Wednesday and an easy Thursday. 

**Figure 1** shows dimensionality reduction via Principal Component Analysis (PCA) of 23 grid, clue and answer-related features obtained from XWord Info. This analysis demonstrates that, while puzzles from a given puzzle day do indeed aggregate with each other in n-dimensional "puzzle property space", the puzzle days themselves nonetheless exist along a continuum. Sunday is well-separated from the other puzzle days in this analysis by PCA1, which undoubtedly incorporates one or more grid size-contingent features.   

**<h4>Figure 1. PCA of Select Puzzle Grid, Clue and Answer Features**                                                                  

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/66495fd6-7d11-4cae-ae80-f1a833420e92)
*<h5>The first 3 principal components accounted for 47.5% of total variance. All puzzles issued from Jan. 1, 2018- Feb. 24, 2024 were included in this analysis (N=2,246).*
###

The overlapping distributions of per puzzle day IS1 solve times across the entire sample period (**Figure 2**) show a parallel performance phenomenon to the continuum of puzzle properties seen in **Fig. 1**. While solve difficulty increased as the week progressed, puzzle days of adjacent difficulty still had substantially overlapping IS1 solve time distributions. Other than for the "easy" days (Monday and Tuesday), distributions of IS1 solve times were quite broad. It should be noted, however, that the broadness of each puzzle day-specific IS1 solve time distribution was also increased by the fairly dramatic improvement in IS1 performance over the full sample period.      

**<h4>Figure 2. Distributions of IS1 Solve Times by Puzzle Day for Full Sample Period**                   

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/120f724e-3782-47f2-948b-cbaa2f17911e)
*<h5>All puzzles completed by IS1 in the sample period were included in this analysis (N=1,197).* 

### Key Outcomes from the IS1 EDA

One of the most important findings from the EDA as far as implications for predictive modeling was that, despite several periods of volatility across puzzle days (e.g., April-May 2023), IS1 demonstrated continual, marked improvement over the course of the sample period across all puzzle days (**Figure 3**). Coupled to the fact that puzzle day-specific recent past performance ('Recent Performance Baseline' [RPB]) was highly positively correlated to performance on the next puzzle both overall (r=.74) and across puzzle days (**Figure 4**), this created an imperative to explore and potentially include different variants of this feature type in the predictive modeling stage.    

**<h4>Figure 3. IS1 Solve Time Overview by Puzzle Day: 10-Puzzle Moving Averages and Distributions of Raw Values**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/3366123c-350a-42f8-908d-9a2a79debd97)
*<h5>All puzzles completed by IS1 in the sample period were included in this analysis (N=1,197).* 

**<h4>Figure 4. Puzzle Day-Specific, Recent Performance Baseline (RPB) Correlation to IS1 Performance on the Next Puzzle**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/1a94caba-53dc-4567-9d1c-ab646bbe00ac)
*<h5> Puzzle-day specific, recent past performance (x-axis) was calculated over the 8 day-specific puzzles previous to the next solve (y-axis) to obtain RPB. All puzzles completed by IS1 from Jan. 1, 2022- Feb. 19, 2024 were included in this analysis (N=967).* 

###
Along with the 'RPB' discussed above, multiple features pertaining to the puzzles themselves demonstrated moderately strong or strong correlations with IS1 performance on individual puzzles (**Figure 5**). Two that stood out in particular for their correlational strength with IS1 performance were 'Average Answer Length' and 'Freshness Factor', the latter of which is a proprietary XWord Info measure of the rareness of a given answer in the NYT puzzle. The strengths of these positive correlations with IS1 solve times (for all 15x15 puzzles: r=.56 for both features) can be seen both in the correlation heatmap (A; top row - 5th and 11th columns) as well as in the overall (black) and per-puzzle day (colors) feature correlation scatterplots in panel B. The feature density plots (C) show that the distributions of these features were well-separated across puzzle days. This is an important property for candidate predictive features to have since, as is shown in **Fig. 2**, distribution peaks of solve times for individual puzzle days were themselves well-separated.   

**<h4>Figure 5. Correlations of Puzzle-Related and Past Performance Features to IS1 Solve Times**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/1aeb9bf0-1ce4-454f-83e4-2626185080ce)
*<h5> All puzzles completed by IS1 from Jan. 1, 2022- Feb. 19, 2024 were included in this analysis (N=967).*

## Methods
### Predictive Feature Generation

For predictive feature generation, all puzzles completed by IS1 from Aug. 8, 2021-Feb. 17, 2024 (N=1,196) were included. This full sample included puzzles issued by NYT as early as August, 2021. The right panel of **Figure 6** summarizes predictive features included in the modeling stage (N=40) by broad class. A few key example features from each class are mentioned below. **Supplementary Table 1** comprehensively lists out, classifies and describes all included features.  

* 'Solver Past Performance' features (n=6) included 'IS_RPB_l8'. This feature captured puzzle day-specific 'Recent Performance Baseline' ('RPB') over the 8 puzzles immediately prior to a puzzle with a time being predicted. A number of temporal integration windows and time-decay weighting curves for this feature were tested in [preliminary univariate linear regression modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/tree/main/notebooks/learning%20rate%20modeling), and an 8-puzzle window (l8) with *no* time-decay weighting yielded the lowest root mean square error (RMSE) mean training error. Furthermore, it was found that predictions with these parameters further improved with adjustment of this past performance feature by the performance of *the GMS* over the same set of puzzles relative to the *GMS' own* recent performance prior to those puzzles. This is referred to as 'Strength of Schedule adjustment' ('SOS adjustment') from here forward, and the left panel of **Fig. 6** depicts creation of this feature. Also note that another feature in this class, which captured normalized past performance against the constructor(s) of a puzzle being predicted ('IS Past Perf vs Constr'), used this 'SOS adjustment' in calculation of the baseline solve time expectation component (see **Supp. Table 1**).   
 
* 'Puzzle: Clue or Answer' features (n=19) included 'Freshness Factor', which measured the aggregate rarity of answers in a given puzzle across all NYT crossword puzzles from before or since the issue date. This class also included other measures of answer rarity, including the number of entirely unique answers in a puzzle ('Unique Answer #') and the 'Scrabble Score', which assigns corresponding Scrabble tile values to each letter in an answer (rarer letters = higher tile values, hence a different angle at assessing answer rarity). On the clue side of the ledger, this class also included a count of the frequency of wordplay in clues for a given puzzle ('Wordplay #'). Later week puzzles contained more such clues, and early week puzzles often contained very, very few. 

* 'Puzzle: Grid' features (n=11) included both the number of answers ('Answer #') and 'Average Answer Length' in a given puzzle. As puzzles got more difficult across the week, the former tended to decrease and the latter tended to increase. This class also included 'Open Squares #', which is a proprietary measure of XWord info capturing white squares not bordered by black squares (tended to increase as puzzles increased in difficulty across the week). Additionally, this category included features capturing other design principles of puzzles, including 'Unusual Symmetry'. This feature captured puzzles deviating from standard rotational symmetry (e.g., those with left-right mirror or diagonal symmetry) that could have had implications for their difficulty.

* 'Circadian' features (n=3) included 'Solve Day Phase', which broke puzzles completion timestamps (obtained per solve via XWStats) into four 6-hour time bins. Per puzzle being predicted, 'IS_per_sdp_avg_past_diff_from_RPB' was a feature that measured how recent puzzle day-specific performance in the pertinent 'Solve Day Phase' compared to 'RPB' across all solve phases. Calculation of this feature used 'SOS-adjustment' (see 'Solver Past Performance' features) in deriving 'RPB'.

* 'Puzzle Day of Week' (n=1) was a class of one ('DOW_num'), simply assigning a number to the puzzle day of week for a given solve.

**<h4>Figure 6. Overview of Solver Past Performance Features Development, and Predictive Features By Class**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/fb48ee5a-3ee3-42af-9a16-59d72bf74ac6)

### Machine Learning Regression Modeling 
For the modeling phase, puzzles completed in the first solve year (2021) were removed to minimize the potential negative effects of high baseline performance volatility (see EDA linked in Intro and **Fig. 3**). This reduced the overall sample size to N=965. Importantly, as they were generated prior to this filtering, 'Solver Past Performance' features included in modeling accrued from the beginning of the solve period (August 2021). Additionally, for the main model 21x21 puzzles (Sun) were also removed from the sample. This resulted in a final modeling 15x15 puzzle N=828. The 21x21 puzzles (N=137) were, however, included in by-puzzle-day modeling. 

After predictive features were generated for each puzzle, the best regression model for prediction of the Target Feature (TF) (raw IS1 solve time, in minutes) was found ('Best Model'). To find 'Best Model', 4 different regression models were explored using [scikitlearn (scikit-learn 1.1.1)](https://scikit-learn.org/stable/auto_examples/release_highlights/plot_release_highlights_1_1_0.html): Linear, Random Forest, Gradient Boosting, and HistGradient Boosting. For evaluation of models including all 15x15 puzzles, an 80/20 training/test split (662/166 puzzles) and 5-fold training set cross-validation were used. Additionally, hyperparameter grid search optimization was used per model as warranted (for ex., for Gradient Boosting the grid search was conducted for imputation type, scaler type, learning rate, maximum depth, maximum features, and subsample proportion used for fitting individual base learners). 'Best Model' (ie, lowest RMSE training error when hyperparameter-optimized) was a Linear Regression model. See the 'Model Metrics' csv files in the 'Reporting' folder for details, including how this model performed relative to the other models). Also, see **Supplementary Figure 1** for some details on the performance of 'Best Model' (ie, Data Quality Assessment, K-best features selection, and Feature Importances).

## Key Modeling Results

**1)** 'Best Model' predicted the TF (raw IS1 solve time, in minutes) more accurately than did a univariate linear model with puzzle day-specific (PDS), mean IS1 solve time *across the entire sample period* as the sole predictive input ('Mean PDS IST'). 'Best Model' also outperformed a variant that simply guessed the mean of the training set TF, *across all 15x15 puzzle days*, for each individual puzzle ('Dummy')(**Figure 7**). The 'Full Model' mean training error of 3.66 minutes, which corresponded to a 33.4% difference from the training set mean across all 15x15 puzzle days. In contrast, the 'Mean PDS IST' and 'Dummy' benchmark models had mean training errors of 3.93 and 5.66 minutes, respectively (corresponding to 35.9% and 51.8% differences from the training set mean). 

**<h4>Figure 7. Best Model Prediction Quality vs Benchmark Models**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/5d88681a-abbe-4ab0-b30a-87d1df7c7ad1)
*<h5> 'Best Model' was a Linear Regression Model (K-best features = 7).* 

###

**2)** When individual feature classes or adjustments were systematically subtracted in the modeling stage ('Subtraction Analysis')(**Figure 8**), subtraction of 'Past Performance' features resulted in by far the largest increase in model error relative to 'Best Model' (8.5%). This increase in model error was increased even more (13.1%) when the 'Puzzle Day' feature class, constituting the only other overt information regarding puzzle day in the feature set, was additionally removed. The only other impactful subtraction was removal of 'SOS adjustment' from 'Past Performance' and 'Circadian' features, which resulted in a 1.5% increase in model error. Each other feature class or adjustment subtracted from 'Best Model' resulted in a <1% increase in model error. **Fig. 8** shows, in decreasing order of negative impact on model prediction quality, the effect of removing individual feature classes or adjustments (hatched bars) from the full 'Best Model'.    

**<h4>Figure 8. Effect on Model Prediction Quality of Removing Individual Feature Classes or Adjustments from the Best Model**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/a77f2862-77ea-4426-b5e1-447846b0a80b)
###
Because subtraction of 'Past Performance' features resulted in substantial reduction in prediction quality, a sub-analysis looked at the impact of removing individual features from this class. 'Past Performance' features included 'SOS-adjusted' IS1 performance on the immediately previous 8 puzzle day-specific puzzles ('Recent Performance Baseline' [RPB]), standard deviation of 'RPB' ('IS_RPB_l8_stdev'), normalized past performance against the constructor(s) of a given puzzle ('IS Past Perf vs Constr'), and number of past solves on both a puzzle day-specific and non-puzzle day-specific basis ('Prior Solves # - DS' and 'Prior Solves # - NDS', respectively; 'IS Solves l7' was the non-puzzle day specific number of solves in the prior 7 days). The left panel of **Figure 9** shows that removal of 'RPB' was responsible for over half (4.9%) of the large increase in model error with the removal of *all* features from this class (8.5%). Removal of 'IS_RPB_l8_stdev' resulted in the only other impactful increase in model error of any feature subtraction in this class (.51%). Removal of the other features in this class each resulted in an increase of model error of <.1%. 

Subtraction of features from classes other than 'Past Performance' that were included in K-best features in 'Best Model' (see **Fig. S1**) had negligible impact on model training error (**Fig. 9**; right panel). Of these features relevant to generation of best model, only removal of the 'Grid' feature indicating number of answers in a puzzle ('Answer #') resulted in so much as a >.1% increase in model error. Notably, though puzzle day of week ('DOW_num') had a large impact on model error when the feature set was already stripped of all 'Past Performance' features, the impact of its removal was negligible when that feature class remained intact. 

**<h4>Figure 9. Effect on Model Prediction Quality of Removing Key Individual Features**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/d73396ed-e91a-4f9e-b6ae-8306e0c38654)

###
**3)** 'Best Model' was discovered on a puzzle day-specific basis ('BPDM'), including for the lone 21x21 puzzle day Sunday (**Figure 10**). Because IS1 mean solve time per puzzle day varied considerably, training errors in **Fig. 10** were normalized to percentage difference from training set mean for that puzzle day. The 'Dummy' model in this puzzle day-specific context is analogous to the 'Mean PDS IST' benchmark model in **Fig. 7**, as the 'Dummy' for all 15x15 puzzles guessed the *overall sample mean* for each puzzle regardless of puzzle day. 

Though the number of puzzles included in the 'BPDMs' (N=140; +- 2) was much smaller than that in the all 15x15 puzzles model, each still outperformed its particular (not so dumb) 'Dummy' with the exception of Thursday (training error nearly equal between 'BPDM' and 'Dummy'). Sunday (21.1% mean 'BPDM' training error) and Monday (14.0%) stood out as the two most predictable individual puzzle days, and the standard deviation of 'BPDM' training error for those days was also relatively small compared to the mean gap between 'BPDM' and 'Dummy'. High performance variability on later week puzzle days (Fri and Sat) can be seen in 'BPDM' standard deviations that overlap extensively with 'Dummy' model performance. It is worth noting, however, that despite the much smaller sample size, each 'BPDM' other than Fri and Sat outperformed the all 15x15 puzzle days 'Best Model' (33.4%; see **Fig. 7** and associated text) on a % of mean solve time basis. 

**Figure 10. Best Puzzle Day-Specific Model (BPDM) Prediction Quality**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/b67516d3-a908-491f-a971-2cabc1aa52f3)
*<h5> 'BPDM' for each day was a Linear Regression Model, with hyperparameter optimization specific to that puzzle day. Due to the relatively small number of puzzles in the sample for each puzzle day, a 90/10 training/testing split was used to find each 'BPDM' (126/14 +-2 puzzles for each puzzle day). Data Quality Assessments across 'BPDMs' uniformly indicated that model quality continued to increase at 90% training set inclusion.* 

## Discussion

#### *Best Model for all 15x15 puzzles outperformed benchmarks* 
The main result of this study was that the full 'Best Model', incorporating both Individual Solver 1 (IS1; me) past-performance features as well as numerous features capturing different aspects of individual puzzle grid, clue and answer properties, outperformed several benchmark models. 'Best Model' greatly outperformed (by ~2 minutes on average) a 'Dummy' model, which guessed the total 15x15 puzzles sample mean solve time for each individual puzzle. This is unsurprising, since different puzzle days included in the modeling set have distinct solve time distributions and peaks. More encouraging that this modeling approach is on the right track is that 'Best Model' also (by ~.3 minutes on average) outperformed a model which guessed the sample mean of the *specific puzzle day* for each individual puzzle. As discussed in the next section, accurately parameterizing 'Recent Performance Baseline' ('RPB') was critical for achieving acceptable model performance for this solver. 

#### *For Best Model, IS1 past performance features were more important than puzzle-specific features*
Another clear finding of this study was that, as a class, 'Past Performance' results had an *overwhelmingly* larger impact on prediction quality than did any other class. Removing this class entirely (subtraction analysis), along with the only other overt encoding of puzzle day in the feature set ('DOW_num'), resulted in a >13% increase in training error (~.55 minutes on average) compared to 'Best Model'. In contrast, removal of no other feature class in isolation resulted in more than a .1% increase in training error. In golf terms, the recent playing form of the golfer was vastly more important than the specific characteristics of the course being played. With that said, it is critical to point out that 'Past Performance' features themselves had puzzle-day specific information built into them. 'RPB', for example, was specific to the puzzle day of the puzzle being predicted. Thus, because "courses" per puzzle day tended to have similar characteristics, many puzzle features themselves may have been somewhat redundant to information carried by 'RPB'. The inverse was clearly not true, however, as puzzle characteristics did not carry any "memory" of how an individual performed relative to their values in the past. 

While 'RPB' proved to be essential to model performance, further improvements to 'RPB' and presumably the model overall, will almost certainly come with adopting a piecemeal approach to discovery of the temporal integration window for past puzzle inclusion in the 'RPB'. The results of the EDA (see **Fig. 3**) made it clear that the slope of IS1 improvement varied across the full sample period. My plan for the next modeling iteration is to respect this nonlinearity in learning rate by deriving the optimal temporal integration window separately by solve period. There's also the potential to integrate recent solve rate (represented as a stand-alone feature in the current project by 'IS Solves l7') into the discovery of this window; with a default hypothesis that periods of higher solve rate per unit time will be associated with smaller optimal temporal integration windows for 'RPB'.     

#### *Accurately  parameterizing puzzle day-specific recent past performance was critical to optimizing Best Model prediction quality*
In previous modeling of the Global Median Solver (GMS; see Intro for link), it was found that the best parameters for capturing 'RPB' were a long temporal integration window (previous 40 puzzle day-specific puzzles) and gradual temporal decay-weighting in that calculation (40, 39,38...). In stark contrast, preliminary univariate linear modeling for IS1 found that a much shorter temporal integration window (previous 8 puzzle day-specific puzzles) and *no* decay weighting was optimal. The learning/improvement rate of the GMS was likely much slower because the sequencing of solves by *puzzle issue* date for the GMS was a necessary approximation (I did not have puzzle completion timestamps for the GMS as I do for IS1). Compounding this are the facts that the GMS is a composite entity and that IS1 is a much more skilled solver than the median solver ('win rates' over the GMS of ~90% across puzzle days; see link in Intro to EDA). What did make a meaningful beneficial difference in prediction quality (1.5% improvement), however, with this short (8-puzzle) optimal temporal integration window for IS1 'RPB' was the use of GMS past performance to generate a 'Strength of Schedule adjustment' for IS1 past performance prior to a given solve. This was possible because of the high positive correlation (see IS1 EDA) between IS1 and GMS solve times on a puzzle-by-puzzle basis, despite the large gap in skill level. It is imperative to note, however, that information on GMS performance on the puzzle being predicted was *not* included in this adjustment (ie, data leakage was steadfastly avoided). 

The lack of impact of the 'Past Performance vs Constructor(s)' feature for IS1 provided another stark contrast with GMS modeling. This was anticipated by differences in correlational strength in the EDA for each solver, and largely likely reflects the fact that the much larger overall sample size for the GMS provided a critical mass of past puzzles per constructor in the context of predictive accuracy. However, there was likely also a meaningful contribution here from the fact that 'RPB' itself was much more effectively modeled in IS1 than in the GMS. Especially in light of the fact that very effective SOS-adjustment was possible for IS1 'RPB' due to the availability of median solver data. The removal of 'RPB' alone resulted in a ~5% increase in model error, which was much larger than the concomitant error increase of its removal in the GMS model. This "unreasonable effectiveness" of 'RPB' modeling for IS1 may also underlie the lack of impact of removing any non-past performance feature. For example, though 'Freshness Factor' (an 'Answer' feature capturing aggregate rarity of answers in a given puzzle) was the second most important feature in 'Best Model' (see Fig. S1), its removal had only a negligible effect on model prediction quality. In contrast, removal of 'Freshness Factor' in the GMS 'Best Model' resulted in a 1.1% increase in error. There is no exaggeration in the statement that well-captured day-specific recent solve form for IS1 simply overwhelmed any other feature or feature class in terms of predictive power.      

 #### *Day-specific models were difficult to interpret, but a larger sample per puzzle day could yield superior model accuracy* 
Unequivocally the most difficult to interpret result of the study was the by-puzzle-day modeling ('BPDM'). The obvious reason for this difficulty was the greatly reduced sample size of each puzzle day relative to the full 15x15 data set employed in generating 'Best Model' (training set sizes of ~125 instead of ~660, despite changing the train/test split from 80/20 to 90/10). To this point, relatively large standard deviations can be seen in **Fig. 10**, and these standard deviations were particularly large for the more heterogenous and more difficult later week puzzle days. Also, though Monday and Sunday appeared to have distinctly higher quality predictions than the other puzzle days, guessing just based on day-specific sample mean ('Dummy') was also proportionately more accurate for those puzzle days than the others. This implies greater homogeneity of puzzle characteristics for those two days relative to the others that led to generally more clustered solve times. Clearly more data would be a great boon to this approach, a fact backed up by the Data Quality Analysis for 'Best Model', where model quality didn't level off at the maximum training set size (see **Fig. S1**). 

It is interesting to point out with regard to the 'BPDMs', however, that despite small sample sizes the majority of these puzzle day-specific models were *more* accurate than the all 15x15 puzzles 'Best Model' in terms of training error as a percentage difference from sample mean. The likely explanation is that, though inadequately powered, each day-specific model contained only puzzles with properties/features highly relevant to those in the puzzle under prediction. On the flip-side, it's likely that the cross-days heterogeneity of puzzles included in the all 15x15 puzzles model added algorithmic challenge even as the increased sample size helped model convergence to a somewhat counterbalancing degree. A reasonable way to "have our cake and eat it too" here would be to have a median-type solver complete many hundreds of puzzles appropriate for individual puzzle days within a reasonably short timeframe (say, months) and then either model them separately or perhaps cluster Mon-Wed in an "easy" model and Thu-Sat in a "hard" model. The main challenge to the first approach would be to accurately model RPB with an unusual rate of solving, and the main challenge to the latter would be that each puzzle day truly does have its own idiosyncrasies (and splitting the sample in half probably doesn't help overcome that).

#### *What's Missing?*
As a percentage of 15x15 puzzle overall sample mean, 'Best Model' for IS1 (33.4% mean training error) was not as accurate as that for the GMS (23.7%). While it is likely that a large part of that discrepancy was due to the ~2x sample size advantage for the GMS, it is equally likely that a substantial amount is also due to the differences in modeling the performance of one individual solver vs modeling a mathematical construct. For one thing, the concept of a "bad day" for the 'GMS' doesn't really exist; if one solver falls off, there's another to take their place smack in the middle of the distribution for that day. While "bad day" is hard to quantify for an individual solver, there are any number of variables that could be measured that might help predict performance variability on a given puzzle. Some examples of what could be plausibly measured consistently and accurately are sleep quality the night prior to a solve, body temperature, heart rate, ambient noise level, degree of caffeination or blood alcohol level, posture while solving, device solving occurred on and lighting conditions. Circadian variables related to completion timestamp were taken into consideration in the present modeling exercise, but did not have a meaningful impact (though some interesting performance patterns at the extremes of performance were detected in the EDA for IS1). Nonetheless, it is almost certain that the performance of an individual solver will be more subject to noisiness that that of the mathematical construct of "median solver", and this difference has very real consequences in the building of an accurate model.    

Of the different classes/subclasses of puzzle-specific features, the one I find to be the most lacking from this modeling iteration is unquestionably the 'Clue' class. The 'Answer' feature 'Freshness Factor' was important for prediction quality for the GMS, and was ranked second in 'Best Model' feature importance for IS1 (though subtraction in isolation had minimal effect). What I'd like to find or create myself is an analogous feature to get at the unusualness of words within clues. One can imagine puzzles where the 'tough' words are in the clues not in the answers, and this would be missed to some extent by the current feature set. Also on the 'Clue' side, simply quantifying 'Average Clue Length' would be quite useful. It stands to reason that the more reading a solver must do to know what to answer, the slower the solve will be. There are also several other types of cross-classification trickiness in puzzle design that would be hard to quantify, but I think worth a shot at doing so. One of these I call 'Answer Ambiguity', which arises when there is more than one plausible answer in an unfilled piece of grid for a given clue. My strong sense as an experienced solver myself is that this is a wheel spin-inducing art form wielded more by some constructors than others, and more typically on later-week puzzle days. There's also another concept that I call 'Design Isolation', where the details of the construction give a solver fewer 'outs' in a tough corner of a puzzle than they'd otherwise have. I don't quite think current features like 'Unusual Symmetry', 'Cheater Square #', or 'Open Squares' are capturing this in a useful way. 

Finally, I'd like to point out that unlike other prediction projects I've carried though (most notably my [men's tennis match prediction project](https://github.com/ursus-maritimus-714/Mens-Tennis-Prediction?tab=readme-ov-file#readme), I have hardly dabbled in the space of "features derivatives"; the taking of "primary" features and combining them in various ways (for ex. creating ratios of puzzle parameters to other puzzle parameters) in hope that one or several just 'click' for a particular model algorithm. That process can be extremely time consuming (the many hundreds of hours I spent on the tennis project attest to that), but also very fruitful and often so in ways that are unexpected going into the modeling stage of a project. 

## Data Supplement

**<h4>Table S1. Features Included in Predictive Modeling**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/62d32cb4-f4ed-4dc0-94be-d3bc571f5ca0)

**<h4>Figure S1. Best Model Metrics**

![image](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/assets/90933302/2096dfac-7c54-405c-9227-89c39f987b8c)
*<h5> The Linear Regression Model yielded 'Best Model', out of the 4 tested. See the Model Metrics file in Reporting folder for full details. Panel A shows K best features selection for 'Best Model' based on mean CV score (k=7). Panel B shows the feature importances for the k best features in 'Best Model'. See Table S1 above for descriptions of these features, and also for those not selected. Panel C shows a Data Quality Assessment for 'Best Model'. It appears that model quality may have been continuing to gradually improve at the number of samples used in the training set (n=~600).*

