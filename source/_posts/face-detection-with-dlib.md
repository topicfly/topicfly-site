---
title: Face Detection with dlib
date: 2018-10-18
author: Isaac Zepeda
author_id: izepeda
tags:
  - face-detection
  - machine-learning
  - computer-vision
  - dlib
---

<img src="/face-detection-with-dlib/cover.jpg" />

Face Detection is the first step towards Face Recognition.

There are a number of algorithms to detect faces: [haar cascade](https://www.quora.com/What-is-haar-cascade), [Histogram of Oriented Gradients](https://en.wikipedia.org/wiki/Histogram_of_oriented_gradients), [Neural Networks](https://en.wikipedia.org/wiki/Artificial_neural_network). There is a [Haar implementation in OpenCV](https://docs.opencv.org/3.4.2/d7/d8b/tutorial_py_face_detection.html), and dlib uses HOG and [Neural Networks](https://towardsdatascience.com/cnn-based-face-detector-from-dlib-c3696195e01c).

We'll learn how to detect faces in an image in a few lines of code using python, [dlib](http://dlib.net/) and [opencv](https://opencv.org/).

<!-- more -->

## The Libraries

[Dlib](http://dlib.net/) is a modern C++ toolkit containing machine learning algorithms and tools for creating complex software to solve real-world problems. We are going to use its [Python API](http://dlib.net/python/index.html).

[OpenCV](https://opencv.org/) is a Computer Vision library also with interfaces in C++, Python, and Java, and contains a lot of functions to help us to manipulate images.

## Installing Dlib and Opencv

Installing Python and its libraries are out of scope of this post. There are some tutorials out there that would help you get python, dlib and OpenCV installed. Here are a couple of links to install them on mac os:

* https://www.learnopencv.com/install-dlib-on-macos/
* https://www.learnopencv.com/install-opencv3-on-macos/

## Face Detection

In dlib API we can find the `get_frontal_face_detector` method which "returns an object detector configured to detect human faces that are looking more or less towards the camera".

The algorithm used to detect faces is [Histograms of Gradients (HOG)](https://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf) plus a [LinearSVM](https://en.wikipedia.org/wiki/Support_vector_machine).

As I mentioned dlib also has a Neural Network implementation, it performs better than HOG to detect non-frontal faces, unfortunately it's computationally heavy, and it's out of the scope in this post, if you are interested you could check the ["Sources"](#sources) section of this post for more info.

In broad terms, the HOG algorithm will iterate on every pixel of a given image, in each pixel, it checks all the pixels around it and figures out how dark the current pixel is compared with the directly pixels surrounding it. Then it draws an arrow showing in which direction is getting darker. The process is repeated on every pixel, then it breaks down in say 16x16 pixels squares, where it counts up how many gradients point in each major direction, then that square is replaced with the arrow direction that was the strongest. Then using a lot of faces, we could train a Linear Support Vector Machine model to detect faces in an image. - *Source:* [Machine Learning is Fun! Part 4: Modern Face Recognition with Deep Learning
](https://medium.com/@ageitgey/machine-learning-is-fun-part-4-modern-face-recognition-with-deep-learning-c3cffc121d78)

<img src="/face-detection-with-dlib/hog.png" />

Dlib provides a the pre-trained model using HOG through the `get_frontal_face_detector` method.

## The Code

We need to code the following steps:

1. Import libraries.
2. Get the dlib frontal face detector.
3. Load the image using OpenCV.
4. Pass the loaded image to the `detector`, it returns an array of rectangles, each rectangle containing the detected face bounds.
5. Iterate over the found faces and draw a rectangle in the original image.
6. Show image with detected faces

```python facedetection.py
# 1. Import libraries
import argparse
import dlib
import cv2

# Use argparse so we can send the image path from the command line
ap = argparse.ArgumentParser()
ap.add_argument("-i", "--image",
    help="Image to input image to detect faces")
args = vars(ap.parse_args())
f = args['image']

# 2. Get the dlib frontal face detector
detector = dlib.get_frontal_face_detector()

print("Processing file: {}".format(f))

# 3. Load the image using OpenCV
img = cv2.imread(f)

# 4. Pass de loaded image to the `detector`
dets = detector(img, 1)
print("Number of faces detected: {}".format(len(dets)))

# 5. Iterate over the found faces and draw a rectangle in the original image.
for i, d in enumerate(dets):
    print("Detection {}: Left: {} Top: {} Right: {} Bottom: {}".format(
        i, d.left(), d.top(), d.right(), d.bottom()))
    cv2.rectangle(img, (d.left(), d.top()), (d.right(), d.bottom()),
                (0, 0, 255), 2)

# 6. Show image with detected faces
cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

## Results

Let's use a couple of images from one of my favorite face-recognition-general-ai-related tv shows, [Person of Interest](https://www.imdb.com/title/tt1839578).

<img src="/face-detection-with-dlib/example001.png" />

We need to run the script in command-line using the `-i` or `--image` argument to the image where we want to detect the faces:

```bash
> python facedetection.py -i example001.png
Processing file: example001.png
Number of faces detected: 2
Detection 0: Left: 476 Top: 150 Right: 631 Bottom: 305
Detection 1: Left: 975 Top: 253 Right: 1130 Bottom: 408
```

We could see in the console output the detected rectangles and the left-top and bottom-right points coordinates. The result looks like this:

<img src="/face-detection-with-dlib/example001_detected.png" />

Another example

<img src="/face-detection-with-dlib/example002_detected.png" />


## <a name="sources"></a>Sources

* http://dlib.net/face_detector.py.html
* https://www.pyimagesearch.com/2018/01/22/install-dlib-easy-complete-guide/
* https://www.pyimagesearch.com/2017/04/03/facial-landmarks-dlib-opencv-python/
* https://www.youtube.com/watch?v=0Zib1YEE4LU
* https://en.wikipedia.org/wiki/Histogram_of_oriented_gradients
* https://www.learnopencv.com/histogram-of-oriented-gradients/
* Paper [Histograms of Oriented Gradients for Human Detection](https://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf)
* https://medium.com/@ageitgey/machine-learning-is-fun-part-4-modern-face-recognition-with-deep-learning-c3cffc121d78
* https://towardsdatascience.com/cnn-based-face-detector-from-dlib-c3696195e01c
* https://www.quora.com/What-is-haar-cascade
* https://en.wikipedia.org/wiki/Artificial_neural_network
