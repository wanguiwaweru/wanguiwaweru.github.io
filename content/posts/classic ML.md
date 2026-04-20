---
title: "Classic ML"
date: "2024-01-02"
draft: true
---

# Classic ML
Classic machine learning (ML) are traditional algorithms and techniques used for predictive modeling before the rise of deep learning. These methods are often based on statistical principles and are designed to work with structured data. Some common classic ML algorithms include:
- Linear Regression
- Logistic Regression
- Decision Trees
- Random Forests
- Support Vector Machines (SVM)
- K-Means Clustering

## Linear Regression
Linear regression is an algorithm used for predicting a continuous target variable based on one or more input variables. It assumes a linear relationship between the input variables and the target variable so that the output can be expressed as a linear combination of the input variables. 

In a simple linear regression with one input variable, the model can be expressed as:
```
y = β0 + β1x + ε
```
Where: 
- `y` is the target variable ie the dependent variable (the value to predict)
- `x` is the input variable ie the independent variable (the feature used for prediction)
- `β0` is the intercept (the value of `y` when `x` is 0)
- `β1` is the slope (the change in `y` for a one-unit change in `x`)
- `ε` is the error term (the difference between the observed and predicted values)

Example of linear regression model to predict the price of a house based on its size using scikit-learn:

```python

import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# Sample data
prices = np.array([150000, 200000, 250000, 300000, 350000])
sizes = np.array([1000, 1500, 2000, 2500, 3000])


# Create and fit the model
model = LinearRegression()
model.fit(X, y)
```

Multiple linear regression is an extension of simple linear regression that allows for multiple input variables. The model can be expressed as:
```
y = β0 + β1x1 + β2x2 + ... + βnxn + ε
```
Where:
- `y` is the target variable
- `x1, x2, ..., xn` are the input variables
- `β0, β1, β2, ..., βn` are the coefficients (parameters) to be estimated
- `ε` is the error term

Example of multiple linear regression model to predict the price of a house based on its size and number of bedrooms using scikit-learn:

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
# Sample data
prices = np.array([150000, 200000, 250000, 300000, 350000])
sizes = np.array([1000, 1500, 2000, 2500, 3000])
bedrooms = np.array([2, 3, 4, 5, 6])
# Create feature matrix
X = np.column_stack((sizes, bedrooms))
# Create and fit the model
model = LinearRegression()
model.fit(X, prices)
```
In both examples, we create a linear regression model using scikit-learn's `LinearRegression` class. We fit the model to the data using the `fit` method, which estimates the coefficients (parameters) of the linear equation that best fits the data. After fitting the model, we can use it to make predictions on new data by calling the `predict` method with new input values.


## Logistic Regression
Logistic regression is a classification algorithm used to predict the probability of a binary outcome (0 or 1) based on one or more input variables. It models the relationship between the input variables and the log-odds of the target variable being in a particular class. 

The logistic regression model can be expressed as:
```
log(p/(1-p)) = β0 + β1x1 + β2x2 + ... + βnxn
```
Where:
- `p` is the probability of the target variable being in class 1
- `x1, x2, ..., xn` are the input variables
- `β0, β1, β2, ..., βn` are the coefficients (parameters) to be estimated
Example of logistic regression model to predict whether a student will pass or fail based on their study hours and attendance using scikit-learn:

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
# Sample data
study_hours = np.array([1, 2, 3, 4, 5])
attendance = np.array([0, 1, 1, 0, 1])
pass_fail = np.array([0, 0, 1, 0, 1])
# Create feature matrix 
X = np.column_stack((study_hours, attendance))
# Create and fit the model
model = LogisticRegression()
model.fit(X, pass_fail)
```
In this example, we create a logistic regression model using scikit-learn's `LogisticRegression` class. We fit the model to the data using the `fit` method, which estimates the coefficients (parameters) of the logistic equation that best fits the data. After fitting the model, we can use it to make predictions on new data by calling the `predict` method with new input values.


## Decision Trees
Decision trees are a type of supervised learning algorithm used for both classification and regression tasks. They work by recursively splitting the data into subsets based on the values of the input features, creating a tree-like structure where each internal node represents a decision based on a feature, and each leaf node represents a class label (for classification) or a continuous value (for regression).

The decision tree algorithm can be expressed as:
```
if feature1 <= threshold1:
    if feature2 <= threshold2:
        return class1
    else:
        return class2
else:
    if feature3 <= threshold3:
        return class3
    else:
        return class4
``` 
Example of decision tree model to predict whether a customer will churn based on their age and monthly spending using scikit-learn:

```python
import numpy as np
from sklearn.tree import DecisionTreeClassifier
# Sample data
age = np.array([25, 30, 35, 40, 45])
monthly_spending = np.array([100, 200, 300, 400, 500])
churn = np.array([0, 0, 1, 1, 1
# Create feature matrix
X = np.column_stack((age, monthly_spending))
# Create and fit the model
model = DecisionTreeClassifier()
model.fit(X, churn)
```
In this example, we create a decision tree classifier using scikit-learn's `DecisionTreeClassifier` class. We fit the model to the data using the `fit` method, which builds the decision tree based on the input features and target variable. After fitting the model, we can use it to make predictions on new data by calling the `predict` method with new input values.

