# Goal
* we want look at a 10 seconds window of acc. data to predict whether someone is intoxicated (binary class)
    * TAC 0.08 is the legal limit for driving - above which is considered intoxicated

# Data

## Data Source
- [This link contains the data for download ](https://archive.ics.uci.edu/ml/datasets/Bar+Crawl%3A+Detecting+Heavy+Drinking)
    - Use the cleaned data (clean_tac)

## Data Files
* accelerameter data - all_accelerometer_data_pids_13.csv - not included in the data folder, go to [source](https://archive.ics.uci.edu/ml/datasets/Bar+Crawl%3A+Detecting+Heavy+Drinking) to download
* clean-tac data - 13 csv files inside the [data/clean_tac](/data/clean_tac) folder
* phone_type - [phone_types.csv](/data/phone_types.csv) - contain info on which participant use iphone vs android
* pids - [pids.txt](/data/pids.txt) - a file listing the participant codes

## Output Files
* merged dataset with both accelerator data with new features and the tacs reading - file size too large so splitted into 2 files:
    * [merged_df1.csv](/data/output_files/merged_df1.csv)
    * [merged_df2.csv](/data/output_files/merged_df2.csv)
* frequency domain features for accerlerator data - [result_freq.csv](/data/output_files/result_freq.csv)
* time domain features for accerlerator data - [result_time.csv](/data/output_files/result_time.csv)


# Jupyter notebooks (contains all the codes, graphs, and models)
* Important notebooks:
    1. Start with [Bar_Crawl_import_initialEDA_featureEngineering.ipynb](/Bar_Crawl_import_initialEDA_featureEngineering.ipynb) - contains all graphs and code used to clean, preprocesse, perform initial EDA, engineer features, and produce the merged dataset used for model building
    2. Go to [Bar_Crawl_finaldataEDA_modelTraining.ipynb](/Bar_Crawl_finaldataEDA_modelTraining.ipynb)-  contains graphs and codes for cleaning up the merged dataset, model building, model selection, and final summary
        * all the models are trained here
* Extras:
    * [splitting_df_into_smaller_files.ipynb](/splitting_df_into_smaller_files.ipynb) - contains only the code use for splitting files into smaller sections

# Apporach to building the model
1. Check for missing data (don't expect any) and then link the accelerometer data from the 13 participants to their TAC readings (merge data)
2. Do EDA - looking at correlation, distribution, etc
3. feature engineering
    * basic summary statistics from time domain
    * some features from frequency domain
4. Pre-process data for modeling - train/test split
5. Build a model
    * Looks like a basic binary classificaiton problem -> TAC above 0.08 or TAC below 0.08
    * basically want to predict: intoxicated / not-intoxicated
5. Try different models, hyperparamter tuning
6. Summarize

# Summary

## Features
* for calculating featuers, first separated by participant id, then calculated the features for a 10 seconds window
* each 10 seconds window for each participant becomes one row, column is the individual features

### features engineered:
* time domain features
    * max
    * min
    * mean
    * meidan
    * variance
    * kurtosis
    * skewness
    * interval
    * Root Mean Square (RMS)
* frequency domain features
    * zero crossings
    * spectral entropy
    * spectral density
    * spectral centroids
    * mfcc factors
    * spectral rolloffs (did not use)

### other potential features that are not engineered:
* gait-like features
    * number of steps
    * step strength
* frequency domain features
    * fft max frequency
    * fft min frequency

## Data Preprocessing / Cleaning / Merging

### data preprocessing
* No missing data as indicated
* Accelerator time is converted from unix timestamp to real time based on ms
* TAC timestamp is converted from unix timestamp to real time based on s
* Min-Max Scaling for the accelerator data for each participant before going through the 10s resampling
* For the TAC data:
    * above or equal to 0.08 is set as intoxicated (case 1)
    * lower than 0.08 is set as NOT intoxicated (case 0)
        
## data merging
1. To merge the accerlerator 10s window data with the TAC reading data (every 30min), I set:
    * TAC start_time = TAC timestamp
    * TAC end_time = TAC timestamp + 30 minutes
2. If the 10s acc window falls within the 30min of the TAC, then get the TAC reading information

## Model Selection
1. Holdout 20% of the data as test set
2. Compare bewteen Logistic Regression, SVM, Decision Tree and Random Forest
    * Decision Tree and Random Forest preforms the best but Decision Tree is simplier and easier to interpret
        * They are overfitting with a training accuracy of 100%
3. Hyper-parameter tuning for the Decision Tree model
    * max_depths
    * minimum samples per leaf
4. Final test accuracy and f1-score
    * Training set:
        * 96.1% accuracy on training
    * Cross Validation on Training Set:
        * 68.5% accuracy with 5-fold cross-validation
        * 0.513 f1-score (out of 1)
    * Test set:
        * 93.2% accuracy on test set
        * 0.896 f1-score (out of 1)

## Other things to think about
* How to deploy the model?
    - Want to be able to input a accerlerator and return prediction
* What features to eliminate / add?
    * There are some features that are highly correlated with each other (multicollineary) that can be eliminated
        * Less of a problem for decision tree since it will randomly pick one as the important feature, but still should get rid of some
        * Can use Recursive Feature Elimination to eliminate some features
    * There are some features that can be added see above section - other potential features that are not engineered

# Result
* Decision Tree model with 78 features
* Binary classificaiton - intoxicated (case 1) or not intoxicated (case 0)
    * We have 67% not intoxicated out of 137333 rows (10s window)
* Final test set accuracy and f1-score
    * **93.2% accuracy on test set**
    * **0.896 f1-score (out of 1)**

