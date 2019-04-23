
# Coding the Naive Bayes Classifier From Scratch

This post will walk through the basics of the Naive Bayes Classifier as well as show a python implementation of coding it from the ground up. While Naive Bayes is a fairly simple and straightforward algorithm, it has a number of real world use cases, including the canonical spam detection as well as sentiment analysis and weather detection. This post will walk through an example using [UCI's Banknote Authentication Dataset](https://archive.ics.uci.edu/ml/datasets/banknote+authentication "UCI ML Repo").

## Banknote Dataset

First, let's get started with the basics and __read in the dataset:__


```python
import pandas as pd
import numpy as np
from sklearn.metrics import accuracy_score
from sklearn.naive_bayes import GaussianNB
```


```python

url = url = 'http://archive.ics.uci.edu/ml/machine-learning-databases/00267/data_banknote_authentication.txt'
cols = ['imgVariance','imgSkewness','imgCurtosis','imgEntropy','Class']
df = pd.read_csv(url, header=None, names=cols)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>imgVariance</th>
      <th>imgSkewness</th>
      <th>imgCurtosis</th>
      <th>imgEntropy</th>
      <th>Class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3.62160</td>
      <td>8.6661</td>
      <td>-2.8073</td>
      <td>-0.44699</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.54590</td>
      <td>8.1674</td>
      <td>-2.4586</td>
      <td>-1.46210</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3.86600</td>
      <td>-2.6383</td>
      <td>1.9242</td>
      <td>0.10645</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.45660</td>
      <td>9.5228</td>
      <td>-4.0112</td>
      <td>-3.59440</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.32924</td>
      <td>-4.4552</td>
      <td>4.5718</td>
      <td>-0.98880</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



The dataset consists of 1372 observations. There are four features which describe images of genuine and forged banknotes, as well as as label indicating whether or not the note is genuine. There are several flavors of the Naive Bayes Classifier each of which make their own assumptions about the data. Because our features are continuous and distributed approximately normally, this example will cover the __Gaussian__ Naive Bayes Classifier. But before diving into the specifics, let's __split our data into training and test sets:__


```python
np.random.seed(94110)
msk = np.random.rand(len(df)) < 0.8
train_df = df[msk]
test_df = df[~msk]
```

## Why is it Naive? Why is it Bayes(ian)?

The Naive Bayes Classifier is a supervised learning algorithm so given a set of datapoints {$${x^1,...x^m}$$} our goal is to predict the correct {$${y^1,...,y^m}$$}. However, unlike discriminative classifier such as logistic regressions or decision trees which directly estimate $$P(Y\mid X)$$ and create a decision boundaries to make predictions, the Naive Bayes Classifier is a __generative classifier__. It uses $$P(X\mid Y)$$ to then estimate $$P(Y\mid X)$$. And here is where good old Bayes Theorem (below) helps you out.

$$\displaystyle P(X\mid Y)={\frac {P(Y\mid X)P(X)}{P(Y)}}$$

### Getting from Bayes' Rule to the Naive Bayes Classifier

We can translate the Bayes' Rule into:

$$\displaystyle posteriorProbability = \frac {conditionalProbability * priorProbability} {marginalProbability}$$

For many cases, the denominator (aka the marginal probability) is impossible or near impossible to calculate. However, this doesn't matter for the purposes of the naive bayes classifier. Let's see why. Using Bayes' Rule as a classifier, our goal is going to be to maximize the posterior probability and predict whichever class has the maximum probability. Mathematically:

$$\displaystyle prediction \leftarrow \underset{C_i}{\operatorname{argmax}} P(C_i \mid f_1,f_2...,f_n)$$

Where:
+ $$C_i$$ represents the ith class
+ $$f_n$$ represents the nth feature vector

So here is where our "naive" assumption comes in. We assume independence between features, which allows us to calculate the conditional probability simply as the product of the individual probabilities of each feature:

$$\displaystyle P(f_1,f_2,...,f_n\mid C_i) = \prod_{j=1}^n P(f_j\mid C_i)$$

Applying this assumption to our banknote dataset, we get the following:

$$\displaystyle posterior(forged) = \frac{P(forged) P(imgVariance\mid forged) P(imgSkewness\mid forged) P(imgCurtosis\mid forged) P(imgEntropy\mid forged)}{marginalProb}$$

$$\displaystyle posterior(genuine) = \frac{P(genuine) P(imgVariance\mid genuine) P(imgSkewness\mid genuine) P(imgCurtosis\mid genuine) P(imgEntropy\mid genuine)}{marginalProb}$$


With the example, we can see clearly how the marginal probability cancels. But how exactly do we calculate the individual conditional probabilities? Here is where we apply our assumption of normality and use the probability density function for the normal distribution.

$$p(x) = \frac{1}{\sqrt{2\pi \sigma^2}} exp(-\frac{(x-\mu)^2}{2\sigma^2})$$

So to calculate $$P(imgEntropy\mid forged)$$ we would plug in the following:

$$P(imgEntropy\mid forged) = \frac{1}{\sqrt{2\pi * variance(imgEntropyInForgedData)}} exp(-\frac{(x-mean(imgEntropyInForgedData)}{2*variance(imgEntropyInForgedData)})$$

Where $$x$$ is the value of an individual observation.

So now we have all background we need write our code.

## Coding it Up!

In order to calculate our conditional probabilities, we need to __calculate the mean and variance for each feature in each class__, which we can then plug into the gaussian probability density function. First we'll separate our training dataframe by class, and then calculate the necessary parameters.


```python
#separate training df by class
pos_df = train_df[train_df['Class']==1]
neg_df = train_df[train_df['Class']==0]

#calculate class probabilities (priors)
prob_pos = pos_df.shape[0] / train_df.shape[0]
prob_neg = neg_df.shape[0] / train_df.shape[0]

#calculate means and variances
pos_class_means = pos_df.mean()
pos_class_means = pos_class_means.drop(index='Class') #dropping class label
pos_class_vars = pos_df.var()
pos_class_vars = pos_class_vars.drop(index='Class') #dropping class label
neg_class_means = neg_df.mean()
neg_class_means = neg_class_means.drop(index='Class')
neg_class_vars = neg_df.var()
neg_class_vars = neg_class_vars.drop(index='Class')

#Store info in dictionary
model_info = {'pos_means':pos_class_means,
				'pos_vars':pos_class_vars,
				'neg_means':neg_class_means,
				'neg_vars':neg_class_vars,
				'prob_pos':prob_pos,
				'prob_neg':prob_neg}

#Show example
pos_class_means
```




    imgVariance   -1.872307
    imgSkewness   -0.984491
    imgCurtosis    2.172635
    imgEntropy    -1.230872
    dtype: float64



Now we write a function that will calculate the condition probability of a given observation using the paramaters calculated from the data.


```python
def conditional_prob(a,b_mean, b_var):
	"""
	This function calculates p(a|b)
	Args:
	  a: A float representing a single datapoint
	  b_mean: A float
      b_var: A float
	Returns:
	  prob: Float representing a probability 
	"""
	prob = 1 / (np.sqrt(2 * np.pi * b_var)) * np.exp((-(a-b_mean)**2) / (2 * b_var))
	return prob
```

In order to make a prediction, we can write another function using the previously calculated parameters. 


```python
def predict_single_datapoint(row, model_info):
	"""
	Makes a prediction for one new row of data using the NB model params
	generated from training data.
	Args:
	  row: A row of data from a pandas df (does NOT include label)
	  model_info: paramters stored in dictionary
	Returns: Prediction (1 or 0)
	"""
    #create empty lists to store conditional probabilities for each class
	cond_probs_pos = []
	cond_probs_neg = []
    
    #loop through features and calculate conditional probabilities for each
    #feature and class
	for i in range(len(row)):
		a = row[i]
		b_mean_pos = model_info['pos_means'][i]
		b_mean_neg = model_info['neg_means'][i]
		b_var_pos = model_info['pos_vars'][i]
		b_var_neg = model_info['neg_vars'][i]
		cond_probs_pos.append(conditional_prob(a,b_mean_pos,b_var_pos))
		cond_probs_neg.append(conditional_prob(a,b_mean_neg,b_var_neg))
        
    #get product of conditional probabilities and weight by probability class
	cond_prob_pos = np.prod(cond_probs_pos) * model_info['prob_pos'] 
	cond_prob_neg = np.prod(cond_probs_neg) * model_info['prob_neg']
    
    #return class with larger product
	if cond_prob_pos > cond_prob_neg:
		return 1
	else:
		return 0
```

Let's check to see if this works on the first row of our test data:


```python
row = test_df.iloc[0]
row
```




    imgVariance    3.62160
    imgSkewness    8.66610
    imgCurtosis   -2.80730
    imgEntropy    -0.44699
    Class          0.00000
    Name: 0, dtype: float64



So we see that we are expecting the model to predict a '0' for this datapoint. Let's see what we actually get:


```python
row = row.drop('Class') #drop target
predict_single_datapoint(row,model_info)
```




    0



This row works as expected, but we want to check and see how the model does on the entire test set and compare our implementation to sklearn's implementation.

## How'd we do?

So we see that for one datapoint, we correctly predict that the data gets a negative label. We can efficiently use pandas to apply our function to the entire test set and evaluate how our classifier did.


```python
y_true = test_df['Class']
test_df = test_df.drop('Class',axis=1)
preds = test_df.apply(lambda row:predict_single_datapoint(row,model_info),axis=1) #get predictions for test set
accuracy_score(y_true,preds)
```




    0.8438661710037175



After making our predictions and comparing them to ground truth simply using accuracy score, we find that we have done reasonably well with this dataset. However, to double check that we've implemented this correctly, we can compare the implementation to the scikit-learn implementation.

### Compare to Scikit-learn Implementation


```python
gnb = GaussianNB()
gnb.fit(train_df[['imgVariance','imgSkewness','imgCurtosis','imgEntropy']].values,train_df['Class'])
sklearn_pred = gnb.predict(test_df[['imgVariance','imgSkewness','imgCurtosis','imgEntropy']])
accuracy_score(y_true,sklearn_pred)
```




    0.8438661710037175



Yay, it same!
