---
title: Training the model and sharing results
toc: true
author: John Payne
---

## Training the model
Once the training and validation sets were created, the model customization was working, and I had learned how to use AML, I trained the model fairly hard on an AML cluster with 4 GPUs.  The process did not go smoothly.  
### Fixing learning rate problems
The model didn't improve much at first, and I realized that the learning rate was probably wrong.  However, I didn't have a good way to visualize results because the Detectron2 logger is self-contained and didn't communicate with the AML cluster.  To use AML's visualizations properly, it would be necessary to intercept the Detectron2 messages on their way to its logger and share them with AML.  I have since figured out how to do that, but at the time, I just wrote a bit of code to visualize both the output and the learning rate schedule once the run was finished. I then experimented quite actively with learning rate schedules.  I found that I had misinterpreted some of Detectron2's learning rate schedule parameters and the initial learning rate was too high.  I eventually figured out where to aim (although it was a moving target, because the best rate dropped as the model improved), and training finally started going well.  
### Dropping the 'boma' class and re-training
In the US, a corral for livestock is usually made with fence posts and wire, but in subSaharan Africa, corrals are often made by piling up brush in a rough circle or rectangle.  In Tanzania, those brush corrals are called 'bomas'.  Howard wanted to be able to identify bomas, and they were included in our object list.  As the training proceeded, the model accuracy got up to 80-90% for most of the important classes, but there were still an unacceptable number of false-positives.  Looking at the output, we quickly realized that most of the problem was in the 'boma' class.  The model obviously recognized straight edges, but sometimes got the scale very wrong because at smaller scales, everything looks like a boma.  We didn't have a very big sample size of labeled bomas, and we decided that it was better to leave the class for another time.  So we dropped it and retrained, starting by training the ROI heads with the rest of the model layers frozen.  That fixed the false-positive problem and the accuracy got quite good.  It's currently above 95% for the common categories, with only 5 - 7% false-positives.  The `05_aml_pipeline.ipynb` notebook describes the training.

![]({{ site.imageurl }}/blog_images/RR19_RKE_20181001A_RSOL_1610_ann_12inch.jpg)

A dusty herd of elephants in forest

### TridentNet's three heads
TridentNet has an unusual feature from which its name is derived: the model has three separate heads that handle small, medium, and large objects, respectively.  The TridentNet creators wrote that once the model was trained, it worked perfectly well to do inference using _only_ the medium head, to save computation.  We compared inference using all of the heads with inference using only the medium head and so far, I agree with their conclusion.

## Uh oh, how do you share results?
Another unanticipated challenge cropped up at this point: how do you share results with colleagues?  It hadn't occurred to me that it would be a problem.  There were two major problems: first, I was turning each full-sized image into 54 tiles and running the tiles through the model.  But would a colleague really want to see 54 different annotations per original image?  I thought not, so I decided to re-combine the annotations (done separately for Detectron2 output and the fastai classification model output).

Secondly, the output from a Detectron2 model is a Python dict containing Detectron2 classes, so you need Detectron2 installed to view the annotations.  Installing Detectron2 is non-trivial, and even if I could translate the annotations to a simpler format, most people don't have access to software that will superimpose bounding boxes onto an image.  So as a stopgap measure, I wrote some code to do the superposition myself, and then shared image pairs consisting of a full-sized, annotated image plus its corresponding original image with my colleagues.  It's not a sustainable solution because the files are so large, so I am really just highlighting the need to think about how you are going to share results with your colleagues.  

![]({{ site.imageurl }}/blog_images/JP-annotation-8_12inch.jpg)


