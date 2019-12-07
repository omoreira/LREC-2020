# GM-RKB WikiText Error Correction Task (LREC-2020)

The *GM-RKB WikiText Error Correction (WEC) Task* is intended to help to deliver an automated system that recognizes and repairs simple typographical errors in [GM-RKB](http://gmrkb.com)  pages WikiText  based on frequent patterns observed in the corpus. 

## Datasets

We created multiple datasets for both training and evaluating our baseline WEC systems and for evaluating.  There are two main types of datasets: 

1. GM-RKB Datasets; 
1. Wikipedia Datasets. 

### GM-RKB Datasets

This dataset type is created using a snapshot of GM-RKB wiki pages in MediaWiki XML format.This dataset type is created using a snapshot of GM-RKB wiki pages in MediaWiki XML format. The current version of GM-RKB related corpus consists of 106,111 MediaWiki-formatted pages of an average size of 976 characters.

### Wikipedia Datasets
We used MediaWiki API, a web service that allows us to download Wikipedia pages as XML files. We have created two Wikipedia datasets:

1. A Wikipedia training dataset containing a total of 35,000 wiki pages. We run the API to create a list of random pages using a special query without specifying any other filters or constraints. This dataset has an average page size of 7,252 characters and a total size of 253,835,323 characters. 
1. a Wikipedia testing dataset containing a list of 5000 random page titles and pages content. This dataset has an average page size of 6164 characters, and the total size of 34,700,225 characters. 

### Dataset Features

GM-RKB and Wikipedia datasets both have English Wiki pages written in Wiki markup language. They both have similar annotation syntaxes, which implies they have the same notion of corrected text. Nevertheless, these have different average page size. The Wikipedia datasets have an average page size of 7,252 characters, while, GM-RKB datasets have an average page size of 976 characters. Only 3,700 out of 10,000 frequent words have common characters between these two dataset types, and these are often ordered completely different in the files.

## Pre-Processing And Error Generation

Given that the absence of manually corrected examples, we have created a tool to synthetically introduce errors. We created a Python library to automatically parse the XML Wiki snapshot, extract the required information, the original content, add noise to the original content and store all the data in ”Parquet” files. These files became our the main data repository. 

Each ”Parquet” files contain: 

- the original content of each page, 
- the version with noise added, 
- an unique ID number, 
- the size of the page in number of characters, 
- a ’k’ value from 0 to 10 to be used with 10-fold cross validation. 

Four types of character-level errors are introduced: *“Changed Character”*, *“Deleted Character”*, *“Inserted Character”*, and *“Swapped Characters”*. When a new character is inserted as noise, it is selected at random from an ASCII non-numeric alphabet. Over time this random selection is adjusted to better simulate naturally occurring errors. The noise is controlled by multiple variables by indicating the level of total noise and the ratio of each type of noise added.

## Baseline Models

We compared the relative performance of a character MLE Language Model-based and a sequence-to-sequence (seq2seq) neural network-based WEC.

### A Character-level MLE Language Model-based WikiFixer

The Character-level MLE Language Model-based WikiFixer is a data-driven implementation based on a simple character-level language model-based (in the form of a lookup table) that predicts the likelihood score for short sub-strings of characters.  This statistical approach assumes that N-gram is a sequence of N characters.  Currently, this function is trained on the GM-RKB dataset. The MLE WikiFixer uses similarity probabilities to detect noise by selecting error correction candidates with the highest probability. This model is controlled by setting thresholds for detecting noise and accepting an error correction action. 

Additional information about this baseline method is available online at [https://tinyurl.com/mle-wikifixer](https://tinyurl.com/mle-wikifixer)

### A Seq2Seq NNet-based WikiFixer
The sequence-to-sequence (seq2seq) neural networks model *" translates"* partially noisy sequences to corrected ones. Our seq2seq model can map a fixed-length input sequence with a fixed length-output. Input and output text lengths may differ. 

The model consists of 3 main components: 

- **Encoder** -- a stack of several recurrent layers. Each encoder-layer accepts a single element of the input sequence, collects information for that element, and propagates it forward.
- **Encoder Vector** --  a vector that is constructed in the encoder's final hidden state. This vector represents the information for all input elements in order to help the decoder make accurate predictions.
- **Decoder** --  a stack of several recurrent layers. Each layer predicts an output element at a time step. Each recurrent unit accepts a hidden state from the previous unit, produces an output as well as its own hidden state.

In the WikiFixer problem, the input sequence is a collection of all characters from the noisy WikiText, while, the output sequence is a collection of all characters of the fixed text. The recurrent units used are LSTMs, an extension for recurrent neural networks built to enhance their memory capacity.

## Evaluation Metric

Each character in the WEC system’s output text is grouped into one of the following five outcomes:

- **True Positive (TP):** the source character was an error; the model correctly fixed it.
- **False Positive (FP):** although the source character was not an error; however, the model tried to fix it intro- ducing a new error instead.
- **True Negative (TN):** the source character was not an error; the model correctly disregarded it as an error and did not try to fix it.
- **False Negative (FN):** the source character was an er- ror; however, the model failed to detect it and pro- cessed the error as the correct character.
- **Detected Not-Fixed (DN):** the source character was an error; the model successfully detected the error but failed to correct it, or used the wrong character to fixed. (e.g. the correct repair was changing ’a’ to ’b’ but the model changed ’b’ to ’c’ instead of ’a’).

We focused on only two of the five classes of TPs and FPs. That is, we focused on the two classes that can help us measure the model's precision. The evaluation score is calculated as follows:
*Evaluation = 1&times;TP count &minus; 5&times;FP count  *
The adopted evaluation metric treats the task as a (near-binary) classification problem.  It measures the changes in text repairs performed on the Wiki pages by rewarding correctly fixed errors and penalizing system added errors. Ultimately, the metric is well-suited at quantifying the detection of orthographic errors rather than grammatical errors. It also operates at the character-level rather than at a token or word-level.
## References

- Gabor Melli, Abdelrhman Eldallal, Bassim Lazem, and Olga Moreira (2020). "GM-RKB WikiText Error Correction Task and Baselines". In: Submitted to LREC 2020. 

## Directory Structure
```bash
├── WikiFixer.py : WikiFixer MLE Model
├── WikiFixerNNet.py : WikiFixer NNet seq2seq Model
├── model_config.py :  WikiFixer NNet configuration file
├── requirements.txt
├── Datasets
│   ├── MWDump.20191001.Noisetest.parquet :  GM-RKB Wiki dataset
│   ├── Wikipedia.Noise.parquet : Wikipedia Training dataset
│   └── WikipediaTest.Noisetest.parquet : Wikipedia Test dataset
├── mle
│   ├── LanguageModel.py : character based language model implementation
│   ├── model6-0.json
│   ├── model7-1.json
├── nnet
│   ├── data
│   │   └── allowed_chars_sm.json
│   ├── data_processing.py
│   ├── data_vectorization.py
│   └── models
│       ├── gm_rkb_nnet_fixer_GMRKB&Wiki7_sm_e22.h5
│       ├── gm_rkb_nnet_fixer_GMRKB2019_sm_e22.h5
│       ├── gm_rkb_nnet_fixer_GMRKB_PREWiki_sm_e12.h5
│       └── gm_rkb_nnet_fixer_Wikipedia_sm_e10.h5
├── tests
│   ├── log_file.txt
│   ├── multiproc_evaluation.py
│   ├── path.py
│   ├── test_config.py
│   ├── test_integration.py
│   ├── test_mle.py
│   ├── test_model.py
│   ├── test_nnet.py
│   ├── test_script.py
│   └── test_tools.py
└── tools
    ├── WikiTextTools.py
    ├── clean_lm.py
    ├── diff_match_patch.py
    ├── enums.py
    ├── fixer_evaluation.py
    ├── path.py
    ├── review_logs_output.py
    └── test_WikiTextTools.py


```
