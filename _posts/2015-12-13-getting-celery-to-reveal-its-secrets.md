---
layout: post
title: "Celery Event Reporting: Getting Your Jobs on that Hotline Bling."
description: ""
category: 
author: "T.J. Alumbaugh"
header-img: "img/2.jpg"
tags: ["python", "celery", "django"]
---

<h2 class="section-heading"> "Celery Event Reporting: Getting Your Jobs on that Hotline Bling." </h2>

There's two things we can all agree on: Drake dropped a massive hit with his single "Hotline Bling". 2: `celery` is a great distributed computing library that has all sorts of bells and whistles to handle a vast range of distributed computing needs. The problem is that, unlike Drake's earstwhile girlfriend, if you don't set things up correctly, your celery jobs won't tell you what they are up to - and THAT my friends, is what this blog post is about.

A lot of the time though, your distributed computing needs are pretty simple. 

- demonstrate how to set up celery for a basic use case of distributed task execution
- monitor celery running live to verify that it is performing as desired

Celery has a section in its documentation for event reporting. Remarkably, when you go to the celery documentation on monitoring events, you won't find the magic command you have to enter so that you can actually
see what your jobs are doing.

For example. on a recent project, we use celery to execute Numba-driven Python calculations of tax revenue estimates. Our jobs have the following characteristics:

- significant memory high watermark
- >20 seconds run time
- I/O bound for some portion of time

For my case, I wanted to make sure that celery did not 

{% highlight python %}
CELERY_PREFETCH_MULTIPLIER=1

{% endhighlight %}

So, the trick is to tell celery that you would like to enable events, and *then* it will allow you to view tasks as they come in:

{% highlight python %}
$ celery -A celery_tasks control enable_events
$ celery -A celery_tasks  events
{% endhighlight %}

{% highlight python %}
CELERY_PREFETCH_MULTIPLIER=1

{% endhighlight %}


What about when your jobs complete? Celery provides a mechanism for you to figure out that you are oh-so-far from having a working program. on the 
off-chance that your function actually succeeds, you can get the results as follows:

Whatever you do, don't do what I did, which was just return the guid associated with the created job, and then, at some later time, create a fresh AsynResult object from that job ID. 

{% highlight python %}

    handle = my_task.delay(arg1, arg2)
    return str(handle)

{% endhighlight %}

{% highlight python %}

    # Use the job_id to create an AsynResult and get its status
    result = celery_app.AsyncResult(job_id)
    if result.ready():
        return resultsresult

{% endhighlight %}

This `result` object will definitely work as long as the only thing that ever happens to your job is that it executes succesfully. If your job does anything other than succeed, this technique will always tell you that your
job is in the PENDING state. Forever. With no error message. So, I gathered that I shouldn't do that.

Instead, you should keep track of what is returned by the `delay` method of a task, maybe in a dictionary:

{% highlight python %}

    job_handle = my_task.delay(arg1, arg2)
    # Keep the handle!
    my_jobs[job_handle.id] = job_handle

{% endhighlight %}

This object will hold the proper status of the task (e.g. 'SUCCESS', 'FAILURE', 'PENDING', etc.) and even 
give you the traceback if an exception is raised:

{% highlight python %}

    # Now it's time to check if the job is done
    task = running_jobs[job_id]

    if task.ready():
        if task.successful():
            return results.result
        elif results.failed():
            return results.traceback
        else:
            # Nothing should get here
            raise

{% endhighlight %}



