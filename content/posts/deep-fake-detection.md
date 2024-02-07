+++
title = 'Detect Deep Fakes in 30 minutes with Computer Vision'
date = 2018-12-01
+++

![Deep Fake Detection](/dfdc.jpeg)

In the realm of digital content, the rise of deep fakes has presented a unique set of challenges.
I joined the [Kaggle Deep Fake Detection Challenge (DFDC)](https://www.kaggle.com/c/deepfake-detection-challenge) and challenged myself to submit a
 working solution in two days maximum. I wrote my approach below. Code is [here](https://github.com/cyrillay/kaggle-dfdc).

## The Challenge at a Glance
The dataset size is a challenge by itself, being a 500GB of video content dataset.

## Approach and Methodology

1. **Frame Extraction**: Capture a few frames from each video to immediately reduce the dataset size by 90%, and obtain something small to iterate with.
2. **Face Recognition**: Use image segmentation techniques to isolate and extract faces from the frames. This step is crucial as GAN-generated videos often exhibit glitches or imperfections in facial features.
3. **Model Training**: I trained a Convolutional Neural Network (CNN) to predict whether the extracted faces were fakes or real. I chose ResNext50 at the time, as it was known for its efficacy in handling image-based datasets.
4. **Inference Pipeline**: The pipeline involved converting videos to frames, detecting faces within those frames, and making predictions on each of these face. The final step averages these predictions to produce an overall output for each video.

## Results and Findings
I was happy with the results for a 2 day project, achieving an accuracy rate of 75%.

## Conclusion
This was a quick POC hacked around in a day or two. To further improve it, I recommend reading through the [competition's discussions](https://www.kaggle.com/c/deepfake-detection-challenge/discussion) and gather the best techniques.