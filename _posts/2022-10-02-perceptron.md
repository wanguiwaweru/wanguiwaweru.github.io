# Perceptron : Understand the building blocks of Neural Networks

A Perceptron is an artificial neuron(based on the idea of biological neurons); thus,a neural network unit. 

The pereptron is a mathematical function that performs computations to detect features or patterns in the input data.
The function can be represented as shown below.

1\.  Input values

The input values are the features in our dataset. n represents the total number of features and X represents the value of the feature(rows).

They form the first layer of the neural network which is refered to as the **input layer**.

2\. Weights and bias

The weights(w<sub>1</sub>,+ w<sub>2</sub>,...,w<sub>n</sub>) are coefficients to be multiplied with the input values. They determine how much a certain feature influences the output.

The weights are initialized as random values as values then they get automatically updated after each training error between the predicted result and the known result. The error is back propagated in order to adjust the weights.

The bias(w<sub>0</sub>)is a constant term added to the network to allows the classifier to move the decision boundary around from its original position to the right, left, up, or down

Training the model involves finding the right values for the weights and bias.

3\. Net sum 

This is the sum of all the inputs multiplied by their weights and the bias term(w<sub>0</sub>)

Net Sum = w<sub>0</sub> + w<sub>1</sub>X<sub>1</sub> + w<sub>2</sub>X<sub>2</sub> +.. + w<sub>n</sub>X<sub>n</sub>

4\. Activation function

The activation function decides whether a neuron should be activated or not by calculating the weighted sum and adding bias to it. It introduces non-linearity into the neural network.

5\. Output

The output can be a single value or multiple values.

## References
- Géron, A. (2019) Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow, 2nd Ed., O'Reilly Media
- [Please Stop Drawing Neural Networks Wrong](https://towardsdatascience.com/please-stop-drawing-neural-networks-wrong-ffd02b67ad77)