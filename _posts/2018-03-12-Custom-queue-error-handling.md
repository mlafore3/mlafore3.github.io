---
layout: code-snippet
title:  "Custom queue error handling"
date:   2018-03-14
excerpt: ""
tag:
- snippet
---
One thing we can all agree on is that we strive our apps to be quick and snappy. We all get frustrated by even the slightest signs of lagging. This is where background processing becomes really important in application development. Completeing tasks asyncronously is even more important in data-driven applications where background processes can take minutes or even days. Being a django app developer I prefer to use django-rq to manage these background tasks. It is very easy to get rq up and running within a couple of hours. 

{% highlight py %}
# In your config file add the following code

INSTALLED_APPS = (
    # other apps
    "django_rq",
)

RQ_QUEUES = {
    'default': {
        'HOST': 'localhost',
        'PORT': 6379,
        'DB': 0,
        'PASSWORD': 'some-password',
        'DEFAULT_TIMEOUT': 360,
    },
    'high': {
        'URL': os.getenv('REDISTOGO_URL', 'redis://localhost:6379/0'), # If you're on Heroku
        'DEFAULT_TIMEOUT': 500,
    },
    'low': {
        'HOST': 'localhost',
        'PORT': 6379,
        'DB': 0,
    }
}

# In a separate file where you keep view helper functions place 

import django_rq

def enqueue(func, *args, **kwargs):
    queue = django_rq.get_queue(kwargs.pop('queue', 'default'))
    job = queue.enqueue(func, *args, **kwargs)
    return job

# Now use the enqueue function in your app whereever you want to complete a background task.
enqueue(start_job, job, timeout=600, ttl=86400)

# You must make sure that you have a redis server running as well as a rq worker. 
{% endhighlight %}

None of this is groundbreaking but I ran into something a little more interesting the other day. I had to add a custom handler to failed jobs. This handler needs to respond to a specific error type (timeout) and update a model accordingly. 

{% highlight py %}
#The config file gets these two handlers. It will pass through these handlers consecttively.

RQ_EXCEPTION_HANDLERS = [
    'probex.task_queue.timeout_handler', # errors will pass through this handler first
    'probex.task_queue.move_to_failed_queue' # catch all error handler
]

#Update the view helper file to 

import django_rq
from rq.timeouts import JobTimeoutException
from app.models import Model

# custom error handler
def timeout_handler(job, exc_type, exc_value, traceback): # the arguments that get passed to the handler
    if isinstance(exc_value, JobTimeoutException): # check that the exception value is JobTimeoutException
        if 'app.background_process.do_long_computation' == job.func_name: # check that the queued function is the one I care about
            job = Model.objects.get(pk=job.args[-1]) # get the model instance that represents the value of the computation
            job.status = 'Error: This process timed out.' # update the job status
            job.save() # save the status
    return True

# catch all handler to move to failed queue
def move_to_failed_queue(job, *exc_info): 
    worker = django_rq.get_worker(job.origin)
    worker.move_to_failed_queue(job, *exc_info)
    return True

def enqueue(func, *args, **kwargs):
    queue = django_rq.get_queue(kwargs.pop('queue', 'default'))
    job = queue.enqueue(func, *args, **kwargs)
    return job
{% endhighlight %}

Now just use the enqueue function as normal and it will hit the custom error handler. 
