# :snake: depyf: Decompile python functions, from bytecode to source code!

This is used primarily to understand the bytecode produced by PyTorch 2.0 Dynamo (PT 2.0 compiler stack).

# Installation

`pip install depyf` or `pip install git+https://github.com/youkaichao/depyf.git`

# Usage

## Simple Usage:

```python
# obtain a callable object or codeobject
def func():
    print("hello, world!")
# import the `decompile` function
from depyf import decompile
# and decompile it into source code!
print(decompile(func))
```

## Used to understand PyTorch generated bytecode

First, run a pytorch program with `torch.compile`:

```python
from typing import List
import torch
from torch import _dynamo as torchdynamo
def my_compiler(gm: torch.fx.GraphModule, example_inputs: List[torch.Tensor]):
    print("my_compiler() called with FX graph:")
    gm.graph.print_tabular()
    return gm.forward  # return a python callable

@torchdynamo.optimize(my_compiler)
def toy_example(a, b):
    x = a / (torch.abs(a) + 1)
    if b.sum() < 0:
        b = b * -1
    return x * b
for _ in range(100):
    toy_example(torch.randn(10), torch.randn(10))
```

Second, get compiled code and guard code from pytorch:

```python
from torch._dynamo.eval_frame import _debug_get_cache_entry_list
cache_entries = _debug_get_cache_entry_list(toy_example._torchdynamo_orig_callable.__code__)
guard, code = cache_entries[0]
```

Third, decompile the code to see how the code works:

```python
from depyf import decompile

print("guard code:")
print(decompile(guard))

print("compiled code:")
print(decompile(code))
```

Hopefully, by using this package, you can understand python bytecode now!

:warning: The above example should be run using pytorch nightly. Some debug functions like `_debug_get_cache_entry_list` might not exist in stable releases yet.

# Python Version Coverage

The following python major versions are tested:

- Python 3.10

You can see the coverage report by simply running `python python_coverage.py`.

# Full Python Syntax Is Not Supported

This package is intended to understand the generated pytorch bytecode, and does not aim to fully cover all the syntax of python. For example, async operations like `async/await` is not supported.