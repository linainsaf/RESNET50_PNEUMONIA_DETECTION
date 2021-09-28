**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Introduction](#introduction)
  - [What problem does ResNet solve](#what-problem-does-resnet-solve)
  - [How does he solve it](#how-does-he-solve-it)
  - [Architecture of ResNet](#architecture-of-resnet)
 - [What is Pneumonia](#what-is-pneumonia)
 - [Dataset](#dataset)
   - [Data exploration](#data-exploration)
   - [Data resizing](#data-resizing)
   - [Data augmentation](#data-augmentation)
 - [Model building](#model-building)
   - [Convolutional Neural Network](#convolutional-neural-network)
   - [ResNet](#resnet)
      - [Transfer learning](#transfer-learning)
      - [Why transfer learning](#why-transfer-learning)
 - [Results and conclusion](#results-and-conclusion) 
   
# Introduction 
<br/>

According to the universal approximation theorem, with sufficient capacity, we know that a feedforward network with a single layer is sufficient to represent any function but the layer can be massive and the network tends to overfill the data. Therefore, there is a common trend in the research community that our network architecture needs to go further. However, increasing the depth of the network doesn't work by just stacking the layers together. Deep networks are difficult to form due to the notorious problem of the leakage gradient - as the gradient is back propagated to the earlier layers, repeated multiplication can make the gradient infinitely small.
<br/>

## What problem does ResNet solve
<br/>

One of the problems solved by ResNets is gradient leakage. Indeed, when the network is too deep, the gradients from which the loss function is calculated easily reduce to zero after several applications of the chain rule. This result on the weights never updates its values and therefore no learning is performed.
<br/>

## How does he solve it
<br/>

Instead of learning a transformation of  x -> y  with a function  H(x)  (Some stacked nonlinear layers). Define the residual function using  F(x) = H(x) - x, which can be cropped to  H(x) = F(x) + x, where F(x) and x represent stacked nonlinear layers and identity function (input = output) respectively.

The author's hypothesis is that it is easy to optimize the residual function F (x) rather than to optimize H (x).

The central idea of ResNet is to introduce a so-called "identity shortcut connection" which skips one or more layers, as shown in the following figure:

<br/>

## Architecture of ResNet
<br/>

Since ResNets can have varying sizes, depending on the size of each layer in the model and the number of layers it has, we will follow the authors' description in this article : https://arxiv.org/pdf/1512.03385.pdf to explain the structure after these networks.



Here we can see that the ResNet (the one below) consists of a convolution and grouping step (on orange) followed by 4 layers of similar behavior.

Each of the layers follows the same pattern. They perform a 3x3 convolution with a fixed feature map dimension (F) [64, 128, 256, 512] respectively, bypassing the input every 2 convolutions. In addition, the width (W) and height (H) dimensions remain constant throughout the layer.

The dotted line is there, precisely because there has been a change in the size of the input volume (of course a reduction due to convolution). Note that this reduction between layers is obtained by increasing the stride, from 1 to 2, at the first convolution of each layer; instead of through a pooling operation, which we're used to seeing as down samplers.

In this table, there is a summary of the output size at each layer and the dimension of the convolutional karnel at each point of the structure :

In what follows we will try to compare the performance of a ResNet vs a CNN on a Dataset.
<br/>


# What is Pneumonia 
<br/>

Pneumonia is an inflammatory condition of the lung mainly affecting the small air sacs called alveoli. Symptoms usually include a combination of productive or dry cough, chest pain, fever, and difficulty breathing. The severity of the condition varies.

Pneumonia is usually caused by infection with viruses or bacteria and less commonly by other microorganisms, certain drugs, or conditions such as autoimmune disease. Risk factors include cystic fibrosis, lung disease Chronic obstructive pulmonary disease (COPD), asthma, diabetes, heart failure, a history of smoking, poor ability to cough as a result of a stroke and a weakened immune system. Diagnosis is often based on symptoms and physical examination.

Chest x-ray, blood tests, and sputum culture can help confirm the diagnosis. The goal of this project is to train a ResNet model to help us detect Pneumonia from chest x-ray.
<br/>

# Dataset 
<br/>

The Dataset is organized in 3 folders (train, test, val) and contains sub-folders for each image category (Pneumonia / Normal). There are 5863 X-ray images (JPEG) and 2 categories (Pneumonia / Normal). Chest (anteroposterior) x-rays were selected from retrospective cohorts of pediatric patients aged one to five years at the Guangzhou Women's and Children's Medical Center in Guangzhou. All chest x-rays were taken as part of routine clinical patient care. For analysis of chest x-ray images, all chest x-rays were initially reviewed for quality control by removing any poor quality or unreadable scans. The diagnoses for the images were then scored by two expert doctors before being validated for training the AI system. In order to take account of possible scoring errors, the evaluation set was also checked by a third expert.
<br/>

## Data exploration
<br/>

We have two classes, Pneumonia and Normal. The data appear to be out of balance. To increase the normal training examples we will use data augmentation.
Some images from the Dataset : 

<br/>

## Data resizing
 <br/>

Now back to our X.train and X.test. It is important to know that the shape of these two arrays is (5216, 224, 224) and (624, 224, 224) respectively. Well, at a glance these two shapes look good as we can just display them using the plt.imshow () function. However, this shape is simply not acceptable to the convolutional layer as it expects a color channel to be included as an input.

So, since this image is mostly colored in rgb, then we have to add 3 new axes with 3 dimension which will be recognized by the convolution layer as the color channels.
<br/>

## Data augmentation 
<br/>

In order to avoid any overfitting problem, we need to artificially expand our dataset. We can make your existing dataset even bigger. The idea is to modify the training data with small transformations to reproduce the variations. Approaches that modify training data to change the representation of the table while maintaining the same wording are called data augmentation techniques. 
Some popular augmentations that people use are : grayscale, horizontal inversions, vertical inversions, random crops, color jitter, translations, rotations, and much more. By applying just a few of these transformations to our training data, we can easily double or triple the number of training examples and create a very robust model.

For the increase of data we have chosen:

- Random zoom of 0.2 on some training images
- Randomly rotate some training images by 30 degrees
- Shift images horizontally randomly 0.1 of the width
- Shift images vertically randomly 0.1 height
- Randomly flip the images horizontally.

<br/>

# Model building 
## Convolutional Neural Network
<br/>

Now is the time to build the architecture of the neural network. Let's start with the input layer (input1). So this layer basically takes all of the image samples in our X data. Therefore, we need to make sure that the first layer accepts exactly the same shape as the image size. It should be noted that what we need to define is only (width, height, channels), instead of (samples, width, height, channels).

Then, this input layer1 is connected to several pairs of convolutional aggregation layers before finally being flattened and connected to dense layers. Note that all of the hidden layers in the model use the ReLU activation function due to the fact that ReLU is faster to compute than sigmoid, and therefore the required training time is shorter.

Finally, the last layer to be connected is output1, which consists of 3 neurons with a sigmoid activation function.

Here the sigmoid is used because we want the outputs to be the probability value of each class.
<br/>
## ResNet

We are now using a pre-trained ResNet50 convolutional neural network model and we are using transfer learning to learn the weights of the last layer of the network only.

### Transfer learning
Transfer learning is a machine learning problem that focuses on retaining the knowledge gained by solving a problem and applying it to a different but related problem. For example, the knowledge gained from learning to recognize cars could be applied when trying to recognize trucks.

Indeed, one of the great motivations of transfer learning is that it takes a large amount of data to have robust models (especially in deep learning). So, if we can transfer some knowledge acquired during the creation of an X model, we can use less data for the creation of a Y model.

### Why transfer learning
Because with transfer learning, you start with an existing (trained) neural network used for image recognition - then modify it a bit (or more) here and there to train a model for your case. particular use. And why are we doing this? Training a reasonable neural network would mean needing around 300,000 image samples, and for very good performance we would need at least a million images.

In our case, we have around 4000 images in our training set - you have a guess as to whether that would have been enough if we had trained a neural network from scratch.

We are going to load a pre-trained ResNet50 network, which has been trained on approximately one million images from the ImageNet database.


# Results and conclusion 
