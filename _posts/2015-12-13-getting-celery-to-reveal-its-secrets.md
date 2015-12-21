---
layout: post
title: "Getting Celery to Reveal Its Secrets"
description: ""
category: 
author: "T.J. Alumbaugh"
header-img: "img/2.jpg"
tags: ["python", "celery", "django"]
---

<h2 class="section-heading">Getting Celery to Reveal Its Secrets</h2>

Celery is a great distributed computing library that has all sorts of bells and whistles to handle a vast range of distributed computing needs.
A lot of the time though, your distributed computing needs are pretty simple. 
The purpose of this blog post is two-fold:

- demonstrate how to set up celery for a basic use case of distributed task execution
- monitor celery running live to verify that it is performing as desired

Remarkably, when you go to the celery documentation on monitoring events, you won't find the magic command you have to enter so that you can actually
see what your jobs are doing.

For example. on a recent project, we use celery to execute Numba-driven Python calculations of tax revenue estimates. Our jobs have the following characteristics:

- significant memory high watermark
- >20 seconds run time
- I/O bound for some portion of time

You might think that 

{% highlight %}
CELERY_PREFETCH_MULTIPLIER=1

{% endhighlight %}

So, the trick is to tell celery that you would like to enable events, and *then* it will allow you to view tasks as they come in:


{% highlight %}
$ celery -A celery_tasks control enable_events
$ celery -A celery_tasks  events
{% endhighlight %}

What about when your jobs complete? Celery provides a mechanism for you to figure out that you are oh-so-far from having a working program. on the 
off-chance that your function actually succeeds, you can get the results as follows:

Whatever you do, don't do what I did, which was just create a new Task object from the job ID that you got back. 

{% highlight python %}



{% endhighlight %}

{% highlight %}
CELERY_PREFETCH_MULTIPLIER=1

{% endhighlight %}

{% highlight python %}

    job_handle = my_task.delay(arg1, arg2)
    # Keep the handle!
    my_jobs[job_handle.id] = job_handle

{% endhighlight %}

When your function raises an exception, the results are available through the `traceback` attribute. 

{% highlight python %}
    task = running_jobs[job_id]
    if task.failed() or task.ready():
        to_pop.append((task, job_id))


    if results.ready() and results.successful():
        tax_result = results.result
        return tax_result
    elif results.failed():
        return results.traceback
    else:

{% endhighlight %}
