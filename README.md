# Prediction of 30-Day Hospital Inpatient Readmission Rates 
# Introduction 
In 2012, the [Centers for Medicare and Medicaid Services (CMS)](https://www.cms.gov/Medicare/Medicare-Fee-for-Service-Payment/AcuteInpatientPPS/Readmissions-Reduction-Program) began reducing Medicare payments for subsection(d) hospitals with excess readmissions within 30 days of discharge, under the [Hospital Readmission Reduction Program (HRRP)](https://www.cms.gov/Medicare/Medicare-Fee-for-Service-Payment/AcuteInpatientPPS/Readmissions-Reduction-Program#:~:text=The%20Hospital%20Readmissions%20Reduction%20Program,in%20turn%2C%20reduce%20avoidable%20readmissions.). CMS has a proprietary formula for determining excess 30-day readmission rate (RR). Total readmissions for cases involving a specific set of conditions for which initial admission occured (heart attack, heart failure, pneumonia, chronic obstructive pulmonary disease, hip/knee replacement, and coronary artery bypass graft surgery) at a given hospital are divided by an expected RR that is based upon RR at hospitals deemed to be similar by CMS.

I was approached (in a purely hypothetical scenario) by the leadership of a medium-large sized (~2,700 inpatient cases in the previous year) public hospital in New Jersey, Community Medical Center (CMC), for assistance. In their most recent review from CMS, CMC was flagged as having an unacceptably high RR (17%), and was informed that they needed to reduce this rate by at least 1% over the next year to avoid Medicare payment reduction under the provisions of HRRP. CMC asked me to help them conduct their own analysis of factors most related to RR, and to potentially assist them in coming up with a data-driven plan to avoid a quality of care-reducing penalty.

# Methods and Approach
## Data Collection
I took two general approaches in data collection and inclusion for this project (summarized in **Figure 1**). The first was to obtain as much data as possible at the individual hospital level, with the logic that quantitative accounting of various outcomes during inpatient stays from admission to release will carry predictive information regarding the likelihood of patients returning to the hospital within 30 days of release. Additionally, a few available outpatient metrics potentially useful as proxies for efficiency of inpatient care at a given hospital (e.g., ER case load volume and mean patient wait times) were also included. For these "Within Hospital" data, a number of csv files(>10) were obtained from the [CMS website data repository](https://data.cms.gov/provider-data/sites/default/files/resources/). 

This "Within Hospital" data included:                                                   
* Proportions of initial admissions due to different reasons, including those in CMS’ own formula (heart failure, COPD, hip-knee replacement, heart attack, pneumonia, COPD, coronary artery bypass graft surgery)
* Rates of different types of infections during inpatient stay
* CMS-computed Patient Safety and Adverse events rates during inpatient stay
* HCAHPS-Related Features (Patient Survey Star ratings from inpatient stay)
* Various other measures from other aspects of hospital care (e.g., emergency department efficiency metrics and hospital worker vaccination rates)

The second data collection approach was focused on demographic and socioeconomic data at the country level. The rationale being that these types of data ("County-Level Demographic and Socioeconomic Data") would be useful for the development of features that provide context to "Within Hospital" data. From the [Bureau of Economic Analysis data archive](https://www.bea.gov/data), I obtained county-level data on median income. From the [US Census Bureau data archive](https://data.census.gov/cedsci/), I obtained data on county-level population and median age. 

The "County-Level Demographic and Socioeconimic Data" sources required substantial cleaning and standardization (especially ensuring county names are consistent across sources, and that same-named counties stayed with their appropriate state) before merging with the "Within Hospital" data tables (many, many Left Joins on multiple columns in this project!). Once cleaned and merged, these data souces were used to create several contextualizing features for each hospital. For example, "population per hospital in county" gave each hospital in the sample a de facto measure of theoretical overall patient burden.

**Figure 1. Data Collection and Inclusion Methodology/Rationale**
![image](https://user-images.githubusercontent.com/90933302/195432222-3250c7f7-efef-46ee-bc9b-b24105317815.png)


## Exploratory Data Analysis
All data manipulation, processing, and analyses were conducted in the Python/Pandas environment and documented with Jupyter Notebooks (anaconda3). Prior to the modeling stage, relationships between individual putative predictive features (~50) and the target feature (RR) were explored visually. Additionally, some categorial features (e.g., ER case load volume) were numerically encoded prior to this stage as well. During the EDA, it was discovered that many of the HCAHPS-Related Features (Patient Ratings) were highly correlated with each other and their presence in the data set was consequently reduced to several aggregate features capturing patient's overall sentiments about their care. This consolidation reduced concerns for multicollinearity in the first modeling stage (Linear Modeling). 

The strongest correlations (Pearson's correlation coefficient [r]) to target feature (RR) in the EDA were all negative correlations (**Figure 2**):
* **A**: Proportion of Initial Inpatient Admissions Due to Hip/Knee Replacement (r = -.22)
* **B**: Overall Hospital Rating (Patient Survey Star Rating from Inpatient Stay) (r = -.19)
* **C**: Proportion of Initial Inpatient Admissions Due to Acute Myocardial Infarction (r = -.18)
* **D**: Proportion of Healthcare Workers Vaccinated (r = -.15)
* Also of note were the fact that both Proportion of Initial Inpatient Admissions Due to COPD (**E**) and Proportion of Patients with Postoperative Respiratory Failure (**F**) had moderate POSITIVE correlations (both r = ~.10) with RR. These features would pop up again as very important in the best model, and would feature in the Client Recommendation (see below)

**Figure 2. Correlations of Select Predictive Features to Target Feature (RR)**
![image](https://user-images.githubusercontent.com/90933302/195627883-1e2d9785-984c-41c7-9e18-d6926c5212d9.png)


## Modeling
Overall, four predictive regression models were created using [scikitlearn (scikit-learn 1.1.1)](https://scikit-learn.org/stable/auto_examples/release_highlights/plot_release_highlights_1_1_0.html): Linear, Random Forest, Gradient Boosting, and HistGradient Boosting. For all models, a 70/30 training-test split and 5-fold training set cross validation were used. Additionally, hyperparameter grid search optimization was used per model as warranted (for example, for HistGradient boost, the grid search was conducted for imputation type, scaler type, learning rate, maximum iterations and maximum tree depth). By RMSE on training set cross-validation, the best model was Random Forest (0.748, 0.029), followed by Gradient Boosting (0.752,0.025), HistGradient Boosting (0.756, 0.031), and Linear Regression (0.779, 0.023). Test set validation predictions were within ~1% of training set predictions for each model, indicating that overfitting to the training set was not a major concern. A data quality analysis also indicted that the sample size was not hampering prediction quality. Importantly, the best model (and all full models) performed substantially better than a "dummy" model whereby RR was predicted simply by training set mean (RMSE 0.825). 

With regard to feature importance in the best model (**Figure 3**), the most important (by a substantial margin) was Proportion of Initial Inpatient Admissions Due to Hip/Knee Replacement. This was followed by Proportion of Initial Inpatient Admissions Due to Acute Myocardial Infarction (AMI), Proportion of Patients with Postoperative Respiratory Failure (Postop RF), Proportion of Patients with Postoperative Pneumonia, and Proportion of Initial Inpatient Admissions Due to COPD. Note that reasons for initial inpatient admission were NOT necessarily mutually exclusive. 

**Figure 3. Best Model Feature Importances**
![image](https://user-images.githubusercontent.com/90933302/195637740-ed674abd-7ff9-46c2-bdb3-98de4b4030bb.png)

*The relatively strong Pearson correlations with RR (see **Figure 2**) for 3 features also in the top 5 in terms of model importance (bold and italics in figure), along with their potential for actionability, provided some additional impetus to focus on these features as potential drivers of reduced 30-day RR (see Business Modeling and Client Recommendation section below for further rationale and discussion).* 

## Business Modeling and Client Recommendation
As a final step, best model was run on the entire dataset save for the client hospital (CMC). RMSE of the training set was slightly improved on the entire sample (0.739, 0.05) relative to the training set cross-validation in the previous step. The client hospital was shown to have a predicted RR of 15.45% with the best model, which is well below the 16.80% actual RR. I therefore recommended to the client that "watch and wait" is a viable strategy over the next year as variance alone could account for the observed RR that was of concern to CMS. 

However, I also ran a simulation in which CMC increased their Proportion of Initial Inpatient Admissions Due to Hip/Knee Replacement to that of the sample median and simultaneously also decreased Proportion of Patients with Postoperative Respiratory Failure to the sample median. In both of these areas, CMC was an outlier in the direction associated with higher RR (**Figure 4**, **Figure 5**). Both of these features were also among the few most important in the best model. In this simulation, we found that CMC could expect a RR reduction 32% of the way to the overall desired reduction. Addition of one more specialist for Hip-Knee Replacement surgery might be sufficient to boost that procedure sufficiently in the case mix. Indeed many hospitals at the low end of RR in the sample, boast a MAJORITY of cases as this type of elective surgery. As for reduction of Proportion of Patients with Postoperative Respiratory Failure, CMC has a higher than expected Proportion of Initial Inpatient Admissions Due to COPD (**Figure 6A**). It was also evident in the EDA stage that COPD is POSITIVELY correlated to Propoprtion of Patients with Postoperative Respiratory Failure (**Figure 6B**). Furthermore, Proportion of Initial Inpatient Admissions Due to COPD was one of the most important features in best model construction (**Figure 3**). Given that CMC itself is high in both COPD Proportion at Admission and Postoperative Respiratory Failure Rate, extra vigilance toward COPD patients undergoing surgery might go a long way toward reducing the latter at the hospital. 

**Figure 4. Distribution of Proportion of Inpatient Cases that Were Hip-Knee Replacements Across Hospitals**
![image](https://user-images.githubusercontent.com/90933302/195429189-d129d4b4-5d79-4c36-a674-23cc072c8f90.png)

*The client hospital (CMC; dashed red line) was an outlier in the direction associated with higher 30-day RR in proportion of inpatient cases that were hip-knee replacements. Simulation suggested that boosting the proportion of total inpatient cases that are this type of procedure could help reduce 30-day RR.*

**Figure 5. Distribution of Postoperative Respiratory Failure Rates Across Hospitals**
![image](https://user-images.githubusercontent.com/90933302/195430534-40d82435-36cf-459d-9af6-533a2485854c.png)

*CMC (dashed red line) was an outlier in the direction associated with higher 30-day RR in postoperative respiratory failure rate. Simulation suggested that measures taken to identify and monitor patients at risk of postoperative respiratory failure (e.g., those with COPD at admission) may help reduce this rate and thereby reduce 30-day RR.*

**Figure 6. Distribution of Rates of Cases Involving COPD at Admission Across Hospitals and Relationship to Postoperative Respiratory Failure**
![image](https://user-images.githubusercontent.com/90933302/195641380-900d05e0-7ea1-4c6d-8c8d-48a385fe61c7.png)

***A**: The previous figure (**Figure 5**) showed that CMC had a postoperative respiratory failure rate in the top several percent of hospitals in the sample. In addition, and perhaps related to this, CMC was in the top third of the sample in terms of COPD as a reason for inpatient admission.*

***B**: Across the entire sample of hospitals, proportion of inpatient cases involving COPD at admission was positively correlated to postoperative respiratory failure rate. Taken together with CMC's high postoperative respiratory failure rate as shown in A, this suggests that focusing on patients with COPD at admission might help reduce postoperative respiratory failure rate and reduce overall 30-day RR.*




## Future Directions
One potential way to improve the model would be to conduct unsupervised learning upfront to cluster hospitals by one or more high-level characteristic(s), and then fit a more bespoke model to each cluster. Currently, the ~2,700 hospitals in the sample (a minimum of 200 inpatient cased per year at a given hospital was used as a highpass filter during the data cleaning and wrangling stages) are all modeled together. While this hetereogeneity may be a strength in some respects, it may also be a hindrance in others. Public vs private or urban vs rural or large vs small hospitals, may systematically vary in some important ways that reduce prediction quality when attempting to model all hospitals in the overall sample together.

Additionally, one clear limitation of this project was the unavailabilty of detailed information about facilities, monetary and human resources on a per-hospital level. The county-level socioeconomic and demographic data brought in from the US Census and Bureau of Economic Analysis were fairly crude proxies for direct knowledge of the resourcing at individual hospitals. An ideal data set would include data related to resourcing at the level of *individual procedure types* at individual hospitals. If high rates of adverse outcomes for specific admission types were associated with staffing issues, for example, predictions could be generated to bear directly on whether simply adding staff in the right places might lower 30-day RR. Perhaps direct queries to individual hospitals can yield at least some of this direct resourcing data not available from the CMS data repository. Some optimism for the availability of this approach can be drawn from availability of data on the "treatment bandwidth" and level of general preparedness of *outpatient* services at a number of hospitals (e.g., mean waiting time for patients in the ER, Proportion of Healthcare Workers Vaccinated) that was included in the CMS data set. These data were used as proxies for overall facility quality in predictive modeling, but obviously there could well be a disconnect between outpatient vs inpatient service quality and staffing on a per hospital basis that would counter the effectiveness of this approach. 
 


     




