# Real Time Object Detection with measurement of distance and localization

## Created and Modified by: <br>
[![Pradumn Mishra](https://img.shields.io/badge/-Pradumn%20Mishra-magneta)](https://www.linkedin.com/in/pradumn203/)<br>

### Made with:<br>

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![OpenCV](https://img.shields.io/badge/opencv-%23white.svg?style=for-the-badge&logo=opencv&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-%23EE4C2C.svg?style=for-the-badge&logo=PyTorch&logoColor=white)

### Deployed with : 
![Flask](https://img.shields.io/badge/flask-%23000.svg?style=for-the-badge&logo=flask&logoColor=white)

## Introduction
This project uses opencv, CNN and YOLO for performing the below tasks:
 - Detecting objects in a video frame or image.
 - Object Count
 - Distance of the object using depth info.
 - Multiple Camera Inferece.
 
For object detection YOLO-V3 has been used which is able to detect 80 different objects. Some of those are-
- person
- car
- bus
- bench
- stop sign
- dog
- bear
- backpack and so on.

### User Instructions

#### The YOLOv4 update

**There is a new update with [yolov4 new release](https://github.com/Tianxiaomo/pytorch-YOLOv4). All you have to do a simple step which is after downloading the project run the following command and follow the rest of the process as it is.**

  ```
    cd YOLOv4
  ```

To execute object_dection.py you require Python version > 3.5 (depends if you are using gpu or not) and have to install the following libraries.

### Instalation
``` python
    $ pip install -r requirements.txt
         or
    $ pip install opencv-python
    $ pip install numpy
    $ pip install pandas
    $ pip install matplotlib
    $ pip install Pillow
    $ pip install imutils
```
<hr>

#### For the installation of torch using "pip" 
``` python
    $ pip3 install torch===1.2.0 torchvision===0.4.0 -f https://download.pytorch.org/whl/torch_stable.html
```
or please follow the instructions from [Pytorch](https://pytorch.org/)
#### For installing the "win32com.client" which is Text-to-Speech module for windows you have follow this
First open the cmd as an administrator, then run
``` python
   $ python -m pip install pywin32
   #After installing open your python shell and run
      import win32com.client
      speaker = win32com.client.Dispatch("SAPI.SpVoice")
      speaker.Speak("Good Morning")
```

After unzipping the project, there are two ways to run this. If want to see your output in your browser execute the "app.py" script or else run "object_detection.py" to execute it locally.


If you want to run object detection and distance measurement on a video file just write the name of the video file to variable <b>id</b> in either "app.py" or "object_detection.py" or if you want to run it on your webcam just put 0 in <b>id</b>.

However, if you want to run the infeence on a feed of <b>IP Camera </b>, use the following convention while assigning it to the variable <b>"id"</b>
``` python
    "rtsp://assigned_name_of_the_camera:assigned_password@camer_ip/"
```

For multiple camera support you need to add few codes as follows in app.py-

``` python
   def simulate(camera):
       while True:
           frame = camera.main()
           if frame != "":
               yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n\r\n')

   @app.route('/video_simulate')
   def video_simulate():
       id = 0
       return Response(gen(ObjectDetection(id)), mimetype='multipart/x-mixed-replace; boundary=frame')
```

Depending on how many feed you need, you have to add the two methods in "app.py" with different names and add a section in index.html.

``` html
<div class="column is-narrow">
        <div class="box" style="width: 500px;">
            <p class="title is-5">Camera - 01</p>
            <hr>
            <img id="bg" width=640px height=360px src="{{ url_for('video_simulate') }}">
            <hr>

        </div>
    </div>
    <hr>
```
#### Note: 
You have to use git-lfs to download the yolov3.weight file.
<hr>

#### Theory behind the whole task
In a traditional image classification approach for object detection there are two well-known strategies.

For single object in a image there are two scenarios.
- Classification
- Localization

For multiple objects in a image there are two scenarios.
- Object detection with localization
- Object segmentation
<p align="center"> 
 <b> For Single Objects </b>
    <img src ="./sample_images/puppy.jpg">
</p> 

<p align="center"> 
<b> For Multiple Objects </b>
    <img src ="./sample_images/detect-segment.jpeg">
</p> 

## Meaurement of the Distance
Ultrasonic sensor is a traditional way of measuring distance but we can use the depth information from cameras to perform the task.

### How the object detection works?
From the initial part we understood that, to measure distance from an image we to localize it first to get the depth information.
<b> Now, how actually localization works?</b>

#### Localization of objects using regression
   Regression is about returning a number istead of a class. The number can be represented as (x0,y0,width,height) which are related to a bounding box. In the images illustrated above for single object if you want to only classify  the object type then we don't need to draw the bounding box around that object that's why this part is known as <b> Classification </b>.
   However, if we are interested to know where does this object locates in the image then we need to know that 4 numbers that a regreesion layer will return. As you can see there is a black rectangle shape box in the image of white dog which was drawn using the regression layer. What happens here is that after the final convolutional layer + Fully connected layers instead of asking for class scores to compare with some offsets a regression layer is introduced. Regression layer is nothing but some rectangular box which represents individual objects. For every frame/image to detect objects the following things happens.
 - Using the inference on any pre-trained imagenet model the last fully connected layer will need to be re-trained to the desired objects. 
 - After that all the proposals (=~2000proposal/image) will be resized to maatch the inputs of the cnn.
 - A SVM is need to be trained to classify between object and background (One binary SVM(Support Vector Machine) for each class)
 - And to put the bounding box perfectly over the image a linear regression classifier is needed to be trained which will output some correction factor.
Problem with this approch is that one part of the network is dedicated for region proposals. After the full connected layers the model tries to propose certain regions on that image which may contain object/objects. So it also requires a high qulaity classifier to filter out valid proposals which will definitely contains object/objects. Although these methos is very accurate but it comes with a big computational cost (low frame-rate) and that's why it is not suitable for embedded devices such as Arduino or Raspberry Pi which has less processing power.
<hr>

#### Localizing with Convolution neural networks

Another way of doing object detection and to reduce this tedious work is by combining the previous two task into one network. Here, instead of proposing regions for every images the model is fed with a set of pre-defined boxes to look for objects. So prior to the training phase of a neural network some pre-defined rectangular boxes that represents some objects are given to the network to train with. So when a image is gone through the network, after the fully connected layer the trained model tries to match predefined boxes to objects on that image by using non-maxima suppression algorithm to completely tied. If the comparison crosses some threshold the model tries to draw the bounding box over the object. For example, in the case of the picture of white dog, the model knows what is the coordinates of the box of the dog object and when the image classification is done the model uses L2 distance to calculate the loss between the actual box coordinates that was predefined and the coordinate that the model gave so that it can perfectly draw the bounding box over the object on that image.

The main idea is to using the convolutional feature maps from the later layers of a network to run small CONV filters over these feature maps to predict class scores and bounding box offsets.
Here, we are reusing the computation that is already made during classification to localize objects is to grab the activation from the final conv layers. At this point we still have the spatial infomation of an image that model start training with but represented in a much smaller scope. So, in the final layers each "pixel" represent a larger area of the input image so we can use those cells to infer object position. Here the tensor that contains the information of the original image is quite deep as it is now squeezed to a lower dimension. At this point a 1x1 CONV layer can be used to classify each cell as a class and also from the same layer we can add another CPNV or FC(Fully Connected) layer to predict 4 numbers( Bounding Box). In this way we get both class scores and location from one. This approach is known as <b> Single Shot Detection </b> . Overall strategy in this approach can be summarised as follows:-
- Train a CNN with regression(bounding box) and classification objective.
- Gather Activation from a particular layer or  layers to infer classification and location with FC layer or another CONV layer that works like a FC layer.
- During prediction use algorithms like non-maxima suppression to filter multiple boxes around same object.
- During training time use algorithms like IoU to relate the predictions during training the the ground truth.

[Yolo](https://pjreddie.com/media/files/papers/YOLOv3.pdf) follows the strategy of Single Shot Detection. It uses a single activation map for prediction of classes and bounding boxes at a time that's why it called "You Only Look Once".

Here pre-trained of <b> yolo-v3 </b> has used which can detect <b>80 different objects</b>. Although this model is faster but it doesn't give the reliability of predicting the actual object in a given frame/image. It's a kind of trade-off between accuracy and precision.

### How the distance measurement works?
This formula is used for determing the distance 

``` python
    distancei = (2 x 3.14 x 180) ÷ (w + h x 360) x 1000 + 3
```
For measuring distance, atfirst we have to understand how a camera sees a object. 
<p align="center">
<img src="./sample_images/camera-sees-object.png">
</p>

One can relate this image the white dog picture where the dog was localized. Again we will get 4 numbers in the bounding box which is (x0,y0,width,height). Here x0,y0 is used to tiled or adjust the bounding box. Width and Height these two variable are used in the formula of measuring the object and actually describing the detail of the detected object/objects. Width and Height will vary depending on the distance of the object from the camera.

As we know an image goes refracted when it goes through a lens because the ray of light can also enter the lens whereas in the case of mirror the light can reflected that's why we get exact reflection of the image. But in the case of lens image gets little stretched. The following image illustrates how the image and the corresponding angles looks when it enters through a lens.
<p align="center">
 <img src="./sample_images/internal.png">
</p>
If we see there are three variable named:

- do (Distance of object from the lens)
- di (Distance of the refracted image from the convex lens)
- f (focal length or focal distance)

So the green line <b>"do"</b> represents the actual distance of the object from the convex length. And <b>"di"</b> gives a sense of how the actual image looks like. Now if we consider a triangle in the left side of the image(new refracted image) with base <b> "do" </b> and draw a opposite triangle similar to the left side one. So the new base of the opposite triangle will also be do with the same perpendicular distance. Now if we compare the two triangles from right side we will see <b> "do"</b> and <b> "di" </b> is parallel and the angle that create on each side of both the triangle are opposite to each other. From which we can infer that, both the triangles on the right side is also similar. Now, as they are similar, ratio of the corresponding sides will be also similar. So do/di = A/B. Again if we compare between two triangles in right side of the image where opposite angles are equal and one angle of both the triangles are right angle (90°) (dark blue area). So A:B is both hypotenuse of the similar triangle where both triangle has a right angle. So the new equation can be defined as :
<p align="center">
 <img src="./sample_images/eq1.png">
</p>
Now, if we derive from that equation we will find:-
<p align="center"> 
 <img src="./sample_images/eq2.png">
</p>
And eventually will come to at 
<p align="center">
<img src="./sample_images/eq3.png">
</p>
Where f is focal length or also called the arc length by using the following formula 
<p align="center">
<img src="./sample_images/eq4.png">
</p>
we will get our final result in "inches" from this formula of distance. 

``` python
    distancei = (2 x 3.14 x 180) ÷ (w + h x 360) x 1000 + 3
```

* Notes - As mentioned earlier YOLO prefers performance over accuracy that's why the model predicts wrong objects frquently.
