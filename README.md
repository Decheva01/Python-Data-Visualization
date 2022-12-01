# (Dataset Exploration Title)
## by (your name here)


## Dataset
Prosper Marketplace, Inc. is a San Francisco, California-based peer-to-peer lending company. More information is available on the Prosper Wiki site: [Prosper Wikipedia](https://en.wikipedia.org/wiki/Prosper_Marketplace).

The data set used in this report contains 113,937 loans with 81 variables on each loan. The last update was made on 03.11.2014

[Data Source](https://s3.amazonaws.com/udacity-hosted-downloads/ud651/prosperLoanData.csv)

[Variable Dictionary](https://docs.google.com/spreadsheets/d/1gDyi_L4UvIrLTEC6Wri5nbaMmkGmLQBk-Yx3z0XDEtI/edit#gid=0)

The outcome variable that we are seeking to predict/explain is loan outcome status. The loan outcome status variable contains multiple categories. They can be separated into ongoing (or current) loans and past (or completed) loans. There are several categories of past/completed loans, which can be compressed to two: repaid vs. not repaid.

Out of the multitude of variables that could be considered predictors of loan outcome, we selected variables that are described in a paper by [Serrano-Cinca, Gutierrez-Nieto & Lopez-Palacios (2015)](https://www.researchgate.net/publication/282426542_Determinants_of_Default_in_P2P_Lending). They grouped predictor variables together in the following manner:

### 1) Borrower Assessment. 
This category consists of customer credit rating and loan interest rate. Prosper uses two different customer scoring systems (one consisting of 7 grades and another consisting of 10 grades) and, additionally, a customer credit score range provided by a consumer credit rating agency.

### 2) Loan Characteristics. 
The characteristics of a loan are its purpose and amount. There is a fairly clear hypothesis related to the relationship between loan purpose and the probability of default, i.e., consumer loans (e.g., a wedding loan) are generally less risky than a loan for a small business. The relationship between the loan amount and the probability of default is less clear (some studies show a positive correlation, while others show a negative correlation). 

### 3) Borrower Characteristics. 
The borrower characteristics are income, housing and employment. 

### 4) Borrower Credit History. 
Borrower credit history is characterized by:

   4.1) Time since first loan (i.e., credit history lenght) and number of total and open loans;

   4.2) Number of open revolving accounts and the monthly payment on those;

   4.3) "Revolving Utilization": "Revolving line utilization rate, or the amount of credit the borrower is using relative to all available revolving credit";

   4.4) Current delinquencies, number of past delinquencies and amount of delinquencies on loans;

   4.5) Number of inquiries by creditors (totally and for a current period);

   4.6) Derogatory public records (Prosper saves data for the last 12 months and for the last 10 years).
   
   The Prosper data set does not contain a "revolving utilization" variable. There is a variable 'BankcardUtilization' ("the percentage of available revolving credit that is utilized at the time the credit profile was pulled.") which could be construed as revolving utilization rate. 
   
The rest of the credit history variables are represented in the Prosper data set.  

### 5) Borrower Indebtedness. 
Borrower Indebtedness variables are the following: loan amount to annual income, annual payment to annual income and debt to income ratio. The Prosper data set only contains one of these three variables: 'DebtToIncomeRatio'.

Additionally, the **loan term** could be one of the factors determining default rate.

### Data Wrangling Steps

#### Cleaning/Wrangling Issue #1
A few ordinal variables of interest, including 'CreditGrade', 'ProsperRating (Alpha)' and 'IncomeRange', are of data type 'object'. We transformed them into ordinal categorical variables. The levels of 'EmploymentStatus' were arranged in a meaningful way (although it is not an ordinal variable).

#### Cleaning/Wrangling Issue #2 
There are three **credit rating variables** (apart from the professional rating scores 'CreditScoreRangeLower' & 'CreditScoreRangeUpper'): 

  - the variable 'CreditGrade' contains only records made **until July 2009**, 
  - 'ProsperRating (Alpha)' and its "clone" 'ProsperRating (numeric)' both apply only to records made **since July 2009**, and 
  - 'Prosper Score' is a 'custom risk score built using historical Prosper data' (according to variable dictionary), it has 10 levels and has been used **since July 2009**.

    According to the variable dictionary, 'Prosper Score'has 10 levels, however, quite a few borrowers have a value of 11 on that variable. After examining these borrowers' 7-level credit score, we suppose that this is a mistake and the real score of these borrowers should be 10.
    
    'CreditGrade' and 'ProsperRating (Alpha)' both have 7 levels, although 'CreditGrade' has an additional level 'NC' which probably corresponds to the 0 of 'ProsperRating (numeric)' and could stand for "no credit (history)". 

