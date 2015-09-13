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

<h2 class="section-heading">Making Python Extensions on Windows</h2>

One of the great things about CPython is its ability to integrate well with other languages. For example, it's very common to integrate some existing C or C++ code with Python. There are a LOT of choices out there for doing this. A less common choice, but still extremely useful, is integrating Python with Fortran. In that case, there aren't that many choices out there. In fact, there's pretty much just f2py. You can find some tips out there for building C/C++ extension packages for Python on Windows, but I haven't found very much out there for Fortran. So, my pain is your gain.

The steps I present here are what would occur to someone who has a more Unix-y philosophy. If that is foreign to you, then you likely already understand Windows enough that this page will be a bit redundant. So, first let's get some Fortran code we want to build and 

<a href="http://www.github.com/talumbau/fortran_extension">Some Fortran code</a>

Well, that was easy.

Next to compile Fortran, you need a Fortran compiler. For Windows. Amazingly, these exist. In addition, because we need to compile code that adheres to the C API of Python, we also need a C compiler.

<h4> How do I know which compiler I should download?</h4>
Answer: First you should find out which compiler (and version number) was used to compile your CPython.

<h4> How am I supposed to know what that means? </h4>
That's a good question. I just looked at this StackOverflow post. 

<h4> So, can I 'yum install' this or what?</h4>
Answer: No. You have goolge "Download Microsoft Visual Studio 2008". I did that and basically the first link is to a download page that doownloads a Powerpoint presentation talking about Visual Studio 2008. Great job.

<h4>Are there any more pitfalls that can cause this to fail</h4>
Answer: Yes, thanks for asking! The Fortran compiler I chose doesn't work with Visual Studio Express

<h4> Ok, I've paid for the compiler. So now, just <code>python setup.py build</code>, right? </h4>
Answer: No. You have to modify your PATH. This is just like modifying your PATH on a Unix system, except you have to type everything inside this little box.


Awesome.


