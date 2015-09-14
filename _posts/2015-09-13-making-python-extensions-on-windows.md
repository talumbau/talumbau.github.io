---
layout: post
title: "Creating Python Extensions on Windows"
date:       2015-09-13 14:00:44
description: ""
category: 
author: "T.J. Alumbaugh"
header-img: "img/nordseestraende.jpg"
tags: ["python", "windows"]
---

<h2 class="section-heading">Creating Python Extensions on Windows</h2>

One of the great things about CPython is its ability to integrate well with other languages. For example, it's very common to integrate some existing C or C++ code with Python. There are a <a href="http://cython.org/">LOT</a> <a href="http://www.boost.org/doc/libs/1_59_0/libs/python/doc/">of</a> <a href="http://www.swig.org/">choices</a> <a href="https://riverbankcomputing.com/software/sip/intro/">out</a> there for doing this. A less common choice, but still extremely useful, is integrating Python with Fortran. For example, the underlying routines in SciPy are Fortran code. You can find some tips out there for building C/C++ extension packages for Python on Windows, but I haven't found very much out there for Fortran. So, my pain is your gain.

The steps I present here are what would occur to someone who has a more Unix-y mindset. If that is foreign to you, then you likely already understand Windows enough that this page will be a bit redundant. 

First let's get some Fortran code we want to build as a Python extension:

<a href="http://www.github.com/talumbau/fortran_extension">Some Fortran code</a>

Well, that was easy. I also included a basic "setup.py" so Python knows how to build the extension.

Next to compile Fortran, you need a Fortran compiler. In this case, we want one that works on Windows. Amazingly, these exist. In addition, because we need to compile code that adheres to the C API of Python, we also need a C compiler.

<h4>Q: How do I know which compiler I should download?</h4>
Answer: First you should find out which compiler (and version number) was used to compile your CPython. Then you need to make sure that the Fortran compiler you select is compatible with that C compiler version.

Let's look at the version string for our default version of Anaconda Python 2.7:

<code>
>>> sys.version
'2.7.10 |Continuum Analytics, Inc.| (default, May 28 2015, 16:44:52) [MSC v.1500 64 bit (AMD64)]'
</code>

<h4>Q: How am I supposed to know what that means? </h4>
That's a good question. I really have no idea. I found the answers by looking at <a href="http://stackoverflow.com/questions/2676763/what-version-of-visual-studio-is-python-on-my-computer-compiled-with">this StackOverflow post on Visual Studio versions</a>, which contains this helpful snippet:

<code>
Visual C++ 2008                 MSC_VER=1500
</code>

Based on that, I know this Python was compiled with MS Visual Studio 2008.

Here is another important tip that can save you time and confusion. On Windows, the 64-bit version of the x86 instruction set is referred to as "amd64" or "AMD64", instead of x86-64. However, in Intel compilers, I just don't think they could handle having a config option with "AMD" in it, so they call the 64-bit option "intel64". Great stuff, right?

<h4>Q: So, can I <code>yum install</code> Visual Studio or what?</h4>
Answer: No. You actually have to google "Download Microsoft Visual Studio 2008". I did that and the first link is to a download page that downloads a Powerpoint presentation talking about Visual Studio 2008. Great job, everyone who made that happen.

I installed the Intel Fortran compiler, which is officially called the "Intel Composer XE 2013 SP1". Could have also gone with "ifortran vXX.X", but they didn't ask me.

<h4>Are there any more pitfalls that can cause this to fail?</h4>
Answer: Yes, thanks for asking! The Intel Fortran compiler doesn't work with Visual Studio Express, so you can't get away with using those versions, which is often the recommended path for folks who want to compile Python extensions on Windows.

<h4>Q: Ok, I've paid for both the Visual Studio C compiler and Intel Fortran compiler. So now, I can just do as follows:</h4>

*   <code>git clone {my Fortran code}</code>

*   Open the Anaconda Command Prompt

*   <code>activate</code> a conda environment with numpy and Python 2.7

*   <code>python setup.py build</code>

*   Live in the bliss of compiled Windows object code for the rest of your days.

Answer: Not even close, friend. To start, you will probably have to modify your PATH to make sure that you can execute <code>cl.exe</code> (the MS compiler) and <code>ifort.exe</code> (the Intel compiler). This is just like modifying your PATH on a Unix system, except you have to type everything inside this little box.

![](img/windows_edit_path.png "Why? What do you have against env variables?")

Awesome.

There's still a whole bunch of things that can go wrong, but, basically, if you have gotten both compilers installed on your system, you just need to make sure they are configured correctly and you can probably figure out the rest.

For MS Visual Studio, it turns out that there is a master config batch script that lives in the <code>VC</code> directory called <code>vcvarsall.bat</code>. Open it up and make sure that the section for <code>amd64</code> is pointing to another batch script that exists and is doing reasonable things.

On the Fortran side, open the Intel compiler prompt window and execute the main configuration script, setting the platform to 64-bit and the version of Visual Studio to talk to as VS 2008:

<code>compilervars.bat intel64 vs2008</code>

If that command succeeds, you are good to go. If not, you won't be able to compile the Python extension so you should debug that first. In my case, the above command failed. I tracked it down to the fact that <code>compilervars.bat</code> could not see my <code>VS90COMNTOOLS</code> environment variable, which is definitely in my environment. But, for whatever reason, it couldn't see this variable and so would not recognize my installation of Visual Studio 2008. By hardcoding the right path in to the script, all was well. Then, I was able to <code>python setup.py build</code> and then install my new package, calling Fortran from the Python prompt!


