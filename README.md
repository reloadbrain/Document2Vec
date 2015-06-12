# Document2Vec
Finding document vectors from pre-trained word2vec word vectors

![Build Status](https://api.travis-ci.org/cemoody/Document2Vec.svg)
![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)

# How to install
Simply install from the git repo like so:

```bash
pip install -e git+git://github.com/cemoody/Document2Vec.git#egg=Package
# on a shared machine without system-python access add --user
```

# How to use
The word2vec file must be a trained gensim Word2Vec file and cannot be Mikolov's
pre-trained vectors. This is because training a new document vector requires
the syn1 layer which the C version of word2vec throws away.

Initialize Document2Vec with pre-trained word vectors from a pre-existing
word2vec training run like so:

```python    
from document2vec.document2vec import Document2Vec
from document2vec.corpora import SeriesCorpus
import pandas as pd
# This must be a gensim Word2Vec or Doc2Vec pickle
d2v = Document2Vec("/home/moody/projects/Parachute/data/data-all-02.py2")
sentences = pd.Series(['i love jackets', 'blue is my favorite color'])
corpus = SeriesCorpus(sentences)
doc_vectors = d2v.transform(corpus)
```

And then semantic similarities can be evaluated directly:

```python
from scipy.spatial.distance import cosine
# vector for 'i love jackets'
v0 = doc_vectors[0, :] 
# vector for 'coat'
v1 = d2v['coat']
similarity = 1 - cosine(v0, v1)
print(similarity) # 0.84
# Note that sentence vector and the coat vector are very similar
# even though the sentence did not literally contain the word `coat`
v2 = d2v['jacket']
similarity = 1 - cosine(v0, v2)
print(similarity) # 0.87
# Of course, the similarity with a word that is included
# is slightly higher
```

# Monitoring training

It can be useful to monitor the training over many iteration
to make sure doc2vec is at (least locally) doing what it should be doing:

```python
from scipy.spatial.distance import cosine
def monitor(model):
    print model.alpha
    for word in ['jackets', 'jacket', 'coats', 'dog']:
        print 1.0 - cosine(model['SENT_0'], model[word]),
    print " "
model.monitor = monitor
doc_vectors = d2v.transform(corpus)
```
 
