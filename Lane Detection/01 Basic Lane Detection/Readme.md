## A) Canny Edge Detection

Canny Edge Detection is a popular edge detection algorithm. It was developed by John F. Canny in

1. It is a multi-stage algorithm and we will go through each stages.
2. **Noise Reduction :** Since edge detection is susceptible to noise in the image, first step is to remove the noise in the image with a 5x5 Gaussian filter. We have already seen this in previous chapters.
3. **Finding Intensity Gradient of the Image :** Smoothened image is then filtered with a Sobel kernel in both horizontal and vertical direction to get first derivative in horizontal direction (Gx) and vertical direction (Gy). From these two images, we can find edge gradient and direction for each pixel as follows:

![image](https://user-images.githubusercontent.com/40880896/120822872-7719f480-c574-11eb-8c95-a7ca921c93fc.png)

Gradient direction is always perpendicular to edges. It is rounded to one of four angles representing vertical, horizontal and two diagonal directions.

4. **Non-maximum Suppression :** After getting gradient magnitude and direction, a full scan of image is done to remove any unwanted pixels which may not constitute the edge. For this, at every pixel, pixel is checked if it is a local maximum in its neighborhood in the direction of gradient. Check the image below:

![image](https://user-images.githubusercontent.com/40880896/120823091-af213780-c574-11eb-985f-fd1884d9b537.png)

 Point A is on the edge ( in vertical direction). Gradient direction is normal to the edge. Point B and C are in gradient directions. So point A is checked with point B and C to see if it forms a local maximum. If so, it is considered for next stage, otherwise, it is suppressed ( put to zero).

In short, the result you get is a binary image with "thin edges".

5. **Hysteresis Thresholding :**
This stage decides which are all edges are really edges and which are not. For this, we need two threshold values, minVal and maxVal. Any edges with intensity gradient more than maxVal are sure to be edges and those below minVal are sure to be non-edges, so discarded. Those who lie between these two thresholds are classified edges or non-edges based on their connectivity. If they are connected to "sure-edge" pixels, they are considered to be part of edges. Otherwise, they are also discarded. See the image below:

![image](https://user-images.githubusercontent.com/40880896/120823308-ed1e5b80-c574-11eb-96d6-1d0cd1d37561.png)

The edge A is above the maxVal, so considered as "sure-edge". Although edge C is below maxVal, it is connected to edge A, so that also considered as valid edge and we get that full curve. But edge B, although it is above minVal and is in same region as that of edge C, it is not connected to any "sure-edge", so that is discarded. So it is very important that we have to select minVal and maxVal accordingly to get the correct result.

This stage also removes small pixels noises on the assumption that edges are long lines.

So what we finally get is strong edges in the image.

### Canny Edge Detection in OpenCV

OpenCV puts all the above in single function, cv.Canny(). We will see how to use it. First argument is our input image. Second and third arguments are our minVal and maxVal respectively. Third argument is aperture_size. It is the size of Sobel kernel used for find image gradients. By default it is 3. Last argument is L2gradient which specifies the equation for finding gradient magnitude. If it is True, it uses the equation mentioned above which is more accurate, otherwise it uses this function: Edge_Gradient(G)=|Gx|+|Gy|. By default, it is False. 

```
import numpy as np
import cv2 as cv
from matplotlib import pyplot as plt
img = cv.imread('messi5.jpg',0)
edges = cv.Canny(img,100,200)
plt.subplot(121),plt.imshow(img,cmap = 'gray')
plt.title('Original Image'), plt.xticks([]), plt.yticks([])
plt.subplot(122),plt.imshow(edges,cmap = 'gray')
plt.title('Edge Image'), plt.xticks([]), plt.yticks([])
plt.show()
```

![image](https://user-images.githubusercontent.com/40880896/120823498-1f2fbd80-c575-11eb-9aef-6b8bdcf7a4e1.png)

## B) Hough Transform

### What is Hough transform?

Hough transform is a feature extraction method for detecting simple shapes such as circles, lines etc in an image.

A “simple” shape is one that can be represented by only a few parameters. For example, a line can be represented by two parameters (slope, intercept) and a circle has three parameters — the coordinates of the center and the radius (x, y, r). Hough transform does an excellent job in finding such shapes in an image.

The main advantage of using the Hough transform is that it is insensitive to occlusion.

Let’s see how Hough transform works by way of an example.

### Hough transform to detect lines in an image

![image](https://user-images.githubusercontent.com/40880896/120840508-fb29a780-c587-11eb-8e09-4896680aa507.png)

### Equation of a line in polar coordinates

![image](https://user-images.githubusercontent.com/40880896/120840783-480d7e00-c588-11eb-96a7-6b0ec2f733ba.png)

### Accumulator

![image](https://user-images.githubusercontent.com/40880896/120840973-89059280-c588-11eb-97e8-894a0ec45da5.png)

![image](https://user-images.githubusercontent.com/40880896/120841035-a0448000-c588-11eb-80b7-8f0b7a3636ab.png)

![image](https://user-images.githubusercontent.com/40880896/120841103-b3575000-c588-11eb-8209-5361a7180252.png)

![image](https://user-images.githubusercontent.com/40880896/120841269-eb5e9300-c588-11eb-8c9a-17e355ecb612.png)

![image](https://user-images.githubusercontent.com/40880896/120841318-fca79f80-c588-11eb-81d6-b52d6db7d01f.png)

![image](https://user-images.githubusercontent.com/40880896/120841429-1943d780-c589-11eb-863c-0fe7ec818d3b.png)

### HoughLine: How to Detect Lines using OpenCV

![image](https://user-images.githubusercontent.com/40880896/120841526-3bd5f080-c589-11eb-8537-629277f572c9.png)

![image](https://user-images.githubusercontent.com/40880896/120841593-4ee8c080-c589-11eb-8c9f-bf45be2a2880.png)

### Line Detection Result

Below we show a result of using hough transform for line detection. Bear in mind the quality of detected lines depends heavily on the quality of the edge map. Therefore, in the real world Hough transform is used when you can control the environment and therefore obtain consistent edge maps or when you can train an edge detector for the specific kind of edges you are looking for.

![image](https://user-images.githubusercontent.com/40880896/120841743-83f51300-c589-11eb-8873-33cc4399b4fa.png)

![image](https://user-images.githubusercontent.com/40880896/120841802-97a07980-c589-11eb-9b09-284f132230b4.png)

