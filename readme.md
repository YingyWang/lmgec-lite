# LMGEC-Lite

This repository contains the Language Model based Grammatical Error Correction system described in:

> Christopher Bryant and Ted Briscoe. 2018. [**Language Model Based Grammatical Error Correction without Annotated Training Data**](http://aclweb.org/anthology/W18-0529). In Proceedings of the 13th Workshop on Innovative Use of NLP for Building Educational Applications. Association for Computation Linguistics, New Orleans, Louisiana, June.  

Unlike many approaches to GEC, this approach does NOT require annotated training data and mainly depends on a monolingual language model. The program works by iteratively comparing certain words in a text against alternative candidates and applying a correction if one of these candidates is more probable than the original word. These correction candidates are variously generated by a spell checker, morphological inflection database, or are otherwise defined manually. Currently, this system only corrects:

1. Non-words (e.g. freind and informations)  
2. Morphology (e.g. eat, ate, eaten, eating, etc.)  
3. Common Determiners and Prepositions (e.g. the, a, in, at, to, etc.) 

The input is a tokenized original sentence, and the output is the equivalent tokenized corrected sentence.  

# Installation

We have only tested the code in Python 3. We also make use of 3 publicly available libraries.

## KenLM

KenLM is a language modelling toolkit. The language model is the core component of our approach and must be trained separately. Be warned that the model we tested takes up over 20GB of hard drive space (in binary format) and must be loaded into memory to be used. The uncompressed model takes up twice this amount. If this is too much for your hardware, it is also possible to train a smaller model on less data, although results may be worse.

There are separate instructions on how to install and use KenLM [here](https://github.com/kpu/kenlm) and [here](http://kheafield.com/code/kenlm/), but we installed it as follows:

Navigate to the directory you want to install KenLM and then run:  
```
git clone https://github.com/kpu/kenlm.git
mkdir kenlm/build
cd kenlm/build
cmake ..
make -j 4
pip3 install https://github.com/kpu/kenlm/archive/master.zip
```
This installs and compiles the main program along with a python wrapper.  

Having installed KenLM, we then need to build a language model from native text. We used the [Billion Word Benchmark](http://www.statmt.org/lm-benchmark/) dataset, consisting of close to a billion words of English taken from news articles on the web. The text can be downloaded and preprocessed as follows:  
```
wget http://www.statmt.org/lm-benchmark/1-billion-word-language-modeling-benchmark-r13output.tar.gz
tar -zxvf 1-billion-word-language-modeling-benchmark-r13output.tar.gz
cat 1-billion-word-language-modeling-benchmark-r13output/training-monolingual.tokenized.shuffled/* > 1b.txt
```
This will produce a 3.86GB text file called 1b.txt which contains the entire training set, one sentence per line.

To train a model on this file with KenLM, run:
```
mkdir tmp
/<your_path>/kenlm/build/bin/lmplz -o 5 -S 50% -T tmp/ < 1b.txt > 1b.arpa
```
-o lets you choose n-gram order (we recommend 5)  
-S lets you specify how much RAM to use in percent (can also say e.g. 10G for 10GB)  
-T lets you choose a directory for temporary files (recommended)  
This may take some time depending on your hardware. On our system, the model took ~90mins to train.

Having trained a model, finally run the following to binarise it.  
`/<your_path>/kenlm/build/bin/build_binary 1b.arpa 1b.bin`  
This will both reduce model loading times and save hard drive space.  

## spaCy

[spaCy](https://spacy.io/) is a Natural Language Processing (NLP) library. We use it mainly for lemmatization.  
We recommend spaCy v1.9.0 which can be installed along with the default English model as follows:  
```
pip3 install -U spacy==1.9.0
python3 -m spacy download en  
```

## CyHunspell

[CyHunspell](https://github.com/OpenGov/cython_hunspell) is a Python wrapper for the Hunspell spell-checking library. We found it performs better than the native python PyEnchant spell-checking library. It can be installed as follows:  

`pip3 install -U CyHunspell`

# Usage

Having installed and trained a model, the correction program can be run as:
```
python3 lmgec.py <text_file> -mdl <model_file> -o <out_file>
```
The input text file must be word tokenized (i.e. plain text separated by whitespace) with one sentence per line. The model file should point to the 1b.bin file (or equivalent) as described earlier. The output file will be the corrected version of the input file.  

### Advanced

In the paper, we found that different datasets required different language model improvement thresholds to optimise performance. In the code, the default threshold is set to 0.96, which requires a candidate correction to improve the language model log probability score of the input sentence by at least 4% in order for a change to be made. You can change this value by adding `-th <float>` to the command line when running the program. Note that values greater than 1.0 will likely cause the program to get stuck in a loop and run forever. Note also that the optimum value for this threshold depends on both the development set and the size of the language model. 

# MIT License

Copyright (c) 2017 Christopher Bryant

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

# Contact

If you have any questions, suggestions or bug reports, you can contact the author at:  
christopher d0t bryant at cl.cam.ac.uk  
