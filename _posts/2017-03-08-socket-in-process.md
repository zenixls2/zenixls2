---
layout: post
title: "socket in processes"
description: "performance issue in socket when fd is not shared"
tags: [web, note]
---
For creating servers in multi-processing environment, 
there are 2 ways to do it. One is to create a socket fd in main process, 
and share this fd among all the child forks. Another is to reuse the port 
and binding address in child processes and create a new socket in each.

Let's create some programs to test the differences between the 2 implementations:

Share fd:
```python
#!/usr/bin/env python3
#
# share same socket fd in multi-processing.
# see https://github.com/channelcat/sanic/blob/master/sanic/server.py#L424
# for how it share a socket among processes in detail.
#
from sanic import Sanic
from sanic.response import text

app = Sanic(__name__)

@app.route("/")
async def test(request):
    return text("hello")

app.run(host='0.0.0.0', port=8080, sock=None, debug=False, workers=4)
```
<br>

And create fd on it's own:
```python
from sanic.response import text
from multiprocessing import Process, Event
from signal import SIGTERM, SIGINT
from signal import signal as signal_func
from sanic.server import serve

app = Sanic(__name__)

@app.route('/')
async def test(request):
    return text("hello")

stop_event = Event()
signal_func(SIGINT, lambda s, f: stop_event.set())
signal_func(SIGTERM, lambda s, f: stop_event.set())

server_settings = app._helper(host='0.0.0.0', port=8080, before_start=None,
    after_start=None, before_stop=None, after_stop=None, ssl=None,
    sock=None, workers=1, loop=None, protocol=HttpProtocol, backlog=100,
    stop_event=None, register_sys_signals=True)
server_settings['reuse_port'] = True
processes = []
for _ in range(4):
    process = Process(target=serve, kwargs=server_settings)
    process.daemon = True
    process.start()
    processes.append(process)

for process in processes:
    process.join()

for process in processes:
    process.terminate()
```
<br>

Both programs create 4 child processes to handle income connections.
For simplicity, we would use wrk to test each:

```bash
wrk --latency -t12 -d10s -c100 http://localhost:8080/
```
<br>

And produce results of RPS(Requests/sec) in each. On my Macbook pro (2016/8G), The RPS of sharing socket is 34492 compared to 22366 for using different socket. Someone might argue that this might be an issue in the underlayer implementation of python socket, but if you modify the second program to run a single server and execute multiple instance of it in your shell, you'll get the same result.

Currently I cannot tell what really happens to the second implementation to have such large performance degradation. Just suspect if the addition algorithm on socket delivery uses too much time on it. This kernel [commit](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=c617f398edd4db2b8567a28e899a88f8f574798d) mentions there's a 4-tuple hash used to uniformly deliver packets to different sockets. Just make sure for the next time when you want to write something from underlayer, remember to share sockets instead of re-creating new ones.
