---
layout: post
title: Development Progress
category : development
tagline:
tags : [development, history]
---
{% include JB/setup %}
## Progress

#####[Report] 2015-03-30

* ***Anh***

  Re-implement the worker and server communication using ZeroMQ.
  Refer to [parameter management]() for details.
  TODO. Test the code and merge with the development branch.

#####[Report] 2015-03-11

* ***Wang Wei***

  Add documentation for [Programming model]({{ BASE_PATH }}{%post_url /docs/2015-03-10-programming-model %})

#####[Report] 2015-03-04

* ***Anh&Wang Sheng&Wang Wei***

  * We discussed on the APIs of ParameterManager (PM) to support flexible
    parameter management and multiple cluster topologies. [See details]().

  * We also discussed **Fault Tolerance** which is not well supported in previous version because MPI crashes
  once a single node fails. We will implement it after more detailed dicussions.


#####[Report] 2015-02-23

* ***Wang Wei***
  New features added:

  * New communication protocol between workers and servers. We replace MPI with ZeroMQ for network communication.
    The are mainly two reasons. First, fault tolerance of MPI is not as well supported as ZeroMQ.
    Second, ZeroMQ provides flexible communication pattens to implement different communication protocols.
    In addition, ZeroMQ's multi-part message can reduce some serialization cost.
    Using ZeroMQ we provide extensible synchronization protocol between workers and servers. Currently,
    we use a simple random sampling protocol to reduce network traffic to increase the scalability of the system.
    Specifically, for each worker, it samples paritial of the model parameters
    and synchronizes them with the servers after each iteration. We are testing different synchronization
    methods (i.e., how to handle synchronization message at server sides and how to handle servers reponse messages at worker sides)
    w.r.t the training convergence time. Using this sampling protocol, we can scale well in terms of processed images per second, as shown in
    the following figure. We used the [MLP example]({{ BASE_PATH }}{%post_url /docs/2015-01-10-quick-start %}) with the mini-bath size being 1000.
    One server with 4 working threads is used in the experiment.

    <img src="{{ BASE_PATH }}/assets/image/history/1server4threads-mlp.png" align="center" width="400px"/>

  * Neural Network Partition. We support layer partition, data partition and hybrid partition.
    Refer to [NeuralNet Partition]({{ BASE_PATH }}{%post_url /docs/2015-02-30-neuralnet-partition %}) for details.

  * Multi-threading. Multi-threading is support to exploit CPUs with multiple (>=4) cores.

  * New Updaters. We have implemented 5 parameter updaters, namely, [AdaDelta](http://arxiv.org/pdf/1212.5701v1.pdf),
    [AdaGrad](http://www.magicbroom.info/Papers/DuchiHaSi10.pdf),
    [RMSProp](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf),
    [Nesterov](http://scholar.google.com/citations?view_op=view_citation&hl=en&user=DJ8Ep8YAAAAJ&citation_for_view=DJ8Ep8YAAAAJ:hkOj_22Ku90C), SGD with momentum.

  TODO. Test the newly added features.


#####[Report] 2015-02-08
* ***Wang Wei***
  Analysis and optimization of scalability.

  <img src="{{ BASE_PATH }}/assets/image/history/ps-framework.png" align="center" width="400px"/>

  The above figure shows the parameter server framwork. The execution flow is:

    * Every worker receives fresh parameters from servers, total size P=40-300 MB.
    Then it computes parameter updates and sends updates to servers, total size P.

    * Every server maintains partial of model parameters, P/#servers. It receives
    parameter updates from workers, updates them and sends back the parameters.

  Communication workload of a worker: send updates (size P), receive parameters (size P).
  Communication workload of a server: receive updates (size #workers x P/#servers),
    send parameters (size #workers x P/#servers).

  1Gbps Ethernet transfers about 100MB data in one second. If P=40MB, #workers=16, #server=1,
  it takes the server 6s to receive and 6s to send P*#workers=640MB data.
  But the computation time on one worker is less than 3s for the [MLP
  example]({{ BASE_PATH }}{%post_url /docs/2015-01-10-quick-start %}).
  Then the worker has to wait for at least 3 seconds to receive the fresh parameters.
  With more workers, the waiting time would be longer. Consequently, we cannot scale well.

  *Solutions*:

  * Increase mini-batch size to increase computation time. But, large mini-batch
      may slow down the convergence rate. Drawback: we may have to run more iterations.

  * Compress the transferred data to reduce the communication workerload.
      Drawback: compression may affect the precision.

  * Replicate servers so that each replica serves a smaller number of workers.
      Drawback: we have to use more machines and synchronize all replicas (Anh is working on this).

  * optimize sending queue of servers and use multicast to send fresh parameters to workers.
      The figure below shows the sending queue of one server, each message consists of one
      parameter and a destination worker. When we insert a new message into the queue,
      we check the queue to replace the old parameter with the new parameter.
      The destination worker is added into the message. The sending thread multicasts
      each message to all destination workers. We can reduce the workload of sending thread to P/#servers.
      But we still need to receive the parameter updates (#workers x P/#servers) using unicast from all workers.
      The waiting time of every worker then increases linearly with the total number of workers.

      <img src="{{ BASE_PATH }}/assets/image/history/multicast.png" align="center" width="500px"/>

  * Reduce the synchronization frequency or synchronize only partial of the
      whole model for each iteration (wangwei and jinyang is working on this).

#####[Report] 2015-01-29
* ***Anh***
  Test H2O with Spakling Water. [See details]({{ BASE_PATH }}{% post_url /blog/2015-01-29-compare-h2o %}).

#####[Report] 2015-01-27
* ***Wang Wei***
  Testing the computation (i.e., Update operation for per tuple) time at server side.
  We only show the cost for large tuples, i.e., matrix. The update for bais parameters
  is within 10^(-6)s.

  <img src="{{ BASE_PATH }}/assets/image/history/serverupdate.png" width="300px"/>
  <img src="{{ BASE_PATH }}/assets/image/history/serverupdatecblas.png" width="300px"/>

  The left figure is tested using foor loop to update parameters. The right
  figure is tested using openblas to do the update which is much faster (the largest
  tuple can be processed within 0.007s). We can observe the
  update time is not large. Take the [Deep MLP](% post_url /docs/2015-01-10-quick-start %)
  as an example, the total number of parameter matrices is 6. If we use 16 workers,
  then the largest number of requests is 96 at the server side (use one server), which
  can be processed within 1s. But, currently, we have to wait for about 7 seconds/tuple
  on the worker side. A detailed analysis shows that most of the time is spent on
  MPI's sending operations, i.e., the bandwidth is saturated. TODO, reduce the size of
  parameters transferred between workers and servers by compression. Alternatively,
  we can replicate the parameter servers to serve a smaller number of workers per server.


#####[Report] 2015-01-25
* ***Wang Wei & Zhaojing***
  Preliminary comparison with [H2O-Hadoop](http://0xdata.com/).

  <img src="{{ BASE_PATH }}/assets/image/history/h2o.png" width="400px"/>

  The experiment is done with 3 server nodes and pure data paralleism. One mini-batch
  has 100 images. We used the
  [Deep Big Simple MLP]() model which
  has 6 hidden layers and about 10 million parameters. The y-axis is the number
  of images processed per second, i.e., throughput of the system. Although SINGA
  outperforms H2O, the scalability is still a problem when the number of worker
  nodes is larger than 16 node. TODO optimize the parameter server to improve the
  scalability, test with hybrid parallelism, and test accuracy.

#####[Report] 2015-01-09
* ***Anh***.
    * Added multi-thread support for table server. A global parameter **_server_threads_**
        determines how many threads running at each table server. Each thread starts a
        dispatch loop that reads requests from the network queue and processes it
        (see _Server_ and _RequestDispatcher_ class).


    * Evaluated multi-threaded table server. The figure below shows the speed-up
        for 16 workers (divided into 4,8 groups respectively). With 8 threads,
        the parameter collection latency at the work drops by a half.

        <img src="http://www.comp.nus.edu.sg/~dinhtta/latency.png" width="300px"/>
        <img src="http://www.comp.nus.edu.sg/~dinhtta/waiting_time.png" width="300px"/>

        The improvement is attributed to the speed-up in per-request processing
        time at each server. To demonstrate this, we measured _waiting
        time_ for each request, computed as the duration from the time the
        request is received to when it is successfully processed (and removed
        from the queue). The result below shows that having more threads
        reduces the per-request processing time at each server.


#####[Report] 2015-01-04
* ***Anh***. Related work: _Minerva_[1,2], another framework for deep learning
    in which _parameter server_ is based on P2P architecture. Each worker runs
    a local parameter server, and the servers propagate updates via a gossip
    protocols (details of which are not clear). The P2P approach implies long
    propagation delays in update which could severely affect the overall convergence.
    Also, the question of how to select a topology that strikes a good trade-off
    between network overhead, update delays and convergence, seems non-trivial.
    One advantage of the P2P approach is to be able to distribute the update
    computation workload which the authors claim to limit scalability of the
    centralized, master-slave parameter server (one that we adopt). However,
    this limitation is exaggerated, as we have shown in running real applications
    that adding more machines to the parameter servers improve throughput almost
    linearly. The source code for this P2P-based parameter server has not been published.

[1] _Minjie Wang et al._ A scalable and Topology Configurable Protocol for Distributed Parameter Synchronization, APSys 2014.

[2] http://sudnya.wordpress.com/2014/01/04/minerva-an-overview/

#####[Report] 2014-12-23
* ***WangSheng***. Redesigned the internal representation of distributed array:
    with the new partition representation, a reshaped array can be easily managed
    to check which part of the content is in local; it eases the process of fetching
    the content in a reshaped array, as we avoid transforming the shape back to
    the original one. Cleaned the API of distributed array. [TODO] The implementation
    of Shape, Partition and LocalArray are almost done, need to finish DistributedArray next.

#####[Plan] 2014-12-16
* Implement Multi-Layer Perceptron and test without parameter servers to compare
    with H2O on MNIST dataset. *by wangwei, 1-2 weeks*.
* Measure how much processing time and how much network transfer time contribute
    to the overall latency. Use multiple threads for processing requests at the server process. *Anh - 1 week*.


#####[Report] 2014-12-16
* ***WangWei***. Updated TableDelegate to support local parameter update which
    is applied when no parameter servers are started. Added a loop thread in
    TableDelegate to send requests and parse the response of get request from
    table servers to fill the fetched parameter data into Param objects. [TODO]
    considering adding a new request type UpdateGet which send a update request
    and get request together to avoid sending the two requests separately.
* ***Anh***. Refactored communication and table server implementation. Underlying
    message passing implementation extends *Network* class (currently MPI
    implementation is used). Messages (both sent and received) are managed by
    a *NetworkService* class which has one thread for sending and another thread
    for receiving messages. Table servers are implemented by *GlobalTable* class
    which contains *Shard* class bor storing key-value tuples. The key is of
    type *TKey*, value is of type *TVal*. Updates, gets, puts operations
    invoke user-defined handler which extends *TSHandler* class.
* ***Anh***. Benchmark table server performance.
  * *Setup*. 1-8 table servers. 16 workers with varying group sizes from 1-8
    (each group containing 16-2 workers). Parameters are split with a threshold *t*
    (number of float values) from 5K-5M.
    * *Methodology*. One worker first populates the table. Then all workers
        continuously update and get the data. Measure the time taken to fetch
        parameters from the server (*collect time*), and the average request *queue length* at the table servers.
    * *Results*: ![Table Server performance](http://www.comp.nus.edu.sg/~dinhtta/table_servers.png)
        * *Discussion*: Adding more table server improves performance. Two factors
        affect performance: request queue length and the network transfer time.

#####[Plan] 2014-12-08
* Adding support of training without parameter server, i.e., only local updates, like H2O. *by wangwei, 1 week*.
* Test and analyze parameter server performance. *by anh, 1 week*.
* Refactoring distributed array using new partition representation. *by wangsheng, 2 weeks*.
* Test H2O and Deeplearning4J (training stacked auto-encoders and multi-layer perceptron on MNIST dataset).  *by zhaojin & kaiping, 3 weeks*.


