# CarND Vehicle Detection and Tracking Project

## Writeup 

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and calculate color histograms.
* Train a XGBoost classifier with the above information.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./images/image01.png
[image2]: ./images/image02.png
[image3]: ./images/image03.png
[image4]: ./images/image04.png
[image5]: ./images/image05.png
[image6]: ./images/image06.png
[image7]: ./images/image07.png
[image8]: ./images/image08.png
[image9]: ./images/image09.png



## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG) and color histogram

#### 1. Explain how (and identify where in your code) you extracted HOG features and color histograms from the training images.

The code for this step is contained at cell codes #11 through #23 in `./P5.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different skimage.hog() parameters (orientations, pixels_per_cell, and cells_per_block). I grabbed random images from each of the two classes and displayed them to get a feel for what the skimage.hog() output looks like.

Here is an example using grayscale and HOG parameters of orientations=9, pixels_per_cell=(8, 8) and cells_per_block=(2, 2) for `vehicle` and `non-vehicle` images:

![alt text][image2]

![alt text][image3]

I transformed the images to a grayscale using  the opencv function `cv2.cvtColor`, then I calculated the HOG features with the following parameters: 

* orientations = 9
* pixels_per_cell = 8
* cells_per_block = 2

I applied a global image normalization equalisation that is designed to reduce the influence of illumination effects. In practice we used the square root. Furthermore we calculated the color histograms. 


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters for HOG features and color channels but I chose the following set up: 

* Three channels from the YCrCb color channel
* orientations = 9
* pixels_per_cell = 8
* cells_per_block = 2
* Square root for normalization


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features and color histogram.

Firstly, I used principal component analysis (cell codes #24 through #26 in `./P5.ipynb`) in order to reduce the dimension of the input space (HOG features + color histogram) , this step was made for theoretical considerations (a big input space is prone to overfitting). 750 components were used which explained 83% of the variability of the data.

Next, XGBoost was used for classifying vehicles and non-vehicles (cell codes #27 through #37 in `./P5.ipynb`). Grid search was used to determine a good model (there is no guarantee of optimality). 

The best model has the following hyperparameters:
* n_estimators = 50
* colsample_bytree = 0.7
* max_depth = 10

Using cross-validation (fold=4) the average of the metric (ROC AUC) on this model is 0.9992 which is acceptable. In the test set the precision, recall, and f1-score was greater or equal than 0.97.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I created the function `find_cars` (cell codes #38 `./P5.ipynb`)  to seach windows and identify cars. The function returns the image with bounding boxes and the coordinates of the bounding boxes. I decided to search windows from the position 400 to 656 in the y-axis. The size of the image in the windows search is 64 pixels with a cells per step equal to 2. 


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

At the beginning the classifier threw few false positives (only HOG features calculated over grayscale were used) then I improve the model adding information of more the YCrCb color channels and color histograms.  Here are some example images:

![alt text][image4]
![alt text][image5]
![alt text][image6]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link](https://raw.githubusercontent.com/ricardoues/CarND-Vehicle-Detection/master/output_videos/outputVideo.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in the images contained in the test_images folder (cell codes #40 through #41 in `./P5.ipynb`). From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here are three frames and their corresponding heatmaps and their resulting bounding boxes:

![alt text][image7]
![alt text][image8]
![alt text][image9]

The function process_image (the pipeline) is defined in the cell code #42 in `./P5.ipynb`. The video is generated at the cell code #43 in `./P5.ipynb`.  

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are several conditions that can could affect the performance of the pipeline such as: brightness, weather, orientation of the cars, and so on. On the other hand the dataset that was used in our application is not big enough. In order to improve the results, it is necessary to review the state of art of object recognition techniques, it can be helpful to use the following python library for data augmentation:

[imgaug](https://github.com/aleju/imgaug)
