.. _quick_start:

Quick Start
===========

Get Prepared
------------
With mimircache, testing/profiling cache replacement algorithms is very easy.
Let's begin by getting a cachecow object from mimircache:

    >>> import mimircache as m
    >>> c = m.cachecow()

Open Trace File
---------------
Now let's open a trace file, your have three choices for opening different types of trace files, choose the one suits your need.

    >>> c.open("trace/file/location")
    >>> c.csv("trace/file/location", init_params={'label_column':x})  # specify which column contains the request key(label),
    >>> c.vscsi("trace/file/location")          # for vscsi format data

.. note::
    for csv data, the column number begins with 0, so the first column is 0, the second is 1, etc. In the init_params, other possible parameters include: header: True/False; real_time_column:x; op_column:x; size_column:x. Header indicates whether the csv file has a header; real_time_column indicates the time of the request, it must be in the form of number, either in microseconds, milliseconds, seconds or some other unit; op_column means the type of operation, it is not supported currently, size_column indicates the column of request size, it is not used in current version


OK, data is ready, now let's play!

If you want to read your data from cachecow, if you simply use cachecow as an iterator, for example, doing the following:

    >>> for request in c:
    >>>     print(c)

.. note::
    If you have a special data format, you can write your own reader in a few lines, see :ref:`here<advanced_usages>` about how to write your own cache reader.



Get Profiler/Do Profiling
----------------------------
Before we calculate/plot something, let's make one thing clear, the calculation and plotting is carried out by something called profiler, so we need to obtain a profiler first.

Profiling with LRU
^^^^^^^^^^^^^^^^^^^

First, let's try LRU(least recently used), we need to get a profiler:
    >>> profiler_LRU = c.profiler('LRU')


* To get reuse distance for each request, simply call the functions below, the returned result is a numpy array:
    >>> reuse_dist = profiler_LRU.get_reuse_distance()
    array([-1, -1, -1, ..., -1, -1, -1], dtype=int64)

* To get hit count, hit rate, miss rate, we can do the following:

* Hit count is a numpy array, the nth element of the array means among all requests, how many requests will be able to fit in the cache if we increase cache size from n-1 to n. The last two elements of the array are different from all others, the second to the last element means the number of requests that needs larger cache size(if you specified cache_size parameter, otherwise it is 0), the last element says the number of cold misses, meaning the number of unique requests.
    >>> profiler_LRU.get_hit_count()
    array([0,  2685,  662, ...,   0,   0,  48974], dtype=int64)

* Hit rate is a numpy array, the nth element of the array means the hit rate we can achieve given cache size of n. Similar to hit count, the last two elements says the ratio of requests that needs larger cache size and the ratio of requests that are unique.
    >>> profiler_LRU.get_hit_rate()
    array([ 0,  0.02357911,  0.02939265, ...,  0.56992061,   0,  0.43007939])

* Miss rate is a numpy array, the nth element of the array means the miss rate we will have given cache size of n. The last two elements of the array are the same as that of hit rate array.
    >>> profiler_LRU.get_miss_rate()
    array([ 1,  0.97642089,  0.97060735, ...,  0.43007939,   0,  0.43007939])

.. note::
    for reuse distance, hit count, hit rate, miss rate, if you don't specify a cache_size parameter or specify cache_size=-1, it will use the largest possible size.


* With the data calculated from profiler, you can do plotting yourself, or any other calculation. But for your convenience, we have also provided several plotting functions for you to use. For plotting hit rate curve(HRC):
    >>> profiler_LRU.plotHRC()

.. figure:: ../images/example_HRC.png
        :width: 50%
        :align: center
        :alt: example HRC
        :figclass: align-center

        Hit rate curve(HRC) of the trace



* Similarly, we can plot miss rate curve(MRC):
    >>> profiler_LRU.plotMRC()


.. figure:: ../images/example_MRC.png
        :width: 50%
        :align: center
        :alt: example HRC
        :figclass: align-center

        Miss rate curve(MRC) of the trace



Except all the default parameter we used for profiling above, you can also provide other keyword arguments, supported keyword arguments are listed below.




    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    | functions/arguments |                              cache_size                              |                                        begin                                        |                            end                            |
    +=====================+======================================================================+=====================================================================================+===========================================================+
    |                     | the size of cache, default is -1, which is the largest possible size | the place begin profiling, number begin with 0, default is 0, the beginning of file | the place stops profiling, default is -1, the end of file |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |  get_reuse_distance |                                  No                                  |                                          No                                         |                             No                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |    get_hit_count    |                                  Yes                                 |                                         Yes                                         |                            Yes                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |     get_hit_rate    |                                  Yes                                 |                                         Yes                                         |                            Yes                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |    get_miss_rate    |                                  Yes                                 |                                         Yes                                         |                            Yes                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |       plotHRC       |                                  Yes                                 |                                          No                                         |                             No                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+
    |       plotMRC       |                                  Yes                                 |                                          No                                         |                             No                            |
    +---------------------+----------------------------------------------------------------------+-------------------------------------------------------------------------------------+-----------------------------------------------------------+




