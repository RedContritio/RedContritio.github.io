---
title: Paddle开发入门
date: 2023-02-08 17:02:29
tags:
---

过年期间看到 Paddle 框架那边发了新的 issue，所以顺手修了几个，这里取几个例子来简单介绍其修复过程，

## paddle.flip

事实上的首个动手目标，它的出错输入是这样的：

```python
import paddle
import numpy as np
from paddle import flip
x = paddle.to_tensor(np.random.uniform(-10, 10, [1, 2, 3]).astype(np.int64)),
axis = paddle.to_tensor(np.random.uniform(-2147483648, 2147483647, [3, 3]).astype(np.int32))
print(x)
print(axis)
flip(x, axis)
```

翻到 Paddle 源码看看

```python
def flip(x, axis, name=None):
    """
    Reverse the order of a n-D tensor along given axis in axis.

    Args:
        x (Tensor): A Tensor(or LoDTensor) with shape :math:`[N_1, N_2,..., N_k]` . The data type of the input Tensor x
            should be float32, float64, int32, int64, bool.
        axis (list|tuple|int): The axis(axes) to flip on. Negative indices for indexing from the end are accepted.
        name (str, optional): Name for the operation (optional, default is None). For more information, please refer to :ref:`api_guide_Name`.

    Returns:
        Tensor, Tensor or LoDTensor calculated by flip layer. The data type is same with input x.

    Examples:
        .. code-block:: python

          import paddle

          image_shape=(3, 2, 2)
          img = paddle.arange(image_shape[0] * image_shape[1] * image_shape[2]).reshape(image_shape)
          tmp = paddle.flip(img, [0,1])
          print(tmp) # [[[10,11],[8, 9]], [[6, 7],[4, 5]], [[2, 3],[0, 1]]]

          out = paddle.flip(tmp,-1)
          print(out) # [[[11,10],[9, 8]], [[7, 6],[5, 4]], [[3, 2],[1, 0]]]
    """
    if isinstance(axis, int):
        axis = [axis]

    if in_dygraph_mode():
        return _C_ops.flip(x, axis)
```

思路很明显，flip 期望输入一个一维的数组，但是错误的案例里输入了二维的。因此解决方案就更明显了，不妨加一个维度检查。

加在哪呢？很直观的，既然上面的 python 代码里处理了输入单个整数时将其转换为整数，那就给它加上一个维度检查，形如 

```python
    if paddle.to_tensor(axis).ndim != 1:
        raise ValueError('维度不对，麻烦再检查检查，只要一维的')
```

非常合理，而且简单易懂，维度不为 1 就报错，于是事情就这样成了（并没有）。

因为 [Good First Issue Lists](https://github.com/PaddlePaddle/Paddle/issues/49927) 里面还有这样的对话。

> > 您好，想问下是要在 Python 端加输入检查，还是要在C++端用 `PADDLE_ENFORCE_EQ` 做检查呢 ?
> 
> 根据情况而定。能够在C++端加的，一定要在C++端加。如果C++端不具备条件的，则在Python端加。

也就是说，应该尽可能在 C++ 端加对应检查。