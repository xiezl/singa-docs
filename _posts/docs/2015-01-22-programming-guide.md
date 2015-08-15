---
layout: post
title: Programming Guide
category : docs
tags : [programming]
---
{% include JB/setup %}

This guide provides instructions of implementing a new model and submitting the
training job in SINGA. The programming model is made almost transparent to the
underlying distributed environment. Hence users do not need to worry much about
the [distributed training]({{ BASE_PATH }}/docs/distributed-training).

### Deep learning training

Deep learning is labeled as a feature learning technique, which usually
consists of multiple layers.  Each layer is associated with a feature
transformation
function. After going through all layers, the raw input feature (e.g., pixels
of images) would be converted into a high-level feature that is easier for
tasks like classification.

Training a deep learning model is to find the optimal parameters involved in
the transformation functions that generates good features for specific tasks.
The goodness of a set of parameters is measured by a loss function, e.g.,
[Cross-Entropy Loss](https://en.wikipedia.org/wiki/Cross_entropy). Since the
loss functions are usually non-linear and non-convex, it is difficult to get a
closed form solution. Normally, people uses the SGD algorithm which randomly
initializes the parameters and then iteratively update them to reduce the loss
as shown in Figure 1.
<img src="{{ BASE_PATH }}/assets/image/sgd.png" align="center" width="400px"/>
<span><strong>Figure 1. SGD flow</strong></span>



### SINGA training flow
<img src="{{ BASE_PATH }}/assets/image/overview.png" align="center" width="400px"/>
<span><strong>Figure 2. SINGA overview</strong></span>

The stochastic gradient descent (SGD) algorithm is used in SINGA to train
parameters of deep learning models. The training workload is distributed over
worker and server units as shown in Figure 2. In each
iteration, every worker calls *TrainOneBatch* function to compute
parameter gradients. *TrainOneBatch* takes a *NeuralNet* object
representing the neural net, and visits layers of the *NeuralNet* in
certain order. The resultant gradients are sent to the local stub that
aggregates the requests and forwards them to corresponding servers for
updating. Servers reply to workers with the updated parameters for the next
iteration. To start a training job, users [prepare training data]({{ BASE_PATH }}/docs/data) and submit a job configuration consisting
of four components:

  * a [NeuralNet]({{ BASE_PATH }}/docs/neural-net) describing the neural net structure with the detailed layer setting and their connections;
  * a [TrainOneBatch]({{ BASE_PATH }}/docs/train-one-batch) algorithm which is tailored for different model categories;
  * an [Updater]({{ BASE_PATH }}/docs/updater) defining the protocol for updating parameters at the server side;
  * a [Cluster Topology]({{ BASE_PATH }}/docs/distributed-training) specifying the distributed architecture of workers and servers.

If some layers are not built-in layers in SINGA, users have to implement them
on their own. [Layer]({{ BASE_PATH }}/docs/layer) describes how to extend the
base layer class to implement new layers.

### Main function

Each training job has a main function that
  * initializes SINGA, e.g., setup logging.

  * registers the layers (and other components) implemented by users.

  * submits the job by providing the job configuration.

  An example driver program is like

    #include "singa.h"
    #include "user.h"  // header for user code

    int main(int argc, char** argv) {
      singa::Driver driver;
      dirver.Init(argc, argv);
      bool resume;
      // parse resume option from argv.

      // register all user defined layers in user-layer.h
      RegisterLayer<FooLayer>(kFooLayer);
      RegisterUpdater<FooUpdater>(kFooUpdater);
      ...
      auto jobConf = diver.job_conf();
      //  update jobConf
      singa::Submit(resume, jobConf);
      return 0;
    }

The Driver class will load a job configuration file which contains at least the
cluster topology. It returns the `jobConf` for users to update or fill in
configurations of neural net, updater, etc. If users define subclasses of
Layer, Updater, Worker and Param, they should register them through the driver.
Finally, the job configuration is submitted to the driver which starts the
training.

We will provide helper functions to make the configuration easier in the
future, like [keras](https://github.com/fchollet/keras).

Users need to compile and link this code (e.g., layer implementation and main
function) with singa library (`.libs/libsinga.so`) to generate an
executable file, e.g., with name `mysinga`.  To launch the program, users just pass the
path of the `mysinga` and base job configuration to `./bin/singa-run.sh`.

    ./bin/singa-run.sh -conf <path to job conf> -exec <path to mysinga> [other arguments]

The default main.cc will be complied and linked to generate an executable
`singa`, which is passed to `singa-run.sh` if `-exec` is not specified by users.
The [RNN application]({{ BASE_PATH }}/docs/rnn) provides a full example of
implementing a new model and submitting a training job.
