---
layout: post
title: "Kaldi lattices"
description: "Introduction to working with Kaldi lattices, and differences to SLF"
category: 
tags: []
---
{% include JB/setup %}

### SLF and Kaldi lattices

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

Lattices are intrinsic to Kaldi, and commonly generated during decoding, but
they are represented as finite-state transducers (FSTs). This is just an
alternative formulation of a directed graph. FSTs are finite-state machines that
produce output when they transition from one state to the next. In this case the
output is the word identities. The transitions are additionally labeled with
weights that are defined by the acoustic and language model scores. The numbers
are a bit different from the scores that one finds in SLF lattices:

1. The weights in Kaldi lattices are costs, i.e. negative log-likelihoods.
2. Kaldi scripts may "push" language model weights towards the beginning of the
   graph, so that they cannot be interpreted as individual word probabilities,
   but only the sum along an entire path through the lattice is meaningful.
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
default silence probability of 0.5 can be compensated by a word insertion
penalty of log(2), and the transition probabilities don't make much difference.

### Kaldi lattice format

There are actually two alternative FST representations for lattices in Kaldi,
implemented by the Lattice and CompactLattice classes. They differ in how they
store the *transition IDs*, which identify the HMM states. Lattice has
transition IDs in input. CompactLattice has the word identity in both input and
output, and transition IDs are included in the weights.

Kaldi stores lattices in its general purpose archive format, which can be either
binary or text. By default archives are saved in binary form. `lattice-copy`
command can be used for converting between binary and text formats, as well as
Lattice and CompactLattice.

Notice that the lattice states do not necessarily correspond to words (as the
nodes usually do in SLF lattices).

### Working with Kaldi lattices

`lattice-best-path` is a simple utility to decode the best path of lattices and
output word IDs. A word symbol table can be provided for mapping the IDs to
words, but that affects the debug output only. In order to write text
transcripts, one can output in text format and convert the word IDs to words
using `int2sym.pl` (excluding the first field, which is the utterance ID):

```bash
lattice-best-path --lm-scale=12
                  --word-symbol-table=LANG-DIR/words.txt \
                  "ark:zcat DECODE-DIR/lat.N.gz |" ark,t:- |
utils/int2sym.pl -f 2- LANG-DIR/words.txt >transcript.ref
```

`lattice-1best` command can be used to write the best path as a linear FST
(an FST with only one path). Transition IDs, language model costs, acoustic
costs, and transcripts can be extracted from the linear FST using
`nbest-to-linear` utility:

```bash
lattice-1best --lm-scale=12
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
First the lattice must be aligned with word boundaries using
`lattice-align-words`, so that the transition IDs on each arc correspond with a
single word. Again the word IDs (the fifth field) need to be converted to words:

```bash
lattice-1best --lm-scale=12
              "ark:zcat DECODE-DIR/lat.N.gz |" ark:- |
lattice-align-words LANG-DIR/phones/word_boundary.int \
                    MODEL-DIR/final.mdl \
                    ark:- ark:- |
nbest-to-ctm ark:- - |
utils/int2sym.pl -f 5 LANG-DIR/words.txt >transcript.ctm
```

The CTM file will contain five fields on each line: utterance ID, audio channel,
begin time in seconds, duration in seconds, and the word.