#### Cleaning/Wrangling Issue #3
The loan term variable, 'Term', is of the data type 'integer', however, there are only three possible terms and we are not interested in mean/median loan term as outcome variable, which is why it should be regarded as an ordinal variable. Therefore, we created an additional ordinal variable, 'term_ordinal', with three ordered string values: '12 months', '36 months', '60 months'.

#### Cleaning/Wrangling Issue #4
The **loan purpose** is represented by the variable 'ListingCategory (numeric)'. The labels of the categories can be found in the variable dictionary. We created a new categorical variable 'loan_purpose' with labeled categories.

#### Cleaning/Wrangling Issue #5
There is no variable 'credit history lenght', but it can be calculated from these two other variables: 'FirstRecordedCreditLine' and 'ListingCreationDate'.

#### Cleaning/Wrangling Issue #6
**Shrinking the data set**

##### Cleaning/Wrangling Issue #6.1
The main variable of interest, 'LoanStatus', has multiple values: 'Cancelled', 'Chargedoff', 'Completed', 'Current', 'Defaulted', 'FinalPaymentInProgress', 'PastDue'. However, in order to answer the research question about factors that determine default, we investigated only loans whose outcome is known (closed loans). In the current data set, closed loans are coded as 'cancelled', 'completed' (meaning repaid), 'defaulted' and 'charged-off'. There is a difference between defaulted and charged-off, however, for our purposes, both can be merged into one category, 'not repaid'. The cancelled loans are a very small group, which does not really contribute to the investigation, therefore, it can be dropped. 

##### Cleaning/Wrangling Issue #6.2
We **excluded** closed loans where the borrowers were rated according to the old 7-level rating ('CreditGrade'): 

  - the variable 'CreditGrade' contains only records made **until July 2009**, 
  - 'ProsperRating (Alpha)' and its "clone" 'ProsperRating (numeric)' both apply only to records made **since July 2009**
    
Therefore, we investigated **only scores from July 2009 on**, because this is when the **current rating system** was first implemented.

##### Cleaning/Wrangling Issue #6.3
The data set contains a lot of variables that we did not investigate, therefore, we saved a new dataframe with less variables.

##### Cleaning/Wrangling Issue #6.4
Additionally, we **removed** any rows containing nulls in the (independent) variables of interest.

#### Cleaning/Wrangling Issue #6.5 
Create dummy outcome variable with labels: 'Repaid' vs. 'Not repaid'.

#### Cleaning/Wrangling Issue #7
After shrinking the dataset to the records we want to analyze, the 'loan purpose' variable contains one empty category ('personal loan') and another one with very few records ('not available'). We removed both.


## Summary of Findings