* An example for how to use these keyword arguments:
    >>> profiler_LRU.get_hit_rate(cache_size=2000, begin=1, end=10)

.. warning::
    Upon testing, using keyword arguments will cause error in some of 32-bit platform, if you get an error, please try not using keyword arguments.


Profiling with non-LRU
^^^^^^^^^^^^^^^^^^^^^^

Apart from LRU, we have also provided a varieties of other cache replacement algorithms for you to play with, including Optimal, FIFO, LRU-2, LRU-K, MRU, LFU_RR, LFU_MRU, LFU_LRU, Random, SLRU, S4LRU, clock, adaptive SLRU.

.. note::
    Check :ref:`here <algorithms>` for detailed information about each cache replacement algorithms.

To play with these cache replacement algorithms, you just substitue 'LRU' in the examples above with cache replacement algorithm you want, then give a cache_size and bin_size. This e reason why we need cache_size and bin_size is that for a general cache replacement algorithm, the profiling is done by sampling at certain points among all cache size, in other words, the nth element in numpy arrays returned represents the result at cache size of n*bin_size.
Some examples are shown below:

* Obtaining a profiler:
    >>> profiler_FIFO = c.profiler('FIFO', cache_size=2000, bin_size=100)

several other parameters and their default values are listed below,

+------------------+------------------+----------------------+
| Keyword Argument | Default Value    | Necessary            |
+==================+==================+======================+
| cache_size       | No default value | YES                  |
+------------------+------------------+----------------------+
| bin_size         | cache_size/100   | No                   |
+------------------+------------------+----------------------+
| cache_params     | None             | Depends on algorithm |
+------------------+------------------+----------------------+
| num_of_threads   | 4                | No                   |
+------------------+------------------+----------------------+

* After obtaining the profiler, everything else is the same as above with LRUProfiler, you can obtain hit_count, hit_rate, miss_rate, you can plotHRC, plotMRC, the only difference is the returned hit_count array, hit_rate array, miss_rate array does not have the last two special elements as above. Some examples are shown below:
    >>> profiler_FIFO.get_hit_count()
    >>> profiler_FIFO.get_hit_rate()
    >>> profiler_FIFO.get_miss_rate()

.. note::
    Reuse distance related operation is only allowed on LRU, so don't call get_reuse_distance on non-LRU cache replacement algorithms.

.. note::
    If you want to test your own cache replacement algorithms, check :ref:`here<create_new_cache_replacement_algorithms>`.


Two Dimension Plotting
----------------------
Mimircache allows you to plot a variety graphs, including some two diemension graphs to help you get better understanding of your data.
Before plotting, we shall talk about the concept of time, there are two types of time concept in mimircache,
the first one is called virtual time, which basically is the order in the request sequence.
Real time, opposite to virtual time is the wall clock time, which is available in some data, for example, vscsi format data.

There are currently two types of 2D plots are supported, Cold miss plot and request num plot, you can plot them by calling:
    >>> c.twoDPlot(time_mode, time_interval, plot_type)

The axis is starting time t, the y axis is the number of cold miss/request number in time *t+time_interval*.

Cold miss plot
^^^^^^^^^^^^^^

* Cold miss plot: the number of cold misses in the given interval.
    >>> c.twoDPlot('v', 1000, 'cold_miss')

.. figure:: ../images/example_cold_miss2d.png
        :width: 50%
        :align: center
        :alt: example cold miss
        :figclass: align-center

        Cold miss count in virtual time


Request number plot
^^^^^^^^^^^^^^^^^^^

* Request number plot: the number of requests in the given interval.
    >>> c.twoDPlot('r', 10000, 'request_num')

.. figure:: ../images/example_request_num.png
        :width: 50%
        :align: center
        :alt: example request num
        :figclass: align-center

        Request number count in real time


Plotting Heatmaps
-----------------
Another great feature about mimircache is that it allows you to incorporate time component of a cache trace file into consideration, make the cache analysis from static to dynamic.
Currently six types of heatmaps are supported:

Plot Types
^^^^^^^^^^

