# Finding Lane Lines Writeup

This is my writeup for Project 1 of the Self-Driving Car Engineer Nanodegree.

[image1]: ./examples/grayscale.png "Grayscale"
[image2]: ./examples/blur.png "Gaussian Blur"
[image3]: ./examples/canny.png "Canny Edge Detection"
[image4]: ./examples/roi.png "Region of Interest"
[image5]: ./examples/hough.png "Hough Transform"
[image6]: ./examples/best-fit.png "Lines of Best Fit"
[image7]: ./examples/final.png "Final Image"

### Reflection

#### Pipeline

1. I convert the image to grayscale so we are only dealing with pixel intensity. This is because we only want the Canny Edge Detection to find edges where the pixel intensity changes and not worry about color.

    ![Step 1][image1]

2. Next, we apply a slight gaussian blur to the whole image. This is to further help the Canny Edge Detection from picking up sharp edges and random high intensity pixels.

    ![Step 2][image2]
    
3. Next, we run the image through Canny Edge Detection with a low threshold of 40 and a high threshold of 80.

    ![Step 3][image3]
    
4. Next, we apply a region of interest mask to remove any edges that aren't likely to be lane lines. A trapezium shape is used to filter out anything but the current lane of the ego vehicle.

    ![Step 4][image4]
    
5. Next, we run the processed image through a Hough Transform to find the straight lines. We then get a series of (x,y) coordinate pairs that make up a single line. We then place the lines into "bins" depending on if the line is on the left or right side of the screen. The image below shows green as being lines detected on the left and blue being on the right.

    ![Step 5][image5]
    
6. As you can see, the green lines on the left are seperated. To make each line a consistent single line, I made some modifications to the "draw_lines()" function. I take all of the (x, y) coordinates that make up lines on the left or right side and use a robust linear regression method (Theil–Sen Estimator) to create a line of best fit. Using Theil-Sen instead of a simple least-squares method allows me to reduce the effect of outliers. The two lines of best fit are shown in red below.

    ![Step 6][image6]
    
7. Finally, I overlay the lines onto the original image. I decided to leave the green and blue hough lines to help with debugging causes of the line moving eratically.

    ![Step 7][image7]

#### Potential Shortcomings

* The elevation of the road ahead will effect the region of interest. If the car was to crest over a hill, the ROI would need to be reduced otherwise other objects such as clouds, hills or trees might be included in the hough transform.

* The suspension of the car (the car tilting forward or backwards under breaking and acceleration) and the mounting of the camera would also affect the region of interest.

* The canny edge detection and hough transform need constant tuning to the lighting conditions and environment, it's nearly impossible to make parameters that fit all scenarios.

* You could only use these lines to tell the car to turn left or right. If we wanted to do more useful trajectory planning, we would need to know the location of the lane lines in meters from the ego vehicle etc. To do this would require camera calibration and removing the distortion of the camera lens.

* Lanes are often not straight. The hough transform used is only detecting straight lines. This means that when going around corners, the detected lane would be extremely short in front of the ego vehicle and not particularly useful.

* This method is only capable of detecting the lane the ego vehicle is currently in.


#### Possible Improvements

* Instead of representing lanes as straight lines, represent them as a 3rd degree (or more) polynomial. This can be done with curve fitting and using a birds-eye projection.

* The slope of the lane lines can move erratically, we could use some sort of moving average from the previous camera frames to smooth this movement.

* Instead of using basic computer vision techniques, use a machine-learning approach removing the need for an ROI.

* Use a birds-eye projection of the front camera to further improve the ability to detect other lanes to the side of the ego vehicle.
