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
you can also print the computed test values (and their `dtype`, `shape`, etc.
attributes).

Evaluation of the expressions using test values is enabled by setting the
`compute_test_value` configuration attribute. Naturally executing the graph
using the test values introduces computational overhead, so you probably don’t
want to keep this enabled except when you need to debug an error in the graph.
The example below prints probably 9, which has been computed using the test
value provided for the input data, while the actual data is not available yet:

```python
import numpy
from theano import tensor, config
config.compute_test_value = 'warn'
data = tensor.matrix('data', dtype='int64')
data.tag.test_value = numpy.random.randint(0, 10, size=(100, 100))
maximum = data.max()
print(maximum.tag.test_value)
```

### Printing

Printing the actual value of a tensor during computation is possible using a
`theano.printing.Print` operation. The constructor takes an optional message
argument. The data that is given to the created operation object as an argument
will be printed during execution of the graph, and also passed on as the output
of the operation.

That’s not very convenient. In order to print something, you have to use the
value returned by the print operation in the computation graph, so it's not
trivial to print e.g. the shape of a matrix. (You could use the value in e.g.
an assertion just to get it printed.) If you encounter an error while compiling
the function, this doesn’t help. In that case you can print the test value only.
But the print operation can be used to print values computed from the actual
inputs, if necessary.

```python
import numpy
from theano import tensor, printing, function
data = tensor.matrix('data', dtype='int64')
identity = tensor.identity_like(data)
print_op = printing.Print("identity:")
identity = print_op(identity)
f = function([data], identity.sum())
toy_data = numpy.arange(9).reshape(3, 3)
f(toy_data)
```

The above example prints the identity matrix in the middle of the graph:

```python
identity: __str__ = [[1 0 0]
 [0 1 0]
 [0 0 1]]
```

### Assertions

Often one would like to make sure that the result of an operation has for
example the shape that was intended. There is an assertion operation that works
in the same way as the print operation. A `theano.tensor.opt.Assert` object is
added somewhere in the graph. The constructor takes an optional message
argument. The first argument is the data that will be passed on as the output of
the operation, and the second argument is the assertion. Note that the assertion
has to be a tensor, so for comparison you'll have to use `theano.tensor.eq()`,
`theano.tensor.neq()`, etc. The example below verifies that the number of output
and input dimensions are equal:

```python
import numpy
from theano import tensor, function
data = tensor.matrix('data', dtype='int64')
identity = tensor.identity_like(data)
assert_op = tensor.opt.Assert("Shape mismatch!")
output = assert_op(identity, tensor.eq(identity.ndim, data.ndim))
f = function([data], output)
toy_data = numpy.arange(9).reshape(3, 3)
f(toy_data)
```

Assertions are not very convenient to use either, and in case an assertion
fails, the printed message usually gives you less information than if you simply
let the computation continue until the next error.

### Unit tests

Testing the correctness of some higher level functions that use neural networks
is difficult because of the nondeterministic nature of neural networks. I have
separated the network structure from the classes that create and use the actual
Theano functions. If I want to write a unit test for a function that performs
some operation on neural network output, I replace the neural network with a
simple dummy network.

So let’s say I have a class `Processor` that uses neural network output to
perform some task. The function `process_file()` reads input from one file and
writes output to another file.

```python
from theano import tensor, function
class NeuralNetwork:
    def __init__(self):
        self.input = tensor.scalar()
        self.output = complex_theano_operation(self.input)
class Processor:
    def __init__(self, network):
        self.compute_output = function([network.input],
                                       network.output)
    def process_file(self, input_file, output_file):
        for line in input_file:
            output_file.write(str(self.compute_output(float(line))
                              + "\n")
```

When writing unit tests for Processor, I would create a dummy neural network
that produces simple deterministic output, then pass that dummy network to
Processor before testing its functions. The trivial example below tries to
illustrate this approach:

```python
class DummyNetwork:
    def __init__(self):
        self.input = tensor.scalar()
        self.output = self.input + 1
class ProcessorTest(unittest.TestCase):
    def test_process(self):
        network = DummyNetwork()
        processor = Processor(network)
        with open('input.txt', 'r') as input_file,
             open('output.txt', 'w') as output_file:
            processor.process(input_file, output_file)
        # Assert that each line in the output file equals to the
        # corresponding line in the input file plus one.
```

### Performance Problems

Performance problems are perhaps even more challenging to track down. Looking at
the computation graph is necessary to know what Theano is actually doing under
the hood. Analyzing the graph is easier if you first try to simplify the
computation, as long as the performance issue won't disappear.

Print the computation graph using theano.printing.debugprint(). You can print
the graph at any point when you’re constructing it, but only the final graph
compiled using theano.function() shows the actual operations and memory
transfers that will take place. You can display the compiled graph of function
*f* using `theano.printing.debugprint(f)`. If you have Graphviz and pydot
installed, you can even print a pretty image using
`theano.printing.pydotprint(f, outfile="graph.png")`.

One thing that can be immediately noted on the graph is the `HostFromGpu` and
`GpuFromHost` operations. These are the expensive memory transfers between the
host computer and the GPU memory. You can also notice from the names of the
operations of the compiled graph, whether they run on GPU or not: GPU operations
have the Gpu prefix. Ideally your shared variables are stored on the GPU and you
have only one `HostFromGpu` operation in the end, as in the graph below:

```
HostFromGpu [id A] ''   136
 |GpuElemwise{Composite{((-i0) / i1)}}[(0, 0)] [id B] ''   133
   |GpuCAReduce{add}{1,1} [id C] ''   128
   | |GpuElemwise{Composite{((log((i0 + (i1 / i2))) + i3) * i4)}}[(0, 3)] [id D] ''   126
   |   |CudaNdarrayConstant{[[  9.99999997e-07]]} [id E]
   |   |GpuElemwise{true_div,no_inplace} [id F] ''   119
   |   | |GpuElemwise{Exp}[(0, 0)] [id G] ''   118
   |   | | |GpuReshape{2} [id H] ''   116
   |   | |   |GpuElemwise{Add}[(0, 1)] [id I] ''   114
   |   | |   | |GpuReshape{3} [id J] ''   112
   |   | |   | | |GpuCAReduce{add}{0,1} [id K] ''   110
   |   | |   | | | |GpuReshape{2} [id L] ''   108
   |   | |   | | |   |GpuElemwise{mul,no_inplace} [id M] ''   66
   |   | |   | | |   | |GpuDimShuffle{0,1,x,2} [id N] ''   58
   |   | |   | | |   | | |GpuReshape{3} [id O] ''   55
   |   | |   | | |   | |   |GpuAdvancedSubtensor1 [id P] ''   35
   |   | |   | | |   | |   | |layers/projection_layer/W [id Q]
```

Some operations force memory to be transferred back and forth. If you're still
using the old GPU backend (`device=gpu`), chances are that reason is that the
GPU operations are implemented for float32 only. Make sure that you set floatX
to float32 and your shared variables are float32. All the floating point
constants should be cast to numpy.float32 as well. Another example is
multinomial sampling—uniform sampling is performed on GPU, but
`MultinomialFromUniform` forces a transfer to host memory:

```
MultinomialFromUniform{int64} [id CH] ''
 |HostFromGpu [id CI] ''
 | |GpuReshape{2} [id CJ] ''
 | ...
 |HostFromGpu [id CT] ''
 | |GPU_mrg_uniform{CudaNdarrayType(float32, vector),inplace}.1 [id CU] ''
 |   |<CudaNdarrayType(float32, vector)> [id CV]
 |   |MakeVector{dtype='int64'} [id CW] ''
```
