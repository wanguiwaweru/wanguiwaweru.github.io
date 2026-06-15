---
title: "PyTorch: A Deep Learning Framework"
date: "2026-06-07"
draft: true
tags: [PyTorch, Deep Learning, Machine Learning, Tensors, GPU]
---

# PyTorch: A Deep Learning Framework

PyTorch is an open-source machine learning library developed by Facebook's AI Research lab (FAIR). It provides a flexible and dynamic computational graph, making it easier for researchers and developers to build and train deep learning models. PyTorch is widely used in both academia and industry for various applications, including computer vision, natural language processing, and reinforcement learning.

## Tensors
Tensors are the fundamental data structure in PyTorch, similar to NumPy arrays but with additional capabilities for GPU acceleration and automatic differentiation. They can be created from Python lists or NumPy arrays and can be manipulated using a wide range of operations.

Use `torch.tensor()` to create a tensor from a Python list or NumPy array. For example, 

```python
import torch

data = [[1, 2], [3, 4]]
tensor = torch.tensor(data)
print(tensor)
```
This will create a 2D tensor with the values from the list. You can also specify the data type of the tensor using the `dtype` argument, such as `torch.float32` or `torch.int64`.

Create a tensor from a desired shape using functions like `torch.zeros()`, `torch.ones()`, or `torch.rand()`. For example, `torch.zeros((3, 4))` will create a 3x4 tensor filled with zeros.

This is useful for initializing weights in neural networks or creating tensors of a specific shape for computations  where you know the desired dimensions but not the specific values.

```python
# Create a tensor of shape (3, 4) filled with zeros
tensor_zeros = torch.zeros((3, 4))
print(tensor_zeros) # Output: tensor([[0., 0., 0., 0.],
              #[0., 0., 0., 0.],
              #[0., 0., 0., 0.]])

tensor_ones = torch.ones((2, 5))
print(tensor_ones) # Output: tensor([[1., 1., 1., 1., 1.],
              #[1., 1., 1., 1., 1.]])

tensor_rand = torch.rand((4, 4))
print(tensor_rand) # Output: tensor([[0.1234, 0.5678, 0.9101, 0.1121],
              #[0.3141, 0.5161, 0.7181, 0.9202],
              #[0.2233, 0.4455, 0.6677, 0.8899],
              #[0.3344, 0.5566, 0.7788, 0.9901]])           
```

Create a tensor by mimicking the shape and properties of an existing tensor using functions like `torch.zeros_like()`, `torch.ones_like()`, or `torch.rand_like()`or `torch.randn_like`. For example, `torch.zeros_like(existing_tensor)` will create a new tensor with the same shape as `existing_tensor` but filled with zeros. This is useful when you want to create a new tensor that has the same dimensions as an existing one but with different values, such as initializing a new tensor for storing gradients or creating a mask tensor based on the shape of an existing tensor.

> This copy the shape, data type (dtype), memory layout, and hardware device (CPU/GPU) of a reference input tensor.

`torch.randn_like` (The "n" stands for Normal) generates values using a standard normal distribution where most generated values cluster tightly around 0.0. Values further from zero (like -2.5 or 3.1) are progressively less likely to occur.

`torch.rand_like` generates values using a uniform distribution between 0.0 and 1.0(inclusive), where all values in this range are equally likely to occur. It will never generate values outside of this range, so you won't see negative numbers or values greater than 1.0.

> Note the core difference between `torch.randn_like` and `torch.rand_like` lies entirely in the statistical distribution they use to sample random values.

Create a tensor from an existing tensor using functions like `torch.clone()`, `torch.detach()`, or `torch.to()`. For example, `tensor.clone()` will create a copy of the original tensor, while `tensor.detach()` will create a new tensor that shares the same data but does not require gradients. This is useful when you want to create a new tensor based on an existing one but with different properties, such as not tracking gradients for backpropagation or moving the tensor to a different device (e.g., from CPU to GPU).

Create a tensor from a file using functions like `torch.load()` or `torch.from_numpy()`. For example, `torch.load('tensor.pt')` will load a tensor that was previously saved to a file using `torch.save()`. This is useful for saving and loading models, checkpoints, or any tensor data that you want to persist across sessions or share with others.

### Tensor Attributes
- `shape`: Returns the dimensions of the tensor as a tuple. For example, if you have a tensor with shape (3, 4), it means the tensor has 3 rows and 4 columns.
- `dtype`: Returns the data type of the tensor, such as `torch.float32` or `torch.int64`. This indicates the type of data stored in the tensor, which can affect memory usage and computational performance.
- `device`: Returns the device on which the tensor is allocated, such as 'cpu' or 'cuda'. This indicates whether the tensor is stored in CPU memory or GPU memory, which can impact the speed of computations performed on the tensor.

