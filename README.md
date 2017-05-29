** Vehicle Detection Project **

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image0]: ./examples/00HOG.png
[image1]: ./examples/01variousBoxes.png
[image2]: ./examples/02searchBoxes.png
[image3]: ./examples/03initialPic.png
[image4]: ./examples/04resize.png
[image5]: ./examples/05HOG.png
[image6]: ./examples/06heatmap.png
[image7]: ./examples/07label.png
[image8]: ./examples/08final.png
[image9]: ./examples/09HSV.png
[video1]: ./project_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points



### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

Histogram of Oriented Gradients was peformed on initial pictures from the training set, which are all 64x64 pixels (see cell # 236 in python notebook: 11_Video_Final.ipynb).

Here is a result of a vehicle image from the test set:

![alt text][image0] 

Besides evaluating different HOG parameters, I also looked into different color spaces, such as HSL (see below).

![alt text][image9] 


#### 2. Explain how you settled on your final choice of HOG parameters.

The pictures were first covnerted to grayscale, then I tried various parameters of HOG to see if I could improve the accuracy of my Support Vector Machine trained on HOG. While having more orientations seemed to net better results, there is a price to pay in the sense that more orientations made training my SVM longer.
I settled on the following:
* Pixels per cell = 8
* Cells per block = 2
* Number of orientations = 9

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features.

I trained a linear SVM using scikitlearn's GridSearchCV, which enabled me to compare different parameters and determine which ones were the best (see cells # 15 - 21).

I used an rbf kernel, and the following parameters:
* C values: [0.1, 1, 10], 
* gamma values: [0.001, 0.01, 0.1]

GridSearchCV found the best parameters were 10 for C and 0.1 for gamma.

I then used these parameters to train the SVM on 80% of the training set, keeping 20% as a testing set, and got an accuracy of 98.99% (see cells # 24 - 25)

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I spent a lot of time determining what window sizes I should use to scan the bottom half of an image (i.e. where cars are). Since training pictures were 64x64 pixels, I decided to use this size as the smallest and then go with multiples of it (i.e. 96, 128, and 192). 

![alt text][image1]

Here is an example of what my scanning windows ultimately looked like (note: I actually used twice as many smaller windows, but I did not draw them on this image for readibility purposes)

![alt text][image2]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?


For the width, I decided to not use a square ratio but rather use a width that was about 130% of height, which I would then 'squeeze' when resizing the picture to 64x64 pixels. I did this because I realized cars are more wide than tall, and also because I could see many images on the training set had thus been squeezed on the x-axis, so the model had already been trained with such pictures.

Here are the steps I took to draw boxes around cars on an image:
1. crop the image to only keep the bottom half and convert it to grayscale (see cell # 238)

![alt text][image3]

2.  Apply sliding windows and resize the resulting images (see cells # 292, and 300 - 301)

![alt text][image4]

3. Get the Histogram of Oriented Gradient from the resized image and let the SVM predict if it contains a car (see cell # 302)

![alt text][image5]

4. Use all the overlapping bounding boxes (i.e. windows where the SVM model detected a car) and create a heat map (see cells # 304 & 305)

![alt text][image6]

4. Use a threshold to get rid of areas that were not selected in enough bounding boxes (to eliminate false positives). Use Python label() to determine areas where a car is positioned (see cell # 306).

![alt text][image7]

6. Use the positions found to draw rectangles wich are superimposed on the original picture (see cell # 307)

![alt text][image8]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_output.mp4). The pipeline for processing video is in cell # 310.


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The biggest difference between the pipeline for the video and one for an individual picture, is that I kept the valid windows (i.e. the ones displayed on screen) from a few previous frames. This enabled me to smooth out the bounding box surrounding a car. I then adjusted my threshold so that I could give a higher value to the bounding boxes from the current frame, versus those from previous frames.


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
The video took a fairly long time to process in it entirety, which made experimentation of parameter more difficult. I even tried to run my pipeline on the AWS EC2 instance I used in previous projects, but didn't see much of an improvement, which leads to think that the software method used is not optimized for parallel processing. This is a big problem in a real-word situation, where cars need to be recognized in real time.

I also do not know how the SVM model would react under different wheather or lighting conditions (i.e. rain, snow, nightime), although I believe this could be solved by either using additional training data, or training different SVM models for different conditions (i.e. one model trained for night time).

I would also like to explore further various color models (i.e. HSV, YCbCr) to see if they can help to further improve model accuracy and reduce false positives.

