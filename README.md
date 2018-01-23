# Project: Finding Lane Lines on the Road

In this project, I used Python and OpenCV to build a pipeline that detects lane lines on road images and eventually applying the pipeline to video (frames) that has roads with lane lines.

Below are the steps involved in the pipeline:
- 1.	Reading an image
- 2.	Conversion of image to grayscale
- 3.	Gaussian blur
- 4.	Canny edge detection
- 5.	Region of Interest selection
- 6.	Hough Line Transform
- 7.	Averaging line segments
- 8.	Drawing lines on lanes
- 9.	Adding (Blending) images

## Reading an image
This step involves reading an image from the working directory

Below are the 2 images I will be using for testing my code as I move forward in the pipeline:

## Conversion of image to grayscale
Once you read the image, convert the image to grayscale to simplify the image processing. It is relatively easier to deal with a single color channel (shades of white/black) than multiple color channels. 

Below are the 2 images after going through this process:

## Gaussian Blur
This step involves using Gaussian blur (smoothing) for edge detection. Most edge-detection algorithms are sensitive to noise and using Gaussian blur filter before edge detection aims to reduce the level of noise in the image, which improves the result of the following edge-detection algorithm. Later in our pipeline we use Canny edge detection algorithm which also applies Gaussian blur but we apply our own Gaussian blur to reduce the noise. 
Choose Kernel_size an odd number, larger kernel size implies averaging or smoothing over larger area. Based on my experiments, I have chosen Kernel size as 7.

Below are the images after applying Gaussian blur to grayscale images:

## Canny Edge Detection
Canny Edge Detection is a popular edge detection algorithm developed to detect edges in an image.  opencv.canny() includes a Gaussian filter internally, but we also di it again for further smoothing or nose reduction. Output of this process is a binary image with white pixels tracing out the detected edges (stromg gradient) and black everywhere else.
A nice read on Canny edge detection can be found [here](https://en.wikipedia.org/wiki/Canny_edge_detector)

Canny recommendation for low threshold to high threshold a low to high ratio of 1:2 or 1:3. I have chosen 50 and 150 as low and high thresholds.

Below are the images after passing through Canny edge detection:

## Region of Interest selection
Once we have the image from canny edge detection, we can only consider pixels for color selection in the region where we expect to find the lane lines. This is done by applying a quadrilateral mask on the edge detected image using opencv [fillPoly](https://docs.opencv.org/3.0-beta/modules/imgproc/doc/drawing_functions.html#fillpoly) and [bitwise_and](https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html).

Below are the images after  applying a quadrilateral mask to edge detected image:

## Hough Line Transform
With all the edges detected and selecting our region of interest to mark lanes, we can now use Hough Line Transform to detect straight lines.
In image space, a line is plotted as x vs y as y=mx+b, here in hough space it is plotted as m vs b instead. A point in image space describes a line in Hogh space. So a line in an image is a point in Hough space
With two kinds of Hough Line Transforms in OpenCV, we use Probabilistic Hough Line Transform, which is the more efficient implementation of the Hough Line Transform and gives as output the extremes of the detected lines (x0,y0,x1,y1). In OpenCV this is implemented with the function HoughLinesP. 
This function takes multiple parameters as inputs and need to be tunes to get the desired output. See [opencv documentation](https://docs.opencv.org/2.4/modules/imgproc/doc/feature_detection.html?highlight=houghlinesp#houghlinesp) for a good read. 
``` 
rho = 1 # distance resolution in pixels 
theta = np.pi/180 # angular resolution in radians
threshold = 10     # minimum number of votes 
min_line_length = 2 #minimum number of pixels making the line
max_line_gap = 2    # maximum gap in pixels between connectable line segments
```
Once fine-tuned the parameters, you can now see the line segments on the images as seen below:

## Averaging line segments
Once we have the Hough Transform image from the above step, our goal is to produce only two lines representing the left and right lanes. This can be done by averaging the lines detected on a lane line. We also need to extrapolate the line to cover full lane line length where the lane lines are partially recognized.
We start by dividing the lines into two groups left and right, with left line having a negative slope and right line having a positive slope and adding weight to lines.

Once we collect all the negative slope lines and positive slope lines, we can take an average to get the left and right line parameters to calculate x coordinates (left and right) using the equation x=(y-b/a)
```
x1_left = int((y_max-b_left)/a_left)
x2_left = int((y_min-b_left)/a_left)
x1_right = int((y_max-b_right)/a_right)
x2_right = int((y_min-b_right)/a_right)
```

## Drawing lines on lanes
Once we have the entire x and y coordinates for left and right lanes from the above step, we can use opencv line function to draw lines on left and right lanes. 

Below are the images after passing through the line function:

## Adding (Blending) images
Our final step in the pipeline is to add the image from the previous step to the original 3 channel image to get a final image with lane lines. This can be achieved by using opencv addWeighted function.
A good read can be found [here](https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html?highlight=addweighted#addweighted)

Below are the final images with lane lines



