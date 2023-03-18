---
layout: post
title: "Sharing memory between Python processes in different containers"
date: 2022-11-03 
tags:
  - Docker
  - Hack
  - Python
---

Nowadays, a microservices-oriented architecture is the norm in terms of software architecture design. This is great, especially compared to those old, huge and difficult monolith projects that "used to" exist before. 

One requirement that comes with using multiple isolated containers is how you communicate them. You need to add some logic into: REST, GRPC or whatever. Well, this post is about a very special way to share things between them, a way you should never use. 

This idea came to me while working for a service that runs inference on a deep learning model. It was a very simple [FastAPI service](https://fastapi.tiangolo.com/) that received a request with an image and returned the inference result. 

Once it was built, I started doing some metrics analysis and discovered that it was spending a lot of time loading the image into a numpy array. This really pissed me off because I really tried to squeeze the time as much as possible: I had already set the half-precision mode and performed weight pruning and layer merging (using the magical [TensorRT SDK](https://developer.nvidia.com/tensorrt)). I literally could not improve efficiency anywhere. As the model was already in memory, the service was limited to open the image in a numpy array, do inference and return the answer. 

I started looking at the other services involved in the overall process. There was one service that was particularly interesting, the one that downloads the image, resizes it, and saves it to a shared EBS volume (where the inference service takes it from).

The reason why this logic is split into different services in separate containers is that GPU time is far more expensive than CPU time. Thus, it is better to have a service that is fully using the GPU all the time (performing inference) and not wasting time downloading or resizing. 


Wouldn't it be great to just open it once (in the resize service) and then share that object between processes? something like sharing the actual memory location or something? We would eliminate the load time of that object in memory, which would be even a better solution than collecting the numpy array in a tmpfs/ramfs and then opening it again in the other service (this was my first real approach).

At first glance this seemed impossible to me. I experimented a bit with sharing memory between python processes and it was working (everything is possible using Python <3). However... What about docker? That's a big nono. Right?
### How?

It is necessary to create docker instances that share the same IPC namespace (Inter-Process Communication) in order to have shared memory segments between dockers. In this case I will use the host for both but you could create a container with the IPC property <i>shareable</i> and another one using that one with <i>“container:<_name-or-ID_>"</i>. 

**TERMINAL 1**
```python
docker run -it --ipc host python
Python 3.9.5 (default, May 12 2021, 15:26:36)
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from multiprocessing import shared_memory
>>> from sys import getsizeof
>>> shm_a = shared_memory.SharedMemory(create=True, size=10)
>>> len(buffer)
10
>>> buffer[:]=bytes(b'HelloWorld')
>>> shm_a.name
'psm_3de3651a'
# now it is time to go to the other terminal and retrieve this string that is already in memory 
```

**TERMINAL 2**
```python
docker run -it --ipc host python
Python 3.9.5 (default, May 12 2021, 15:26:36)
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from multiprocessing import shared_memory
>>> existing_shm = shared_memory.SharedMemory(name='psm_3de3651a')
>>> bytes(existing_shm.buf[:])
b'HelloWorld' 
```

Isn't it wonderful? This can also be used with numpy arrays very easily by creating a numpy array backed by a shared memory segment. 

**TERMINAL 1**
```python
>>> import numpy as np
>>> a = np.array([1, 1, 2, 3, 5, 8])  # Start with an existing NumPy array
>>> shm_a = shared_memory.SharedMemory(create=True, size=a.nbytes)
>>> # Now create a NumPy array backed by shared memory
>>> b = np.ndarray(a.shape, dtype=a.dtype, buffer=shm_a.buf)
>>> b[:] = a[:]  # Copy the original data into shared memory
>>> b
array([1, 1, 2, 3, 5, 8])
>>> shm_a.name
'psm_3de3651a'
```

Now just open the existing shared memory (as we did before) and instantly retrieve the numpy array from memory: 

**TERMINAL 2**
```python
existing_shm = shared_memory.SharedMemory(name='psm_3de3651a')
c = np.ndarray((6,), dtype=np.int64, buffer=existing_shm.buf)
```

> Note that the creation of the shared memory object may take a few precious milliseconds (at least the first time). This is just me messing around. 
