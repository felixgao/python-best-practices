!!! Summary

    :white_check_mark: Use the standard python [cProfile] library

    :white_check_mark: Use PyCharm pstat profile viewer
    
    :white_check_mark: Ensure entire program can be easily tested for performance


# Profiling Tools & Strategy

Profiling code is a hugely important part of data science code since machine
learning and AI algorithms are typically compute heavy processes. Performance
tuning is relevant during both training and prediction time.

During training time, performant code allows developers to have shorter 
development/training cycles. The difference between a function call that takes
10 seconds vs 1 second has a huge impact on the development experience. An 
example that many people have likely run into is when loading huge word vector 
models into memory. A load of GoogleNews W2V using Gensim can take multiple 
minutes before even being able to do a computation.

During prediction time, performance typically dictates how much a model will
cost to host. Performance is often ignored in favor of scaling out horizontally
and just paying the cost of hosting. Besides saving on hosting costs, the 
biggest motivating factor for performance tuning code would be the ability to
have a better/faster development experience (just like during training).

# Tools

There are a variety of profiling tools for python which fall under two 
different categories:

- **Scope-Based Profilers**: These profilers generally use the built-in 
    [sys.setprofile] or [sys.settrace] python library calls to track the 
    duration of each stack frame. Every time a function enters a scope, a start
    time is recorded, and every time the function exits, the end time is 
    recorded. This allows the profiler to record a hierarchical view of where 
    time is spent. These types of profilers may also introduce a lot of 
    overhead since every single scope entrance/exit is tracked. For programs
    which have many function calls (such as mapping a function over a giant 
    collection) the overhead of a scope-based profiler may give inflated
    measurements. Another limitation is that many python functions do not 
    generate a stack frame (for example built-in functions). This means that
    scope-based trackers may not give a granular enough idea of where
    time is being spent. Scope-based profilers cannot give line-level profiling
    information.
    
- **Sampling Profilers**: These profilers generally use the built-in 
    [sys._getframe] python library call to get the current state of the 
    application. In contrast to a scoped-based profiler, sampling profilers only
    periodically query the running python application to determine where time is
    being spent. Rather than tracking the duration of a function scope, a 
    sampling profiler tracks the number of times it sees a specific stack frame.
    This means that a sampling profiler will not track a very accurate measure
    of the overall method time. A huge benefit of a sampling profiler is that
    it has the ability to give line-level profile information. It also has 
    almost none of the overhead issues that a scope-based profiler has. Some
    sampling profilers are even designed to be run on long-running applications 
    for production monitoring.
    
The mode widely used tools are the following:

- [cProfile] (Scope-Based): The python standard library profiler.
- [pprofile] (Scope-Based & Sampling): A pure python profiler.
- [yappi] (Scope-Based & Sampling): A profiler with multi-threading support.
- [scalene] (Sampling): A profiler with built-in memory usage profiling.

In general, it is recommended to start with [cProfile] and move on to other 
tools only if there is a need. Almost all use-cases are well-suited to the most 
basic scoped-based profiler and there are only infrequent times when more 
specialized profilers should be used. For example [yappi] may be well suited to 
testing a multi-threaded web server with extremely high traffic. Though 
sampling profilers can give extremely granular information, it is unlikely 
that the issues will not show up clearly in a scope-based profile.


[sys.setprofile]: https://docs.python.org/3/library/sys.html#sys.setprofile
[sys.settrace]: https://docs.python.org/3/library/sys.html#sys.settrace
[sys._getframe]: https://docs.python.org/3/library/sys.html#sys._getframe

[cProfile]: https://docs.python.org/3/library/profile.html#profile
[yappi]: https://github.com/sumerc/yappi
[scalene]: https://github.com/emeryberger/scalene
[pprofile]: https://github.com/vpelletier/pprofile