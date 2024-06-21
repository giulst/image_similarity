# image_similarity

### Installation 
It is possible to install the repository from Github:

git clone https://github.com/giulst/image_similarity/

The repository contains:
- data.txt: file that contains the link to download the data
- Image_similarity_pipeline.ipynb: a jupyter notebook with the image similarity pipeline.
- Image_similarity_pipeline.html: the html from the notebook, with some examples already saved and displayed
- requirements.txt: file that contains the packages used to build and run the pipeline.

Data can be downloaded and saved in a folder called "data".

### Idea
The idea behind the pipeline is divided in three main step:
# 1. Extract from each figure some important features that can be used to define similarity between shapes.
Among them, the color of the shape and of the background, the center of gravity of the feature which determines the position, the dimension of the shape.
To obtain the colors, I simply found the two unique colors used, then the background color was set as the color present in the majoprity of corners (to avoid cases where the image covers one of the corners).
The dimension of the shape was obtained counting the number of pixels with the shape color.
The gravity center was found checking the mean of x and y axis of pixels colored with the shape color.

# 2. Implement a CNN to classify the geometric shape
This CNN is then used as a first step to define the most similar images to the input ones, taking advantage of the geometric family class already available in the dataset.

Before applying the CNN, some preprocessing of the images was done, to "simplify" the work for the neural network, so it could focus only on the geometric family recognition (all the other important features have been already saved in step 1). 
The preprocessing consists in: 1. converting the image in black (for background) and white (for shape) and 2. enlarge the shape to the image dimension (so the CNN is not confounded by dimensions, and actually this step boosted the performences of the CNN). To do this, I cropped the largest square around the object and the resized the image to 200 x 200.
To avoid overfitting each layer of the CNN was built with a not too high number of neurons and with a final dropout layer. The output is the shape geometric family, prediction is built using softmax to obtain probaility for each possible class. The geometric family with higher probability is given as final output.

The CNN was trained on 1000 images (80% as train set and 20% as validation set). Cross Entropy was used as loss, since we have a multiclass classification and accuracy on the validation set was used as performance metrics.

# 3. Similarity measure
Once that images have been classified in one specific geometric family, only the images of that classes are retained for further checks.
Also for test images the preprocessing steps are applied.
The features obtained in step 1 are then used to measure the distance between the new object and all the images used to train the model.
A weighted euclidean distance was used, giving a higher weight to:
1. shape color (weight=2)
2. dimension (weight=1)
3. gravity center (weight=1)
4. background color (weight=0.5)

The 3 most similar images (minimum distance) are provided for a check, together with the original one.
