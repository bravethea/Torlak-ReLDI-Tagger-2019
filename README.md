# Part of speech tagger and lemmatizer for Torlak <a><img src="https://traceba.net/wp-content/uploads/2019/06/TraCeBa-logo-L.png" width="150" align="right" title="hover text"></a>





Custom model of the [ReLDI tagger](https://github.com/clarinsi/reldi-tagger) adapted for Torlak dialect.

Python version 2.7

Python modules:

* sklearn(>=0.15) (necessary only if you want to perform lemmatisation as well)
* marisa_trie (https://github.com/pytries/marisa-trie)
* pycrfsuite (https://github.com/tpeng/python-crfsuite)

## Citing the tagger

The tagger for Torlak and the underlying training files are described in the paper:

**Vuković, Teodora (submitted). Representing variation in a spoken corpus of an endangered dialect. The case of Torlak.**

To cite the ReLDI tagger, see the original [GitHub repository and the README file](https://github.com/clarinsi/reldi-tagger)

## Tagger files and scripts

The original [ReLDI tagger files and instructions](https://github.com/clarinsi/reldi-tagger) are adapted for Torlak.

Torlak model code is called `torsr` ('tor' + 'sr', because the training data contains Torlak and Serbian merged).
The files can be downloaded here, added to the [ReLDI tagger repository](https://github.com/clarinsi/reldi-tagger) and used accordingly.
The files for Torlak available here are:
```
torLex.gz
torsr.lemma_freq
torsr.msd.model
torsr.train
```
Another file necessary for using the tagger is `torsr.marisa`, which is too large to be uploaded to GitHub, and needs to be created by training the tagger as explained below OR downloaded [here](https://drive.switch.ch/index.php/s/FcjizszEoVtO1l5).

You can create the `torsrLex.gz` file used for training and referenced throughout the text that follows, by merging `torLex` file and the [Serbian lexicon srLex_v1.2](https://www.clarin.si/repository/xmlui/bitstream/handle/11356/1073/srLex_v1.2.gz).

## Modifying the `tagger.py` to include Torlak

In order to run `tagger.py` with the Torlak files containing `torsr` in the title, you first need to add the Torlak language code to the list of possible language code arguments in the `tagger.py` script. To do so, change the following line from the original script:
```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr'])
```
so that the list of possible choices contains `torsr`, as follows:

```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr', 'torsr'])
```

## Running the tagger

If you have the necessary files for tagging for Torlak `torst.msd.model` and `torsr.marisa`), you can run the tagger in the terminal by entering the text in the terminal word by word and pressing CTRL+D at the end:
```
$ ./tagger.py torsr
u
selo
mi
živimo
ja
i
dedava
.

CTRL+D
u	Sa
selo	Ncnsa
mi	Pp1-pn
živimo	Vmr1p	živeti
ja	Pp1-sn
i	Cc
dedava	Ncmsn_v
.	Z
```

You can include the lemmatizer, using the `-l` flag. Lemmatization requires the `torsr.lexicon.guesser` file, which can be created by training the lemmatiser as described below.
```
$ ./tagger.py torsr -l
u
selo
mi
živimo
ja
i
dedava
.

CTRL+D
u	Sa	u
selo	Ncnsa	selo
mi	Pp1-pn	mi
živimo	Vmr1p	živeti
ja	Pp1-sn	ja
i	Cc	i
dedava	Ncmsn_v	deda
.	Z	.
```

You can also use a tokenized verticalized file as input be tagged, as in the example below,:

```
$ cat file.txt | ./tagger.py torsr -l
u	Sa	u
selo	Ncnsa	selo
mi	Pp1-pn	mi
živimo	Vmr1p	živeti
ja	Pp1-sn	ja
i	Cc	i
dedava	Ncmsn_v	deda
.	Z	.
```

Uout can also output the results  to another file using `> newfile.txt`:

```
$ cat file.txt | ./tagger.py torsr -l > newfile.txt
```

The text can be processed using the tokenizer, as explained in the original instructions for the ReLDI tagger. Bear in mind that the togenizer from the ReLDI tagger package has not been adapted to parse Torlak transcripts. It works for written standard BKMS or Slovene.
Update July 2023 - Tokenizer is missing from the original Reldi repository.

## Training the `torsr` model 

In case you want to train the Torlak tagger, you can use the `torsr.train` and `torsrLex.gz` files or a different, modified input. As stated in the ReLDI Taggger instructions, the input files need to be in "in the one-token-per-line, empty-line-as-sentence-boundary format. The tagger training data should be, with the token, lemma and the tag separated by a tab." See `torsr.train` file as an example. 

Once you have the necessary file in the required format, you may proceed with the training based on the ReLDI tagger instructions and using the adapted commands for the `torsr` model.

### Modifying the `train_tagger.py` and and `train_lemmatizer.py` from to include Torlak

In order to run `train_tagger.py` and `train_lemmatizer.py` with the Torlak files containing `torsr` in the title, you first need to add the Torlak language code to the list of possible language code arguments in the `tagger.py` script. To do so, change the following line from the original script:
```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr'])
```
so that the list of possible choices contains `torsr`, as follows:

```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr', 'torsr'])
```


### Preparing the lexicon trie used by the tagger

The lexicon trie is used both during training the tagger and during tagging.

The lexicon file should be formatted in the same manner as the training data, just with no sentence boundaries. Additionally to the words, tags and lemmas, it contains information about the frequency of the word in the training data. To prepare the `torsr` lexicon, run the following command:
```
$ gunzip -c torsrLex.gz | cut -f 1,2,3 | ./prepare_marisa.py torsr.marisa

```

### Training the tagger

The only argument given to the script is the language code. In case of Torlak (language code `torsr`) the corpus training data is expected to be in the file `torsr.train`, while the lexicon trie is expected to be in the file `torsr.marisa`.
```
$ ./train_tagger.py torsr

```

### Preparing the lexicon for training the lemmatiser

The first step in producing the lexicon for lemmatisation is to calculate the lemma frequency list from the tagger training data. The data in the same format as for training the tagger should be used.

```
$ ./lemma_freq.py torsr.lemma_freq < torsr.train
```

The second step produces the lexicon in form of a `marisa_trie.BytesTrie`. The lemma frequency information is used in case of `(token,msd)` pair collisions. Only the most frequent lemma is kept in the lexicon.
```
$ gunzip -c torsrLex.gz | cut -f 1,2,3 | ./prepare_lexicon.py torsr.lemma_freq torsr.lexicon
```


### Training the lemmatiser

The lemmatiser of unknown words is trained on the lexicon prepared in the previous step. The lexicon used for training the lemma guesser has a suffix `.train`. A Multinomial Naive Bayes classifier is learned for each MSD. The classes to be predicted are quatruple transformations in form `(remove_start,prefix,remove_end,suffix)`. The transformation is being applied by removing the first remove_start characters, adding the prefix, removing the last remove_end characters and adding the suffix.

The output of the lemmatiser learning process is a file with the `.lexicon.guesser` suffix.
```
$ ./train_lemmatiser.py torsr.lexicon
```
