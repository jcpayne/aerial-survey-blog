---
title: Current status
toc: true
author: John Payne
---

## Current status: ready for large batches
After months and months of work, the project is finally at the point where we can run large batches of images through the pipeline.  
- **We have two working models**: a heavy bounding-box model (TridentNet), and a much lighter and faster multi-label classification model (ResNet50 from fastai).  
- **Both can be used _as is_**; they both have high enough accuracy (>90% for the most important categories) and low enough false-positive rates (<5%) to be useful.  
- **The classification model reduces the workload for the bounding box model by 95%**.  The multi-label classification model reduces the number of tiles that the big Detectron2 bounding-box model needs to check by almost 95% (>98% reduction if you’re willing to ignore ‘uncertain’ tiles), and it is fast enough to be used in production.  Both of those points are very good news, and it means we could even afford to throw in a random sample of ‘empty’ tiles for the big bounding-box model to double-check.  
- **It may not reduce the workload for humans as much.** Only 4.8% of the total tiles are classified as ‘uncertain’, meaning that the model doesn’t classify them as anything, not even ‘empty’.  Unfortunately, despite their modest numbers, those ‘uncertain’ tiles are scattered very evenly across files, with the result that about 1/3 of the images are empty, 1/3 contain objects, and 1/3 are a mix of ‘empty’ and ‘uncertain’ tiles.  That means that effectively, the multilabel classification model only gives us a 1/3 reduction in the number of full-size images that we need to check.  Depending on how we design the pipeline and what we decide to double-check, that may mean that the workload for humans is not reduced as much as the workload for the Detectron2 model.
- **Our primary need at the moment is to find more training images**.  We will need to use the models to help with that process by churning through big piles of new images. 
- **We need to develop an environment for human-in-the-loop work**.  We have been exploring [AIDE](https://github.com/microsoft/aerial_wildlife_detection) and Howard has also begun to work with the team at [WildMe](https://wildme.org/#/).  Keeping track of edits to annotations is a very complicated problem.
- **We need to improve the existing code**.  It needs to be made more modular, more stable, and to have some of the functions packaged properly, to make them easier to use by other people. 
-**We need a more stable and solid pipeline**.  I have used the word 'pipeline' rather usely throughout this blog, but in reality we would like to move towards an automated Azure pipeline that makes it easier to pass data from one step to the next.

In summary, the future is looking quite good.  I look forward to seeing whether the big model can eliminate some of the ‘uncertain’ tiles and thereby reduce the number of fullsized images (although it could add new false-positives, too).  There are still quite a few categories where the number of training images was very low and the models are still very confused about them.  I expect that as the models improve and that confusion is reduced, the models will get less unsure about whether a file is empty, which will help to shrink the number of ‘uncertain’ tiles.
