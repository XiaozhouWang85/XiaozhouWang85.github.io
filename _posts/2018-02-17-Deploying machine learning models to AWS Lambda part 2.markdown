---
layout: post
title:  "Deploying machine learning models to AWS Lambda (part 2)"
date:   2018-02-15 13:21:40 +0800
categories: datascience machinelearning AWSLambda Deployment
---


Before we can begin discussing how to implement a machine learning model, we need to build one. The model shown below is built using Naive Bayes which is commonly used in text based predictive challenges. Technically it is not a machine learning model but uses the same dependencies within Python so serves as a good stand-in.

The data and solution were both taken from Kaggle. Here is the [competition](https://www.kaggle.com/c/spooky-author-identification) and [solution](https://www.kaggle.com/sudalairajkumar/simple-feature-engg-notebook-spooky-author). This article will only provide a very brief walkthrough of how the model is built since it is assumed the reader already knows this. If this is not then case then Kaggle contains many tutorials and worked examples that is a great learning resource.

#### Building a model with NLTK, Scikit-learn and Pandas

The Kaggle challenge is based around the works of horror authors Edgar Allan Poe, HP Lovecraft and Mary Shelley. The task is to identify the author when given an excerpt take from one of their books.

The first step is to import the dependent libraries.


```python
#Import dependencies
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import itertools
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split, KFold
from sklearn import naive_bayes
from sklearn.metrics import confusion_matrix, auc, log_loss
```

Pandas is used to read the input data and convert it into a list. Stop words (common words in English such as "the" or "a") are removed and stemming is performed (converting running and ran into run). Then a count vectorizer is used to convert the words into a bag of words matrix.


```python
#Import data from csv and store as text in a list
train = pd.read_csv("train.csv")
le=LabelEncoder()
train_y =le.fit_transform(train['author'])
```


```python
cnt_vec = CountVectorizer(stop_words='english', ngram_range=(1,3))
cnt_vec.fit(train['text'])
train_vec = cnt_vec.transform(train['text'])
```

Once the data has been transformed into a matrix format, a kfolds approach is used train a Naive Bayes model. Kfolds splits the dataset into k equal sized parts, trains the model on k-1 of the parts and validates on the remaining part. The mean performance indicates predictiveness of the model and standard deviation is a measure of how stable the model is. A high standard deviation can indicate overfitting of the model.


```python
def runMNB(train_X, train_y, test_X, test_y):
    model = naive_bayes.MultinomialNB()
    model.fit(train_X, train_y)
    pred_test_y = model.predict_proba(test_X)
    return pred_test_y, model
```


```python
cv_scores = []
models=[]
pred_train = np.zeros([train.shape[0], 3])
kf = KFold(n_splits=5, shuffle=True, random_state=2017)
for dev_index, val_index in kf.split(train):
    dev_X, val_X = train_vec[dev_index], train_vec[val_index]
    dev_y, val_y = train_y[dev_index], train_y[val_index]
    pred_val_y,model = runMNB(dev_X, dev_y, val_X, val_y)
    pred_train[val_index,:] = pred_val_y
    cv_scores.append(log_loss(val_y, pred_val_y))
    models.append(model)
print("Mean cv score : ", np.mean(cv_scores))
print("SD cv score : ", np.std(cv_scores))

```

    Mean cv score :  0.45389257569874275
    SD cv score :  0.005560531509619231
    

Performing kfolds indicates the model performs well and model has very low standard deviation in performance. This is not surprising given that the Naive Bayes model is very simple and has almost no tunable parameters. The model is rebuilt using the whole dataset for training.


```python
clf = naive_bayes.MultinomialNB()
clf.fit(train_vec, train_y)
cm=confusion_matrix(train_y, clf.predict(train_vec))
```

Matplotlib is used to visualise the confusion matrix. Code for graphing a confusion matrix taken from this [example](http://scikit-learn.org/stable/auto_examples/model_selection/plot_confusion_matrix.html#sphx-glr-auto-examples-model-selection-plot-confusion-matrix-py ).


```python
def plot_confusion_matrix(cm, classes,
                          title='Confusion matrix',
                          cmap=plt.cm.Blues):
    """
    This function prints and plots the confusion matrix.
    """

    plt.imshow(cm, interpolation='nearest', cmap=cmap)
    plt.title(title)
    plt.colorbar()
    tick_marks = np.arange(len(classes))
    plt.xticks(tick_marks, classes, rotation=45)
    plt.yticks(tick_marks, classes)

    fmt = '.0f'
    thresh = cm.max() / 2.
    for i, j in itertools.product(range(cm.shape[0]), range(cm.shape[1])):
        plt.text(j, i, format(cm[i, j], fmt),
                 horizontalalignment="center",
                 color="white" if cm[i, j] > thresh else "black")

    plt.tight_layout()
    plt.ylabel('True label')
    plt.xlabel('Predicted label')
```


```python
plt.figure(figsize=(8,8))
plot_confusion_matrix(cm, classes=['EAP', 'HPL', 'MWS'],
                      title='Confusion matrix')
plt.show()
```

![Confusion Matrix]({{site.url}}/assets/spooky_cm.png){:class="img-responsive" width="50%"}

The results are fairly good, perhaps surprisingly so. By using count vectorizer with ngrams, vocabulary and style of phrasing will be captured. Therefore it is not surprising that this will be able to predict the author of the text.

#### Deploying the model normally

After completing the model build process a number of Scikit-learn objects were created. These objects can be persisted to disk using joblib. Once the objects have been saved, they can be loaded and re-used.


```python
#Persisting Sckitlearn objects to disk
from sklearn.externals import joblib
joblib.dump(cnt_vec, 'cnt_vectorizer.pkl')
joblib.dump(clf, 'clf.pkl')
joblib.dump(le, 'le.pkl')

cnt_vec = joblib.load('cnt_vectorizer.pkl') 
clf = joblib.load('clf.pkl')
le = joblib.load('le.pkl') 
```

The various objects can be linked together in a function that now takes text as input and returns a prediction

```python
def lda_pipeline(inlist):
    tf_ex=cnt_vec.transform(inlist)
    clf_ex=clf.predict(tf_ex)
    return list(le.inverse_transform(clf_ex))
lda_pipeline(train[:3])

['EAP', 'HPL', 'EAP']
```

This function can now be easily deployed using Flask or Django if deploying in a traditional way. The easiest tutorial I have seen on this is [here](https://impythonist.wordpress.com/2015/07/12/build-an-api-under-30-lines-of-code-with-python-and-flask/). This concludes part 2 of this series. In the next article, we will see how AWS Lambda can be used to turn a very simple function (not the model we developed here) into an API.
