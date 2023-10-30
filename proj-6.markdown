---
layout: page
title: J/psi production project
permalink: /Jpsi-production-project/model-development
---

## Model Development with scikit-learn
[< Prev](proj-5.markdown) | [Next >](proj-7.markdown)

### Model Choice: Random Forest Regressor

For this project, we've chosen the `RandomForestRegressor` from the Scikit-learn library. The Random Forest algorithm is an ensemble learning method, which combines multiple decision trees to produce a more accurate and robust model. It is well-suited for our dataset because:

- It can handle high-dimensional data efficiently.
- It provides a natural resistance to overfitting.
- It offers insights into feature importance, allowing for a better understanding of our data.

### Model Training

The dataset was split into training and test sets, with 80% of the data used for training and 20% reserved for testing.

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

The model parameters were initialized as follows:

- number of estimators (trees): 100
- maximum depth of the tree: 10

These parameters were chosen to balance the accuracy and computational efficiency.

The model was then trained on the data.

```python
rf = RandomForestRegressor(n_estimators=100, max_depth=10, random_state=42)
rf.fit(X_train, y_train.values.ravel())
```

### Model Evaluation

Model performance was evaluated using the mean squared error (MSE) metric for both training and test data.

```
Training MSE: 0.0005671118252182426
Test MSE: 0.0005891680348386154
```

### Feature Importance

One of the significant advantages of using Random Forest is its ability to provide insights into which features are more influential in making predictions.  Below are the feature importances:

```
Importance of x: 0.6807263601754133
Importance of z: 0.006080546460394571
Importance of Q: 0.19588435868521697
Importance of PT: 0.11730873467897507
```

This feature importance analysis can guide further feature engineering or even experimental design, shedding light on which phase space variables play a critical role in determining cross sections.