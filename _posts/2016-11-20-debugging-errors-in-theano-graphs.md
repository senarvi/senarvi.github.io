---
layout: post
title: "Debugging errors in Theano graphs"
description: "Notes about debugging errors when building computation graphs using Theano"
category: 
tags: []
---
{% include JB/setup %}

### Theano and NumPy

Theano interface is made as similar to NumPy as possible. Theano code often
closely resembles NumPy code, but the interface is limited and some differences
are necessary because of how Theano works. It might not always be clear that the
matrix operations do what you intended. It's easy to test them by running the
same operations in an interactive Python session using NumPy.

While Theano documentation is not perfect, it often helps to look a the
corresponding NumPy documentation. If you’re new to both Theano and NumPy, you
should at least familiarize yourself with
[broadcasting](http://docs.scipy.org/doc/numpy/user/basics.broadcasting.html), and
[slicing and indexing](https://docs.scipy.org/doc/numpy/reference/arrays.indexing.html),
which are explained more thoroughly in NumPy documentation.

A couple notable deviations from NumPy are worth mentioning, though. Theano uses
integers to represent booleans. Thus you cannot index a matrix with a boolean
matrix. You’ll have to convert the boolean matrix to a matrix of indices with
`nonzero()`. And since Theano never modifies a tensor, for creating a tensor
where a subset of the matrix elements have been updated, you need to call
`set_subtensor()`. For example, this would set the elements of a matrix to zero,
where mask is nonzero:

```python
from theano import tensor
indices = mask.nonzero()
matrix = tensor.set_subtensor(matrix[indices], 0)
```
