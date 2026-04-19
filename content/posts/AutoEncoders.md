---
title: "AutoEncoders"
date: "2023-11-16"
draft: false
---

# AutoEncoders

Autoencoders (AE) are generative models used in unsupervised learning. AutoEncoders are neural network that compresses the input data into a lower dimension and uses it to recreate the original input.

Their architecture consists of an encoder and a decoder.

![Autoencoders](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*vRfGgPmRBAC7Gz_HQ5ldoQ.jpeg)

Autoencoders enable you to learn the structure of unlabeled data and the interesting features in the data which can help identify anomalies and extract important features.

The encoder and decoder work together to find the most efficient way to condense the input data into a lower dimension.

The difference between the attempted recreation output and the original input is the reconstruction error.

The network finds the lower dimensional representation by minimizing the reconstruction error as the network learns to exploit the structure in the data.

## The Encoder

The encoder consists of a series of neural network layers which extract features from a dataset and embed or encode them to a low-dimensional latent space.

For instance, if we have a dataset containing information about countries and their capital cities our entries might include Kigali Rwanda, Berlin Germany etc. It is conceptually possible to have an entry like Moscow,Spain but we don't have such a scenario in real data because real data is not spread out across all possibilities and is constrained.

Real data often lies on a lower dimensional subspace within the full dimensionality of the input. In our case, the Moscow, Spain space is unoccupied. Based on the constraints in the data, we can describe the same data but on a lower dimension without losing important information.

Autoencoders attempt to find the function that maps the data from its full input space into a lower dimension that takes advantage of the structure in the data.

Real data is not random but instead has structure; therefore, we don't need every part of the full input space to represent data.

Generally, the architecture of the encoder is converging, as the latent space is lower-dimensional than the input space.

## The Decoder

The decoder attempts to recreate the original input using the encoder's output which is of a lower dimensionality than the original.

If there is no information lost between the encoder and decoder then the network could simply learn to multiply the input by one and reconstruct the input. This would be an unnecessary use of neural networks.

Auto-encoders enforce information loss with the network bottleneck which is created by tuning the network architecture such that the inner dimension is less than the dimension needed to express our data.

To ensure the network does not learn the trivial solution of multiplying by one we add noise to the input and the network learns to remove the noise.

## Applications of Autoencoders

- **Anomaly detection**: If a data point is sufficiently distinct(anomalous) from the other dataset the auto-encoder will have difficulties reproducing it with its learned weights, and the reconstruction loss will be high. The reconstruction loss is used as the anomaly score and above a specific loss the input can be considered an anomaly. This can be applied in fraud detection.

- **Filling in missing values in a dataset**: The models can be used to predict what the missing values are likely to be because it has learned the structure of the data.

- **Feature extractor**: Autoencoders can be used as feature extractors by using the encoder part to extract meaningful features from the data.

## References

1. [Autoencoders](https://www.youtube.com/watch?v=3jmcHZq3A5s&t=109s&pp=ygUhYXV0b2VuY29kZXIgZm9yIGFub21hbHkgZGV0ZWN0aW9u)
2. [Masked Autoencoders Are Scalable Vision Learners](https://openaccess.thecvf.com/content/CVPR2022/papers/He_Masked_Autoencoders_Are_Scalable_Vision_Learners_CVPR_2022_paper.pdf)