Sentiment Analysis
==================

Sentiment analysis is a broad term for a set of tasks with the purpose of identifying an emotion or opinion in a text.

In this repository we provide an overview of open sentiment analysis models and dataset for Danish. 

| Model                                                        | Model type    | License                                                      | Trained by                                                | Dimension          | Tags                                                         | DaNLP |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ | --------------------------------------------------------- | ------------------ | ------------------------------------------------------------ | ----- |
| [AFINN](#afinn) | Wordlist | [Apache 2.0](https://github.com/fnielsen/afinn/blob/master/LICENSE) | Finn Årup Nielsen                                         | Polarity           | Score (integers)                                             | ❌     |
| [Sentida](#sentida) | Wordlist | [GPL-3.0](https://github.com/esbenkc/emma/blob/master/LICENSE) | Jacob Dalsgaard, Lars Kjartan Svenden og Gustav Lauridsen | Polarity           | Score (continuous)                                           | ❌     |
| [BERT Emotion](#bert-emotion) | BERT     | CC-BY_4.0                                                    | Alexandra Institute                                       | Emotions           | glæde/sindsro, forventning/interesse, tillid/accept,  overraskelse/forundring, vrede/irritation, foragt/modvilje, sorg/skuffelse, frygt/bekymring, No emotion | ✔     |
| [BERT Tone](#bert-tone) (beta) | BERT     | CC-BY_4.0                                                    | Alexandra Institute                                       | Polarity, Analytic | ['postive', 'neutral', 'negative'] and ['subjective', 'objective] | ✔     |
| [SpaCy Sentiment](#spacy-sentiment) (beta) | spaCy    | MIT                                                          | Alexandra Institute                                       | Polarity           | 'postive', 'neutral', 'negative'                             | ✔     |
| [Senda](#senda) | BERT | MIT | Ekstra Bladet | Polarity | 'postive', 'neutral', 'negative'  | ❌ | 

### Use cases 

Sentiment analysis is a very useful tool for a company which wants to gain insight about its brand or products as it can be used, for example, for : 

 * monitoring social media (e.g. quickly identifying posts that generate strong emotions)
 * monitoring brand and managing reputation (e.g. analysing the global sentiment of people tweeting about a brand or a product)
 * customer support (e.g. identifying happy or angry customers for better adapted responses)
 * customers or employees feedback (e.g. analysing the global output of a satisfaction survey)

## Models 

### AFINN

The [AFINN](https://github.com/fnielsen/afinn) tool [(Nielsen 2011)](https://arxiv.org/abs/1103.2903) uses a lexicon based approach for sentiment analysis.
The tool scores texts with an integer where scores <0 are negative, =0 are neutral and >0 are positive. 


### Sentida
The tool Sentida  [(Lauridsen et al. 2019)](https://tidsskrift.dk/lwo/article/view/115711)
uses a lexicon based approach to sentiment analysis. The tool scores texts with a continuous value. There exist both an R version and an implementation in Python.  In these documentations we evaluate the python version from [sentida](https://github.com/guscode/sentida). 

### 🔧 BERT Emotion {#bert-emotion}

The emotion classifier has been developed in collaboration with Danmarks Radio, which has granted access to a set of social media data. 
The data has been manually annotated for 2 tasks:  

* to detect whether there is emotion or not in a text (binary classification)
* to classify the text among 8 emotions (`Glæde/Sindsro`, `Tillid/Accept`, `Forventning/Interrese`, `Overasket/Målløs`, `Vrede/Irritation`, `Foragt/Modvilje`, `Sorg/trist`, `Frygt/Bekymret`).

The BERT [(Devlin et al. 2019)](https://www.aclweb.org/anthology/N19-1423/) emotion model(s) have been finetuned on this data using the [Transformers](https://github.com/huggingface/transformers) library from HuggingFace, and it is based on a pretrained  [Danish BERT](https://github.com/botxo/nordic_bert) representations by BotXO. 
The model classifying amongst eight emotions achieves an accuracy on 0.65 and a macro-f1 on 0.64 on the social media test set from DR's Facebook containing 999 examples (we do not have permission to distributing the data). 

You can load the models (as one) through the load_bert_emotion_model. 
Or you can find the models on our HuggingFace page: [detection of emotion](https://huggingface.co/DaNLP/da-bert-emotion-binary); [classification of emotion](https://huggingface.co/DaNLP/da-bert-emotion-classification).

 Below is a small snippet for getting started using the BERT Emotion models. 
 Please note that the BERT model can maximum take 512 tokens as input, however the code allows for overfloating tokens and will therefore not give an error but just a warning. 

```python
from danlp.models import load_bert_emotion_model
classifier = load_bert_emotion_model()

# using the classifier
classifier.predict('der er et træ i haven')
''''No emotion''''
classifier.predict('jeg ejer en rød bil og det er en god bil')
''''Tillid/Accept''''
classifier.predict('jeg ejer en rød bil men den er gået i stykker')
''''Sorg/trist''''

# Get probabilities and matching classes names
classifier.predict_proba('jeg ejer en rød bil men den er gået i stykker', no_emotion=False)
classifier._classes()
```


### 🔧 BERT Tone {#bert-tone}

The tone analyzer consists of two BERT [(Devlin et al. 2019)](https://www.aclweb.org/anthology/N19-1423/) classification models:

* a polarity detection model (classifying between `positive`, `neutral` and `negative`);
* a subjective/objective classification model. 

Both models have been finetuned on annotated twitter data using the [Transformers](https://github.com/huggingface/transformers) library from HuggingFace, and it is based on a pretrained  [Danish BERT](https://github.com/botxo/nordic_bert) representations by BotXO. 
The data used for training is manually annotated data from [Twitter Sentiment](../datasets.md#twitter-sentiment) (train part) and [EuroParl sentiment 2](../datasets.md#europarl-sentiment2)), both datasets can be loaded with the DaNLP package.  

You can load the models (as one) through the load_bert_tone_model method of DaNLP.
Or you can find the models on our HuggingFace page: [polarity detection](https://huggingface.co/DaNLP/da-bert-tone-sentiment-polarity); [subjective/objective classification](https://huggingface.co/DaNLP/da-bert-tone-subjective-objective).

Below is a small snippet for getting started using the BERT Tone models. 
Please note that the BERT model can maximum take 512 tokens as input, however the code allows for overfloating tokens and will therefore not give an error but just a warning. 

```python
from danlp.models import load_bert_tone_model
classifier = load_bert_tone_model()

# using the classifier
classifier.predict('Analysen viser, at økonomien bliver forfærdelig dårlig')
'''{'analytic': 'objektive', 'polarity': 'negative'}''' 
classifier.predict('Jeg tror alligvel, det bliver godt')
'''{'analytic': 'subjektive', 'polarity': 'positive'}'''

# Get probabilities and matching classes names
classifier.predict_proba('Analysen viser, at økonomien bliver forfærdelig dårlig')
classifier._classes()
```


### 🔧 SpaCy Sentiment {#spacy-sentiment}

SpaCy sentiment is a text classification model trained using spacy built in command line interface. It uses the CoNLL2017 word vectors (read about it [here](embeddings.md)).

The model is trained using hard distil of the [BERT Tone](#bert-tone) (beta) - Meaning,  the BERT Tone model is used to make predictions on 50.000 sentences from Twitter and 50.000 sentences from [Europarl7](http://www.statmt.org/europarl/). These data is then used to trained a spacy model. Notice the dataset has first been balanced between the classes by oversampling. The model recognizes the classes: 'positiv', 'neutral' and 'negative'.

It is a first version. 

Read more about using the Danish spaCy model [here](../frameworks/spacy.md).

Below is a small snippet for getting started using the spaCy sentiment model. Currently the danlp packages provide both a spaCy model which do not provide any classes in the textcat module (so it is empty for you to train from scratch), and the sentiment spacy model which have pretrained the classes 'positiv', 'neutral' and 'negative'. Notice it is possible with the spacy command line interface to continue training of the sentiment classes, or add new tags. 

```python
from danlp.models import load_spacy_model
import operator 

# load the model
nlp = load_spacy_model(textcat='sentiment') # if you got an error saying da.vectors not found, try setting vectorError=True - it is an temp fix

# use the model for prediction
doc = nlp("Vi er glade for spacy!")
max(doc.cats.items(), key=operator.itemgetter(1))[0]
'''positiv'''
```

### Senda

[Senda](https://github.com/ebanalyse/senda) is a python package developed by Ekstra Bladet which helps with fine-tuning transformers for text classification.
A model trained on [DaNLP's sentiment datasets](../datasets.md) is available through [HuggingFace](https://huggingface.co/pin/senda). 



## 📈 Benchmarks  

### Benchmark of polarity classification

The benchmark is made by converting the relevant models scores and relevant datasets scores 
into the there classes 'positive', 'neutral' and 'negative'.

The tools are benchmarked on the following datasets:

- [LCC Sentiment](../datasets.md#lcc-sentiment) contains 499 sentences from the proceedings of the European Parliament annotated with a sentiment score from -5 to 5 by Finn Årup Nielsen.
- [Europarl Sentiment](../datasets.md#europarl-sentiment1) contains 184 sentences from news and web pages annotated with sentiment -5 to 5 by Finn Årup Nielsen.
- [Twitter Sentiment](../datasets.md#twitter-sentiment) contains annotations for polarity (positive, neutral, negative) and annotations for analytic (subjective, objective) made by Alexandra Institute. 512 examples of the dataset are defined for evaluation. 

A conversion of the scores of the LCC and Europarl Sentiment dataset and the Afinn model is done in the following way: a score of zero to be "neutral", a positive score to be "positive" and a negative score to be "negative". 

A conversion of the continuous scores of the Sentida tool into three classes is not given since the 'neutral' class  can not be assumed to be only exactly zero but instead we assume it to be an area around zero.  We looked for a threshold to see how closed to zero a score should be to be interpreted as neutral.   A symmetric threshold is found by optimizing the macro-f1 score on a twitter sentiment corpus (with 1327 examples (the corpus is under construction and will be released later on)) . The threshold is found to be 0.4, which makes our chosen conversion to be:  scores over 0.4 to be 'positive', under -0.4 to be 'negative' and scores between to be neutral. However note, the original idea of the tools was not to convert into three  class problem. 

The scripts for the benchmarks can be found [here](https://github.com/alexandrainst/danlp/blob/master/examples/benchmarks/). There is one for the europarl sentiment and LCC sentiment data and another one for the twitter sentiment. This is due to the fact that downloading the twitter data requires login to a twitter API account. The scores below for the twitter data is reported for all the data, but if tweets are deleted in the mean time on twitter, not all tweets can be downloaded. 
In the table we consider the accuracy and macro-f1 in brackets, but to get the scores per class we refer to our benchmark script.

| Tool/Model                          | Europarl Sentiment | LCC Sentiment   | Twitter Sentiment (Polarity) | Sentences per second (CPU*) |
|-------------------------------------|--------------------|-----------------|------------------------------|-----------------------------|
| AFINN                               | 0.68 (0.68)        | 0.66 (0.61)     | 0.48 (0.46)                  |~1707                        |
| Sentida (version 0.5.0)             | 0.67 (0.65)        | 0.58 (0.55)     | 0.44 (0.44)                  |~751                         |
| BERT Tone (polarity, version 0.0.1) | **0.79** (0.78)    | **0.74** (0.67) | 0.73 (0.70)                  |~5                           |
| spaCy sentiment (version 0.0.1)     | 0.74 (0.73)        | 0.66 (0.61)     | 0.66 (0.60)                  |~139                         |
| Senda                               | 0.75 (0.74)        | 0.68 (0.59)     | **0.76** (0.73)              |~5                           |

*Sentences per second is based on a Macbook Pro with Apple M1 chip.

### Benchmark of subjective versus objective classification

The data for benchmark is: 

- [Twitter Sentiment](../datasets.md#twitter-sentiment) contains annotations for polarity (positive, neutral, negative) and annotations for analytic (subjective, objective) made by Alexandra Institute. 512 examples of the dataset are defined for evaluation. 

The script for the benchmarks can be found [here](https://github.com/alexandrainst/danlp/blob/master/examples/benchmarks/) and it provides more detailed scores. Below is accuracy and macro-f1 reported:

| Model                               | Twitter sentiment (analytic) |
| ----------------------------------- | ---------------------------- |
| BERT Tone (analytic, version 0.0.1) | 0.90 (0.77)                  |




## Zero-shot Cross-lingual transfer example

An example of utilizing a dataset in another language to be able to make predictions on Danish without seeing Danish training data is shown in this 
[notebook](<https://github.com/alexandrainst/danlp/blob/master/examples/example_zero_shot_sentiment.ipynb>). 
It is trained on English movie reviews from IMDB, and it uses multilingual embeddings from [Artetxe et al. 2019](https://arxiv.org/pdf/1812.10464.pdf) 
called [LASER](<https://github.com/facebookresearch/LASER>)(Language-Agnostic SEntence Representations).


## 🎓 References 

- Mikel Artetxe, and Holger Schwenk. 2019. 
  [Massively Multilingual Sentence Embeddings for Zero-Shot Cross-Lingual Transfer and Beyond.](https://arxiv.org/pdf/1812.10464.pdf). 
  In **TACL**.
- Erik Velldal, Lilja Øvrelid, Eivind Alexander Bergem, Cathrine Stadsnes, Samia Touileb and Fredrik Jørgensen. 2018. [NoReC: The Norwegian Review Corpus.](http://www.lrec-conf.org/proceedings/lrec2018/pdf/851.pdf) In **LREC**.
- Gustav Aarup Lauridsen, Jacob Aarup Dalsgaard and Lars Kjartan Bacher Svendsen. 2019. [SENTIDA: A New Tool for Sentiment Analysis in Danish](https://tidsskrift.dk/lwo/article/view/115711). In **Sprogvidenskabeligt Studentertidsskrift**.
- Finn Årup Nielsen. 2011. [A new ANEW: evaluation of a word list for sentiment analysis in microblogs](https://arxiv.org/abs/1103.2903). In **CEUR Workshop Proceedings**.
