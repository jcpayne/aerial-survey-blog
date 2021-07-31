---
title: Creating a classification model
toc: true
author: John Payne
---

## Adding a classification model as a preliminary filter
In desperation, I decided to add a multi-label classification model as a preliminary filter. The idea was that if it was accurate, it could be used to eliminate most of the empty tiles, thereby greatly reducing the workload on the slow Detectron2 model.  I chose a ResNet50 model from `fastai`, because a ResNet50 should be fast and fastai's training environment is superb.  

To make a longish story short, I created a new VM, installed the fastai environment, ported the training images, annotations and model file to the new environment, converted the annotations to the multi-label classification format, and trained the model, as described in `06_fastai_model.ipynb`.  It soon reached an acceptable level of accuracy (about the same as the Detectron2 model) and so I ran an initial batch of test images through the model.  To my horror, inference took 0.37 seconds per image.  Several desperate hours later, thanks to the wonderful `fastai` community I discovered a hidden parameter whose default was  `cpu=True` in the function that re-loaded a saved model.  In other words, it wasn't using the GPU!  I changed that to `cpu=False` and *Hey Presto!* It ran 150 X faster.  

## Doing inference with the classifier model
It was time to put the classification model to a more serious test.  I ran a batch of 20,000 full-size images from a study site that the model had never seen before.  Because I hadn't yet figured out how to pass in-memory tiles to a `fastai`dataloader, I wrote just over 1 million tiles (20,000 X 54) to disk.  That tiling took nearly 12 hours, which is a problem, but I was very relieved to find that it took less than 2 hours to run 1 million tiles through the model on an Azure NC6_v3 machine (which has an Nvidia V100 GPU).  Finally, we had a model that was fast enough to be really useful!  But was it accurate enough?  

## Assessing a multi-label classification model.  Surprising edge cases.
The standard 'confusion table' method for assessing the output of a single-label classification model does not work in the multi-label case.  In a multi-label model, you never know whether an object was _missed_ (not detected at all), or _misclassified_ (detected, but classified as the wrong category).  I haven't found a good way to assess a multi-label model, but I wrote some code to try to separate missed from misclassified objects, as shown in the .  In the process, I discovered two surprising edge cases:
1) Some tiles were not classified as anything.  No category _including the 'empty' category_ had a score that was higher than the detection threshold.  I named these predictions 'uncertain'.
2) A few tiles were classified as both empty _and_ as containing an object.  That wasn't really a problem because we could just ignore the 'empty' classification in these cases.

### Is fastai's implementation of 'empty' a feature or a bug?
Clearly, the fastai model's empty' category is not just a remainder that is created when the model fails to find any object in an image; it's actually a _learned_ category itself.  That architectural choice generates the strange edge cases that make counting results much harder.  But is it a bug or a feature?  

We examined the 'uncertain' tiles and discovered that in many cases, there were interesting objects in them; sometimes objects that were not in our list, and other times false-positives.  Essentially, the 'uncertain' category is quite useful because it highlights images where 'something is not quite right,' and we therefore decided to treat it as a feature, not a bug.  As our model improves (particularly for rare categories), we expect that the number of 'uncertain' predictions will shrink, but they will still highlight tiles that are worth examining more closely.
