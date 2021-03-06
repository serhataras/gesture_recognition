# Gesture Recognition Program

This program aim to recognize hand gestures based on a small dataset of images.

This code uses the following frameworks and libraries :

* Tensorflow
* Keras
* OpenCV

# Thinking Process

## OpenCV

At first, I thought that OpenCV could have been enough to solve the problem. By applying several image processing techniques — such as Gaussian mixture-based background subtraction algorithm and convolution — a pretty good job was done !

**[Watch the video](https://github.com/OmarAflak/gesture_recognition/blob/master/res/video1.mp4?raw=true)**


The idea here was to capture a still background then "subtract" it from the other frames which would eventually seperate the user from the rest.
Then converting the image into gray scale and thresholding at a certain value would result in a black and white image.

<img src="https://github.com/OmarAflak/gesture_recognition/blob/master/res/image1.jpg?raw=true" />


Then I would find the biggest contour using a convex hull finder and assume that it is the user's hand. Finally I would count the number of edges found in the previous step and deduce the number of fingers on the screen and thus the shape of the hand.

<img src="https://github.com/OmarAflak/gesture_recognition/blob/master/res/image2.jpg?raw=true" />

Unfortunately, this could not work in a real life situation because of the background which wouldn't have been static.

## Machine Learning

The next try was all about machine learning. I took several videos of my hand in different positions and I injected this dataset into a neural network which had to classify the each picture into a gesture category (e.g. rock, paper, scissors).

This didn't work either. As the images contained a complex background (e.g people moving, the sky, the lights, etc.) and because I had too few images, the network couldn't generalize well.

## OpenCV + Machine Learning

I knew that machine learning was the right method to use, but I had no dataset. I had to make it somehow easier for the network to learn. The solution was to process the images before feeding them into the network.

If I could extract the shape of the hand from an image without its background, then grayscale the image so colors don't really matter, it would have been great !

First problem : **how to extract the hand from the background ?**

A good solution was to filter the image based on a **HSV skin color** range. It worked for me with empirical values but for someone with a lighter/darker skin it could have failed. Fortunately, OpenCV has a built-in face recognition tool. This allowed me to track the user's face, pick its color, and filter the image based on this color.
This technique works while the assumption that our **face** and our **hands** have **almost the same color** is true.

The **HSV range** used for filtering is defined as follow :
```
hsv_face = detect_face_hsv()
hsv_lower_range = hsv_face - [10, 100, 100]
hsv_upper_range = hsv_face + [10, 255, 255]
```

Once the user's hand extracted from the background, a **gray scaling** is applied, the image is labeled and it's ready for the neural network.

<img src="https://github.com/OmarAflak/gesture_recognition/blob/master/res/image3.png?raw=true" />

### Network Architecture

I used **Keras** which is a high level framework built on top of **Tensorflow** and which allows you to create complex neural networks easily.
For image recognition problems, **convolutional neural networks** have proven to be very efficient (no implementation of CapsNet for now...). Therefore, the neural network I used for this program has the following architecture :

```
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d_1 (Conv2D)            (None, 32, 32, 64)        640       
_________________________________________________________________
max_pooling2d_1 (MaxPooling2 (None, 16, 16, 64)        0         
_________________________________________________________________
dropout_1 (Dropout)          (None, 16, 16, 64)        0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 16, 16, 128)       73856     
_________________________________________________________________
max_pooling2d_2 (MaxPooling2 (None, 8, 8, 128)         0         
_________________________________________________________________
dropout_2 (Dropout)          (None, 8, 8, 128)         0         
_________________________________________________________________
conv2d_3 (Conv2D)            (None, 8, 8, 256)         295168    
_________________________________________________________________
max_pooling2d_3 (MaxPooling2 (None, 4, 4, 256)         0         
_________________________________________________________________
dropout_3 (Dropout)          (None, 4, 4, 256)         0         
_________________________________________________________________
flatten_1 (Flatten)          (None, 4096)              0         
_________________________________________________________________
dense_1 (Dense)              (None, 256)               1048832   
_________________________________________________________________
dropout_4 (Dropout)          (None, 256)               0         
_________________________________________________________________
dense_2 (Dense)              (None, 5)                 1285      
=================================================================
Total params: 1,419,781
Trainable params: 1,419,781
Non-trainable params: 0
_________________________________________________________________
```

<img src="https://github.com/OmarAflak/gesture_recognition/blob/master/res/accuracy.png?raw=true" />

<img src="https://github.com/OmarAflak/gesture_recognition/blob/master/res/loss.png?raw=true" />

With this architecture we manage to get a 96.03% accuracy on validation set.

## Image Augmentation with Keras

The dataset generated using videos of my hands may not be enough. Fortunately, Keras implements a powerful tool called **image augmentation**. This api can generate many images by applying transformations such as rotations, translations, zooming, shifting etc. to an existing image.

# How to use

Four files are included in this repository :

* **dataset_builder.py** is used to build a dataset using the camera
* **cnn_trainer.py** is used to train the neural network using the previously generated dataset
* **cnn_tester.py** is used to test the previously trained neural network
* **skin_reco.py** is an "utils" file that contains functions used for skin subtraction

## Step 1 : Build a dataset

In **dataset_builder.py** there is a **dataset_folder** variable which is set by default to **gestures**.

This folder should include a different folder for each class e.g.

```
--- dataset/
    --- cars/
        - car1.png
        - car2.png
    --- humans/
        - human1.png
        - human2.png
    --- chickens/
       - chicken1.png
       - chicken2.png
```

Run the program :

```shell
python dataset_builder.py
```

Put your hand in the red rectangle and press the `r` key to start recording your first gesture. Press `r` again when you're done.

Repeat the whole process for every category (class) you want to classify with the network.

## Step 2 : Train the network

As training a neural network is computationally intensive and can take a lot of time, I would **highly recommend** to use a **GPU** if you have one : [A VERY USEFUL LINK](https://github.com/williamFalcon/tensorflow-gpu-install-ubuntu-16.04)

Open **cnn_trainer.py** and set **nb_classes** according to the number of gestures you recorded earlier.

Then run the program :

```shell
python cnn_trainer.py
```

Once the network has trained, it should have generated two folders **model** and **out**.
* **model** contains the whole network (model + weights) saved into files, so you can reload it later without retraining it.
* **out** contains several files. Amongst them, a file starting with **tensorflow_lite_** which is the network model for Tensorflow Lite. Tensorflow Lite was developed by Google and allows you to run pretrained tensorflow models on mobile devices (Android and IOS).  

## Step 3 : Test the network

This script loads the neural network from the generated files.

To execute the script, simply run :

```shell
python cnn_tester.py
```

# Mobile integration

Some points to emphasize if the model is used on mobile phones :

* The network **cannot** process images of a **different** size that those it was trained on (currently **32x32** pixels but can be easily changed).
* The image passed to the network should be filtered based on **hsv** skin color, in **gray scale** mode and normalized (divide every value of the matrix by 255).
* The output of the prediction is a vector of size (1,n) where n is the number of classes the network was trained on. The network will output numbers between 0 and 1 (a number close to 0 means low probability, a number close to 1 means high probability). The mapping between the output vector and the classes is available in the file **out/labels.txt** after the training.

# Pre-trained model

I also included a pre-trained model that you can find here : [Google Drive](https://drive.google.com/drive/folders/1m-eXc9gCy95Fl5Koc3R2fyCULjBoXvlh?usp=sharing)
