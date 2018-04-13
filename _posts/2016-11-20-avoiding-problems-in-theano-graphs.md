---
title: "Avoiding problems in Theano computation graphs"
classes: wide
---

## Notes on writing, testing, and debugging Theano computation graphs

### NumPy as a reference

Theano interface is made as similar to NumPy as possible. Theano code often
closely resembles NumPy code, but the interface is limited and some differences
are necessary because of how Theano works. If you're not confident that the
matrix operations do what you intended, you can esily test them by running the
same operations in an interactive Python session using NumPy.

While Theano documentation is not perfect, it often helps to look at the
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
is obviously that when expressing a mathematical operation using Theano, the
Python code doesn’t process the actual data, but symbolic variables that will be
used to construct the computation graph. The concept is easy to understand, but
also easy to forget when you’re writing a Theano function, and makes debugging
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
of the operation. In order to print something, the computation graph has to use
the value returned by the print operation. Printing e.g. the shape of a matrix
would be difficult, but luckily the `Print` constructor takes another parameter
`attrs` for the purpose of printing certain attributes instead of the value of a
tensor.

If you encounter an error while compiling the function, this doesn’t help. In
that case you can print the test value only. But the print operation can be used
to print values computed from the actual inputs, if necessary. This example
prints `identity shape = (3, 3)` during the execution of the graph:

```python
import numpy
from theano import tensor, printing, function
data = tensor.matrix('data', dtype='int64')
identity = tensor.identity_like(data)
print_op = printing.Print("identity", attrs=['shape'])
identity = print_op(identity)
f = function([data], identity.sum())
toy_data = numpy.arange(9).reshape(3, 3)
f(toy_data)
```

### Assertions

Often one would like to make sure for example that the result of an operation
is of the correct shape. There is an assertion operation that works in the same
way as the print operation. A `theano.tensor.opt.Assert` object is added
somewhere in the graph. The constructor takes an optional message argument. The
first argument is the data that will be passed on as the output of the
operation, and the second argument is the assertion. Note that the assertion has
to be a tensor, so for comparison you'll have to use `theano.tensor.eq()`,
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

### Performance issues in computation graph

Performance problems can be very challenging to track down. Looking at the
computation graph is necessary to know what Theano is actually doing under the
hood. Analyzing the graph is easier if you first try to simplify the
computation, as long as the performance issue won't disappear.

Print the computation graph using `theano.printing.debugprint()`. You can print
the graph at any point when you’re constructing it, but only the final graph
compiled using `theano.function()` shows the actual operations and memory
transfers that will take place. You can display the compiled graph of function
`f` using `theano.printing.debugprint(f)`. If you have Graphviz and pydot
installed, you can even print a pretty image using
`theano.printing.pydotprint(f, outfile="graph.png")`.

One thing that can be immediately noted on the graph is the `HostFromGpu` and
`GpuFromHost` operations. These are the expensive memory transfers between the
host computer and the GPU memory. You can also notice from the names of the
operations of the compiled graph, whether they run on GPU or not—GPU operations
have the Gpu prefix. Ideally your shared variables are stored on the GPU and you
have only one `HostFromGpu` operation in the end, as in the graph below:

```
HostFromGpu [id A] ''   136
 |GpuElemwise{Composite{((-i0) / i1)}}[(0, 0)] [id B] ''   133
   |GpuCAReduce{add}{1,1} [id C] ''   128
   | |GpuElemwise{Composite{((log((i0 + (i1 / i2))) + i3) * i4)}}
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
using the old GPU backend (`device=gpu`), chances are that the reason is that
the GPU operations are implemented for float32 only. Make sure that you set the
flags `floatX=float32` and that your shared variables are float32. All the
floating point constants should be cast to `numpy.float32` as well. Another
example is multinomial sampling—uniform sampling is performed on GPU, but
`MultinomialFromUniform` forces a transfer to host memory:

```
MultinomialFromUniform{int64} [id CH] ''
 |HostFromGpu [id CI] ''
 | |GpuReshape{2} [id CJ] ''
 | ...
 |HostFromGpu [id CT] ''
 | |GPU_mrg_uniform{CudaNdarrayType(float32, vector),inplace}.1
 |   |<CudaNdarrayType(float32, vector)> [id CV]
 |   |MakeVector{dtype='int64'} [id CW] ''
```

### Profiling performance

Profiling is important after making changes to a Theano function, to make sure
that the compiled code won't run inefficiently. Profiling can be enabled by
setting the flag `profile=True`, or for certain functions individually by
passing the argument `profile=True` to `theano.function()`.

When profiling is enabled, the function runs very slowly, so if your program
calls it repeatedly, you probably want to exit after a few iterations. When the
program exits, theano prints several tables. I have found the Apply table to be
the most useful. It displays the time spent in each node of the computation
graph, in descending order:

```
Apply
------
<% time> <sum %> <apply time> <time per call> <#call> <id>
  89.2%    89.2%     383.470s       2.52e-01s   1523   116
    input 0: dtype=float32, shape=(10001,), strides=(1,)
    input 1: dtype=float32, shape=(18000,), strides=(1,)
    input 2: dtype=int64, shape=(18000,), strides=c
    output 0: dtype=float32, shape=(10001,), strides=(1,)
   3.6%    92.8%      15.575s       1.02e-02s   1523   111
    input 0: dtype=float32, shape=(10001,), strides=(1,)
    input 1: dtype=float32, shape=(720,), strides=(1,)
    input 2: dtype=int64, shape=(720,), strides=c
    output 0: dtype=float32, shape=(10001,), strides=(1,)
