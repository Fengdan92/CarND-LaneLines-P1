# **Finding Lane Lines on the Road**
Fengdan Ye, March 26, 2018

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


---

### Reflection

### 1. Pipeline

My pipeline consisted of 6 steps:
1. Grayscale the image.
2. Smooth the image with Gaussian blurring.
3. Canny edge detection
4. Mask the image with region of interest
5. Perform Hough transform on the region of interest to identify line segments and draw them on a blank image. Optionally, draw extrapolated lines on a blank image.
6. Overlay the identified lines with the original image.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function. See draw_lines_extrapolate() in P1_fengdan.ipynb. This function contains the following steps:
1. Loop through each line segment detected.
2. For this line segment, calculate its slope using ((y2-y1)/(x2-x1)).
3. Decide if the segment is noise, i.e. horizontal segments. This is done by setting a slope threshold.
4. If the segment is not noise, decide if it belongs to the right line or the left line based on the sign of slope.
5. If it belongs to the right line, make sure it is not on the left side of the image (if x1 > 440 and x2 > 440). Then, append x values to x_right, and y values to y_right. It is similar if the segment belongs to the left line.
6. After looping through all segments, find linear fit on (x_left, y_left) and (x_right, y_right), respectively. This gives us the slope (m) and intercept (b) of the linear fit.
7. Decide the maximum and minimum y values for drawing the extrapolated lines. Here I choose y_max = 539 and y_min = 330.
8. Calcualte the corresponding x values. That is, x=(y-b)/m.
9. Turn the x values into integers as requested by cv2.line.
10. Plot the extrapolated lines using cv2.line.

Here is how the pipeline works in details:

Grayscale:
![](./test_images_output/grayscale.jpg)
Gaussian blur:
![](./test_images_output/gaussian-blur.jpg)
Canny edge detection:
![](./test_images_output/canny.jpg)
Hough transform line detection in region of interest and extrapolation:
![](./test_images_output/hough_extrapolated.jpg)
Final result (overlay):
![](./test_images_output/solidWhiteRight_extrapolate.jpg)

### 2. Identify potential shortcomings with your current pipeline

The pipeline works fine on images. On the videos, it correctly recognizes the lines, but the frame-to-frame transition is not smooth (lines being "jumpy").

Another shortcoming would be that, when the color difference between roads and lines are not obvious, the pipeline might fail to detect some of the edges.

It is also an issue if there are other confusing factors such as shadow on the ground, or cars immediately in front. In this case, Canny edge detection will pick up the edges from the shadows/cars too, and a region of interest mask seems not enough to filter the noise.

Some of the above issues are already seen in the challenge video (see P1_fengdan.ipynb).


### 3. Suggest possible improvements to your pipeline

One possible solution to "jumpy" lines would be to average over frames.

As for the case when the color of the lines and the roads are similar, a color conversion can be done. For example, in the challenge video, I have converted RGB colors to HSV, and only kept the V channel. This is because under a different color profile the difference between lines and roads can be more obvious.

Solution to shadow suppresion can be tricky. People mentioned RGB normalization method, but that did not work on the challenge video in my case. I could filter noise based on the location of the detected segments, but that does not seem to be the optimal solution, especially when there are many line segments detected due to noise. Another possibility is to fine tune the parameters of Canny edge detection and Hough transform to reduce shadows and other confusing information.
