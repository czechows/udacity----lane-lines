**Finding Lane Lines on the Road -- writeup**
**Aleksander Czechowski**
**Udacity SD Car Nanodegree, Term 1**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on performed work in a written report


---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline of video processing for lane line detection was applied on a per-frame basis, so each frame was treated as an image to process.
The pipeline for each frame consisted of the following steps:


1. First step was to create a working copy of the image. All the following steps were performed on the copy.

2. Then, I applied a color mask (the maskYellow function), which masks all colors in the picture, which are below a fixed, high treshold of V=225 (value) in the HSV scale.
In practice, this leaves white-ish and yellow-ish colors (i.e. the colors of lane lines) and blackens out everything else.

3. Next, I converted the image to grayscale.

4. Next, I applied gaussian blur (kernel size set to 5).

5. Next, I applied the Canny transform. The low treshold set to 50 and the high treshold set to 150 gave good results.

6. Next, I applied two polygonal masks (region_of_interest): 
one for the region in the left side of the image where the left lane line is expected and one for the region in the right side of the image,
where the right lane line is expected. The regions were treated separately for the purposes of the Hough transform, in order to eliminate
the appearance of "bogus lane lines" crossing the road perpendicullary during video playback.

7. The Hough transform with parameters rho=2, theta=2pi/180, treshold=50, min_line_length=10, max_line_gap=140 was applied separately 
to the left masked region and to the right masked region. 

8. From within the Hough transform, the draw_lines function was called (again, for each of the regions separately).
The purpose of the function was to average over the lines identified by the Hough transform, so that a single line is formed,
and to extend the aggregate line so it would at the bottom of the image, and end roughly at 4/5 of the visible road segment.
The averaging was done  by taking the mean over the set of the top and the bottom ends of the lines from the Hough algorithm.
The extension was performed by computing the slope and the intercept of the aggregate line, and using these to recalculate
the desired end points (y=slope \* x + intercept). 

8a. As an additional feature of the draw_lines function, a global variable linesG was stored. If there are at least two lines to average at a given time step,
then linesG was updated with the aggregate line of the time step. If there was only one or zero lines to average,
then averaging was not performed, and instead the most recent value of linesG served as a guess for the postion of the lines.
The purpose of this feature was to stabilize the algorithm, so it will not crash if the Hough transform outputs zero lines,
and so it will not propose a "bogus lane line" in case the Hough transform outputs one line, which may be an artifact of other white objects in the image.
The underlying assumption behind this feature is that the video is continuous, so the lane line from the previous time step is a reasonable substitute.

9. The left lane line and the right lane line are merged to a single image, and then superimposed over the original image (using the weighted_img function).




###2. Identify potential shortcomings with your current pipeline

The line memory feature described in 8a has two shortcomings. First one is that it relies on a global variable.
The second one, is that it relies on the fact that the first image in the stream correctly identified the lane lines.

The polygonal masks are quite narrow. This yields accurate results for a car already driving on a lane,
but if the car is performing a maneuver like merging into traffic or changing lanes, then the lane lines will not be found.

In the last video, on several frames where the trees overshadowed the road,
the algorithm would not produce a lane line. 

###3. Suggest possible improvements to your pipeline

Addressing the last shortcoming: it would be good to apply some transform to get rid of the shadows.

A possible improvement with regard to the first shortcoming would be to handle the memory feature in a more structured way, without global
variables, and also disable it for the first video within the stream.

As for the polygonal masks, it would be good to make their position continuously recalculated in a region around lane lines from the previous time step.
Ideally, the algorithm would also recognize the steering motion and use it to guess a correct placement of the masks.