Generally, a statistical model can be defined as a prediction model, an inference model, or a combination of the two. A prediction model is focused on how to best predict an outcome based on a combination of predictors without looking too much into the relationships among the predictors. An inference model, on the other hand, is also focused on the way predictors influence each other 
as well as on the influence of each separate predictor variable on the outcome 
[James, G. et al., 2021](https://www.amazon.com/Introduction-Statistical-Learning-Applications-Statistics-ebook/dp/B01IBM7790) <a name="statlearning"><sup>1</sup></a> 

Including all of the predictor variables in a regression model of the Prosper data proved to be difficult.

The first issue we encountered was the variety of non-normal distributions of the continuous variables. Furthermore, many of these variables were count data containing a lot of zeros.

The zeros together with multimodal and strongly skewed distributions (which were visibly not resembling a log-normal distribution)
precluded log-transformation. A possible solution would be to use a specific regression model for count data which is often encountered in life science <a name="countdata"><sup>2</sup></a>.

However, the aim of an exploratory study like this one is to gain insight into the relationships among variables. We have built neighter a prediction nor an inference model here. Instead, continuous (and discrete) variables were transformed into ordinal variables and the percentage of unrepaid loans was computed for each level of each variable. Plotting the percentage proved to be very illustrative and allowed us to classify the variables as possible strong or weak predictors of loan outcome status.

The transformation was performed either by using percentiles or equal intervals, depending on the distribution of the rough data. In addition, there are unordered categorical variables, such as loan purpose and homeownership which influence the ordered ones.   

Correlation measures such as Pearson or Spearman are not suitable for proving the relationship between loan outcome and predictors,
because of the assumption of linearity (Spearman uses ranked scores, but applies the same assumption to them). 

A chi-square test of independence proved that the hypothesis of independence of the outcome and most of the predictor variables can be rejected (i.e., most chi-square tests were statistically significant, except two: loan outcome vs. credit history lenght and loan outcome vs. employment status duration).

However, in order to make complete sense of these results, a correction of the p-value for multiple comparisons should be applied.
Due to the exploratory aim of this study, we refrained from applying such correction.

The following classification into 'weak' and 'strong' predictors is based only on inspection of plots. 

#### Strong predictors:
- interest rate (quadratic,  with a peak at approximately 30-32%)
- 7-level rating or 'Prosper Rating' (linear)
- 10-level rating or 'Prosper Score' (could be interpreted as linear, quadratic or cubic)
- customer credit rating by a professional credit rating agency (linear)
- loan term (with three categories, 12, 36, and 60 months, trend linear or exponential)
- income range (linear)
- sum of derogatory public records for the last 10 years (linear)
- sum of derogatory public records for the last 12 months (there is a difference between the two categories)
- debt-to-income ratio (could be interpreted as linear or cubic)

#### Weak predictors:
- loan amount (could be interpreted as quadratic, probably moderated by other variables such as purpose, income and debt-to-income ratio)
- employment status duration (default rate seems to grow with this variable, however the trend is not clear)
- homeownership (very small difference in default rate between the categories, possibly moderated by other variables)
- borrower credit history (default rate seems to grow from a certain value of this variable onward)
- number of current credit lines (one value is associated with a much higher default rate than the rest: zero)
- number of open credit lines (the same as current credit lines)
- sum of loans for the last 7 years (seemingly linear, but very weak)
- number of open revolving accounts (one value is associated with a much higher default rate than the rest: zero)
- open revolving monthly payment (same as open revolving accounts)
- revolving utilization or 'Bankcard Utilization' (two values are associated with a much higher default rate than the rest: 0% and 100% or higher)
- number of current delinquencies on loans (possibly quadratic with peak at 3-5 delinquencies)
- amount delinquent of loans (possibly quadratic with a peak in the category \\$352-3,040)
- sum of delinquencies for the last 7 years (no obvious trend)
- sum of inquiries by creditors for the last 6 months (one value is associated with a much higher default rate than the rest: 7 or more inquiries)
- total sum of inquiries by creditors (one value is associated with a much higher default rate than the rest: 20 or more inquiries) 

#### Unordered categorical variables:
- loan purpose (some categories have very low default rates, while others have much higher ones,
which is why we also investigated the combination between loan purpose and other predictors)


<sup>[1](#statlearning)</sup> [James, G., Witten, D., Hastie, T., & Tibshirani, R. (2021). An introduction to statistical learning. New York: Springer.](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0139427)

<sup>[2](#countdata)</sup> [How to Deal with Count Data](https://pubmed.ncbi.nlm.nih.gov/34104569/)


## Key Insights for Presentation

Strong predictors of default would be redundant in a prediction model, especially the different credit rating variables (all of which are strongly correlated to the interest rate, as theoretical models propose). Deciding which of them to include into a model would go beyond the score of this discussion. The 'weak' predictors, on the other hand, could have important influence as part of a regression model. A necessary preparatory step of such a model would be to decide on a suitable transformation method.

Unordered categorical variables, as, for instance, the loan purpose should also be included in a prediction model. Plotting the combinations of loan purpose with other variables showed that certain categories of loan purpose are clearly distinct in terms of default rate, interest rate, loan size, income and so forth. 