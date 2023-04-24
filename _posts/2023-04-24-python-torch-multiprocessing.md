---
layout: post
title:  Multiprocessing in PyTorch
date:   2023-04-24 17:40:16
description: study notes of multiprocessing in pytorch. references see below.
tags: programming
categories: mix-posts
---

# Table of Content
- [Introduction](#introduction)
__getattribute__
# References
 [official-docs](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing); [ref-1](https://superfastpython.com/multiprocessing-in-python/); [ref-2](https://www.digitalocean.com/community/tutorials/python-multiprocessing-example).

# Thread vs. Process
A process refers to a computer program. In Python a process is in fact one instance of the python interpreter that executes python instructinos (byte-code, not the code written in the program).

A thread always exists within a process and represents the manner in which instructions or code is executed. A process will have at least one thread, called the main thread. Any additional threads that we create within the process will belong to that process.

More on this see [here](https://superfastpython.com/thread-vs-process).


# Life-cycle of a Process
A process in Python is represented as an instance of the `multiprocessing.Process` class.

The life-cycle of a process is as follows:
## New child process
It is a process that has been constructed and configured by creating an instance of the `multiprocessing.Process` class.

## Running process
A new process can transition to a running process by calling the start() method.
This also creates and starts the main thread for the process that actually executes code in the process.

There are three main techniques used to create a child process, referred to as process start methods. They are: **spawm**, **fork**, and **forkserver**.

### spawm
The parent process starts fresh Python interpreter process. The child process will only inherit those resources necessary to run the process object’s run() method. In particular, unnecessary file descriptors and handles from the parent process will not be inherited. 
*Note that starting a process using this method is rather slow compared to using fork or forkserver.*. 
**Default on Windows and MacOS**.

### fork
The parent process uses `os.fork()` to fork the Python interpreter. The child process, when it begins, is effectively **identical** to the parent process. All resources of the parent are inherited by the child process. 
*Note that safely forking a multithreaded process is problematic.*
**Default on Unix**.

### forkserver
When the program starts and selects the **forkserver** start method, a server process is started. From then on, whenever a new process is needed, the parent process connects to the server and requests that it fork a new process. The fork server process is single threaded so it is safe for it to use `os.fork()`. No unnecessary resources are inherited.

## Blocked process (optional)
A running process may block in many ways if its main thread is blocked, such as reading or writing from a file or a socket or by waiting on a concurrency primitive such as a semaphore or a lock. After blocking, the process will run again.

## Terminated process
A process may terminate once it has finished executing its code or by raising an error or exception.
Note that a process cannot be terminate until:
- All non-daemon threads have terminated, including the main thread.
- All non-daemon child processes have terminated, including the main process.


# Basic Usage

## Run a Function in a Process
To run a function in another process:

1. Select a start method with `set_start_method()`.
2. Create an instance of the `multiprocessing.Process` class. 
3. Specify the name of the function via the “**target**” argument. The function executed in another process may have arguments in which case they can be specified as a *tuple* and passed to the “**args**” argument of the `multiprocessing.Process` class constructor or as a *dictionary* to the “**kwargs**” argument.
4. Call the `start()` function.
5. We can explicitly wait for the new process to finish executing by calling the `join()` function.

```python
import multiprocessing as mp

if __name__ == '__main__':
    mp.set_start_method('spawn')
    p = mp.Process(target=func, args=(arg1, arg2))
    # run the process
    p.start()
    # wait for the process to finish
    print("Waiting for the process to finish...")
    p.join()
```

## Communication (Exchanging objects) between processes
1. `Queue()`
2. `Pipe()`
```python
import multiprocessing as mp

def foo(q, i):
    q.put(f'hello from process {i}')
    
if __name__ == '__main__':
    mp.set_start_method('spawn')
    # create data holder
    q = mp.Queue()
    procs = []
    for i in range(10):
        # print(i)
        # create a new process
        p = mp.Process(target=foo, args=(q, i))
        procs.append(p)
        # run the process
        p.start()
    # get data
    print(q.qsize()) # Because of multiprocessing semantics, this number is not reliable.
    for i in range(10):
        print(q.get())
    # wait for the process to finish
    print("Waiting for the process to finish...")
    for p in procs:
        p.join()
```

## Synchronization between processes
Use `Lock()`. It allows code to claim lock so that no other process can execute the similar code until the lock has be released. 
To claim lock the, acquire() function is used and to release lock release() function is used.

```python
import multiprocessing as mp

def foo(l, q, i):
    l.acquire()
    try:
        q.put(f'hello from process {i}')
    finally:
        l.release()

if __name__ == '__main__':
    mp.set_start_method('spawn')
    lock = mp.Lock()

    # create data holder
    q = mp.Queue()
    procs = []
    for i in range(10)
        # create a new process
        p = mp.Process(target=foo, args=(lock, q, i))
        procs.append(p)
        # run the process
        p.start()
    # get data
    print(q.get())
    # wait for the process to finish
    print("Waiting for the process to finish...")
    for p in procs:
        p.join()
```

## Share State between processes
When donig concurrent programming, it is usually the best to avoid using shared state as far as possible. Especially when there are lots of processes.

### Shared Memory
Data can be stored in a shared memory map using `Value()` or `Array()`.

```python
from multiprocessing import Process, Value, Array

def f(n, a):
    n.value = 3.1415927
    for i in range(len(a)):
        a[i] = -a[i]

if __name__ == '__main__':
    num = Value('d', 0.0)
    arr = Array('i', range(10))

    p = Process(target=f, args=(num, arr))
    p.start()
    p.join()

    print(num.value)
    print(arr[:])
```

### Server Process
A manager object returned by `Manager()` controls a server process which holds Python objects and allows other processes to manipulate them using proxies.
It supports types `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` and `Array`.

```python
from multiprocessing import Process, Manager

def f(d, l):
    d[1] = '1'
    d['2'] = 2
    d[0.25] = None
    l.reverse()

if __name__ == '__main__':
    with Manager() as manager:
        d = manager.dict()
        l = manager.list(range(10))

        p = Process(target=f, args=(d, l))
        p.start()
        p.join()

        print(d)
        print(l)
```

# Advanced Usage
TO BE ADDED...
