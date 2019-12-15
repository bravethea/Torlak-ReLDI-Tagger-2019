# Part of speech tagger and a lemmatizer for Torlak

Custom model of the ReLDI tagger adapted for Torlak dialect.

## Citing the tagger

The tagger and the underlying training files are described in the paper:
Vuković, Teodora (Submitted). Representing variation in a spoken corpus of an endangered dialect. The case of Torlak.

## Tagger files and scripts

The original ReLDI tagger files and instructions are taken from from `https://github.com/clarinsi/reldi-tagger` and adapted for Torlak.

Torlak model code is `torsr` (tor + sr, because the training data contains Torlak and Serbian merged).
The files can be downloaded here and added to the ReLDI tagger repository, available from the GitHub repository. 
The necessary files for Torlak are:
```
torsr.lemma_freq
torsr.lexicon
torsr.lexicon.guesser
torsr.lexicon.train
torsr.marisa
torsr.msd.model
torsr.train
torsrLex.gz
```

## Modifying the `tagger.py` from the script to include Torlak

In order to run `tagger.py` with the Torlak files containing `torsr` in the title, first you need to add this label to the list of possible language codes in the tagger script. Change the following line from the original script:
```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr'])
```
so that the list of possible choices contains `torsr`, as follows:

```
parser.add_argument('lang', help='language of the text', choices=['sl', 'sl.ns', 'sl.ns.true', 'sl.ns.lower', 'hr', 'sr', 'torsr'])
```

Bear in mind that the script is written in Puthon 2.7., so you 

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

You can include the lemmatizer, using the `-l` flag:
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

You can also send the tokenized verticalized file to be tagged to stdin, as in the following example (and optionally redirect the output to another file using `> newfile.txt`):

```
$ cat file.txt | ./tagger.py torsr -l > newfile.txt
u	Sa	u
selo	Ncnsa	selo
mi	Pp1-pn	mi
živimo	Vmr1p	živeti
ja	Pp1-sn	ja
i	Cc	i
dedava	Ncmsn_v	deda
.	Z	.
```

The text can also be processed using the tokenizer, as explained in the original instructions for the tagger. Bear in mind that the togenizer from the ReLDI tagger package has not been adapted to parse Torlak transcripts. It works for written standard BKMS or Slovene.

## Training the `torsr` model 

In case you want to re-train the Torlak tagger you can use the `torsr.train` and `torsrLex.gz` files or with modified input. As stated in the ReLDI Taggger instructions, the input files need to be in "in the one-token-per-line, empty-line-as-sentence-boundary format. The tagger training data should be , with the token, lemma and the tag separated by a tab." See `tors.train` file as an example. 

Once you have the necessary file in the required format, proceed with the training based on the ReLDI tagger instructions and using the adapted commands for the `torsr` files.

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
