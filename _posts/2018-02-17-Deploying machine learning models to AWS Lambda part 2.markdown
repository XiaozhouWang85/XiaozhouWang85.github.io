---
layout: post
title:  "Deploying machine learning models to AWS Lambda (part 2)"
date:   2018-02-15 13:21:40 +0800
categories: datascience machinelearning AWSLambda Deployment
---

Before we can begin discussing how to implement a machine learning model, we need to build one. The model shown below is a fairly simple one and has fairly low predictive power. However, it uses NLTK, Scikit-learn and Pandas so serves as a good example model to implement. The data and solution were both taken from Kaggle. Here is the [competition](https://www.kaggle.com/c/spooky-author-identification) and [solution](https://www.kaggle.com/arthurtok/spooky-nlp-and-topic-modelling-tutorial). This article will only provide a very brief walkthrough of how the model is built since it is assumed the reader already knows this. If this is not the case then Kaggle contains many tutorials and worked examples that are a great learning resource.

#### Building a model with NLTK, Scikit-learn and Pandas

The Kaggle challenge is based around the works of horror authors Edgar Allan Poe, HP Lovecraft and Mary Shelley. The task is to identify the author when given an excerpt taken from one of their books.

The first step is to import the dependent libraries.


```python
#Import dependencies
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import itertools
from nltk.stem import WordNetLemmatizer
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.decomposition import LatentDirichletAllocation
from sklearn.preprocessing import LabelEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.metrics import confusion_matrix
```

Pandas is used to read the input data and convert it into a list. Stop words (common words in English such as "the" or "a") are removed and stemming is performed (converting "running" and "ran" into "run"). Then a count vectorizer is used to convert the words into a bag of words matrix. 


```python
#Import data from csv and store as text in a list
train = pd.read_csv("train.csv")
text = list(train.text.values)

#Create a count vectorizer class that also applies stemming to 
#the words in each document
lemm = WordNetLemmatizer()
class LemmaCountVectorizer(CountVectorizer):
    def build_analyzer(self):
        analyzer = super(LemmaCountVectorizer, self).build_analyzer()
        return lambda doc: (lemm.lemmatize(w) for w in analyzer(doc))

#Create object and apply to text data
tf_vectorizer = LemmaCountVectorizer(
    max_df=0.95,min_df=2,stop_words='english', decode_error='ignore'
)
tf = tf_vectorizer.fit_transform(text)

#Conduct an LDA transformation on the data
lda = LatentDirichletAllocation(n_topics=50, max_iter=5,
                                learning_method = 'online',
                                learning_offset = 50.,
                                random_state = 0)
X=lda.fit_transform(tf)

#Convert author to integer
le=LabelEncoder()
Y=le.fit_transform(train['author'])
```

Once the data has been transformed into a matrix format, it is split into train and test groups. This is so that cross validation can be performed on the model once it is built. Random forest is used as the classifier and once fit to the data, it is used to make predictions on the data that had been set aside for testing. The accuracy scores and confusion matrix are produced.


```python
#Split data into test and train
train_x, test_x, train_y, test_y = train_test_split(
    X, Y,train_size=0.25
)
clf = RandomForestClassifier(n_jobs=200,min_samples_split=100,min_samples_leaf=100)
clf.fit(train_x, train_y)
pred=clf.predict(test_x)
train_acc=accuracy_score(train_y, clf.predict(train_x))
test_acc=accuracy_score(test_y, clf.predict(test_x))
cm=confusion_matrix(test_y, clf.predict(test_x))
```

Matplotlib is used to visualise the confusion matrix.


```python
#Print model results and confusion Matrix
print('Train: {0:.0f}%, Test: {1:.0f}%'.format(train_acc*100, test_acc*100))

plt.imshow(cm, interpolation='nearest', cmap=plt.cm.Blues)
classes=['EAP', 'HPL', 'MWS']
plt.title('Confusion Matrix')
plt.colorbar()
tick_marks = np.arange(len(classes))
plt.xticks(tick_marks, classes, rotation=45)
plt.yticks(tick_marks, classes)

fmt = 'd'
thresh = cm.max() / 2.
for i, j in itertools.product(range(cm.shape[0]), range(cm.shape[1])):
    plt.text(j, i, format(cm[i, j], fmt),
             horizontalalignment="center",
             color="white" if cm[i, j] > thresh else "black")

plt.tight_layout()
plt.ylabel('True label')
plt.xlabel('Predicted label')
plt.show()

Out: Train: 53%, Test: 50%
```

![Confusion Matrix]({{site.url}}/assets/spooky_cm.png)


The results are fairly poor. Edgar Allen Poe can be identified a bit more reliably but the other two authors are not identified well. However, this will not stop us from using this as an example model to implement.

#### Deploying the model normally

After completing the model build process a number of Scikit-learn objects were created. These objects can be persisted to disk using joblib. Once the objects have been saved, they can be loaded and re-used.


```python
#Persisting Sckitlearn objects to disk
from sklearn.externals import joblib
joblib.dump(tf_vectorizer, 'tf_vectorizer.pkl')
joblib.dump(lda, 'lda.pkl') 
joblib.dump(le, 'le.pkl') 
joblib.dump(clf, 'clf.pkl')

tf_vectorizer = joblib.load('tf_vectorizer.pkl') 
lda = joblib.load('lda.pkl') 
le = joblib.load('le.pkl') 
clf = joblib.load('clf.pkl') 
```

The various objects can be linked together in a function that now takes text as input and returns a prediction


```python
def lda_pipeline(inlist):
    tf_ex=tf_vectorizer.transform(inlist)
    lda_ex=lda.transform(tf_ex)
    clf_ex=clf.predict(lda_ex)
    return list(le.inverse_transform(clf_ex))
lda_pipeline(text[:3])

Out:['EAP', 'EAP', 'EAP']
```

This function can now be easily deployed using Flask or Django if deploying in a traditional way. The easiest tutorial I have seen on this is [here](https://impythonist.wordpress.com/2015/07/12/build-an-api-under-30-lines-of-code-with-python-and-flask/). This concludes part 2 of this series. In the next article, we will see how AWS Lambda can be used to turn a very simple function (not the model we developed here) into an API.
