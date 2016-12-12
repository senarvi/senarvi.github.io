---
layout: post
title: "Kaldi lattices"
description: "Practical knowledge about Kaldi lattices and converting them to SLF"
category: 
tags: []
---
{% include JB/setup %}

During speech recognition, the decoder explores a search space that in principle
contains every possible sentence. It is often useful to save a subset of this
search space as a graph that can be efficiently rescored and decoded using
different, perhaps computationally more demanding, language models.

HTK recognizer defined the Standard Lattice Format (SLF), which has been widely
adopted. It is basically a text file that lists the graph nodes and links. The
nodes may contain time stamps so that the words can be aligned with audio.
Usually the word identities, and language model and acoustic scores
(log-likelihoods) are stored in the links.

Lattices are intrinsic to Kaldi, and commonly generated during decoding, but
they are represented differently, as finite-state transducers (FSTs). Acoustic
and language model scores define the weights of the FSTs, but the numbers
are a bit different from those that one finds in SLF lattices::

1. The numbers in Kaldi lattices are costs, i.e. negative log-likelihoods.
2. When a neural network acoustic model is used, the probabilities are not
   correctly normalized. The neural network predicts posterior probabilities,
   which will be divided by the prior probabilities of the HMM states to obtain
   pseudo-likelihoods (likelihoods scaled by a constant scaling factor). It is
   possible for the pseudo-probabilities to be higher than one.
3. Language model probability is incorporated in *graph cost*, which also
   includes pronunciation and transition probabilities. In order to preserve the
   rest of the graph cost when rescoring using a new language model, one would
   have to first subtract the old language model cost. Usually the other costs
   are not that important, though.

There are actually two alternative FST representations for lattices in Kaldi,
implemented by the Lattice and CompactLattice classes. Lattice is an FST whose
input is *transition IDs*, which correspond to HMM states, and output is word
identities. CompactLattice has the word identity in both input and output, and
transition IDs are included in the weights.