### Boosting Methods
Boosting is an ensemble learning technique that combines multiple weak learners (models that perform slightly better than random guessing) to create a strong learner (a model that performs significantly better than random guessing). The idea behind boosting is to sequentially train weak learners on the data, where each subsequent learner focuses on the mistakes made by the previous learners. This iterative process helps to improve the overall performance of the model.
The boosting algorithm can be expressed as:
```


## Random Forests
Random forests are an ensemble learning method that combines multiple decision trees to improve the accuracy and robustness of predictions. Each tree in the random forest is trained on a random subset of the data and a random subset of the features, which helps to reduce overfitting and increase generalization.
The random forest algorithm can be expressed as:

```
for i in range(n_trees):
    # Create a random subset of the data
    X_subset, y_subset = bootstrap_sample(X, y)
    # Create a random subset of the features
    features_subset = random_subset(features)
    # Train a decision tree on the subset of data and features
    tree = DecisionTreeClassifier()
    tree.fit(X_subset[:, features_subset], y_subset)
    # Add the tree to the forest
    forest.append(tree)
```
Example of random forest model to predict whether a customer will churn based on their age and monthly spending using scikit-learn:

```python
import numpy as np
from sklearn.ensemble import RandomForestClassifier
# Sample data
age = np.array([25, 30, 35, 40, 45])
monthly_spending = np.array([100, 200, 300, 400, 500])
churn = np.array([0, 0, 1, 1, 1])
# Create feature matrix
X = np.column_stack((age, monthly_spending))
# Create and fit the model
model = RandomForestClassifier(n_estimators=100)
model.fit(X, churn)
```

In this example, we create a random forest classifier using scikit-learn's `RandomForestClassifier` class. We specify the number of trees in the forest using the `n_estimators` parameter. We fit the model to the data using the `fit` method, which builds multiple decision trees based on random subsets of the data and features. After fitting the model, we can use it to make predictions on new data by calling the `predict` method with new input values.

## Support Vector Machines (SVM)
Support Vector Machines (SVM) are a type of supervised learning algorithm used for classification and regression tasks. SVMs work by finding the hyperplane that best separates the classes in the feature space. The hyperplane is chosen to maximize the margin between the classes, which helps to improve generalization and reduce overfitting.
The SVM algorithm can be expressed as:
```maximize margin
subject to:
    w * x_i + b >= 1 for y_i = 1
    w * x_i + b <= -1 for y_i = -1
```
Where:
- `w` is the weight vector that defines the hyperplane
- `b` is the bias term that shifts the hyperplane
- `x_i` is the input feature vector for the i-th training example
- `y_i` is the class label for the i-th training example (1 for positive class, -1 for negative class)
Example of SVM model to predict whether a customer will churn based on their age and monthly spending using scikit-learn:

```python
import numpy as np
from sklearn.svm import SVC

# Sample data
age = np.array([25, 30, 35, 40, 45])
monthly_spending = np.array([100, 200, 300, 400, 500])
churn = np.array([0, 0, 1, 1, 1])
# Create feature matrix
X = np.column_stack((age, monthly_spending))
# Create and fit the model
model = SVC(kernel='linear')
model.fit(X, churn)
```

In this example, we create a support vector machine classifier using scikit-learn's `SVC` class. We specify the kernel type as 'linear' to use a linear SVM. We fit the model to the data using the `fit` method, which finds the optimal hyperplane that separates the classes in the feature space. After fitting the model, we can use it to make predictions on new data by calling the `predict` method with new input values.
## K-Means Clustering
K-Means clustering is an unsupervised learning algorithm used to partition a dataset into K distinct clusters based on the similarity of the data points. The algorithm works by iteratively assigning each data point to the nearest cluster center and then updating the cluster centers based on the mean of the assigned points.The K-Means algorithm can be expressed as:
```
1. Initialize K cluster centers randomly
2. Repeat until convergence:
    a. Assign each data point to the nearest cluster center
    b. Update the cluster centers by calculating the mean of the assigned points
```

Example of K-Means clustering to group customers based on their age and monthly spending using scikit-learn:

```python
import numpy as np
from sklearn.cluster import KMeans
# Sample data
age = np.array([25, 30, 35, 40, 45])
monthly_spending = np.array([100, 200, 300, 400, 500])
# Create feature matrix
X = np.column_stack((age, monthly_spending))
# Create and fit the model  
model = KMeans(n_clusters=2)
model.fit(X)
```
In this example, we create a K-Means clustering model using scikit-learn's `KMeans` class. We specify the number of clusters as 2 using the `n_clusters` parameter. We fit the model to the data using the `fit` method, which partitions the data into 2 clusters based on the similarity of the age and monthly spending features. After fitting the model, we can access the cluster centers using the `cluster_centers_` attribute and the assigned cluster labels for each data point using the `labels_` attribute.



## Resources
- [Master the Machine: A Practical Guide to Every Major Machine Learning Algorithm](https://medium.com/@nomannayeem/master-the-machine-a-practical-guide-to-every-major-machine-learning-algorithm-88d2376c2aa8)
- [Mastering Decision Tree Models for Smarter Decisions](https://medium.com/@sachinsoni600517/mastering-decision-tree-models-for-smarter-decisions-451ef8b75836)
- [Mastering Random Forests: A Comprehensive Guide to Ensemble Learning](https://medium.com/@sachinsoni600517/mastering-random-forests-a-comprehensive-guide-to-ensemble-learning-9c8e5b1a2c3e)
