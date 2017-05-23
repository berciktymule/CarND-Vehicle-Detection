## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

##  [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###  Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###  Writeup / README

####  1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###  Histogram of Oriented Gradients (HOG)

####  1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code 5 cell of the IPython notebook.  
I used `skimage.feature.hog()` function to extract the features from individual images.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

#### 2. Explain how you settled on your final choice of HOG parameters.

I started off with:
- `RGB` color space with all channels
- 8 orientations
- 8 pixels per cell
- 3 cells per block

Then I started training my classifier.
Having experimented with different color spaces, and other parameters I have compared the test accuracy and noted what gives the highest probability.

I finally settled for:
- `HSV` color space with all channels
- 12 orientations
- 8 pixels per cell
- 2 cells per block

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using HOG features, spatial fetures (18, 18) and histogram features (52 bins).
The code is in cell 7 of the notebook. The parameters are defined in cell 6.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search the lower right quarter of the image for performance reasons as that is where the target cars are on the video.

Cell 8 of the notebook contains the HOG subsampling based window searched as described in the course. Computing a single HOG for the entire searched area proven to be multiple times faster than computing HOG for individual windows.
The gained performance enabled me to use a bigger overlap (step was a single cell) resulting in more prominent heatmaps.
I wanted to keep the scale as big as possible because the bigger the window the fewer total windows we need to check. However I have noticed that the lack of small windows resulted in no detection when the car was far away.
I settled for 1, 1.5 and 2 as they covered the entire range with best accuracy and fewest false positives.
In case we needed more performance I think we could get away with single 1.5 scale and experiment more with compounding multiple frames.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on three scales using HSV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image3]
![alt text][image4]

Like I've mentioned before in order to gain some performance I have used HOG subsampling.
I also limited the search area (but this is only applicable to this video).
I also used small spatial binning (18, 18).
I also gathered few samples around the spots that initially had false positives and added them to the non-car training set.

---

###  Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.
I compounded the detections across 3 frames. From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The lessons in the course gave me some solid starting point. Using the code from the lessons it only took me a couple of hours to create the pipeline. It took me a lot longer to find the problems with it since it wasn't working.
Most of my problems were around color conversion.
I missed the fact that changing color space using `cv2.cvtColor` changes the scale of the image back to 0-255.
Once I made sure to rescale the converted image to 0-1 scale the pipeline started working on a single image.
But it would not work on the video. It turned out that I had to rescale the frames back to 0-255 range.

This pipeline is tuned to be used in for the project video. Since it's limited to lower right part of the video it wouldn't pick up cars on the rest of the image.

The parameters could be tuned to further improve the performance. I'm sure we could lower the orientations and histogram bins as well as the step in HOG subsampling.

Another thing to do could be not to include the 1 or 2 border pixels of the hog subsamples to get rid of the difference introduced by the pixels surrounding the window which are not empty unlike the training images.

Also it would be good to be aware of the detected cars, remember their positions and anticipate their position to limit the search windows to their proximity and edges of the road.
