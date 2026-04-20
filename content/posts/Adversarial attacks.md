---
title: "Adversarial Attacks on Neural Networks"
date: "2023-10-22"
draft: true
---

# Adversarial Attacks on Neural Networks

Neural networks have proven to be very capable of achieving various tasks and sometimes outperform humans in tasks such as [classifying Covid-19 images¹](https://doi.org/10.1155/2021/5527271).

However, the modelsare prone to error through attacks that confuse the model into making wrong predictions when small changes are introduced to the input. The small perturbations are designed to have a significant effect on the model's performance even when the change is not visible to us.

> Adversarial attacks are inputs intentionally crafted to cause a machine learning model to make incorrect predictions while appearing normal or imperceptible to humans.

Adversarial attacks have been demonstrated in both computer vision and natural language processing domains.

In computer vision, there are numerous adversarial examples where models are unable to make correct predictions when noise is introduced to the input. The models make incorrect predictions when changes such as image rotation and changing lighting conditions are introduced².

![misclassified image](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*vRfGgPmRBAC7Gz_HQ5ldoQ.jpeg)

Morris et al.³ demonstrated how natural language processing classification models including state-of-the-art models can be fooled into misclassifying sentiments by changing one word in a sentence.

## Adversarial Attacks in Computer Vision

An adversarial attack refers to an attack on a neural network where a subtle carefully designed perturbation to the input leads to incorrect predictions while the original input is still classified correctly.

The change may be invisible to the human eye, but the model considers it important enough to change its prediction.

The attacks are classified into white box attacks, black box attacks and grey box attacks.

In a white box attack, the attacker knows the model's architecture and its parameters which they can adjust accordingly.

In blackbox attacks the attacker does not know the model architecture and relies on the outputs of the model.

In grey-box attacks, the attacker has partial information about the model, such as some parameters, architecture knowledge, or query budget limits.

Adversarial examples refer to input that has been transformed by adding some distortion to the original input that causes the model to make an incorrect prediction.

## Attack Methods

Adversarial attacks on computer vision models can be achieved through different approaches including:

- Changing pixel values of an input image slightly to trick the model⁷.
- Generating patches that are digitally placed on the image fooling the model⁴. The patches can also be printed and placed on an object.
- Optimizing the texture of a 3D model and presenting images of the printed 3D model to the classifier⁸.
- Generating a single universal image that can be used as an adversarial perturbation on different images that fools a state-of-the-art deep neural network classifier on all natural images⁹.

Most adversarial attack techniques described above use gradient-based methods where the attackers modify the image in the direction of the gradient of the loss function with respect to the input image.

### Gradient-Based Attacks
The attacker uses the gradient of the loss function with respect to the input to create an adversarial example. Computing the gradient of the loss function with respect to the input image and then modifies the image in the direction that increases the loss, which leads to misclassification by the model.


The loss function is a measure of how well the model's predictions match the true labels. By maximizing the loss, the attacker can find an input that causes the model to make an incorrect prediction. The gradient of the loss function is calculated as the partial derivative of the loss function with respect to the input image. The attacker can then modify the input image by adding a small perturbation in the direction of the gradient, which can cause the model to misclassify the image.

#### FGSM (Fast Gradient Sign Method)


### Generating an Adversarial Example

When training a neural network we are trying to reduce the loss function by minimizing the error between the actual and the predicted value. We adjust the weights and bias to optimize the model's ability to make the correct prediction.

Adversarial attacks take advantage of this technique and try to increase the loss function such that the weights and target class remain fixed and we try changing input to maximize loss.

One algorithm used to achieve this is the Fast Gradient Sign Method which works as shown in the image below.

![adversarial example](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*vRfGgPmRBAC7Gz_HQ5ldoQ.jpeg)

1. Take partial derivative of loss with respect to input data (the image). Move the image pixels in the direction that will increase the loss.
2. Apply sign function on this gradient which produces -1 and 1 for negative and positive values, respectively.
3. Multiply this signed gradient with small epsilon value.
4. Add this value (perturbation) to the image pixel values.


### Why Adversarial attacks are a concern.

Neural networks have applications in safety critical systems such as self-driving cars and facial recognition systems and adversarial attacks on these real-world applications can have detrimental effects.

Here are some examples of attacks on safety critical systems.

- Neural networks classifying stop signs as speed limits when the images are perturbed⁴.
- 3D printed glasses that make people unrecognizable to facial recognition models⁵.
- Printed patches that fool person detection models allowing people to avoid surveillance cameras⁶.

## Defences

- **Adversarial training**: This involves training models with adversarial examples enabling them to generalize well. Adversarial training is effective in defending against whitebox attacks but less effective on black box attacks. Generating the adversarial examples can be expensive as it requires high computation.

- **Defensive distillation**: This involves using another model that is trained to detect adversarial examples because the first model is trained with "hard" labels (100% probability that an image belongs to one class or the other) and then provides "soft" labels (95% probability) that is used to train the second model. This technique is more adaptable to unknown threats but is still limited to the parameters of the first model and adds additional costs.

### Conclusion

Neural networks are powerful deep learning models with many real-world applications but we should be aware of their vulnerabilities. Adversarial attacks pose a significant security concern for neural networks because small input changes can fool the model to make incorrect predictions.

Implementing defences against adversarial attacks increases the model's robustness allowing it to handle noisy input while maintaining its accuracy. Current techniques can handle some attacks though there are opportunities for developing more effective defences.

## References

[1] [Radiologists versus Deep Convolutional Neural Networks: A Comparative Study for Diagnosing COVID-19](https://doi.org/10.1155/2021/5527271).

[2] [Adversarial Examples In The Physical World](https://arxiv.org/pdf/1607.02533.pdf).

[3] [TextAttack: A Framework for Adversarial Attacks, Data Augmentation, and Adversarial Training in NLP](https://doi.org/10.48550/arXiv.2005.05909)

[4] [Robust Physical-World Attacks on Deep Learning Visual Classification](https://arxiv.org/pdf/1707.08945.pdf)

[5] [Accessorize to a Crime: Real and Stealthy Attacks on State-of-the-Art Face Recognition](https://doi.org/10.1145/2976749.2978392).

[6] [Fooling automated surveillance cameras: adversarial patches to attack person detection](https://doi.org/10.48550/arXiv.1904.08653)

[7] [Explaining and Harnessing Adversarial Examples](https://arxiv.org/pdf/1412.6572.pdf)

[8] [Synthesizing Robust Adversarial Examples](https://arxiv.org/pdf/1707.07397.pdf)

[9] [Universal adversarial perturbations](https://arxiv.org/pdf/1610.08401.pdf)

Consider this paper below for further reading: [Real attackers do not compute gradients.](https://arxiv.org/pdf/2212.14315.pdf)
Let us explore an example of an adversarial attack where pixels are changed slightly.
