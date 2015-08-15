---
layout: post
title: TrainOneBatch
category : docs
tags : [neural net]
---
{% include JB/setup %}

For each SGD iteration, every worker calls the *TrainOneBatch* function to
compute gradients of parameters associated with local layers (i.e., layers
dispatched to it). SINGA has implemented two algorithms for the
*TrainOneBatch* function. Users select the corresponding algorithm for
their model in the configuration.

The above algorithm shows the back-propagation (BP) algorithm for
feed-forward models and recurrent neural networks. It forwards (Line 1-3)
features through all local layers and backwards gradients in the reverse order
(Line 4-6). Since RNN models are unrolled into feed-forward models
(*ComputeFeature* and *ComputeGradient* functions compute for all
internal layers), the algorithm runs as the back-propagation through
time (BPTT) algorithm for them.


The above algorithm illustrates the
contrastive divergence (CD) algorithm for
energy models. Parameter gradients are computed (Line 7-9) after the positive
phase (Line 1-3) and negative phase (Line 4-6). *kCD* controls the number
of Gibbs sampling iterations in the negative phase. In both algorithms, the
*Collect* function is blocked until the parameters are fetched from
servers. The *Update* function returns immediately after sending the
gradients.
