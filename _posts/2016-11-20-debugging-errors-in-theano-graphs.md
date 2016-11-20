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
`set_subtensor()`. The following example sets the data elements to zero where
the mask is nonzero:

```python
import numpy
from theano import tensor, function
data = tensor.matrix('data')
mask = tensor.matrix('mask', dtype='int8')
indices = mask.nonzero()
output = tensor.set_subtensor(data[indices], 0)
f = function([data, mask], output)
toy_data = numpy.arange(9).reshape(3, 3)
toy_mask = numpy.zeros((3, 3), dtype='int8')
numpy.fill_diagonal(toy_mask, 1)
print(f(toy_data, toy_mask))
```

The example produces a matrix whose main diagonal has been zeroed out:

```python
[[ 0.  1.  2.]
 [ 3.  0.  5.]
 [ 6.  7.  0.]]
```

In NumPy you could simply modify the array in place, using
`data[mask == 1] = 0`.

### Test values

The biggest difference when writing writing a function with Theano versus NumPy
is of course that, when expression a mathematical operation using Theano, the
code doesn’t handle the actual data, but symbolic variables that will be used to
construct the computation graph. The concept is easy to understand, but also
easy to forget when you’re writing a Theano function, and makes debugging
somewhat harder.

The fact that the actual data is not known when building the computation graph
makes it difficult for Theano to produce understandable error messages. A
solution to this is to always set a test value, when creating a shared variable.
This allows Theano to produce an error message at the exact location where you
are adding an invalid operation to the graph.

The test value is a NumPy matrix. Usually random numbers work. The important
thing is that the shape and data type (and maybe the range of values) correspond
to the data used in the actual application. Typical errors are caused by a
mismatch in the dimensionality or shape of the arguments of an operation. The
error messages can still be quite difficult to interpret. The nice thing is that
you can also print the computed test values (and their data type).

Evaluation of the expressions using test values is enabled by setting the
`compute_test_value` configuration attribute. Naturally executing the graph
using the test values introduces computational overhead, so you probably don’t
want to keep this enabled except when you need to debug an error in the graph.
The example below prints probably 9, which has been computed using the test
value provided for the input data, while the actual data is not available yet:

```python
import numpy
from theano import tensor, function, config
config.compute_test_value = 'warn'
data = tensor.matrix('data', dtype='int64')
data.tag.test_value = numpy.random.randint(0, 10, size=(100, 100)).astype('int64')
maximum = data.max()
print(maximum.tag.test_value)
```