```

In the above example, the most expensive operation is node 116, which consumes
89 % of the total processing time, so there's clearly something wrong with this
operation. The ID 116 can be used to locate this node in the computation graph:

```
GpuAdvancedIncSubtensor1{inplace,inc} [id CY] ''   116
 |GpuAdvancedIncSubtensor1{inplace,inc} [id CZ] ''   111
 | |GpuAlloc{memset_0=True} [id DA] ''   22
 | | |CudaNdarrayConstant{[ 0.]} [id DB]
 | | |Shape_i{0} [id DC] ''   13
 | |   |bias [id BU]
```

It is essential to name all the shared variables by providing the `name`
argument to the their constructors. This makes it easier to understand which
function calls generated a specific part of the graph. In this case, the graph
shows that the bottleneck `GpuAdvancedIncSubtensor1` operates on the `bias`
variable and we can find the code that produced this operation. It looks like
this:

```python
value = numpy.zeros(size).astype(theano.config.floatX)
bias = theano.shared(value, name='bias')
...
bias = bias[targets]
bias = bias.reshape([minibatch_size, -1])
```

`GpuAdvancedIncSubtensor1` is responsible for updating the specific elements of
the bias vector (the elements indexed by `targets`), when the bias parameter is
updated. So how can the performance be improved? It can be difficult to know
what's wrong, especially while Theano is still under quite heavy development and
some things may be broken. If you have a working version without the performance
problem, the best bet might be to make small changes to the code to see what
causes the problem to appear.

In this particular case, turned out that a faster variant of the op,
`GpuAdvancedIncSubtensor1_dev20` was implemented only for two-dimensional input,
and the performance was radically improved by first converting the bias to 2D:

```python
bias = bias[:, None]
bias = bias[targets, 0]
```

### Memory usage

The memory usage of a neural network application can be crucial, current GPU
boards usually containing no more than 12 GB of memory. When Theano runs out of
memory, it throws an exception (`GpuArrayException` in the new backend) with the
message `out of memory`. It means that some of the variables, either the tensor
variables used as the input of a function, shared variables, or the intermediate
variables created during the execution of the graph, do not fit in the GPU
memory.

The size of the shared variables, such as neural network weights, can be easily
observed, and it is clear how the layer sizes affect the sizes of the weight
matrices. Weight matrix dimensions are defined by the number of inputs and the
number of outputs, so the weight matrices get large when two large layers follow
each other (or the number of inputs/outputs and the first/last layer are large).

The shared variables and inputs constitute just part of the memory usage,
however. Theano also needs to save the intermediate results of the graph nodes,
e.g. outputs of each layer. The size of these outputs depend on the batch size,
as well as the layer size. When Theano fails to save an intermediate result, it
prints a lot of useful information, including the opration that produced the
data, the node in the computation graph, and all the variables in the memory:

```python
Apply node that caused the error:
  GpuDot22(GpuReshape{2}.0, layer_1/W)
Toposort index: 62
Inputs types: [GpuArrayType<None>(float32),
               GpuArrayType<None>(float32)]
Inputs shapes: [(482112, 200), (200, 4000)]
Inputs strides: [(800, 4), (16000, 4)]
Inputs values: ['not shown', 'not shown']
Inputs type_num: [11, 11]
Outputs clients: [[GpuReshape{3}(GpuDot22.0,
                   MakeVector{dtype='int64'}.0)]]

Debugprint of the apply node:
GpuDot22 [id A] <GpuArrayType<None>(float32)> ''
 |GpuReshape{2} [id B] <GpuArrayType<None>(float32)> ''
 | |GpuAdvancedSubtensor1 [id C] <GpuArrayType<None>(float32)> ''
 | | |projection_layer/W [id D] <GpuArrayType<None>(float32)>
 | ...
 |layer_1/W [id BA] <GpuArrayType<None>(float32)>

Storage map footprint:
 - GpuReshape{2}.0, Shape: (482112, 200),
   ElemSize: 4 Byte(s), TotalSize: 385689600 Byte(s)
 - layer_1/W, Shared Input, Shape: (200, 4000),
   ElemSize: 4 Byte(s), TotalSize: 3200000 Byte(s)
```

The above message (slightly edited for clarity) shows that the product of two
matrices, layer 1 weight and the output of the projection layer would not fit in
the GPU memory. The output of the projection layer is 482112✕200, which takes
482112✕200✕4 = 386 MB of memory. The weight matrix is 200✕4000, so the result
would require 482112✕4000✕4 = 7714 MB of memory. Either the batch size or the
layer size needs to be reduced.

If you have multiple GPUs, the new gpuarray backend allows defining the
_context_ of shared variables, instructing Theano to place the variable in a
specific GPU. This way you can split a large model over multiple GPUs. This also
causes the computation to be performed and the intermediate results to be saved
in the corresponding GPU, when possible.

If your program is working but you want to observe the memory usage, you can
enable memory profiling by setting the flags `profile=True,profile_memory=True`.
Theano will print the peak memory usage of each function, and a list of the
largest variables.
