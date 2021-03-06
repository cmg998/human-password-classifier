# Human Password Classifier
Creates a classification model of passwords with labels "h" (human generated), "m" (machine-generated). The generated model can be used to classify passwords for human likeliness.

Additional features:
* Apple's coreml model export
* pipeline tests for different vectorizers and classifiers

CLI commands refer to macOS.

# Dependencies

Python 3.6
CoreML dependency: Python 2.7 (NOT 3+ otherwise coremltools won't work)

Packages:
* coremltools
* argparse
* scikit-learn
* pandas
* numpy

For later iOS usage:
* XCode 9+
* iOS 11+

# Setup

## Installation using Anaconda

Create virtual environment named "ml-passwords":

`$ conda create -n ml-passwords anaconda`

If you need CoreML support use:

`$ conda create -n ml-passwords python=2.7 anaconda`

Activate newly created environment:

`$ source activate ml-passwords`

Install libraries: (coremltools is optional)

`$ pip install numpy pandas argparse scikit-learn`

If you need CoreML support additionally use:

`$ pip install coremltools`

## Use newly created environment for later python execution

`$ source activate ml-passwords`

# Training data

Password files require a tab-separated file with unix line breaks. There have to be two columns: The passwords and the corresponding labels ("h": human-generated, "m": machine-generated).

## Preparing the dataset

Generate a labeled dataset from two password files, one containing human- and the other one containing machine-generated passwords.

`$ cat datasets/human-passwords.txt | sed -e $'s/$/\th/' > datasets/passwords-labeled-h.tsv`

`$ cat datasets/machine-passwords.txt | sed -e $'s/$/\tm/' > datasets/passwords-labeled-m.tsv`

`$ cat datasets/passwords-labeled-h.tsv datasets/passwords-labeled-m.tsv | uniq -u | sort -R > datasets/passwords.tsv`


# Usage

See CLI help of scripts by issuing "-h" parameter.

## Exported files

* .pkl: model file
* CoreML: ...-coreml.mlmodel: coreml machine learning model
* CoreML: ...-features.txt: ordered feature columns for vectorizer

## Train model

### Train and export model

Shows training stats and optionally saves model file for later classification.

`$ python train.py --output_prefix models/model datasets/passwords.tsv`

If you want to export the coreml model to use on iOS 11+:

`$ python train.py --export_coreml --output_prefix models/model datasets/passwords.tsv`

#### Show pipeline statistics

`$ python train.py --test_pipelines passwords.tsv`

#### Show static tests at the end of training

`$ python train.py --output_prefix models/model --static_tests_file static-tests/example.tsv datasets/passwords.tsv`

## Classify a password

Uses previously trained and saved model with train.py to classify the password "123456".

`$ python classify.py models/model.pkl "123456"`

## Apply human-likeliness to an existing password list

Adds probabilities of each password if it was generated by a human.

Please note that not all models support the output of probabilities!

`$ cat datasets/passwords.txt | python apply-human-likeliness-to-list.py models/model.pkl`

Sort by likeliness:

`$ cat datasets/passwords.txt | python apply-human-likeliness-to-list.py models/model.pkl | sort -t $'\t' -k 2 -r > outputs/passwords-human-likeliness-sorted.tsv`

# Evaluate model by labeled password list

`$ cat datasets/passwords.tsv | python evaluate-model-by-list.py models/model.pkl`

# Sources

model based on [faizann24's Machine-Learning-based-Password-Strength-Classification](https://github.com/faizann24/Machine-Learning-based-Password-Strength-Classification/blob/master/script.py)

pipeline combinations inspired by [llSourcell's A_guide_to_coreML](https://github.com/llSourcell/A_guide_to_coreML/blob/master/spam_detection.py) and [an official scikit-learn example](http://scikit-learn.org/stable/auto_examples/model_selection/grid_search_text_feature_extraction.html)
