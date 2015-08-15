---
layout: post
title: Neural Net
category : docs
tags : [installation, examples]
---
{% include JB/setup %}

SINGA represents a neural net using a data structure called *NeuralNet*,
which consists of a set of unidirectionally connected layers. Users configure
the *NeuralNet* by listing all layers of the neural net and specifying
each layer's source layer names.

    net {
      layer {
        name : 'layer1"
        type : kType1
      }
      layer {
        name : 'layer2"
        type : kType2
        srclayer: 'layer1'
      }
      layer {
        name : 'layer3"
        type : kType3
        srclayer: 'layer1'
        srclayer: 'layer2'
      }
    }

<img src="{{ BASE_PATH }}/assets/image/rbm-rnn.png" align="center" width="400px"/>
<span><strong>Figure 1. </strong></span>

This representation is natural for
feed-forward models, e.g., CNN and MLP. For energy models including RBM, DBM,
etc., their connections are undirected. To represent these models using
*NeuralNet*, users can simply replace each connection with two directed
connections, as shown in Figure 1a. In other words, for each pair of connected layers, their source
layer field should include each other's name.  The full [RBM example]({{ BASE_PATH }}/docs/rbm) has
detailed neural net configuration for a RBM model, which looks like

    net {
      layer {
        name : "vis"
        type : kVisLayer
        srclayer: "hid"
      }
      layer {
        name : "hid"
        type : kHidLayer
        srclayer: "vis"
      }
    }

For recurrent neural networks,
users can remove the recurrent connections by unrolling the recurrent layer.
For example, in Figure 1b, the original layer is unrolled into a new
layer with 4 internal layers. In this way, the model is
like a normal feed-forward model, thus can be configured similarly. The
[RNN example]({{ BASE_PATH }}/docs/rnn}) has a full neural net configuration for a RNN model.

### Training, validation and test net
Typically, a training job includes three neural nets for training, validation
and test respectively. The three neural nets share most layers except the
data layer, loss layer or output layer.

	static shared_ptr<NeuralNet> Create(const NetProto& np, Phase phase, int num);

The above function creates a `NeuralNet` for a given phase, and returns a
shared pointer to the neural net instance. The phase is in {kTrain,
kValidation, kTest}. `num` is used for net partitioning which indicates the
number of partitions.

### Neural Net Partitioning

{% comment %}
When SINGA
creates the *NeuralNet* instance, it also partitions the original neural
net according to user's configuration to support parallel training of large
models. Partitioning strategies include:

  1. Partitioning all layers into different subsets.%, one worker per subset.
  2. Partitioning each singe layer into sub-layers on batch dimension.
  3. Partitioning each singe layer into sub-layers on feature dimension.
  4. Hybrid partitioning of strategy 1, 2 and 3.
{% endcomment %}
