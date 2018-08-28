---
title: Classify MNIST handwritten digits with Keras
date: 2018-07-24
author: Isaac Zepeda
author_id: izepeda
tags:
    - machine-learning
    - deep-learning
    - keras
    - convolutional-neural-network
---

One of the basic examples in Machine Learning is to classify handwritten digits using the MNIST dataset. Here I'll show you how to solve this problem using a Convolutional Neural Network (CNN) with [Keras](https://keras.io/) a high-level neural networks API.

<!-- more -->

## The MNIST Dataset

The [MNIST Dataset](http://yann.lecun.com/exdb/mnist/) contains 60,000 examples of handwritten digits images normalized to fit a 28x28 pixel bounding and anti-aliased which introduced grayscale levels.

<img src="/classify-mnist-digits-keras/mnist-examples.png" />

Keras already has a [datasets module](https://keras.io/datasets/) from which I'll load the MNIST dataset and automatically get my training and test datasets.

## Convolutional Neural Networks

Convolutional Neural Networks (CNN) make the assumption that the input are images.

The CNN architecture is defined by different layers: Input Layer, Convolutional Layer, Activation Layer (ReLU), Pooling Layer, Fully-Connected Layer.

The **input layer** normally has the shape `height x width x depth`, in this post example it's *28x28x1* but in color images the *depth* would be 3, one for each value in the RGB color model.

The input then is passed to a **convolutional layer** using `n` filters, it creates another matrix with different height, width and as deep as many filters defined. A filter is a small window normally *5x5xinput_depth* in size, it traverses all the image to get its features. Its output goes into and activation function, in this case a <a href="https://en.wikipedia.org/wiki/Rectifier_(neural_networks)">ReLu function</a>.

The resulting layers then goes into a **pooling layer** that reduces dimensionality. Then the **fully-connected layer** is a normal feed forward layer for classification.

We could combine the conv layer with another conv to a pooling layer and repeat it *n* times before going to the fully-connected layer that at the same time could have several fully-connected layers.

<img src="/classify-mnist-digits-keras/conv.png" />

For further detail about Convolutiona Neural Networks you can visit http://cs231n.stanford.edu/ or go the [bibliography section](#bibliography).


## Libraries and modules

```python
import numpy as np
np.random.seed(123)  # for reproducibility

from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras.utils import np_utils
from keras.datasets import mnist

from matplotlib import pyplot as plt
```

I import [Numpy](http://www.numpy.org/) for matrix manipulation, [pyplot](https://matplotlib.org/api/pyplot_api.html) from [matplotlib](https://matplotlib.org/) to render some graphic representation of the data, `keras.util` to more data manipulation, and the [MNIST dataset](https://keras.io/datasets/#mnist-database-of-handwritten-digits) provided by keras.

The keras [Sequential](https://keras.io/models/sequential/) model allows to `add` layers, each layer having its own architecture and purpose.

The idea is to sequentially add different layers to our model util having a Neural Network architecture that can process and classify images of handwritten digits.

## Data Preparation

Every Machine Learning model needs some data preparation. Sometimes this is the most exhaustive and underrated (for beginners) part of the process.

1. Load dataset.
2. Separate dataset into training and test datasets.
3. Visualize data to get some intuition.
4. Prepare input data to feed the input layer.
5. Prepare label data.

```python
# load dataset
(X_train, y_train), (X_test, y_test) = mnist.load_data();
```

The `load_data()` function return two tuples, already splitting the data in training and test collections.

The `X_train.shape` will be `(60000, 28, 28)`, 60,000 examples of 28x28 pixes grayscale images of the 10 digits while `y_train` is an vector fo 60,000 examples where each element is an integer from 0 to 9, and (`X_test` and `y_test` contains 10,000 examples. Remember that `X_train[n]` corresponds to `y_train[n]`.

Let's visualice the 4 first samples as images using `pyplot`.

```python
plt.subplot(221)
plt.imshow(X_train[0], cmap=plt.get_cmap('gray'))
plt.subplot(222)
plt.imshow(X_train[1], cmap=plt.get_cmap('gray'))
plt.subplot(223)
plt.imshow(X_train[2], cmap=plt.get_cmap('gray'))
plt.subplot(224)
plt.imshow(X_train[3], cmap=plt.get_cmap('gray'))
```

<img src="/classify-mnist-digits-keras/pyplot-digits.png" width="300px" />

The next step is to reshape the input matrix to have the shape `batch x height x width x channels`, since these are grayscale images the *channels* will be 1, in other images the *channels* normally would be 3, one for each color in the RGB color model.

*Note: I'm using tensorflow as my keras backend, for tensorflow the input shape needs to have the shape (batch, height, width, channels) this is the current default input shape for `Conv2D`, but other backends accept the shape (batch, channels, height, width) as default. You could change this value by setting `data_format` parameter in the `Conv2D` initialization to `channels_last` (current default) or `channels_first`. Also by modifying the value in the Keras config file.*

```python
X_train = X_train.reshape(X_train.shape[0], 28, 28, 1)
X_test = X_test.reshape(X_test.shape[0], 28, 28, 1)
print(X_train) # (60000, 28, 28, 1)
```

Also I'll normalize the input data to be in the range from `0` to `1`, first I'll change the type to `float32` and divide all elements by 255.

```python
X_train = X_train.astype('float32')
X_test = X_test.astype('float32')
X_train /= 255
X_test /= 255
```

Remember the label data `y_train` is a 1-dimensional array with an integer value from 0 to 9, I'm converting to an 10-dimensional class matrices. So if the `y_train[0]` is `5` it will be an array of 10 elements where the 4th element would be `1.0` and the rest `0.0`, *this way I can compare my output layer of 10 nodes with this labeled data*.

```python
Y_train = np_utils.to_categorical(y_train, 10)
Y_test = np_utils.to_categorical(y_test, 10)
print(Y_train[0]) #[0. 0. 0. 0. 0. 1. 0. 0. 0. 0.]
```

## Neural Network architecture

There are several layer architectures that I could implement, I decided to go as follows:

```python
model = Sequential()
model.add(Conv2D(30, (5, 5), activation='relu', input_shape=(28,28,1)))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Conv2D(15, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2,2)))
model.add(Dropout(0.2))

model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dense(50, activation='relu'))
model.add(Dense(10, activation='softmax'))
```

The [keras Sequential model](https://keras.io/models/sequential/) allows to generate a model for training by adding layers to it.

The first layer is a [Conv2D](https://keras.io/layers/convolutional/#conv2d) layer with a <a href="https://en.wikipedia.org/wiki/Rectifier_(neural_networks)">ReLU activation function</a>, `30` output filters and a 5x5 convolutional window. The `input_shape` is the same shape as each element in `X_train` and `X_test`, notice that I only defined the `input_shape` in the first layer.

Then comes the Pooling layer, I'm using the [MaxPooling2D](https://keras.io/layers/pooling/#maxpooling2d) keras layer with a 2x2 pool size.

After that I connected another `Conv2D` with ReLU activation, 15 filters and 3x3 window and connected it to a 2x2 `MaxPooling2D` layer.

I use a [Dropout](https://keras.io/layers/core/#dropout) layer for regularization to prevent overfitting.

After the convolutions I want to classify the data with fully connected layers, thus I use a [Flatten](https://keras.io/layers/core/#flatten) layer to flat the `Dropout` output to a 1-dimensional vector to be used as input for two [Dense](https://keras.io/layers/core/#dense) (fully-connected) layers with ReLU activation and with 128 and 50 nodes respectively.

The output layer is a `Dense` layer with 10 nodes, same dimension as an element in `Y_train` and `Y_test`, and [softmax](https://en.wikipedia.org/wiki/Softmax_function) activation function.

I could use more or less Conv2D layers, or add other Dropout between the MaxPooling2D and the second Conv2D, I could change the hyperparameters in each layer, or the total nodes in one of the Dense layers, or use one Dense layer instead of two. Trying several architectures and measuring which generalize better is part of the Data Scientist/Machine Learning Engineer daily job.

## Compile model

In the compilation section I'll define some model configuration like loss and optimizer.

```python
model.compile(loss='categorical_crossentropy',
             optimizer='adam',
             metrics=['accuracy'])
```

I'll go with categorical [cross-entropy](http://ml-cheatsheet.readthedocs.io/en/latest/loss_functions.html#cross-entropy) as loss function, and [Adam algorithm](https://machinelearningmastery.com/adam-optimization-algorithm-for-deep-learning/) to optimize it.

## Train model

The model is defined and configured, now it needs to fit the training data through some "epochs" until the network weights can generalize our data.

```python
history = model.fit(X_train, Y_train,
                    validation_data=(X_test, Y_test),
                    epochs=5,
                    batch_size=200,
                    verbose=1)
```

You could select and tweak the `epochs` and `batch_size` (numbers of examples in each epoch). I like the to use `verbose=1` because it gives you a nice animated training status and progress in the console or jupyter notebook output.

<img src="/classify-mnist-digits-keras/keras-fit.png" />

All this data is helpful because when I'm training my model I want the loss to be as close to zero as possible and accuracy close to `1.0` (100%). Also it displays how much time each epoch and step took (50s approx per epoch).

I have assigned the `fit` returned value to `history` variable, since I'm going to print a nice plots in the further section.

## Evaluate model

Now I'm using my test dataset to `evaluate` the model.

```python
score = model.evaluate(X_test, Y_test, verbose=0)
```

The `score` contains an array where the first element is the loss and the second is accuracy, you could check the metrics name in `model.metrics_name`.

```python
print("CNN Error: %.2f%%" % (100-score[1]*100))
# CNN Error: 0.74%
print("Loss: %.2f" % score[0])
# Loss: 0.03
print("Accurracy: %.2f%%" % (score[1]*100))
# Accurracy: 99.06%
```

## Visualize training history

Using the `history` variable I could create a couple of plots to check the model accuracy and model loss during training.

```python
# summarize history for accuracy
plt.plot(history.history['acc'])
plt.plot(history.history['val_acc'])
plt.title('model accuracy')
plt.ylabel('accuracy')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
# summarize history for loss
plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('model loss')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```

<img src="/classify-mnist-digits-keras/plots.png" width="320px" />

## Complete Code

```python
import numpy as np
np.random.seed(123)  # for reproducibility

from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras.utils import np_utils
from keras.datasets import mnist

from matplotlib import pyplot as plt

(X_train, y_train), (X_test, y_test) = mnist.load_data();

plt.subplot(221)
plt.imshow(X_train[0], cmap=plt.get_cmap('gray'))
plt.subplot(222)
plt.imshow(X_train[1], cmap=plt.get_cmap('gray'))
plt.subplot(223)
plt.imshow(X_train[2], cmap=plt.get_cmap('gray'))
plt.subplot(224)
plt.imshow(X_train[3], cmap=plt.get_cmap('gray'))

X_train = X_train.reshape(X_train.shape[0], 28, 28, 1)
X_test = X_test.reshape(X_test.shape[0], 28, 28, 1)
X_train = X_train.astype('float32')
X_test = X_test.astype('float32')
X_train /= 255
X_test /= 255

Y_train = np_utils.to_categorical(y_train, 10)
Y_test = np_utils.to_categorical(y_test, 10)

model = Sequential()
model.add(Conv2D(30, (5, 5), activation='relu', input_shape=(28,28,1)))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Conv2D(15, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2,2)))
model.add(Dropout(0.2))

model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dense(50, activation='relu'))
model.add(Dense(10, activation='softmax'))

model.compile(loss='categorical_crossentropy',
             optimizer='adam',
             metrics=['accuracy'])

history = model.fit(X_train, Y_train,
                    validation_data=(X_test, Y_test),
                    epochs=5,
                    batch_size=200,
                    verbose=1)

score = model.evaluate(X_test, Y_test, verbose=0)
print("CNN Error: %.2f%%" % (100-score[1]*100))
print("Loss: %.2f" % score[0])
print("Accuracy: %.2f%%" % (score[1]*100))

plt.plot(history.history['acc'])
plt.plot(history.history['val_acc'])
plt.title('model accuracy')
plt.ylabel('accuracy')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()

plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('model loss')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```

## Conclusion

I really like keras to create deep learning models quickly and easily, thus testing ideas and hypothesis in datasets become a more efficient and pleasant work.

I know we didn't cover more in depth topics like what's the loss function or the math behind Adam optimizer, or more about Convolutional Networks theory but I'll leave all those topics for future posts.

## <a name="bibliography"></a> Bibliography

Here are some links that helped me put together this post:

* http://cs231n.github.io/convolutional-networks/
* https://www.youtube.com/watch?v=jajksuQW4mc
* https://medium.freecodecamp.org/an-intuitive-guide-to-convolutional-neural-networks-260c2de0a050
* https://elitedatascience.com/keras-tutorial-deep-learning-in-python#step-7
* https://machinelearningmastery.com/handwritten-digit-recognition-using-convolutional-neural-networks-python-keras/
* https://medium.com/the-theory-of-everything/understanding-activation-functions-in-neural-networks-9491262884e0
* http://ml-cheatsheet.readthedocs.io/en/latest/loss_functions.html#cross-entropy
* https://machinelearningmastery.com/adam-optimization-algorithm-for-deep-learning/
* https://machinelearningmastery.com/display-deep-learning-model-training-history-in-keras/
* http://yann.lecun.com/exdb/mnist/
