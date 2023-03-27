---
title: "Paddle开发入门"
tags: []
date: 2023-02-08T17:02:29+08:00
layout: 'post'
draft: false
---

过年期间看到 Paddle 框架那边发了新的 issue，所以顺手修了几个，这里取几个例子来简单介绍其修复过程，

<!--more-->

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

C++ 端加检查难吗？不难，顺着 flip 查找内核实现，就能看到 `flip_kernel.cc` 里面对应的代码：

```cpp
template <typename T, typename Context>
void FlipKernel(const Context& dev_ctx,
                const DenseTensor& x,
                const std::vector<int>& axis,
                DenseTensor* out);

PD_REGISTER_KERNEL(flip,
                   CPU,
                   ALL_LAYOUT,
                   phi::FlipKernel,
                   float,
                   double,
                   int32_t,
                   int64_t,
                   bool,
                   phi::dtype::complex<float>,
                   phi::dtype::complex<double>) {}
```

也就是说，只要看 `FlipKernel` 就行了。

逻辑很合理，但是失败了，gdb 调试跑了一遍（gdb 7 支持了 python 调试），看到了和 **#49922** 相同的调用栈。

```cpp
AddressSanitizer:DEADLYSIGNAL
=================================================================
==92083==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000008 (pc 0x7faceb79eab3 bp 0x7ffe95a59990 sp 0x7ffe95a598e0 T0)
==92083==The signal is caused by a READ memory access.
==92083==Hint: address points to the zero page.
    #0 0x7faceb79eab3 in paddle::pybind::PyObject_CheckLongOrToLong(_object**) /home/work/yakun/paddle-2.4.0/Paddle/paddle/fluid/pybind/op_function_common.cc:69:8
    #1 0x7faceb7a4dd5 in paddle::pybind::CastPyArg2Ints(_object*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, long) /home/work/yakun/paddle-2.4.0/Paddle/paddle/fluid/pybind/op_function_common.cc:357:11
    #2 0x7facea7c4caf in paddle::pybind::eager_api_flip(_object*, _object*, _object*) /home/work/yakun/paddle-2.4.0/Paddle/paddle/fluid/pybind/eager_op_function.cc:1216:29
    #3 0x5025f3 in PyCFunction_Call (/usr/bin/python3.8+0x5025f3) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #4 0x500db4 in _PyObject_MakeTpCall (/usr/bin/python3.8+0x500db4) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #5 0x566223 in _PyEval_EvalFrameDefault (/usr/bin/python3.8+0x566223) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #6 0x55f470 in _PyEval_EvalCodeWithName (/usr/bin/python3.8+0x55f470) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #7 0x5016c5 in _PyFunction_Vectorcall (/usr/bin/python3.8+0x5016c5) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #8 0x560136 in _PyEval_EvalFrameDefault (/usr/bin/python3.8+0x560136) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #9 0x55f470 in _PyEval_EvalCodeWithName (/usr/bin/python3.8+0x55f470) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #10 0x55f102 in PyEval_EvalCode (/usr/bin/python3.8+0x55f102) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #11 0x62a1ef  (/usr/bin/python3.8+0x62a1ef) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #12 0x62a179  (/usr/bin/python3.8+0x62a179) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #13 0x47a7f2  (/usr/bin/python3.8+0x47a7f2) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #14 0x47a5cb in PyRun_SimpleFileExFlags (/usr/bin/python3.8+0x47a5cb) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #15 0x4247dc in _init (/usr/bin/python3.8+0x4247dc) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #16 0x5fb9b8 in Py_BytesMain (/usr/bin/python3.8+0x5fb9b8) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
    #17 0x7fad0bf9783f in __libc_start_main /build/glibc-S7Ft5T/glibc-2.23/csu/../csu/libc-start.c:291
    #18 0x5fb8b8 in _start (/usr/bin/python3.8+0x5fb8b8) (BuildId: 69b06f9a4b2e8428d7e32aa682c34a91dc0b961e)
AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/work/yakun/paddle-2.4.0/Paddle/paddle/fluid/pybind/op_function_common.cc:69:8 in paddle::pybind::PyObject_CheckLongOrToLong(_object**)
==92083==ABORTING
```

可以看到，调用栈完全没进入到 `FlipKernel` 中，而是停在了 `eager_api_flip`，这个自动生成的文件里，并且更具体的，是 `CastPyArg2Ints` 这个函数。

这个函数简单来说，是将类型为 `PyObject` 的 `python` 变量转换成 `std::vector<int>` 的 `cpp` 值，首先判断输入变量是否可迭代，如果可迭代，就将每个元素转换为 `int`（`long`）。

而问题就出在这个转换过程：先检查是否能转换为 `long`，如果能，则进行转换，否则抛出错误。

```cpp
bool PyObject_CheckLongOrToLong(PyObject** obj) {
  if ((PyLong_Check(*obj) && !PyBool_Check(*obj)) ||
      PyObject_IsInstance(*obj, (PyObject*)g_vartype_pytype) ||  // NOLINT
      PyObject_IsInstance(*obj, (PyObject*)g_varbase_pytype) ||  // NOLINT
      PyObject_IsInstance(*obj, (PyObject*)p_tensor_type)) {     // NOLINT
    return true;
  }
```

如果 `obj` 是（能转换到） `long` 并且 `obj` 不是 `bool`，则返回 `true`。
**或者，如果 `obj` 是 `Variable` 或 `Tensor`，返回 `true`**。

- 当 `item` 为 `long` 的时候
  - 检查能否转换 `PyObject_CheckLongOrToLong` 结果为 `true`, 检查通过
  - 进行 `PyLong_AsLong`，**得到一个 long 变量的指针**
- 当 `item` 为 tensor<long> 的时候
  - 检查能否转换 `PyObject_CheckLongOrToLong` 结果为 true（不确定内部逻辑通过原因）, 检查通过
  - 进行 `PyLong_AsLong`，**得到一个空指针**

因而出错。

简单来说，`PyObject_CheckLongOrToLong` 对 `Variable` 和 `Tensor` 这种复杂类型的检查，不够详尽。

最后进行的修复策略也同样简单。

```cpp
  if ((PyLong_Check(*obj) && !PyBool_Check(*obj)) ||
      PyObject_IsInstance(*obj, (PyObject*)g_vartype_pytype) ||  // NOLINT
      PyObject_IsInstance(*obj, (PyObject*)g_varbase_pytype) ||  // NOLINT
      PyObject_IsInstance(*obj, (PyObject*)p_tensor_type)) {     // NOLINT
      (PyObject_IsInstance(*obj, (PyObject*)p_tensor_type) &&    // NOLINT
       (((TensorObject*)(*obj))->tensor.numel() == 1))) {        // NOLINT
    return true;
  }
```

即检查是否是仅存在单个元素，如果是，则表明其支持转换。

通过添加该检查，能够解决该问题。

但是通过敏锐的观察力，不难发现，对于其他数据类型，如 `float` 等，同样存在该问题。

同时，也不难发现，对于 `Variable` (`static`)，同样存在该问题。

因此短期小目标是对这些检查代码进行修改，以提高其稳健性。
