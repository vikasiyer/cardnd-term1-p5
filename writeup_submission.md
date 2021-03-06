##  Writeup Submission

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
[image1]: ./writeup-images/car-notCar.png
[image2]: ./writeup-images/hogCar.png
[image3]: ./writeup-images/slidingWindow.png
[image4]: ./writeup-images/outputtest1.jpg
[image5]: ./writeup-images/outputtest6.jpg
[image6]: ./writeup-images/heatMap.png
[image7]: ./writeup-images/testImageWithDetails.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 5th and 7th code cell of the IPython notebook. The 'get_hog_features()' method extracts the hog features'  

I started the project by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of the `vehicle` class.

![alt text][image1]

I then explored different color spaces(RGB, LUV, YUV,HLS and YCrCb) and different `skimage.hog()` parameters (`orientations` (9 and 12)).  I grabbed a random image and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried a couple of combinations of parameters. I set orientations to 9 and 12. I stuck with pixels_per_cell=(8, 8)(since the image size is 64x64) and cells_per_block=(2, 2)


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I split the datasets 80(training)-20(test), randomly shuffled them and trained using linear SVM. You can see that in the iPython notebook cell 8. In addition to HOG features, I also used spatial color binning and histogram feature extraction.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I implemented Sliding Window search in code cell: . I decided to go with 0.5 overlap since that's halfway and gives good coverage. I experimented a scale of 1, 1.5 and 2. Seems like 1.5 performed the best for me. 2 was too big and 1 was too small.
**EDIT (submission 2):** Based on the project reviewer's feedback, I increased the sliding window search to search for different scale values. I used the values : 1, 1.5,2,2.5 and 3 as seen in code cell 17. (scaleArray = [1,1.5,2,2.5,3]). I then loop through for each scale value, and call the "find_cars() method."
![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using HLS 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
![alt text][image5]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.
I tried different color spaces - starting from RGB, and then I tried LUV, HLS and YCrCb.

I noticed that there would be more false positives at the stretch of the freeway where the road's shade turns lighter. I created some 64x64 frames from this patch (about 15 of them from different places in the patch) to augment my non-vehicle data set. I also tried adjusting the threshold from 1 to 4. Clearly 2 seemed like the best number.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

**EDIT (Following project reviewer's suggestions after submission 1)**:  I also tried YUV and YCrCb. I noticed that the results improved with YUV, so I picked this for submission 2.
I also implemented a history queue that keeps track of the heatmaps across consecutive frames to help differentiate between real boxes and false positive boxes. This seemed to have improved the stability of the bounded boxes.
I also played with the "C" value of LinearSVC. Tried values of 1, 10, 100 and 1000. Stuck with 1000. Not too sure if that made much difference.  The code for this is in cell 8.
I also used the output of "decision_function() method to set threshold for window display. This helped removing some false positives. The code for this is in code cell 17 in the find_cars() method.

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here is a frame and its corresponding heatmap:

![alt text][image6]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

I changed  the image reading logic from mpimg.imread() to cv2.imread() to avoid dealing with normalizing conversions.

I first started out using the RBG color space to detect vehicles. It didn't work so well for me. So I experimented other color spaces like LUV, HLS and YCrBr.HLS and YCrBr performed the best, each with tradeoffs.
HLS produced more false positives but more car detection.
YCrBr produced lesser false positives but lesser occasions of car detection. I stuck with HLS.

I also adjusted the heatmap threshold values several times; set them to: 1, 2, 3 and 4. 1 and 2 seemed the best. 3 and 4 would stop detecting cars often. 1 seemed to produce more false positives. While I could get to avoid the duplicate boxes, the boxes are still quite wobbly.

Like I mentioned above, I saw that the false positives increased at the lighter patch of the freeway. So I augmented by non-vehicle data set with more about 15-20 frames of the same patch. Each frame of size 64x64. I also included a couple of siderail frames from the video as I noticed some false positives over the railing. This seemed to help more in YCrBr color space, but just a little in HLS color space.

In the future, I could try augmenting the dataset with Udacity's extra dataset. This should make it more robust.

**EDIT (Submission 2)**
Following project reviewer's suggestions, I did the following to reduce the wobbling and false positives:
- Added a history queue to keep track of heatmaps across consecutive frames to filter out false positives and keep the windows consistent.
- Adjusted C parameters of LinearSVC
- Used the decision_function() method to set thresholds on window display.

These steps reduced wobbling of the windows and reduced false positives. However, I still see false positives emerge in some areas. Also, the car detection fails for a few seconds in the frame of the video where the road has a white patch. Don't know what is causing this. Tweaking C values : 1, 10, 100 and 1000 didn't seem to improve detection on this patch. So not sure what is causing this.

Also in the area where the tree shadows show up on the road, I saw some false positives emerge. To get rid of these, I augmented my 'non-vehicle' dataset with more images of the shadow. That didn't seem to help much. I tried tweaking the decision_function() threshold values to improve this. It seemed to have reduced the false positives marginally at this section, but not significantly. In future, I'd like to investigate this some more.

Setting the decision_function() threshold > 0.75 would make the bounding rectangles go smaller than ideal. Setting it < 0.25 introduces more false positives. So not sure if decision_function() can bring any more improvements to reduce false positives. I intend to investigate this in future.