#### Data Types
The data type of a tensor determines the type of values it can hold and how much memory it consumes. Common data types in PyTorch include:
- `torch.float32`: A 32-bit floating-point number, commonly used for neural network weights and activations.
- `torch.float64`: A 64-bit floating-point number, used for higher precision calculations.
- `torch.int64`: A 64-bit integer, used for indexing and counting.  
- `torch.bool`: A boolean data type, used for binary values (True/False). Used for masks to create other tensors, conditionional filtering, and logical operations.

### Parameter tensors
In PyTorch, a parameter tensor is a special type of tensor that is used to represent the learnable parameters of a model, such as weights and biases. 

To make a tensor a parameter tensor, set `requires_grad=True` when creating the tensor. This tells the `Autograd` engine that this tensor should be tracked for gradient computation during backpropagation, allowing it to be updated during the training process. 

> PyTorch `Autograd` is PyTorch's built-in automatic differentiation engine that powers neural network training. It eliminates the need to manually compute calculus derivatives by automatically calculating the gradients (derivatives) required to optimize model parameters during backpropagation.
For example, when defining a neural network layer, you can create weight and bias tensors as parameter tensors. 

```python
import torch

tensor = torch.randn(3, 4, requires_grad=True)
print(tensor)
```

This will create a tensor with random values and set it to require gradients, making it a parameter tensor that can be optimized during training. when ``requires_grad=True``, PyTorch will automatically track all operations on this tensor, allowing you to compute gradients and update the parameters during the training process.

### How Autograd Works
1. Dynamic Computational Graph: As you perform math operations on tensors, Autograd tracks them in a Directed Acyclic Graph (DAG). This graph is built dynamically "on the fly" during the forward pass.
 
 2. The Chain Rule: When you trigger the backward pass, Autograd walks backward through this graph. It applies the calculus chain rule to compute partial derivatives.
 
 3. Parameter Updates: It automatically saves these calculated derivatives into each tensor's `.grad` attribute, allowing optimizers to update weights.

 > ``grad_fn`` is an attribute of a tensor that references the function that created the tensor. It is part of the computational graph that Autograd builds to track operations for automatic differentiation. When you perform an operation on a tensor that requires gradients, PyTorch creates a new tensor and sets its `grad_fn` to the function that was used to create it. This allows Autograd to know how to compute gradients during the backward pass. For example, if you perform a power operation on a tensor, the resulting tensor will have a `grad_fn` that references the power function, which will be used to compute the gradient when you call `backward()`.


```python
import torch

# 1. requires_grad=True tells Autograd to track operations on this tensor
x = torch.tensor(3.0, requires_grad=True)

# 2. Forward Pass: Autograd builds the graph for this calculation
y = x ** 2  # y = 9.0 

# 3. The grad_fn tracks the backward operation (e.g., <PowBackward0>)
print(y.grad_fn) # Outputs: <PowBackward0 object at 0x...> because y was created by squaring x, which is a power operation.

# 4. Backward Pass: Computes the derivative dy/dx = 2x
y.backward()

# 5. Access the calculated gradient (2 * 3.0 = 6.0)
print(x.grad)  # Outputs: tensor(6.)
```

##### Importance of Autograd
1. Automates Backpropagation: You do not have to write mathematical derivative formulas for complex, multi-layer neural networks.
2. Handles Dynamic Models: Because the graph is recreated every iteration, you can safely use Python loops, if conditions, and varying tensor shapes in your model.
3. Memory Control: You can wrap evaluation or inference code in a with torch.no_grad(): block. This stops Autograd from building graphs, which reduces memory usage and speeds up performance.

### Device Management
Use `torch.device` to specify the device (CPU or GPU) on which tensors should be allocated. 

> Note that this means the tensor will be in the CPU RAM or GPU VRAM, and the operations on the tensor will be performed on the respective device. At this point you have not yet touched the GPU or CPU, you have just created a small object describing which compute device you want to use for the tensor. The actual data will be allocated on the specified device when you create the tensor or move it to that device.

When you create a tensor, you can specify the device using the `device` argument. For example, to create a tensor on the GPU, you can use `torch.tensor(data, device='cuda')`. If you want to create a tensor on the CPU, you can use `torch.tensor(data, device='cpu')`.

Tensors and models must be in the same device to perform operations on them. You can move tensors and models to the desired device using the `.to()` method. For example, to move a tensor to the GPU, you can use `tensor.to('cuda')`.

PyTorch automatically handles the data transfer between devices when necessary, allowing you to write code that can run on both CPU and GPU without modification.

