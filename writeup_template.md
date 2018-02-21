# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/my-results/color-detection.jpg "Color-detection"
[image2]: ./examples/my-results/canny.jpg "Canny"
[image3]: ./examples/my-results/canny-masked.jpg "Canny-masked"
[image4]: ./examples/my-results/raw-Hough-lines.jpg "Raw-Hough-lines"
[image5]: ./examples/my-results/extrapolated-lines.jpg "Extrapolated-lines"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

First: I created a lane lines detection pipeline using Color Detection, by defining RGB thresholds for both white & yellow colors, 
and then drawing the detected lines (in red) on a 3-channel empty image and combining it with the input image.

![alt text][image1]

The output of this method is pretty accurate, but it's not clear how it could be beneficial, since we can not create mathematical curves from it's output directly, unless we input it to the Canny detection method, which in any case makes no difference since Canny detection is accurate enough on it's own, hence I just left it there and didn't use it's output for the next step.

Second: I created a Canny detection pipeline, consisting of the below steps:
* Convert the image to grayscale, since Canny detects the brightness gradient and color makes no difference.
* Define Canny thresholds by trial and error.
* Apply Canny filter using the given helper function to detect the lane lines edges.

![alt text][image2]

* Mask irrelevant parts of the output by using the given region_of_interest helper function.

![alt text][image3]

* Define parameters for Houghlines function by trial and error.
* Apply hough_lines function using the given helper function to create mathematical lines from the detected edge pixels.
* I modified the hough_lines function to return raw Hough lines, instead of calling the draw_line function and returning the output image, and created another draw_line_extrapolated function, this way after extracting the Hough lines, I can draw either the raw Hough lines, or the extrapolated lines, for comparing results, and fine tuning.
* The draw_lines_extrapolated function works as follows: it reads all the raw Hough lines, computes the slope of each line, dismisses any line that has a near horizontal slope (since this must be noise), and then separates the lines into 2 groups of positive slope lines and negative slope lines, then it computes the average slope of each group, and the average intercept with y-axis (to define the line position), and then draws 2 extrapolated lines based on these average values, extending from the bottom of the image, till the upper limit of the region of interest (which is an input to the function).
* Draw the extrapolated lines (or the raw lines) on an empty 3-channel image (in red), combine it with the input image and return the result.

![alt text][image4]
![alt text][image5]

Finally: test the lane_lines_detection function by reading in all the images in the test_images directory, using a helper function I created for reading multiple images, and then save all the output images to the test_images_output directory.

For testing the videos just call the defined lane_lines_detection function inside the process_image function and return it's output.

Note: After testing the challenge video, I went back and added a step to standarize the resolution of all the input images, which wasn't required for the first 2 videos since they both already have the same resolution as the test_images.


### 2. Identify potential shortcomings with your current pipeline

* The output allows only for extrapolated straight lines, in case of curved road sections, the output won't be accurate.
* This pipeline won't detect lane lines correctly under different lighting conditions; for example under heavy shadows, or under intense sunlight, which can be seen in the challenge video output in which the output is correct most of the time, except for 2 major errors; under the tree shadow at (00:03:500) and when the road barrier is passing intense light at (00:04:100).
* The extrapolated lines vibrate quite a bit, and might not be ideal for further processing.

### 3. Suggest possible improvements to your pipeline

* Use higher degree curves instead of straight line outputs.
* We can define adaptive Canny thresholds: when you can't recognize the lane lines correctly (by correctly I mean you can't see the lines at all or you can see lines but there is a sudden change in slope, so it's probably not correct), lower (loosen) the Canny thresholds, since the edges are probably washed out in regions under heavy shadows or heavy light and don't have very strong gradients, and then can go back to the normal (optimal for most conditions) Canny thresholds after a couple of seconds, to avoid detecting too much noise unnecessarily.
* We don't need to update the lines positions for each video frame (20+ times per second), 5-10 updates per second will probably be enough and will reduce lines vibration making the output more stable.