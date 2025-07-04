## 1. 讲一下Python的GIL？
* GIL（Global Interpreter Lock）是Python解释器的一个重要机制，它确保了在任何时刻只有一个线程执行Python字节码。这意味着在CPython解释器中，无论有多少个CPU核心，Python解释器都只会使用一个线程来执行Python字节码。
* GIL的存在使得Python在多线程场景下性能下降，因为一个线程在执行Python字节码时，其他线程无法执行Python字节码。但是，GIL的存在也使得Python在多线程场景下的一些场景下可以使用，比如I/O密集型的场景。
* 在Python 3.2版本中，GIL被移除了，引入了一个新的机制叫做PyPy，它是一个Python解释器，它使用了JIT（Just-In-Time）编译器，它可以在运行时将Python字节码编译为机器码，从而提高性能。
* 在Python 3.3版本中，GIL被重新引入，但是它的引入使得Python在多线程场景下的性能下降更加明显。
* 在同一个进程中，多个线程可以同时执行Python字节码，但是只有一个线程可以执行Python字节码。
* 可以使用multiprocessing模块来创建多个进程，每个进程都有自己的Python解释器，每个进程都有自己的GIL，所以多个进程可以同时执行Python字节码。

## 2. 讲一下Tornado？
* Tornado是一个Python Web框架，它是一个异步非阻塞的Web服务器，它的特点是高性能、高并发。
* 它使用了异步非阻塞的I/O模型，它可以同时处理多个请求，而不需要为每个请求创建一个线程。
## 2. 讲一下Tornado?
