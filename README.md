# CNN-HowManyFingers-PyTorch
A convolutional neural network (CNN) trained and implemented in PyTorch to predict the number of fingers being help up.

## Key Contents
* **app.py** - application to collect data and predict number of fingers 
* **train_model.ipynb** - jupyter notebook used for training model
* **images.tgz** - compressed tarball of complete training and validation/testing data
* **models/model_Final.pth** - final trained model (Can be used as a pretrained model for further exploration)
* **process_and_optimization** - see draft training notebook with notes for each step,model optimization notes (training iterations and hyperparameters changes), and more
* **requirements.py** - complete list of packages used (with versions)

## Demo
Demo of model predicting the number of fingers being held up in a hand.

> Without (left) and with (right) binary mask visible.

![normal](process_and_optimization/demo/ordered.gif)
![binary mask](process_and_optimization/demo/binarymask.gif)

# Application
An application within this repository uses the CNN and openCV to the make predictions real time of the number of fingers being held up.

## Usage
Run `app.py` and hold your hand up within the highlighted (red box) prediction region / region of interest (ROI).
The model performs best when provided a plain white background without many features.

### Keyboard Shortcuts
* The prediction region on the screen can be moved in different directions by pressing: 
    >  `i` key (up) --- `l` key (down) --- `j` key (left) --- `k` key (right)
* Display the binary mask being applied to the prediction region by toggling with the:
    >  `b` key
* Quit the application by pressing:
    >  `q` key

# Feature Input
The dataset contains grayscaled images that were collected by applying a binary mask to the hands edges, as this simplified the number of inputs and computations used by the CNN. 

The data was collected by holding up 0 to 5 fingers in the region of interest and storing the images. 
The original dataset from the baseline project was used, for which each label contains about 1500 images plus an additional 600 images for validation.

![training_images](process_and_optimization/readme-support/training_images.png)

# Convolution Neural Net

The CNN used for this projects consists of 4 convolutional layers with 3x3 kernels, RELU activations and integer multiples of 32 filters in each layer. Between each convolutional layer a MaxPooling is applied to reduce the models dimensionality. The feature maps produced by the convolutions are passed to a dense layer with 512 nodes and RELU activation before being fed to a softmax output layer with 6 nodes, defining each class.

```
class CNN(nn.Module):
    def __init__(self):
        super(CNN, self).__init__()
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3)
        self.pool1 = nn.MaxPool2d(2)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3)
        self.pool2 = nn.MaxPool2d(2)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3)
        self.pool3 = nn.MaxPool2d(2)
        self.conv4 = nn.Conv2d(128, 128, kernel_size=3)
        self.pool4 = nn.MaxPool2d(2)
        self.hidden= nn.Linear(128*16*16, 512) 
        self.drop = nn.Dropout(0.3) 
        self.out = nn.Linear(512, 6)

    def forward(self, x):
        x = F.relu(self.conv1(x))
        x = self.pool1(x)
        x = F.relu(self.conv2(x))
        x = self.pool2(x)
        x = F.relu(self.conv3(x))
        x = self.pool3(x)
        x = F.relu(self.conv4(x))
        x = self.pool4(x)
        x = x.flatten(start_dim=1)
        x = F.relu(self.hidden(x))
        x = self.drop(x)
        x = self.out(x)
        x = F.softmax(x, dim=1)
        return x
```

> ![cnn_summary](process_and_optimization/readme-support/cnn_summary.png)

# Training
The model is trained with PyTorch using an NVIDIA Tesla P100 for 40 epochs using batches of 128 images each.
To give variation to the training set, the input was augmented with shifts, rotations, zooms, as well as a mirroring to prevent biased to any right or less hand.

Training used a cross entropy loss function, and adadelta optimizer with a starting learning rate set to 1. A learning scheduler was applied to decreased the learning rate by 0.1 every 15 epochs.

> The model was trained on the Google Cloud AI Platform. The cost to have this VM notebook running is estimated to be about $0.266/hour (around time of training). 
![GoogleCloud_VM](process_and_optimization/readme-support/GoogleCloud_VM.png)

# Results and Performance
A validation testing performance of greater than 99% accuracy was achieved during training. There seems to be an issue with loss as it is converging near 1. This was not a critical issue as model performance within the application is as working well as expected with the accuracy results, hence it is deemed acceptable for purposes of this prototype project, however further investigation might be helpful for the future.

![training_Final](process_and_optimization/results/training_Final.png)

![normal testing within app](process_and_optimization/demo/ordered_2.gif)

# Error Analysis 
Applying image augmentations to the testing set allowed a deeper understanding in model performance. It seems that applying these image transformations by torchvision decreased the accuracy by < 1%. This could likely due to a finger being covered by one of the augmentations. This issue should not be of concern as a human can make the same misclassification error if provided with partial information.

The testing accurancy for every label showed that the model has a slight difficulty in classifying four fingers being held up. This was a signal of an issue to further investigate with a confusion matrix and by observing some of the bad predictions.

> ![testing_accuracy for each label](process_and_optimization/readme-support/testing_accuracy.png)

![bad_predictions](process_and_optimization/readme-support/bad_predictions.png)

To better understand model performance outside normal conditions, further testing was done holding up fingers in uncommon positions (ex. only the index and picky). The model was able to classify better than expected in these corner cases, however additional data will might be useful to improve the performance of the model under these scenarios.

![corner case](process_and_optimization/demo/cornerCase.gif)

The confusion matrix shows that the slight error in the model happens when attempting to classify four fingers, and usually confused for three fingers.

![confusion_matrix](process_and_optimization/readme-support/confusion_matrix.png)

Further testing of the model within the application with the binary mask visible helped understand the probable cause of the issue. It seems most common when the four fingers are rotated to the side. This could be improved with more data or by making adjustments to the model. For the scope of this project this matter is deemed acceptable but could be improved in the future.

![four finger error observation](process_and_optimization/demo/misclassification.gif)

Furthermore, with more testing of the model within the application, it can be observed that the predictions of the model are sensitive to any background objects, including shadows and too much lighting.
This was expected as the original dataset for the baseline tensorflow protype had a similar issue. 
To re-emphasize, isolation of hands features can be improved with refined image processing to the lighting. 
Also, the model could be improved by gathering more training data as well as augmentating the images with busy backgrounds and various lighting.

# Acknowledgement
This project was completely built in PyTorch and used as baseline an implemention that was done on Keras and TensorFlow 1. If you would like to see how this project is built in these other technologies, check it out:
https://github.com/jaredvasquez/CNN-HowManyFingers
