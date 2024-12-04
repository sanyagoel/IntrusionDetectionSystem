# Intrusion Detection System Using Spark ML

## Overview

This project involves building a Machine Learning-based Intrusion Detection System (IDS) to classify network traffic data and identify potential attacks. The dataset used consists of labeled network traffic data with both binary and multi-class classification targets. We apply various preprocessing, transformation techniques, and machine learning models, including Random Forest, Decision Tree, Logistic Regression, and XGBoost, to achieve high classification accuracy.

### Dataset
The dataset is a network intrusion dataset containing 18,893,708 rows and 47 columns. It includes multiple types of attacks along with normal traffic ("benign"). The dataset features both continuous and categorical data, including IP addresses that need transformation for model applicability.

Dataset source:  
[Network Intrusion Dataset](https://staff.itee.uq.edu.au/marius/NIDS_datasets/)

## Problem Statement

### Problem 1: Reading the Dataset

Initially, reading the dataset with `inferSchema` set to `True` was very slow due to Spark’s need to manually infer the data types of each column. To speed up the process, we manually defined a schema and passed it to Spark, which significantly reduced the reading time.

### Problem 2: Classification Type

The target variable is represented by two columns:
1. **label**: Binary values (0 or 1).
2. **attack**: String representing the type of attack.

We decided to perform multi-class classification based on the 15 distinct values in the `attack` column, including "benign" (no attack).

### Problem 3: High Overfitting

An initial model resulted in perfect accuracy (1), indicating significant overfitting. This happened because the dataset was large, and the model was too complex.

### Problem 4: Long Preprocessing Time

The large dataset made basic preprocessing (like count, distinct) very time-consuming. We used stratified sampling to reduce the dataset size and ensure each class was well-represented.

## Data Preprocessing

### Sampling and Data Reduction

After examining the distribution of classes in the dataset (with "benign" having 16 million rows), we decided to apply stratified sampling to reduce the dataset. We adjusted the sampling fractions based on the class distribution.

After two iterations of sampling, we reduced the dataset to 111,271 rows, which allowed for more efficient processing.

### Missing Values

We checked for missing values using Spark’s Imputer class from `spark.ml.feature`. However, the dataset didn’t have missing values, so no further action was required.

### One-Hot Encoding

We applied one-hot encoding to the `attack` column since it had no ordinal relationship between categories. We used Spark’s `Pipeline` to apply the transformation efficiently.

### Outlier Detection

Using the Interquartile Range (IQR) method to remove outliers did not change the dataset, indicating that most values were within the acceptable range.

### Handling IP Address Columns

The `IPV4_SRC_ADDR` and `IPV4_DST_ADDR` columns were originally string-based and contained over 100,000 unique values. One-hot encoding these columns would lead to excessive dimensionality and overfitting.

We transformed these columns by mapping IP addresses to their corresponding regions using the `geojs` API. This reduced the number of unique values and prevented overfitting.

### Data Caching

Due to the repeated nature of some operations, we used Spark’s dataset caching feature to speed up preprocessing tasks.

### Shuffle Partitions Configuration

The initial configuration had only 50 shuffle partitions, leading to poor parallelism. After increasing the shuffle partitions to 150, the Spark session performed much more efficiently.

## Feature Transformation

### Principal Component Analysis (PCA)

We initially applied PCA for dimensionality reduction. However, the presence of high-dimensional one-hot encoded columns (`IPV4_SRC_ADDR` and `IPV4_DST_ADDR`) led to the curse of dimensionality, which caused PCA to have low explained variance (around 6%).

Switching from one-hot encoding to label encoding on these columns increased the explained variance to around 85%. We used the transformed features in our final models.

## Models and Results

### Models Used

We tested the following models on the preprocessed data:

1. **Random Forest Classifier**
2. **Decision Tree Classifier**
3. **Logistic Regression**
4. **XGBoost**

### Model Performance

The results on unseen/test data were as follows:

- **Random Forest**: 82.4% accuracy
- **Decision Tree**: 71.6% accuracy
- **Logistic Regression**: 87.02% accuracy
- **XGBoost**: 90% accuracy

## Challenges and Solutions

- **Slow Dataset Loading**: Reduced dataset reading time by manually defining a schema.
- **Overfitting**: Used sampling and dimensionality reduction techniques to handle overfitting.
- **Handling IP Address Columns**: Mapped IP addresses to regions to avoid high cardinality.
- **Preprocessing Time**: Used caching to speed up repeated operations and increased shuffle partitions to improve parallel processing.

## References

- [Network Intrusion Dataset](https://staff.itee.uq.edu.au/marius/NIDS_datasets/)
- [Geolocation API - geojs](https://geojs.io/)
- [XGBoost GitHub Issue](https://github.com/dmlc/xgboost/issues/3662)
- [Stack Overflow Discussion on IP Address Features](https://stackoverflow.com/questions/48259511/how-to-use-ip-address-as-a-feature-in-a-neural-network)
- [Medium Article on Spark Performance](https://medium.com/vedity/python-data-preprocessing-using-pyspark-cc3f709c3c23)



