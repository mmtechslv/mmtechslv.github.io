---
layout: post
title: Guide to improve Python performance.
tags: programming python concurrency parallelism
output: true
description: The primary objective of this guide is to summarize all of the popular approaches for boosting the execution speed of your Python code.
---

Python is an amazing programming language, but it has two huge handicaps compared to compiled languages.

- The first one is GIL or Global Interpreter Lock. GIL is an actual process lock that forces a Python interpreter to work on a single process and use only one of the cores in your CPU. Due to this lock, Python is both simple and stable but also is very slow compared to other programming languages like C or Java.
- The second handicap is rather universal for non-statically typed aka dynamically typed programming languages. Simply put, when you do not specify the data type of the variable that you are going to use and rely on dynamic type assignment, you end up with much slower execution performance.

Luckily there are several ways to speed up your Python code.

### Bypass GIL (Concurrency/GIL persist)

In this approach, the execution of the target code is performed in such a way that data is processed in parallel or concurrently. Essentially, it simply means breaking a single task into multiple separate sub-tasks and processing each in a different thread or process. This is also known as multithreading or multiprocessing. This approach is very effective, only if your task can be separated into different tasks.

Before moving on it is important to distinguish thread and process. A thread and a process are two different things, which simply can be used in a similar fashion, but for different purposes. A single process can create multiple threads, which are limited by OS. All of the threads of any single process share the same memory heap. However, in Python new thread does not mean new CPU core; hence, the thread does not give you an actual performance boost and rather helps you focus on several tasks simultaneously. Therefore, threads are cheap to create and mostly applied for I/O operations. In contrast, when you create a new process the original memory heap is copied and a new one is created. Two processes cannot see each other's memory heap, but do work on separate CPU cores. Creating a new process is expensive and ends up in larger overall memory usage. However, the new process provides you with additional CPU cores (up to maximum) and can be created locally or in different computers within a cluster. [3,4]

Here are some examples:

- Say that you have a very large file on a disk or maybe a list/array on your memory that you would like to process. Instead of processing all of the data in one single loop and using a single core on your CPU, you can break your objective/data into smaller tasks/chunks. Finally, you can then process each chunk separately in different threads or a process.
- Similarly, you can run some piece of Python code inside a single thread that would listen to a network socket for the packages that you might receive from your TCP connection.
- Another popular application of concurrency can be seen in data analysis problems. Say that you want to perform computationally expensive operations on large amounts of data but in a short time inside a cluster of computers/servers. In such a case, each process can be run inside a different computer and data can be shared via a network interface.

Python provides several built-in packages for concurrent processing.
  - **`concurrent.futures`** - This module is very easy to use but lacks advanced control.

  - **`threading`** - Useful if you want to do more complicated things with threads like locking threads or running tasks in a queue.
  
  - **`multiprocessing`** - Useful if you want to do more complicated things with processes like sharing a memory or running tasks in a queue.
 

> **Concurrency vs True Parallelism**: Parallelism based on concurrency is not true parallelism. A piece of code is executed in a truly parallel way when all of the CPU cores literally work on the same task simultaneously. True parallelism is based on the actual hardware and in general a more complicated topic than concurrency. Python's GIL prevents true parallel execution of the code. However, fortunately, there are ways to release the GIL and achieve truly parallel processing. [1]


### No GIL (True parallelism) + Optional Static Typing
In order to release GIL and boost your Python code, you are going to need to get your hands at least a little bit dirty. Python is a great high-level programming language, which is also written in other low-level programming languages. The original and most popular Python is in fact is implemented in C and this reference implementation is called CPython. However, there are other implementations of Python with its own advantages and issues. Each is developed by a different community; hence, every Python implementation is as strong as its community. [5]

> **What about Python packages?** At this point, you may ask yourself a question. Are Python packages that I currently use are also available in other Python implementations? Unfortunately, the answer is ambiguous, since it really depends on the implementation. It is definite that there are going to be some incompatibilities. In order to make sure, always check the documentation material.

Here are some popular alternative implementations of Python.
  - **IronPython** - is .NET Framework-based implementation of Python written in C# language. It is great if you love writing Python code and need to use features of the .NET Framework.

  - **Jython** - is a JVM-based implementation of Python written in Java. It is great if you love writing Python code and need to tightly integrate it with your pure Java backend.

  - **PyPy**- is an alternative implementation of the Python with a just-in-time (JIT) compiler. Compared to the reference implementation of Python (CPython), PyPy makes use of JIT that provides better performance than the original bytecode compiler implemented in CPython. Therefore, PyPy/Python has better performance than CPython/Python. Also according to PyPy documentation, PyPy supports almost all Python packages so it is a huge plus for PyPy. However, PyPy supports only 32-bit architecture and is not updated as frequently as Python.

Following are not Python implementations

- **NumPy/Pandas/Dask** - "Correct" usage of the many methods implemented in these data analysis packages will almost always give you a significant performance improvement.

