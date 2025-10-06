---
title: "Find first occurrences in PyTorch"
---

Recently I needed to solve a seemingly simple problem in PyTorch. The input is a 1-dimensional tensor containing integer values. To make the discussion easier, let's call them codes. The task is to find the first location where each code occurs. In NumPy, this can be done easily.

```python
import numpy as np

def find_first_occurrences(codes: np.ndarray) -> np.ndarray:
    return np.unique(codes, return_index=True)[1]
```

Given the input `[2, 1, 9, 1, 7, 2, 2, 1, 5]`, the function would return `[1, 0, 8, 4, 2]`. The indices are returned in that order, because `np.unique()` sorts the codes first. That is, the first item is the first index of 1, the second item is the first index of 2, the third item is the first index of 5, and so on.

This doesn't work in PyTorch, because `torch.unique()` doesn't support the `return_index` argument. I found a very clever solution on [StackOverflow](https://stackoverflow.com/a/72005790) and thought it deserves some attention. This is the algorithm (with a small change to make sure that this works also when the input tensor is not on CPU):

```python
import torch

def find_first_occurrences(codes: torch.Tensor) -> torch.Tensor:
    _, inverse_idxs, counts = codes.unique(return_inverse=True, return_counts=True)
    grouped_idxs = inverse_idxs.argsort(stable=True)
    group_start_idxs = counts.cumsum(0).roll(1)
    group_start_idxs[0] = 0
    return grouped_idxs[group_start_idxs]
```

`codes.unique()` finds the unique codes and sorts them (but we don't need them). We use the `return_inverse` argument, which makes the function return the inverse mapping—from an input location to the unique codes—and the `return_counts` arguments, which returns the counts of the codes.

Let's take as an example the input `[2, 1, 9, 1, 7, 2, 2, 1, 5]`. In this case, `inverse_idxs` would be `[1, 0, 4, 0, 3, 1, 1, 0, 2]`. This tells us, for example, that the smallest code appears at the second, fourth, and eighth position. `counts` would be `[3, 3, 1, 1, 1]`. This tells us that the two smallest codes appear three times each and the other three appear once in the input.

The idea is to group together items that point to the same code. This is done by `inverse_idxs.argsort(stable=True)`. It produces a list where we first have the indices of the smallest code, then the second smallest code, and so on. Following the same example, `grouped_idxs` would be `[1, 3, 7, 0, 5, 6, 8, 4, 2]`. Based on the counts, we know that the first three items are the indices of code 1, the next three items are the indices of code 2, and these are followed by the index of code 5, 7, and 9. Since we use stable sorting, we know that the first item is the first index of code 1 and the fourth item is the first index of code 2, and generally *the first item in every group is the first index of the corresponding code*.

We use the counts to construct `group_start_idxs`, which can be used to collect the first item in each group. `counts.cumsum(0)` returns the cumulative sum of the code counts, which is almost what we need. In the above example, it would return `[3, 6, 7, 8, 9]`. The last item is not needed, but the other items tell where each group starts, excluding the first group which obviously starts at index 0. So we shift the values and set the first value to zero, obtaining `[0, 3, 6, 7, 8]`.

Finally, we read the first index of each code from `grouped_idxs`, from the locations pointed out by `group_start_idxs`. In our example, we would get `[1, 0, 8, 4, 2]`.