+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| plot type                      | x axis                                  | y axis                                | plot detail                                                                                                                                          | Other                                  |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| hit_rate_start_time_end_time   | start time (real or virtual) in percent | end time (real or virtual) in percent | pixel (x, y) means the hit rate from time x to time y                                                                                                |                                        |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| rd_distribution                | start time (real or virtual) in percent | reuse distance                        | reuse distance distribution graph, pixel (x, y) represents at time x+time_interval, the number of requests have reuse distance of y (shown in color) |                                        |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| rd_distribution_CDF            | start time (real or virtual) in percent | reuse distance                        | similar to reuse distance distribution graph, but each points (x, y) represents the percent of requests have reuse distance less than or equal to y  |                                        |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| future_rd_distribution         | start time (real or virtual) in percent | reuse distance                        | future reuse distance distribution graph, future reuse distance is defined as how far in the future, it will be accessed again.                      |                                        |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| hit_rate_start_time_cache_size | start time (real or virtual) in percent | cache size                            | each vertical line x=t is a hit rate curve of trace starting at t                                                                                    | currently not tested, might have bugs  |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+
| avg_rd_start_time_end_time     | start time (real or virtual) in percent | end time (real or virtual) in percent | pixel (x, y) means average reuse distance of requests from time x to time y                                                                          | currently not tested, might have bugs  |
+--------------------------------+-----------------------------------------+---------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------+


How to Plot
^^^^^^^^^^^
Plotting heatmaps are easy, just calling the following function on cachecow,
    >>> c.heatmap(mode, time_interval, plot_type...):

The first three parameters are the same as before, which are time mode (r or v), time interval, the types of plot(see table above)
Besides these three parameters, there are several keywords arguments listed below.
**Attention**: cache_size is necessary for hit_rate_start_time_end_time graph.


+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| Keyword Arguments | Default Value | Possible Values                            | Necessary                                                  |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| algorithm         | "LRU"         | All available cache replacement algorithms | No                                                         |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| cache_params      | None          | Depends on cache replacement algorithms    | Depends on cache replacement algorithms, for example LRU_K |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| cache_size        | -1            | Positive integer                           | Necessary for plot "hit_rate_start_time_end_time"          |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| figname           | heatmap.png   | Any, remember to include suffix            | No                                                         |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+
| num_of_threads    | 4             | Positive integer except 0                  | No                                                         |
+-------------------+---------------+--------------------------------------------+------------------------------------------------------------+


Ploting Examples
^^^^^^^^^^^^^^^^
    >>> c.heatmap('r', 10000000, "hit_rate_start_time_end_time", cache_size=2000, figname="heatmap1.png", num_of_threads=8)

.. figure:: ../images/example_heatmap.png
        :width: 50%
        :align: center
        :alt: example hit_rate_start_time_end_time
        :figclass: align-center

        Hit rate of varying start time and end time


Another example

    >>> c.heatmap('r', 10000000, "rd_distribution")

.. figure:: ../images/example_heatmap_rd_distibution.png
        :width: 50%
        :align: center
        :alt: reuse distance distribution graph
        :figclass: align-center

        Reuse distance distribution graph


Plotting Differential Heatmaps
------------------------------
Want to know which algorithm is better? Not satisfied with hit rate curve or miss rate curve because they only show you the result over the whole trace?
You are in the right place! Differential heatmaps allow you to compare cache replacement algorithms with respect to time.


Currently we only support differential heatmap of hit_rate_start_time_end_time, and the function to plot is shown below:

    >>> c.differential_heatmap(mode, interval, plot_type, algorithm1, cache_size...)

The first three parameters are the same as before, which are time mode (r or v), time interval, the types of plot(only support hit_rate_start_time_end_time for now)
algorithm1 is the first algorithm, algorithm2 is the second algorithm (default to be Optimal), cache_size is a **necessary** parameter here and it can only be used as keyword argument.
Besides these parameters, there are several keywords arguments listed below.


+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| Keyword Arguments | Default Value            | Possible Values                            | Necessary                                                  |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| algorithm1        | "LRU"                    | All available cache replacement algorithms | Yes                                                        |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| cache_params1     | None                     | Depends on cache replacement algorithms    | Depends on cache replacement algorithms, for example LRU_K |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| algorithm2        | "Optimal"                | All available cache replacement algorithms | No                                                         |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| cache_params2     | None                     | Depends on cache replacement algorithms    | Depends on cache replacement algorithms, for example LRU_K |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| cache_size        | No Default Value         | Positive integer                           | Yes                                                        |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| figname           | differential_heatmap.png | Any name, remember to include suffix       | No                                                         |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+
| num_of_threads    | 4                        | Positive integers except 0                 | No                                                         |
+-------------------+--------------------------+--------------------------------------------+------------------------------------------------------------+


Example:
    >>> c.differential_heatmap('r', 1000000, "hit_rate_start_time_end_time", algorithm1="LRU", cache_size=2000)

.. figure:: ../images/example_differential_heatmap.png
        :width: 50%
        :align: center
        :alt: example differential_heatmap
        :figclass: align-center

        Differential heatmap, the value of each pixel is (hit_rate_of_algorithm2 - hit_rate_of_algorithm1)/hit_rate_of_algorithm1





Congratulations! You have finished the basic tutorial! Check Advanced Usage Part if you need.