## Writeup

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

- [x] Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
- [x] Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
- [x] Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
- [x] Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
- [x] Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
- [x] Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./writeup_images/car_not-car_examples.png
[image2]: ./writeup_images/HOG_channel_experiment.png
[image3]: ./writeup_images/color_spaces.png
[image4]: ./writeup_images/sliding_window.png
[image5]: ./writeup_images/detection.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

In the second cell in my IPython notebook, P5.ipynb, I call `extract_features()` at lines 46 & 55 to extract the HOG, histogram of color, and spatial binning features used in training my Support Vector Machine.  The first call extracts features from the vehicle images while the second extracts them from the non-vehicle images.  The feature extration routines are defined in the first cell of the notebook.  In `extract_features()` starting at line 60, I cycle through each image.  The first step is to read in the image so here are examples of a vehicle and a non-vehicle:

||
|:---:|
|![alt_text][image1]|

After reading in the image, I convert it to the applicable color space in lines 65-75.  Next I apply spacial binning in lines 78-80.  Experimentation with an early version of the code showed beneficial results at an image size of 18x18.  Next, in lines 81-84, I apply histograms of color features.  These lines call `color_hist()` in lines 41-49 which creates a histogram with 32 bins for each color channel.  Note that my tuning parameters are defined in the second cell at lines 33-43.

Lastly, I extract the Histogram of Oriented Gradient (HOG) features.  I experiemented using each of the three channels and then all of them.  Individual channel results were good but using all of them provided the best result as indicated here:
![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG and color parameters.

I ran my classifier using each color option and found two to be the best choices and comparable to each other.  The are YUV and YCrCb.  As I proceeded developing my code and further training, I experiement on both of those color choices to see which would give me an edge.  In the end, I chose YUV.

Here are examples of each color space choices run through my classifier:
![alt_text][image3]

My HOG parameters didn't require much tuning.  I experimented with each HOG channel as indicated above and settled on using them all.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM in the second cell of my notebook.  After reading in the image filename and separating them into vehicle and non-vehicle arrays in lines 19-26, I set my parameters in lines 34-43 and then extract vehicle features in line 46 and non-vehicle features in line 55.  In line 66, I use `StandardScalar().fit()` to compute the mean and standard deviation.  In line 68, I use `transform()` to standardize the data by centering and scaling.  If features were not standardized, those that were much larger might skew the results keepin the classifier from learning the much smaller features.

In line 71, I set up the labels vector.  Line 74 shuffles to the data and line 75 splits it into 80% training and 20% test.  Lines 82 & 85 train the SVM and here's an example of one run:

0.16 Seconds to read in cars/notcars image filenames
cars size:  8792
non-cars size:  8968
163.01 Seconds extract car features
147.86 Seconds to extract notcar features
Using: 11 orientations 12 pixels per cell and 4 cells per block
Feature vector length: 3180
12.08 Seconds to train SVC
Test Accuracy of SVC =  0.9887

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

My first implementation of sliding windows used 128x128-pixel windows with 50% overlap in the x and y directions.
![alt text][image4]

Since cars usually drive on the road and to not fly, I next limited the search to just the bottom portion of the image where the road is in order to reduce search time.  My final implementation uses HOG subsampling to extract hog features once and then sub-sample to get each overlaying window.  This happens in my notebook's third cell titled "Extract features & predict".  The window is set to a sampling rate of 64 in line 38.  cells_per_step is set to 2 in line 40 so that the window steps by 2 rather than by a percent as in the first implementation.  

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image5]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result that does not maintain a frame history](./output_images/project_video_result_YUV_withHeat.mp4)

Here's a [link to my video result that does maintain a frame history](./output_images/project_video_result_YUV_withHeatSum.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

In the fourth cell titled Heatmap functions, I took in the list of positive detections at line 60.  I created a heatmap from that list and applied a threshold in line 74 to help remove false positivies.  In lines 77-78, I summed up the heatmaps over the current and previous 9 frames storing them in a dequeue created in line 57.  In line 85, I used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap and drew the related boxes around each vehicle in line 86.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Prior to storing vehicle detection history, my pipeline detected vehicles reasonably well but the bounding boxes were jumpy.  It found minimal false positives.  After storing detection history and usig that to track where vehicles were, the boxes stabilized a bit but more false positives started being detected.  I need to further explore how to keep the improved vehicle detections while at the same time keeping the false positivies low.

There is also a brief segment where I lose the white vehicle as it gets farther away from my vehicle.  Although I have 8792 vehicle images, I may need more training images for that vehicle. 
