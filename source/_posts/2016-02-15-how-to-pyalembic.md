---
layout: post
title:  "How to PyAlembic - Finally overcome Alembic Python bindings"
date:   2016-02-15 14:52:04
excerpt: "I recently needed to recompile the Alembic library and its Python version, under Fedora at work, and encountered several issues I finally overcame. So I thought about sharing it..."
categories: 
- fix
tags:
- alembic
- c++
- python
- pyalembic
- ilmbase
- pyilmbase
- fedora
comments: true
---

Due to some changes at work, I had to recompile Alembic and had problems on the python bindings for PyAlembic.
I used to version that was on the master branch ([commit](https://github.com/alembic/alembic/commit/3682461d016b188f83e4639d08ffd784e85b3af9)), because a lot of things were fixed since the last release ([1.5.8](https://github.com/alembic/alembic/releases/tag/1.5.8)).

Arrived at the PyAlembic step, everything *seemed* to compile and link correctly. 

And then, when I ran abcview

{% highlight bash %}
> ./abcview
{% endhighlight %}

{% highlight bash %}
> Traceback (most recent call last):
File "/home/alembic/python/examples/AbcView/bin/abcview", line 84, in <module>
      verbose=args.verbose
File "/home/alembic/python/examples/AbcView/lib/abcview/app.py", line 1432, in create_app
      win = AbcView()
File "/home/alembic/python/examples/AbcView/lib/abcview/app.py", line 432, in __init__
      self.viewer = GLWidget(self)
File "/home/alembic/python/examples/AbcView/lib/abcview/widget/viewer_widget.py", line 532, in __init__
      self.setup_default_camera()
File "/home/alembic/python/examples/AbcView/lib/abcview/widget/viewer_widget.py", line 545, in setup_default_camera
      self.add_camera(GLCamera(self, name="interactive"))
File "/home/alembic/python/examples/AbcView/lib/abcview/widget/viewer_widget.py", line 590, in add_camera
      camera.add_view(self)
File "/home/alembic/python/examples/AbcView/lib/abcview/gl.py", line 348, in add_view
      self.views[viewer].setTranslation(self._translation)
ArgumentError: Python argument types in
      GLCamera.setTranslation(AbcGLCamera, V3d)
did not match C++ signature:
      setTranslation(AbcOpenGL::v1::GLCamera {lvalue}, Imath::Vec3<double> trans)
{% endhighlight %}

Like on this post: [Problem Running AbcView](https://groups.google.com/forum/#!topic/alembic-discussion/A5QkgC0iKrc) on the Alembic forum. The problem appeared to be **the auto import imath from the alembic module that was not working**. 

I gave it a try and import alembic to see what would happen.

{% highlight bash %}
> python
{% endhighlight %}

{% highlight python %}
>> import alembic
>> Traceback (most recent call last):
   File "<stdin>", line 1, in <module>
   AttributeError: 'NoneType' object has no attribute 'BaseExc'
{% endhighlight %}

So, yeah there's an error on the python alembic module. I came across this post [PyAlembic import quirk](https://groups.google.com/forum/#!topic/alembic-discussion/EUekCcYeEQQ) and got every same error.
Importing the imath module before the alembic seemed to work, while just the iex then alembic made an error.

---

I tried the trick from the post to import imath and alembic without any error:

{% highlight bash %}
> python
{% endhighlight %}

{% highlight python %}
>> import DLFCN
>> import sys
>> sys.setdlopenflags(DLFCN.RTLD_NOW | DLFCN.RTLD_GLOBAL) 
>> import imath
>> import alembic
{% endhighlight %}

Here, Lucas Miller suggested to *always* **manually** import imath before the alembic module : [Python AttributeError when importing alembic](https://github.com/royedwards/alembic/issues/335)

> *They are importing imath first.*

---

But I did not like this solution (which is not one). Plus, I remembered that I used to compile the alembic module that ran by itself without needing any of those tricks. So, my guess was that the problem started at pyilmbase with the imath module. I continued digging.

So I tried like on this post: [Python Bindings - unit test meshOut method fails](https://groups.google.com/d/msg/alembic-discussion/8wSs0L45md0/QfFPDfMyAwAJ) and got the same error.

{% highlight bash %}
> python
{% endhighlight %}

{% highlight python %}
>> from imath import *
>> ImportError: /home/alembic/lib/libPyImath.so.2: undefined symbol: _ZNSo9_M_insertIdEERSoT_
{% endhighlight %}

Yeah, the problem definitely starts from the imath module. 

#### ***What did I do wrong?***

---

After investigation <i class="fa fa-search"></i>, my problems were:

Wrong Boost Python version
--------------------------
*I was compiling using Boost (Python) 1.55.* 
While the recommanded compatible versions are Boost 1.48 - 1.52 (*see Ryan Galloway's sayings on [Abc.ChildIterator Error: This class cannot be instantiated from Python](https://groups.google.com/d/msg/alembic-discussion/jLwgzpZRjus/VDmPb73KtyIJ)*). 

#### **I, then, used Boost 1.48**

---

"_POSIX_C_SOURCE" and "_XOPEN_SOURCE" redefined
-----------------------------------------------
I also finally pay attention to some warning during the compilation time

{% highlight bash %}
> In file included from /usr/include/python2.7/pyconfig.h:6:0,
                   from /home/boost/1.48.0/include/boost/python/detail/wrap_python.hpp:50,
                   from /home/boost/1.48.0/include/boost/python/detail/prefix.hpp:13,
                   from /home/boost/1.48.0/include/boost/python/args.hpp:8,
                   from /home/boost/1.48.0/include/boost/python.hpp:11,
                   from /home/alembic/python/PyAlembic/Tests/main.cpp:3:
> /usr/include/python2.7/pyconfig-64.h:1182:0: warning: "_POSIX_C_SOURCE" redefined [enabled by default]
> #define _POSIX_C_SOURCE 200112L  
> ^  
> In file included from /usr/include/string.h:25:0,
                   from /home/alembic/python/PyAlembic/Tests/main.cpp:2:
> /usr/include/features.h:165:0: note: this is the location of the previous definition  
> #define _POSIX_C_SOURCE 200809L 
> ^
> In file included from /usr/include/python2.7/pyconfig.h:6:0,
                   from /home/boost/1.48.0/include/boost/python/detail/wrap_python.hpp:50,
                   from /home/boost/1.48.0/include/boost/python/detail/prefix.hpp:13,
                   from /home/boost/1.48.0/include/boost/python/args.hpp:8,
                   from /home/boost/1.48.0/include/boost/python.hpp:11,
                   from /home/alembic/python/PyAlembic/Tests/main.cpp:3:
> /usr/include/python2.7/pyconfig-64.h:1204:0: warning: "_XOPEN_SOURCE" redefined [enabled by default]
> #define _XOPEN_SOURCE 600
> ^
> In file included from /usr/include/string.h:25:0,
                   from /home/alembic/python/PyAlembic/Tests/main.cpp:2:
> /usr/include/features.h:167:0: note: this is the location of the previous definition
> #define _XOPEN_SOURCE 700
> ^
{% endhighlight %}

And I got the **exact same warning** when ***compiling pyilmbase***...

---

After Google suggested me this Stackoverflow thread: [g++ with python.h, how to compile](http://stackoverflow.com/questions/10056393/g-with-python-h-how-to-compile), 

#### **in every file from *Ilmbase* or *Alembic* having \<Python.h> or \<boost/python/\*>, *I put those headers on top.***

After these 2 things, I managed to compile everything smoothly (ilmbase + alembic) and ran abcview without any issue <i class="fa fa-angellist"></i>. **And the alembic module, in python, imports the imath automatically as expected!** <i class="fa fa-child"></i>

---

To sum up
---------

 1. Be sure to **use a compatible Boost version** for the Alembic Python bindings : **1.48 - 1.52**

 2. Put **Python and Boost Python headers on top of all headers** in Ilmbase and Alembic files
