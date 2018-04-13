---
title: "Kaldi lattices"
classes: wide
math: true
---

## Introduction to working with Kaldi lattices, and differences to SLF

### Differences between SLF and Kaldi lattices

During speech recognition, the decoder explores a search space that in principle
contains every possible sentence. It is often useful to save a subset of this
search space as a directed graph, where the nodes or edges correspond to word
hypotheses. The scores computed using acoustic and language models are saved in
the graph so that it can be efficiently rescored and decoded using different,
perhaps computationally more demanding, language models.

HTK recognizer defined the Standard Lattice Format (SLF), which has been widely
adopted. It is basically a text file that lists the graph nodes and edges. The
nodes may contain time stamps so that the words can be aligned with audio.
Usually the word identities, and language model and acoustic scores
(log-likelihoods) are stored in the edges.

Kaldi represents many components of the speech recognition system as
finite-state transducers (FSTs), including hidden Markov models, language model,
and pronunciation dictionary. FSTs are finite-state machines that produce output
when they transition from one state to the next. This is nothing but a certain
kind of directed graph, so lattices in Kaldi are naturally also represented as
an FST. In this case the output is the word identities.

The FSTs in Kaldi are weighted, meaning that each arc is associated with a cost
of taking that transition. In lattice FSTs the weights are defined by the
acoustic and language model scores. The numbers are a bit different from the
scores that one finds in SLF lattices:

1. Kaldi lattices use costs, i.e. negative log-likelihoods, instead of
   log-likelihoods.
2. Kaldi scripts may "push" weights towards the beginning of the graph, so that
   for example language model probabilities cannot be interpreted as individual
   word probabilities, but only the sum along an entire path through the lattice
   is meaningful.
3. When a neural network acoustic model is used, the probabilities are not
   correctly normalized. The neural network predicts posterior probabilities,
   which will be divided by the prior probabilities of the HMM states to obtain
   pseudo-likelihoods (likelihoods scaled by a constant scaling factor). It is
   possible for the pseudo-probabilities to be higher than one.
4. Language model probability is incorporated in *graph cost*, which also
   includes pronunciation, transition, and silence probabilities.

The last item requires some consideration when rescoring language model
probabilities. In order to preserve the rest of the graph cost, one could
subtract the old language model cost from the graph cost, and add the remaining
value to the acoustic cost (which will not be modified during rescoring).
However, if the lexicon doesn't include pronunciation probabilities, and silence
probabilities are not estimated, this might not be worth the trouble. The
default silence probability of <span>$0.5$</span> can be compensated by a word
insertion penalty of <span>$\log(2)$</span>, and the transition probabilities
don't make much difference.

### A look inside Kaldi lattices

There are actually two alternative FST representations for lattices in Kaldi,
implemented by the Lattice and CompactLattice classes. They differ in how they
store the *transition IDs*, which identify the HMM states. Lattice has
transition IDs in input, CompactLattice includes transition IDs in the weights.
(The concept of weight in Kaldi is pretty liberal in what it may contain.
Multiplying the weights of a CompactLattice will concatenate the transition ID
sequences.)

Kaldi stores lattices in its general purpose archive format, which can be either
binary or text. Usually the lattices are saved in binary CompactLattice form.
`lattice-copy` command can be used for converting between binary and text
formats, as well as Lattice and CompactLattice. One file may contain multiple
lattices. The decode scripts in the example recipes save the recognition
lattices from job N into `lat.N.gz`.

In order to see what a lattice file contains, one can simply convert it to text
form using `lattice-copy`. Words are stored in lattices as integer IDs, so the
output makes more sense if the IDs (third field) are converted to words using
the mapping found from `words.txt` in the lang directory:

```bash
lattice-copy "ark:zcat DECODE-DIR/lat.N.gz |" ark,t:- |
utils/int2sym.pl -f 3 LANG-DIR/words.txt
```

The firts line of the output contains the utterance ID. The following lines each
contain start and end state, word, and a comma-separated list of weights. If the
lattice is in CompactLattice format, the weights include acoustic cost, graph
cost, and transition IDs:

```
1-1417560
0 1 tästä 14.6843,-224.556,3_12_18_17_17_17_17_17_17_17_17_17_17_.
0 7275 täst 16.5506,-256.026,3_12_18_17_17_17_17_17_17_17_17_17_..
1 2 on 2.84427,-10.4875, 
2 3 hyvä 3.4889,-118.49,22729_22738_22884_3502_3718_3788_17302_...
3 4 jatkaa 11.5558,-299.218,25331_3552_3636_3635_3738_3737_11434_.
```

In the above example, the first transition from state 0 to state 1 is associated
with language model cost 14.6843. The graph cost -224.556 is negative, which is
possible because a neural network acoustic model has been used to create the
lattice. The transition IDs start with a long sequence of silence states. There
are no transition IDs associated with the word `on`. This is because generally
the weights are not synchronized with each other and the word identities—it is
only meaningful to look at a whole path from the initial node to the final node.

Often it is useful to align the transition IDs with word boundaries, so that one
arc corresponds to one word, by calling `lattice-align-words` first:

```bash
lattice-align-words LANG-DIR/phones/word_boundary.int \
                    MODEL-DIR/final.mdl \
                    "ark:zcat DECODE-DIR/lat.N.gz |" ark,t:- |
utils/int2sym.pl -f 3 LANG-DIR/words.txt
```

Now the output starts with two transitions that produce the `<eps>` token. It is
a special token meaning that there is no word on this arc. The arcs correspond
to the silence at the beginning of the utterance. The transition IDs on the
other arcs correspond to the words:

```
1-1417560
0 4 <eps> 14.6843,-224.556,3_12_18_17_17_17_17_17_17_17_17_17_17_.
0 1 <eps> 16.5506,-256.026,3_12_18_17_17_17_17_17_17_17_17_17_17_.
1 2 täst 0,0,22026_22025_22025_22025_22025_22152_22151_22314_22313
2 3 on 8.15453,-140.39,17450_17449_17606_17714_16186_16226_16462
3 7 hyvä 11.6047,-299.218,7496_7596_7760_25852_25906_25905_25972_.
4 5 tästä 6.33317,-128.978,22026_22025_22025_22025_22025_22152_...
5 6 on 0,0,17302_17538_17656_16186_16226_16462
6 7 hyvä 11.5558,-299.218,7496_7596_7760_25852_25906_25972_25226_.
7 8 jatkaa 0,0,11434_11433_11433_11482_11534_11533_11533_1910_1909
```

### Decoding best path and N-best lists

`lattice-best-path` is a simple utility to decode the best path of lattices and
output word IDs. A word symbol table can be provided for mapping the IDs to
words, but that affects the debug output only. In order to write text
transcripts, one can output in text format and convert the word IDs to words
using `int2sym.pl` (excluding the first field, which is the utterance ID):

```bash
lattice-best-path --lm-scale=12 \
                  --word-symbol-table=LANG-DIR/words.txt \
                  "ark:zcat DECODE-DIR/lat.N.gz |" ark,t:- |
utils/int2sym.pl -f 2- LANG-DIR/words.txt >transcript.ref
```

N-best list in Kaldi are represented as lattices with n distinct (linear) paths.
A lattice with n best paths can be decoded using `lattice-nbest`, and the single
best path can be decoded using `lattice-1best`. Transition IDs, language model
costs, acoustic costs, and transcripts can be extracted from linear FSTs using
`nbest-to-linear` utility:

```bash
lattice-1best --lm-scale=12 \
              "ark:zcat DECODE-DIR/lat.N.gz |" ark:- |
nbest-to-linear ark:- \
                ark,t:transition-ids.txt \
                ark,t:- \
                ark,t:lm-costs.txt \
                ark,t:acoustic-costs.txt |
utils/int2sym.pl -f 2- LANG-DIR/words.txt >transcript.ref
```

A linear lattice can be converted into time marked CTM transcript using
`nbest-to-ctm`, which is useful if you need to know when each word was spoken.
First the lattice must be aligned with word boundaries. Again the word IDs (the
fifth field) need to be converted to words:

```bash
lattice-1best --lm-scale=12 \
              "ark:zcat DECODE-DIR/lat.N.gz |" ark:- |
lattice-align-words LANG-DIR/phones/word_boundary.int \
                    MODEL-DIR/final.mdl \
                    ark:- ark:- |
nbest-to-ctm ark:- - |
utils/int2sym.pl -f 5 LANG-DIR/words.txt >transcript.ctm
```

The CTM file will contain five fields on each line: utterance ID, audio channel,
begin time in seconds, duration in seconds, and the word.

### Pruning and evaluating lattices

Some operations, for example rescoring with a neural network language model, can
take a lot of time if the lattices are large. `lattice-prune` can be used to
prune paths that are not close enough to the best path, in the same way as
`lattice-tool -posterior-prune` does (from SRILM toolkit). The threshold for
pruning is given in the `--beam` argument:

```bash
lattice-prune --inv-acoustic-scale=12 --beam=5 \
              "ark:zcat DECODE-DIR/lat.N.gz |" \
              "ark:| gzip >PRUNED-DIR/lat.N.gz"
```

When optimizing the lattice size, it is helpful to know the oracle word error
rate. `lattice-oracle` computes the minimum error rate that can be obatined from
the lattice, in a similar way that can be done using the SRILM command
`lattice-tool -ref-file`. The command takes as input the lattice file and the
reference transcript. Transcripts are expected to be in form of utterance ID
followed by word IDs, so `utils/sym2int.pl` should be used to map the words to
word IDs. Any out-of-vocabulary words in the reference transcripts need to be
mapped to the `[oov]` tag first:

```bash
utils/sym2int.pl --map-oov [oov] -f 2- LANG-DIR/words.txt \
                 <transcript.ref |
lattice-oracle --word-symbol-table=LANG-DIR/words.txt \
               "ark:zcat PRUNED-DIR/lat.N.gz |" \
               ark:- \
               ark,t:oracle-transcript.int
```

The program also displays the oracle word sequence, and writes the word IDs to
the file given in the third positional argument.

### Converting Kaldi lattices to SLF

Kaldi lattices can be converted to SLF for processing with external tools. There
are two scripts in the `egs/wsj/s5/utils` directory that are designed for that:
`convert_slf.pl` converts a single lattice, and `convert_slf_parallel.sh`
supports converting a batch of lattices in a compute cluster.

The command for submitting batch jobs is given to `convert_slf_parallel.sh` in
the `--cmd` argument as usual. The script also takes path to the data directory
just for checking that the number of lattices matches the number of utterances,
lang directory that contains the necessary word tables, and decode directory,
where the lattices are:

```bash
utils/convert_slf_parallel.sh --cmd BATCH-CMD \
                              DATA-DIR \
                              LANG-DIR \
                              DECODE-DIR
```

Note that the language model scores in the created SLF lattices are actually
graph scores, as explained above.
