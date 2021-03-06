Processing background jobs
===========================

*Hangfire Server* part is responsible for background job processing. It is an optional part, and does not being started automatically -- there is no magic, just additional lines of code.

The Server does not depend on ASP.NET and can be started anywhere, from a console application to Microsoft Azure Worker Role. Single API for all applications is being exposed through the ``BackgroundJobServer`` class:

.. code-block:: c#

   // Creating an instance with `JobStorage.Current` instance.
   // Please look at ctor overrides for advanced options like 
   // explicit job storage instance.
   var server = new BackgroundJobServer(); 

   // Starting to process background jobs (non-blocking).
   server.Start();

   // Initiating server shutdown (non-blocking).
   server.Stop();
   
   // Waiting for graceful server shutdown.
   server.Dispose();

Hangfire Server consist of different components that are doing different work: workers listen to queue and process jobs, recurring scheduler enqueues recurring jobs, schedule poller enqueues delayed jobs, expire manager removes obsolete jobs and keeps the storage as clean as possible, etc.

.. admonition:: You can turn off the processing
   :class: note

   If you don't want to process background jobs in a specific application instance, just don't create an instance of the ``BackgroundJobServer`` class or simply don't call the ``Start`` method in that instance.

Its ``Start`` and ``Stop`` methods are working in a **non-blocking manner** and send corresponding signal to the server. This may require some additional blocking code in a :doc:`console applications <processing-jobs-in-console-app>`, but is more efficient for :doc:`web apps <processing-jobs-in-web-app>`.

The ``Dispose`` method is a **blocking** one, it waits until all the components prepare for shutdown (for example, workers will place back interrupted jobs to their queues). So, we can talk about graceful shutdown only after waiting for all the components.

.. admonition:: Always dispose your background server
   :class: note

   Try to call the ``Dispose`` method where possible to have graceful shutdown features working.

Strictly saying, *you are not required* to invoke the ``Dispose`` method. Hangfire can handle even unexpected process terminations almost fine, and will retry interrupted jobs automatically. However, if you are not calling the ``Stop`` method, some things like :doc:`cancellation tokens <../background-methods/using-cancellation-tokens>` or automatic job re-queueing on shutdown (instead of waiting for a job invisibility timeout) -- in other words graceful shutdown -- **will not work**.
