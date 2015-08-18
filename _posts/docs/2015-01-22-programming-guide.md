---
layout: post
title: Programming Guide
category : docs
tags : [programming]
---
{% include JB/setup %}


Each training job has a main function that

  * initializes SINGA, e.g., setup logging.

  * registers the layers (and other components) implemented by users.

  * passes the job configuration to SINGA driver, including,

    * a [NeuralNet]({{ BASE_PATH }}/docs/neural-net) describing the neural net structure with the detailed layer setting and their connections;
    * a [TrainOneBatch]({{ BASE_PATH }}/docs/train-one-batch) algorithm which is tailored for different model categories;
    * an [Updater]({{ BASE_PATH }}/docs/updater) defining the protocol for updating parameters at the server side;
    * a [Cluster Topology]({{ BASE_PATH }}/docs/distributed-training) specifying the distributed architecture of workers and servers.

An example main function is like

    #include "singa.h"
    #include "user.h"  // header for user code

    int main(int argc, char** argv) {
      singa::Driver driver;
      driver.Init(argc, argv);
      bool resume;
      // parse resume option from argv.

      // register all user defined layers
      driver.RegisterLayer<FooLayer>(kFooLayer);

      // register user defined updater
      driver.RegisterUpdater<FooUpdater>(kFooUpdater);
      ...
      auto jobConf = driver.job_conf();
      //  update jobConf

      driver.Submit(resume, jobConf);
      return 0;
    }

The Driver class' Init method will load a job configuration file provided by
users as one command line argument (`-conf <job conf>`). It contains at least the
cluster topology and returns the `jobConf` for users to update or fill in
configurations of neural net, updater, etc. If users define subclasses of
Layer, Updater, Worker and Param, they should register them through the driver.
Finally, the job configuration is submitted to the driver which starts the
training.

We will provide helper functions to make the configuration easier in the
future, like [keras](https://github.com/fchollet/keras).

Users need to compile and link their code (e.g., layer implementations and the main
file) with singa library (`.libs/libsinga.so`) to generate an
executable file, e.g., with name `mysinga`.  To launch the program, users just pass the
path of the `mysinga` and base job configuration to `./bin/singa-run.sh`.

    ./bin/singa-run.sh -conf <path to job conf> -exec <path to mysinga> [other arguments]

The default main.cc is complied and linked to generate an executable
`singa`, which is passed to `singa-run.sh` if `-exec` is not specified by users.
The default main.cc can be used only if there are no
user classes to register, i.e., the model can be trained using SINGA built-in
classes. The [RNN application]({{ BASE_PATH }}/docs/rnn) provides a full example of
implementing new layers, and the main function for training a specific RNN
model.
