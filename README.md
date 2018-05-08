# Named Entity Recognition with Tensorflow



This repository contains a NER implementation using Tensorflow (based on BiLSTM + CRF and character embeddings) that is based on the implementation by [Guillaume Genthial](https://github.com/guillaumegenthial/sequence_tagging). We have modified this implementation including its documentation. The major changes are listed below:


Mainly, we have done the following changes:
- convert from python 2 to python 3
- extract parameters from source code to a single config file
- create new script for testing new files
- create new script and modify source code for simple transfer learning
- support for several embeddings (GloVe, fasttext, word2vec)
- support to load all embeddings of a model
- support to dynamically load OOV embeddings during testing

Currently, we only provide models for contemporary German and historic German texts.


Table of Content
================


 - [Task of Named Entity Recognition](#task-of-named-entity-recognition)
 - [Machine Learning Model](#machine-learning-model)
 - [Requirements](#requirements)
 - [Run an Existing Model](#run-an-existing-model)
 - [Download Models and Embeddings](#download-models-and-embeddings)
   * [Manual Download](#manual-download)
   * [Automatic Download](#automatic-download)
 - [Train a New Model](#train-a-new-model)
 - [Transfer Learning](#transfer-learning)
 - [Predict Labels for New Text](#predict-labels-for-new-text)
 - [Parameters in the Configuration File](#parameters-in-the-configuration-file)
 - [Requirements](#requirements) 
 - [Citation](#citation)
 - [License](#license)
  







## Task of Named Entity Recognition

The task of Named Entity Recognition (NER) is to predict the type of entity. Classical NER targets on the identification of locations (LOC), persons (PER), organization (ORG) and other (OTH). Here is an example

```
John   lives in New   York
B-PER  O     O  B-LOC I-LOC
```


## Machine Learning Model

The model is similar to [Lample et al.](https://arxiv.org/abs/1603.01360) and [Ma and Hovy](https://arxiv.org/pdf/1603.01354.pdf). A more detailed description can be found [here](https://guillaumegenthial.github.io/sequence-tagging-with-tensorflow.html).

- concatenate final states of a bi-lstm on character embeddings to get a character-based representation of each word
- concatenate this representation to a standard word vector representation (GloVe, Word2Vec, FastText here)
- run a bi-lstm on each sentence to extract contextual representation of each word
- decode with a linear chain CRF

## Requirements


To run the python code, you need python3 and the requirements from the following [file](https://github.com/riedlma/sequence_tagging/blob/master/requirements.txt) which can be easily installed:

```
pip3 install -r requirements.txt
```

In addition, you need to build fastText manually, as described [here](https://github.com/facebookresearch/fastText/tree/master/python), which are the following commands:

```
git clone https://github.com/facebookresearch/fastText.git
cd fastText
pip3 install .
```



## Run an Existing Model

To run pre-computed models, you need to install the [required python packages](#requirements) and you need to download the model and the embeddings. This can be done automatically with a python script as described [here](#automatic-download). However, the models and the embeddings can also be downloaded manually as described [here](#manual-download).


Here, we will fully describe, how to apply the best performing GermEval model to a new file.
First, we need to download the project, the model and the embeddings:

```
git clone https://github.com/riedlma/sequence_tagging
cd sequence_tagging
python3 download_model_embeddings.py GermEval
```

Now, you can create a new file (called test.conll) that should contain one token per line and might contain the following content:

```
Diese 
Beispiel
wurde
von
Martin
Riedl
in
Stuttgart 
erstellt
.
``` 

To start the entity tagging, you run the following command:

```
python3 test.py model_transfer_learning_conll2003_germeval_emb_wiki/config test.conll 
```

The output should be as following:

```
Diese diese	KNOWN	O
Beispiel beispiel	KNOWN	O
wurde wurde	KNOWN	O
von von	KNOWN	O
Martin martin	KNOWN	B-PER
Riedl riedl	KNOWN	I-PER
in in	KNOWN	O
Stuttgart stuttgart	KNOWN	B-LOC
erstellt erstellt	KNOWN	O
. .	KNOWN	O
```
The first column is the input word, the second column specifies the pre-processed word (here lowercased). The third column contains a flag, whether the word has been known during training (KNOWN) or not (UNKNOWN). If labels are assigned to the input file they will be in the third column. Otherwise, they will not be contained. The last column contains the predicted tags.



## Download Models and Embeddings
We provide the best performing model for the following datasets:

| Name| Language | Description| Webpage| 
|-----|----------|------------|---------|
| CoNLL 2003 | German | NER dataset based on Newspaper | [link](https://www.clips.uantwerpen.be/conll2003/ner/)
| GermEval 2014 | German | NER dataset based on Wikipedia | [link](https://sites.google.com/site/germeval2014ner/)|
| ONB| German |NER dataset based on texts of the Austrian National Library from 1710 and 1873 |[link](http://github.com/KBNLresearch/europeananp-ner/)|
| LFT | German | NER dataset based on text of the Dr Friedrich Teßmann Library from 1926 | [link](http://github.com/KBNLresearch/europeananp-ner/)|

All provided models are trained using transfer learning techniques. The models and the embeddings can be downloaded [manually](#manual-download) or [automatically](#automatic-download).


### Manual Download

The models can be downloaded as described in the table. The models should be stored directly on the project directory. Furthermore, they need to be uncompressed (tar xfvz \*tar.gz)

|  Optimized for | Trained | Transfer Learning |Embeddings| Download|
|----------------|------------|---------|-----|------|
| GermEval 2014 | CoNLL2003| GermEval 2014 | German Wikipedia|[link](http://www2.ims.uni-stuttgart.de/data/ner_de/models/model_transfer_learning_conll2003_germeval_emb_wiki.tar.gz) |
| CoNLL 2003 (German) | GermEval 2014 | CoNLL 2003 | German Wikipedia|[link](http://www2.ims.uni-stuttgart.de/data/ner_de/models/model_transfer_learning_conll2003_germeval_emb_wiki.tar.gz) |
| ONB | GermEval 2014 | ONB | German Europeana |  [link](http://www2.ims.uni-stuttgart.de/data/ner_de/models/model_transfer_learning_germeval_onb_emb_euro.tar.gz) |
| LFT | GermEval 2014 | LFT | German Wikipedia | [link](http://www2.ims.uni-stuttgart.de/data/ner_de/models/model_transfer_learning_germeval_lft_emb_wiki.tar.gz) |

The embeddings should best be stored in the folder *embeddings* inside the project folder.
We provide the full embeddings (named Complete) and the filtered embeddings, which only contain the vocabulary of the data of the task. These filtered models have also been used to train the pre-computed models. The German Wikipedia model is provided by [Facebook Research](http://github.com/facebookresearch/fastText/blob/master/pretrained-vectors.md).

| Name | Computed on | Dimensions | Complete  | Filtered|
|------|-------------|------------|-----------|---------|
| Wiki | German Wikipedia | 300   | [link](https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.de.zip)  |  [link](http://www2.ims.uni-stuttgart.de/data/ner_de//embeddings/fasttext.wiki.de.bin.trimmed.npz)|
| Euro | German Europeana | 300   |  [link](http://www2.ims.uni-stuttgart.de/data/ner_de//embeddings/fasttext.german.europeana.skip.300.bin) | [link](http://www2.ims.uni-stuttgart.de/data/ner_de//embeddings/fasttext.german.europeana.skip.300.bin.trimmed.npz) |

### Automatic Download

Using the python script *download_model_embeddings.py* the models and the embeddings can be donwloaded automatically. In addition, the files are placed at the recommended location and are uncompressed.  You can choose between the several options:

```
~ user$ python3 download_model_embeddings.py 

No download option has been specified:
python download_model_embeddings.py options

Following download options are possible:
all            download all models and embeddings
all_models     download all models
all_embed      download all embeddings
GermEval       download best model and embeddings for GermEval
CONLL2003      download best model and embeddings for CoNLL 2003
ONB            download best model and embeddings for ONB
LFT            download best model and embeddings for LFT

```


## Train a New Model

We will describe how a new model can be trained and describe it based on training a model on the GermEval 2014 dataset using pre-computed word embeddings from German Wikipedia. First, we need to download the training data. For training a model, we expect files to have two columns with the first column specifying the word and the second column containing the label.

```
mkdir -p corpora/GermEval
wget -O corpora/GermEval/NER-de-train.tsv  https://sites.google.com/site/germeval2014ner/data/NER-de-train.tsv
wget -O corpora/GermEval/NER-de-dev.tsv  https://sites.google.com/site/germeval2014ner/data/NER-de-dev.tsv
wget -O corpora/GermEval/NER-de-test.tsv  https://sites.google.com/site/germeval2014ner/data/NER-de-test.tsv
cat corpora/GermEval/NER-de-train.tsv  | grep -v "^[#]" | cut -f2,3 |  sed "s/[^ \t]\+\(deriv\|part\)$/O/g" > corpora/GermEval/NER-de-train.tsv.conv
cat corpora/GermEval/NER-de-test.tsv  | grep -v "^[#]" | cut -f2,3 |  sed "s/[^ \t]\+\(deriv\|part\)$/O/g" > corpora/GermEval/NER-de-test.tsv.conv
cat corpora/GermEval/NER-de-dev.tsv  | grep -v "^[#]" | cut -f2,3|  sed "s/[^ \t]\+\(deriv\|part\)$/O/g" > corpora/GermEval/NER-de-dev.tsv.conv
```


For the training we use the German Wikipedia embeddings from the Facebook Research group. These (and all other embeddings) can be downloaded with the following command:

```
python3 download_model_embeddings.py all_models
```

If you want to train on a different language, you can also check if there are pre-computed embeddings available [here](https://github.com/facebookresearch/fastText/blob/master/pretrained-vectors.md). To compute new embeddings you can follow the [manual from fastText](https://github.com/facebookresearch/fastText).

Next, the configuration needs to be edited. First, we create a directory where the model will be stored:

```
mkdir model_germeval
```

Then, we create the configuration file. For this, we use the configuration template ( [config.template](https://github.com/riedlma/sequence_tagging/blob/master/config.template)) and copy it to the model folder:

``` 
cp config.template model_germeval/config
```

At least, all parameters that have as value *TODO* need to be adjusted. Using the current setting, we adjust following parameters (a more detailed description of the configuration is found [here](#parameters-in-the-configuration-file)):

```
[PATH]
#path where the model will be written to
dir_model_output = model_germeval

...
filename_train = corpora/GermEval/NER-de-train.tsv.conv 
filename_dev =   corpora/GermEval/NER-de-dev.tsv.conv 
filename_test =  corpora/GermEval/NER-de-test.tsv.conv 
... 

[EMBEDDINGS]
# dimension of the words
dim_word = 300
# dimension of the characters
dim_char = 100
# path to the embeddings that are used
filename_embeddings = ./embeddings/wiki.de.bin
# path where the embeddings defined by train/dev/test are written to
filename_embeddings_trimmed = ./embeddings/wiki.de.bin.trimmed.npz
...

```

Before we train the model, we build a matrix of the embeddings that are contained in the train/dev/test in addition to the vocabulary, with the *build_vocab.py* script:

```
python3 build_vocab.py model_germeval/config
```

If you want to apply the model to other vocabulary then the one specified in train/dev/test, the model will not have any word representation and will mainly rely on the character word embedding. To prevent this, the easiest way is to add them in the CoNLL format as further parameters to the *build_vocab.py* script:

```
python3 build_vocab.py model_germeval/config vocab1.conll vocab2.conll
```


After that step, the new model can be trained, using the following command: 

```
python3 train.py model_germeval/config
```

The model can be applied to e.g. the test file as follows:

```
python3 test.py model_germeval/config corpora/GermEval/NER-de-test.tsv.conv
```



## Transfer Learning

For performing the transfer learning you first need to train a model e.g. based on the GermEval data as described [here](#train-a-new-model). Be aware, that you added the vocabulary and the tagsets when training the basic model. The easiest way is to add them as additional parameters, when building the vocabulary, e.g.:

```
python3 build_vocab.py model_germeval/config transfer_training.conll transfer_dev.conll test_transfer.conll
```

Whereas there is not explicit parameter fitting to these words, in this way the embeddings will be available for the model. 

After the model has been trained the transfer learning step can be accomplished with the *transfer_learning.py* script, that expects the following parameters:

```
python transfer_learning.py configuration transfer_training.conll transfer_dev.conll
```

After the training, new text files in the domain of the transfer learning files as described [here](#test-a-model).


## Predict Labels for New Text

To test a model, the *test.py* script is used and expects, the configuration file of the model and the test file

``` 
python3 test.py model_configuration test_set
```

## Parameters in the Configuration File

The configuration file is divided in three sections. The section *PATH* contains all variables that specify the locations of the model and labeled data. The *EMBEDDINGS* section contains all parameters for the word embeddings and the *PARAM* section contains all further parameters for the machine learning as well as pre-processing.

```
[PATH]
#path where the model will be written to
dir_model_output = TODO
dir_vocab_output = %(dir_model_output)s
dir_model = %(dir_model_output)s/model.weights/
path_log = %(dir_model_output)s/test.log


filename_train = TODO
filename_dev =   TODO
filename_test =  TODO

# these are the output paths for the vocabulary, the 
# tagsets and the characters used in the train/dev/test set
filename_words = %(dir_vocab_output)s/words.txt
filename_tags = %(dir_vocab_output)s/tags.txt
filename_chars = %(dir_vocab_output)s/chars.txt

[EMBEDDINGS]
# dimension of the words
dim_word = 300
# dimension of the characters
dim_char = 100
# path to the embeddings that are used 
filename_embeddings = TODO
# path where the embeddings defined by train/dev/test are written to
filename_embeddings_trimmed = TODO 
# models can also be trained with random embeddings that are 
# adjusted during training
use_pretrained = True
# currently we support: fasttext, glove and w2v
embedding_type = fasttext
# if using embeddings larger than 2GB this option needs to be switched on
use_large_embeddings = False
# number of embeddings that are dynamically changed during testing
oov_size = 0


# here, several parametesr of the machine learning and pre-processing
# can be changed
[PARAM]
lowercase = True
max_iter = None
train_embeddings = False
nepochs = 15
dropout = 0.5
batch_size = 20
lr_method = adam
lr = 0.001
lr_decay = 0.9
clip = -1
nepoch_no_imprv = 3
hidden_size_char = 100
hidden_size_lstm = 300
use_crf = True
use_chars = True

```





## Citation


If you use this model cite the source code of [Guillaume Genthial](https://github.com/guillaumegenthial/sequence_tagging). If you use the German model and the extension, you can cite our paper:

```
@inproceedings{riedl18:_named_entit_recog_shoot_german,
  title = {A Named Entity Recognition Shootout for {German}},
  author = {Riedl, Martin and Padó, Sebastian},
  booktitle = {Proceedings of Annual Meeting of the Association for Computational Linguistics},
  series={ACL 2018},
  address = {Melbourne, Australia},
  note = {To appear},
  year = 2018
}

```


## License

This project is licensed under the terms of the Apache 2.0 ASL license (as Tensorflow and derivatives). If used for research, citation would be appreciated.


