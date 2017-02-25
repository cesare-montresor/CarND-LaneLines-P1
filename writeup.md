**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline is composed of main 6 steps:
1. Image conversion to grayscale and blur
2. Edge detection
3. Image masking
4. Lines detection 
5. Lines filtering and clustering
6. Final frame assembly

The original image is turned to grayscale and blurred in order to perform the edge detection. 
Once the edges are extracted, the image is masked to be able to select only the region of the image where we expect to find the lane lines.
The line detection is performed using hough transform, that return an unordered list of segments represented by a pair of coordinates.
Those segments, are first filtered in order to remove some noise, based on steepness of the line. Then they are aggregated into 2 homogeneous groups using kmeans on the M and Q parameters of the lines.
For each cluster is draw a line, on and empty image, using the averaged M and Q from the beginning of the region of interest to the bottom of the original frame.
In order to stabilize the lines and reduce the flickering, the lines are averaged with the one draw on the previous frame. 
The original image and the one just generated are then blend into one and returned to MoviePy to be reassembled into a video.

###2. Identify potential shortcomings with your current pipeline

- This pipeline as it is right now is manually tuned to work in the given condition of the video, but as it show in challenge.mp4 negligible (to a human) changes of color in the image or additional noise easily disturb the detection. It is very likely that this pipeline would fail even for simple changes of lighting or weather conditions.

- The number of expected lanes is 2 and they are expected to be found respectively in the left and right half of the image, which even if it might be the most occurring case while driving, for sure doesn't take into considerations several other cases. Among others, possible case scenario when this pipeline can fail are when overtaking other cars or at a crossroad, in case of discoloring of the paint of the lanes or simply with lanes marked by a double line. 

###3. Suggest possible improvements to your pipeline

- Clustering
Is a a bit of a weak solution as it is based on the assumption that there will be exactly 2 lines on the image.
I believe it could be improved by aggregating the segments in clusters with the goal of reducing the variance of the cluster both as a mean to split a cluster and as way filter out noise.
Also, for better clustering would be probably a better choice to replace M with it's polar equivalent in degree(or rads) as it grows linearly and in a limited interval.
Perhaps other algorithms could be also tested, like DBSCAN as it takes into account the density, doesn't require to se a prior number of cluster, among other benefits.

- Average previous N frames
It provides a great benefit in the final outcome, even just by averaging the lines found in the current frame with the one found in the previous one, the result is much more stable. I though of averaging the last N frames but I soon realized that even if it delivers a better result on the example, it might have other side effects on a real case as it would make it less reactive (slower) to detect and respond to rapid changes, proportionally to the number of frames averaged.

- Adaptive ROI and Canny
To limit the amount of potential noise could be interesting to try to adapt the ROI based on the previous frame using the intersection of the lines as tip and the end of the lines as base of the triangular area, plus a margin. Perhaps with a fallback to the default area in case of no lines detected. Given this smaller ROI, trying to use a few ways to enhance contrast and brightness to facilitate edge detection, but also try to adjust the threshold of canny according to overall brightness of the ROI.

- Debugging tools and automated testing
Part of the issues I had while tuning the parameters of the various functions was to see the result of tuning a single parameter only after a few seconds of elaboration and often just so see that it was worst but without understanding too much as I could see one layer at the time. I understand now that is very useful to setup from early development a solid debugging mode, for example adding some "picture in picture" for the intermediate layers directly on the video.

### Considerations

This way of approaching the problem require a great deal of tuning and identify possible case scenario but as it is in an open environment the overall feeling is that will always leave an exception case uncovered and sooner or later the system will fail due to lack of flexibility. 
