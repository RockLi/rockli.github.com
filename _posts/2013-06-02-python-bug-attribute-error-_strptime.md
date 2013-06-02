---
layout: post
title: "Python Bug - Attribute Error _strptime"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Background
===

Last Friday, I met a Python Bug, so this weekend I spent some time to investigate this bug and wrote this post to explain the root cause. I'm not a Python specialist, a C programmer, instead. If you found anything error please correct me.

I extracted the minimize POC here:

<pre>
#!/usr/bin/env python
import thread
import time

def thread_fn():
    for _ in xrange(1, 10):
        for _ in xrange(1, 100):
            time.strptime("2013-06-02", "%Y-%m-%d")

for _ in xrange(10):
    thread.start_new_thread(thread_fn, ())

time.sleep(1)
</pre>

Upper codes sometimes will throw out the exception: ```AttributeError: _strptime_time```, you can run it in your environment and check the output. 

I checked Python-2.7.2(Mac Default) and Python-2.7.3(Compiled from the source code). I got this error randomly, which means sometimes this script works fine!

Debug && Workaround
===

You should realized this will be a multithread issue, right? Here is the implementation of ```time_strptime```,

<pre>
static PyObject *
time_strptime(PyObject *self, PyObject *args)
{
    PyObject *strptime_module = PyImport_ImportModuleNoBlock("_strptime");
    PyObject *strptime_result;

    if (!strptime_module)
        return NULL;
    strptime_result = PyObject_CallMethod(strptime_module,
                                            "_strptime_time", "O", args);
    Py_DECREF(strptime_module);
    return strptime_result;
}
</pre>

Each time when this function is called, it will try to load the module "_strptime". The algorithm of the API ```PyImport_ImportModuleNoBlock``` is if there's a thread is importing that module, it will throw out the exception instead of blocking there. This avoids the duplicate module importing and potential deadlock.

But in multithreads environment, when one thread is tring to import ```_strptime```, but has not been fully imported, another threads tried to call ```strptime_module._strptime_time``` directly. This is why the bug happened. 

If you understand well why this bug happend, you should already have the workaround in your heart. Actually it's really straightforward. All you need to do just is call once ```strptime``` before starting your threads.

How to fix
===
It's easy to find bugs in opensource projects. But how to fix bugs ideally is much more difficult which involved the art of writing a patch for open source projects. I will write a new post on this topic in the future, maybe, I'm so lazy in writing articles!

I gave several optional fixes here, all are not ideal.

1. Move module import ```_strptime``` to the init functions in modules: ```time``` and ```datetime```
2. Rewrite ```_strptime``` not as a Py Module
3. Carefully modify invoke the API ```PyImport_ImportModuleNoBlock``` to the synchrous version
4. More and more methods...

Wrap Up
===

Not sure whether Py3 has fixed this issue, haven't check that.

Module ```time``` and ```datetime``` all are impacted by this issue, for they all use module ```_strptime```.

By the way, everyone knows that the interpreted languages like Python, PHP is much more slower than compiled laungages. The technolody JIT is used widely nowadays. For example: LuaJIT, PyPy. Facebook also launched HipHop to accelerate the speed of PHP. I don't like PyPy, because it's written in Rython(a subset of Python?). I stil think C is the best tool to get this job done with few assembly codes to improve the critical performance point.

If someone is also interested in creating a new Project to speed up the Python language, we can discuss together, please notice it's interests driven and in the spare time. My email can be found in the <a href='http://code-trick.com/about.html'>about page</a>.


