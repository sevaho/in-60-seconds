## Overview

- do more at once vs do things faster:

| horizontal | vertical |
| ---------       | ---------        |
| asyncio         | multiprocessing  |
| threads         | cython           |

---
## Generator

- a function containing yield

```python
def fib():
    cur, nex = 0, 1
    while 1:
        cur, nex = nex, cur + nex
        yield cur


for i in fib():
    if i > 10000:
        print(i, flush=True)
        exit(1)
```

---
## give me some code

```python
import asyncio

loop = asyncio.get_event_loop()
data = asyncio.Queue()
task1 = loop.create_task(func)
task2 = loop.create_task(func)
tasks = asyncio.gather(task1, task2)
loop.run_until_complete(tasks)
```

---
## Debug async code

- Setting the PYTHONASYNCIODEBUG environment variable to 1.
- Using the -X dev Python command line option.
- Passing debug=True to asyncio.run().
- Calling loop.set_debug().


---
## using aiohttp

- async variant of requests
- You may want to install optional cchardet library as faster replacement for chardet:
- For speeding up DNS resolving by client API you may install aiodns as well. This option is highly recommended:

```shell
$ pip install cchardet
$ pip install aiodns
```

---
## using aiohttp

simple usecase:

```python
import aiohttp

async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        resp = await response.json()
```

vs.

```python
session = aiohttp.ClientSession()

async with session.get('...'):
    # ...
await session.close()
```

---
## using aiohttp

Basic API is good for performing simple HTTP requests without keepaliving, cookies and complex connection stuff like properly configured SSL certification chaining

```python
from aiohttp import request

async with request("GET", url) as response:
```

---
## GIL

threads dont add computational speed because of GIL
GIL makes python the way it is, thanks to the GIL python is fast (single threaded), you dont need to MM

---
### webscraping demo

```python
import asyncio


async def get_some():

    tasks = []
    for n in range(10):
        tasks.append((n, asyncio.create_task(get_web(n))))

    for n, t in tasks:
        html = await t
        parsed = await parse_html(html)


async def get_web():
    pass


async def parse_html():
    pass
```

---
### threads

> Note: daemon = True ()

```python
threads = [
    Thread(target=<func>, args=(), daemon=True), 
    ...
]

[t.start() for t in threads]
[t.join() for t in threads]
```

What if you want to cancel all threads (if you start 100 you need to control c 100)
you start another thread lets call it abort_thread()

```python
while any([t.is_alive() for t in threads)]:
    [t.join() for t in threads]
    if not abort_thread.is_alive():
        print("cancel all threads", flush=True)
        break
```

---
### RLock

best: (if code fails you should be able to release lock)

```python
with transfer_lock:
    # do some thread safe stuff
```

---
### multiprocessing

```python
from multiprocessing.pool import Pool

pool = Pool(processes=4)
tasks = []
for n in range(4):
    task = pool.apply_async(func=dosomething, args(0, 100))
    tasks.append(task)
pool.apply_async(func=dosomething, args(101, 101))
pool.close()
pool.join()

for t in tasks:
    print(t.get())
```

---
### multiprocessing and asyncio api

SAME API FOR MULTIPROCESSES AND THREADS!

```python
from concurrent.futures import Future
from concurrent.futures.thread \
import ThreadPoolExecutor as PoolExecutor
# from concurrent.futures.thread \
import ProcessPoolExecutor as PoolExecutor

tasks = []
with PoolExecutor() as executor:
    for url in urls:
        f: Future = executor.submit(<func>, <args>)
        tasks.append(f)
for t in tasks:
    print(f.result())
```

---
### unsync

combines the power of multiprocces, threads and async
@unsync(cpu_bound=True)

---
### Cython

file <file>.pyx

setup.py
```python
from disutils.core import setup
from Cython.Build import cythonize

setup(ext_modules = cythonize("<file>.pyx"))
```

pip install cython
python setup.py build_ext --inplace

one can use a 
```
with nogil:
```

---
### asyncio libs

- [https://github.com/aio-libs]
- [https://github.com/timofurrer/awesome-asyncio]

- aioconsole
- asyncpg / aiopg
- aiofiles