This allows for seamless switching between devices, enabling efficient computation on GPUs when available.

`inputs.to(device)` is used to move the input tensors to the specified device

`model.to(device)` is used to move the model's parameters and buffers to the specified device. This is important because the model's parameters need to be on the same device as the input tensors for the computations to work correctly.

`batch.to(device)` is used to move a batch of data (e.g., a batch of input tensors) to the specified device. This is typically done during the training loop to ensure that the data is on the same device as the model for efficient computation.

> Note that when you move a tensor or model to a different device, it creates a new copy of the data on that device. Therefore, it's important to manage device placement carefully to avoid unnecessary data transfers, which can impact performance.

Use `non_blocking=True` when moving tensors to the GPU to allow for asynchronous data transfer, which can improve performance by overlapping data transfer with computation. This is particularly beneficial when using a GPU, as it allows the CPU to continue executing other tasks while the data is being transferred to the GPU. 

When `non_blocking=True` is set, the data transfer will be performed asynchronously, allowing for better utilization of the GPU and potentially reducing the overall training time. However, it's important to note that this option is only effective when the source and destination devices are different (e.g., moving from CPU to GPU) and may not have an impact when moving tensors within the same device.

`pin_memory=True` is a configuration used within a `PyTorch DataLoader` to accelerate data transfers from the CPU host memory to the GPU device memory. It locks the data into "page-locked" (pinned) system memory, allowing the GPU to use Direct Memory Access (DMA) to copy the data much faster than standard pageable memory. 

> Normally, your system RAM uses pageable memory. The operating system can shift this memory around or swap it to the hard drive, meaning the GPU cannot safely copy it directly.When you activate pinning, PyTorch forces the memory allocation to remain fixed in physical RAM:Standard Copy: CPU (Pageable) → CPU (Pinned Staging) → GPU (Device).Pinned Copy: CPU (Pinned) → GPU (Device) (skips the extra staging copy).Best 

> To get the full speed benefits, combine `pin_memory=True` in your data loader with `non_blocking=True` when pushing tensors to the GPU. This allows the host CPU to keep loading the next batch while the GPU processes the current one.

Avoid it when: 
- You are training exclusively on a CPU (it introduces useless overhead), or your system is running critically low on RAM. 
- Overusing pinned memory can freeze your operating system because it prevents RAM from being swapped out or managed dynamically.

`detach()` is used to create a new tensor that shares the same data but does not require gradients. This is useful when you want to perform operations on a tensor without tracking its history for gradient computation, which can save memory and improve performance during inference or when you don't need to compute gradients.

`detach().cpu()` is a common pattern used to move a tensor from the GPU back to the CPU while also detaching it from the computation graph. This is often done when you want to convert a tensor to a NumPy array or perform operations that are more efficient on the CPU. By detaching the tensor, you ensure that it does not require gradients, and by moving it to the CPU, you can work with it in a more flexible way outside of the GPU context.

This is useful for tasks such as monitoring, logging intermediate results, or when you want to perform operations that are not supported on the GPU. It allows you to efficiently transfer data back to the CPU while ensuring that the tensor is no longer part of the computation graph, which can help reduce memory usage and improve performance in certain scenarios.


For efficient device management:
- Always ensure that your tensors and models are on the same device to avoid unnecessary data transfers.
- Only move tensors to the GPU when necessary to avoid increasing costs for tasks that don't require GPU acceleration, and consider using `non_blocking=True` for faster data transfer.
- Dynamically check for GPU availability using `torch.cuda.is_available()` and set the device accordingly to ensure your code can run on both CPU and GPU environments without modification.

### Working with Multiple GPUs
PyTorch provides several ways to work with multiple GPUs, including `torch.nn.DataParallel` and `torch.nn.parallel.DistributedDataParallel`. These modules allow you to parallelize your model across multiple GPUs, enabling faster training and inference.

# Resources
- [PyTorch Documentation](https://pytorch.org/docs/stable/index.html)
- [PyTorch Tutorials](https://pytorch.org/tutorials/)
- [PyTorch for Deep Learning: A Gentle Introduction](https://www.youtube.com/watch?v=o5gPABcGZwc&t=399s)
- [What are Float32, Float16 and BFloat16 Data Types?](https://www.youtube.com/watch?v=7q1Gh1KOlzw)

- [PyTorch Pin Memory and Non-Blocking Transfers](https://docs.pytorch.org/tutorials/intermediate/pinmem_nonblock.html)
- [What is the disadvantage of using pin_memory?](https://discuss.pytorch.org/t/what-is-the-disadvantage-of-using-pin-memory/1702)