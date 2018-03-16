
## Vehicle Detection Project

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG), along with color transformation and append spatial binned color  feature and histogram of the colorspace to be used as features
* Use labeled training set of vehicle and non-vehicle images and train a classifier Linear SVM classifier
* Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
* Run the pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

### Step 1: Feature Engineering (Extraction & Selection)

Training data(cars and not-cars) provided from GTI vehicle image database & KITTI vision benchmark has been used for the project.

The three feature components include - 

1. Histogram of Oriented Gradients
Three parameters determine the dimension of the feature vector and also the seperability - 

    a) Number of pixels per cell pixels_per_cell, 

    b) Number of cells per block cells_per_block, and

    c) Number of orientations.
    

2. Spatial Binning of Transformed color-space
The parameter to choose was first the color transform itself, i.e. HSV, LUV, HLS, YUV, YCrCb. Secondly the number of spatial bins of the actual image. Spatial Binning alone doesn't seem to be a reliable feature vector.

3.  Histogram of Transformed color-space
Number of histogram bins was a parameter to chose. Although the spatial binning and the histogram approach could have chosen from different colorspaces, here I have chosen the same color space.


#### Optimal Parameters for features

Following are the parameters that were tweaked to find an optimal classifier - 

    a) Number of pixels per cell pixels_per_cell - 16, 
    
    b) Number of cells per block cells_per_block - 2
    
    c) Number of orientations - 15
    
    d) Spatial bins - (32,32)
    
    e) Histogram bins - 32
    
    f) Colorspace - 'YUV'

By brute-force method the parameters were searched to find the one that provided the optimal classifier. 'YUV' colorspace was found to give the least false alarm, in the test image #5 was quite challenging due to shadows where other colorspaces had issues with false alarm. Eventually 'YUV' and 'YCrCb' colorspace was chosen to have minimal false alarm but on the project video 'YCrCb' generated many more false alarms. The other hyper-parameter to tweak was the C parameter of the Linear SVM model to achieve 98.5% + accuracy. Additionally another consideration was paid to the time taken to generate a model, this somehow had an effect (along with bounding boxes chosen) on the inference time causing the detection video to take very very long. In general low C values generated same accuracy models rather quickly hence was prefered.

### Step 2: Training the classifier

The training and test data set were randomly shuffled since the images were cropped from closely spaced images spaces. The extracted car and not-car features were generated using the above parameters and then the training dataset was fed into a linear SVM. Parameter C = 0.001 was found optimal for the SVM classifier. 2500 examples from both the vehicles and non-vehicles were sufficient for the 98.5%+ accuracy. 

### Step 3: Sliding Window implementation

Bounding boxes
Took cues from the bounding box results on test images to decide on the scale and the start and stop boundaries. The final chosen ones are the following. More types of boxes were avoided to speed up the processing thus a trade-off of performance and speed to reach an acceptable solution. Following 6 types of bounding boxes were used for classification on each one of them -
     
                    Scale    ystart     ystop
    Type I.           0.8      400        520
    Type II.          1.2      420        560
    Type III.         1.4      425        600
    Type IV.          1.6      450        600
    Type V.           2.0      400        600
    Type VI           2.5      400        600 

### Step 4: Heatmaps and Label

After positive detections in each image, the last step is to combine the detections to actually identify vehicle locations and generate the final bounding boxes around the detection, also use thresholding technique to remove potential false alarms.
    
    a) Step 1 - Heatmap and Thresholding - From the test_images, threshold of 2 was chosen 
    b) Step 2 - scipy.ndimage.measurements.label() function was used to identify individual blobs in the heatmap.
    c) Step 3 - Bouding boxes around each detected clustered boxes were drawn indicated a detected vehicle and its associated position. 

#### Notes for video processing -

For the video, the idea of successive frames having the vehicle in the neighborhood of the present detected vehicle position is used. Thus a dequeue of 8 is used to remove extraneous false alarms that show up in one frame alone

### Conclusion & Discussion

1. At the conclusion, a reasonable pipeline for vehicle detection has been built. Although it has lots of scope for improvements. 
2. There are false alarms in the first few frames since the last 8 frames wasn't filed in the history queue.
3. In practice, vehicles and non-vehicles are imbalanced classes that would appear in a image, using such distribution to sample input data to classifier might be useful.
4. More advanced and SoA neural netowrk based architecture such as Yolo and SSD can be used to improve the performance further.
