# OpenCV-Find-Lane-for-Self-Drving-Cars
Finding Lane for self Driving Cars


## What are the prerequisites? 
Some basic knowledge of OpenCV would be good.

## Step 1: Edge Detection
```
def canyEdgeDetector(image):
    edged = cv2.Canny(image, 50, 150)
    return edged
```
    
## Step 2: Define ROI(Region of Interest)
```
def getROI(image):
    height = image.shape[0]
    width = image.shape[1]
    # Defining Triangular ROI: The values will change as per your camera mounts
    triangle = np.array([[(100, height), (width, height), (width-500, int(height/1.9))]])
    # creating black image same as that of input image
    black_image = np.zeros_like(image)
    # Put the Triangular shape on top of our Black image to create a mask
    mask = cv2.fillPoly(black_image, triangle, 255)
    # applying mask on original image
    masked_image = cv2.bitwise_and(image, mask)
    return masked_image
```

## Step 3: Get Lines
```
def getLines(image):
    lines = cv2.HoughLinesP(image, 0.3, np.pi/180, 100, np.array([]), minLineLength=70, maxLineGap=20)
    return lines
```

## Step 4: Some utility functions
```
def displayLines(image, lines):
    if lines is not None:
        for line in lines:
            x1, y1, x2, y2 = line.reshape(4) #converting to 1d array
            cv2.line(image, (x1, y1), (x2, y2), (255, 0, 0), 10)
    return image
```

## Step 5: Getting Smooth line
```
def getSmoothLines(image, lines):
    left_fit = []  # will hold m,c parameters for left side lines
    right_fit = []  # will hold m,c parameters for right side lines

    for line in lines:
        x1, y1, x2, y2 = line.reshape(4)
        parameters = np.polyfit((x1, x2), (y1, y2), 1)
        slope = parameters[0]
        intercept = parameters[1]

        if slope < 0:
            left_fit.append((slope, intercept))
        else:
            right_fit.append((slope, intercept))

    left_fit_average = np.average(left_fit, axis=0)
    right_fit_average = np.average(right_fit, axis=0)

    # now we have got m,c parameters for left and right line, we need to know x1,y1 x2,y2 parameters
    left_line = getLineCoordinatesFromParameters(image, left_fit_average)
    right_line = getLineCoordinatesFromParameters(image, right_fit_average)
    return np.array([left_line, right_line])
```
