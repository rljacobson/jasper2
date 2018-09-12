---
title: "Using Mathematica From Python"
date: 2014-12-10 22:27
cover: assets/images/posts/Spikey10.jpg
MathJax: false
tags: [Computer science, Programming, Mathematica, Python]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
author: robert
---

In which I show you how to programmatically interface with a Mathematica kernel from Python.<!--more-->

## Mathematica, Python, and Scientific Computation

Mathematica is the flagship product of [Wolfram Research](http://www.wolfram.com/mathematica/). It's a very sophisticated computer algebra system with the best notebook interface on the market if you ask me. It provides the computational power to [WolframAlpha](http://www.wolframalpha.com/) and is available on "thousands of colleges and universities in over 50 countries" according to Wolfram Research's ad copy. Unfortunately Mathematica is not open source and comes with a hefty price tag, but your institution might have a site license that allows you to install it on your personal computer.

Python has gained a lot of ground in the scientific computation space. With packages like [SciPy](http://www.scipy.org) and SymPy and the comprehensive computer algebra system [Sage](http://www.sagemath.org), Python has access to very sophisticated computational abilities. You know what would be really great, though? If we could interact with a Mathematica kernel directly from our Python code. 

## The Good News

It turns out we can. Mathematica ships with something called MathLink that allows developers of C and C++ (and some other languages) to communicate with a Mathematical kernel programmatically. If you poke around in the Mathematica installation directory you will discover Python bindings for MathLink and a couple of example programs.

## The Bad News—And a Fix

Unfortunately the MathLink Python bindings are undocumented, unsupported, and *very* outdated. Here I show you how I got it working on my system running Mac OS X Yosemite, Mathematica 10.0, and Python 2.7. It's not too hard. I'll assume you have the usual [command line developer tools](http://www.sagemath.org/doc/installation/source.html#section-macprereqs) installed.

First we locate the necessary MathLink library, namely libMLi3.a. On my system it is found here:

    /Applications/Mathematica.app/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/AlternativeLibraries


I'm using the "AlternativeLibraries" version as per the README file located in that directory. We'll also need the mathlink.h header file here: 


    /Applications/Mathematica.app/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions

Now we find the MathLink Python bindings and example code:

    /Applications/Mathematica.app/SystemFiles/Links/Python

Since we will be editing these files, go ahead and copy them to a folder in which to work. That way, if we mess something up we have a backup of the original files.

One of those files, the setup.py file, uses Python's distutils facility to compile and install the Python MathLink bindings extension to our Python environment's site-packages directory. Edit the file to reflect your mathematica version (this might not be strictly necessary):

    mathematicaversion = "10.0"

Now find your platform in the `if-elif` block (in my case "darwin") and edit `include_dirs` and `library_dirs` to be the location of the mathlink.h header file and the library file libMLi3.a respectively. Here's what that piece of code now looks like for my system:

```python
elif(re.search(r'darwin', sys.platform)):
  setup(name="mathlink", version=pythonlinkversion,
    ext_modules=[
      Extension(
        "mathlink",
        ["mathlink.c"],
        include_dirs = ["/Applications/Mathematica.app/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/AlternativeLibraries"],
        library_dirs = ["/Applications/Mathematica.app/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/AlternativeLibraries"],
        libraries = ["MLi3"],
      )
    ]
  )
```

The Python extention is mathlink.c. We need to make a minor adjustment to mathlink.c by defining `MLINTERFACE` to be 3 *before* we include the mathlink.h header file:

```c
#define MLINTERFACE 3
#include "mathlink.h"
```

That's it for the Python bindings! Let's compile and install:

    $ python setup.py build
    $ sudo python setup.py install

You should now be able to run the example script with the following command:

    python textfrontend.py -linkname "math -mathlink"

If you look inside textfrontend.py you'll find a strange attempt by the author to add the Mathematica bin directory to the path. Since the `math` command is already in my path, this line is unnecessary. The script sets up the mathlink connection using the -linkname argument. Consider these things as opportunities for improvement as you monkey with this example code.

Now go write some good Python code!