- **Cython** - is not an implementation of Python. It is rather a superset of Python that compiles to C. Its syntax is almost the same as Python's but more C-like because it can understand static typing. Almost any Python code can be compiled using Cython to generate modules with the same functionality but with faster execution time. Similarly, most of the time it is easy to rewrite your C code to Cython code. In addition, many of the built-in modules and methods are optimized for Cython to produce minimal C code before compilation. Furthermore, Cython also supports NumPy and runs very efficiently when using `numpy.ndarray`. [2]
  
- **Numba** - is a NumPy-aware JIT compiler for Python. Numba is very user friendly and can be easily applied to specific pieces of Python code that you would like to speed up.


> **Important Note:**  Every approach described in this section has comparable advantages and disadvantages, best usage applications, best practices, learning curves. However, you will need to get your hands dirty if you want to get the best performance out of any approach.


### No GIL (True parallelism) + Mandatory Static Typing
The following methods are different from all the methods above because they require sufficient knowledge of a second low-level programming language like C/C++. This section rather describes tools/methods that act as a "bridge" between Python and extension written in a low-level programming language. Therefore, performance is only limited by the low-level programming language and the bridge. At this point, two important questions must be stated before we continue. [5]

- If the compiled languages like C, require compilation and linkage of the source code, is it possible to automate compilation during package distribution?
- Since compiling C code can be different based on the OS, does that mean that your extension code will not be cross-OS anymore?
- How to convert Python data types to C types and vice versa?

There is no single answer to these questions because it depends on what you want to do. Both compiler configuration and automation can be managed using a built-in Python module called `distutils`, which is very easy to use. However, it really depends on how you design your extension if you want to use it cross-OS. Nevertheless, when it comes to data types there are three logical approaches.

*Let C side handle data types:*

- **Python API (CPython)** - CPython is a reference implementation of the Python programming language. In other words, CPython is essentially the primary C code, which when compiled generates a Python interpreter and its bytecode compiler. Luckily, CPython/Python developers provide access to the core Python API for C/C++ programmers via `Python.h` include file. Extension written using this approach can be easily distributed using `distutils`. It is also possible to embed Python functionality using CPython into your C program.[5]

*Let Python side handle data types*

- **`cffi`** (C Foreign Function Interface for Python) - A very powerful tool with a world of its own. CFFI provides several modes for both extending Python with C extension and embedding Python inside the C program. Furthermore, in addition to API mode CFFI also provides ABI mode that can be used to access functions available in any compiled libraries like `.dll` for Windows or `.so` for Linux/macOS.
- **`ctypes`** - This is a built-in Python package that provides a foreign function interface for Python-C. However, this package only provides ABI access to libraries similar to CFFI.

*Let the bridge handle data types.*

- **SWIG** (Simplified Wrapper and Interface Generator) - This is an old but a popular tool that lets you interconnect different programming languages including Python and C. SWIG is one of the oldest tools that made possible interfacing different programming languages but compared to other newer options to bridge Python with C it seems to lose its popularity.

- **Cython** - Cython deserves to re-appear here because of its flexibility. Cython extensions also allow using Python API via `Python.h` and even can act as an interface for pure C extensions. Cython is very good at understanding both C and Python. [2]


## Conclusion
To summarize, let's make very rough generalizations. Bypassing GIL is very easy to implement and can significantly improve overall performance. In addition, multithreading allows simultaneous I/O without blocking your main code. However, concurrency does not mean true parallelism and can be applied only for specific tasks. Nevertheless, in the second section, we learned about different ways on how to release GIL and still keep it Python. Alternative Python implementations can achieve true parallelism by releasing GIL without losing the flexibility of Python. On the other hand, Cython may require some experience to get the most out of it. Similarly, data analysis packages like Pandas can produce lightning performance if used correctly, but its application depends on the nature of the task. Finally, for those who are not afraid to get hands dirty, GIL can be released via C extensions. C extensions can provide total control over your code and the best execution speed. However, C knowledge is required because too much control is too much responsibility.



## References

  1. “Concurrency vs. Parallelism,” HowToDoInJava. [Online]. Available: https://howtodoinjava.com/java/multi-threading/concurrency-vs-parallelism/. [Accessed: 18-Mar-2020].
  2. K. W. Smith, Cython: a guide for python programmers, First Edition. Beijing Cambridge Farnham Köln Sebastopol Tokyo: O’Reilly, 2015.
  3. A. J. J. Davis, “Grok the GIL: Write Fast and Thread-Safe Python.” [Online]. Available: https://emptysqua.re/blog/grok-the-gil-fast-thread-safe-python/. [Accessed: 18-Mar-2020].
  4. “Java Multi-threading Tutorials,” HowToDoInJava. [Online]. Available: https://howtodoinjava.com/java/multi-threading/. [Accessed: 18-Mar-2020].
  5. M. Summerfield, Python in practice: create better programs using concurrency, libraries, and patterns. Addison-Wesley, 2013.

