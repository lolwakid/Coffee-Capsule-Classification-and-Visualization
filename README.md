# Coffee Capsule Identification & Visualisation:

Milestone of the project:
1.	Use the captured videos to train a model for recognizing and counting the number of capsules according to different types of colors and shapes. 
2.	Create a dashboard for data visualization.
Data Format:
Data was in the form of videos from different camera angles.

Classes for the Project:

We have 7 coffee flavours ('altissio', 'alto_dolce', 'bianco_leggero', 'chiaro', 'diavolito', 'lid', 'scuro', 'voltesso') and we will include one more class, 'lid', which will be acting as unknown (top part of the capsule and/or unknown class) category.

Data Labelling:
Microsoft’s VoTT (https://github.com/microsoft/VoTT) was used to label/annotate the objects (in our cases, coffee capsules) and the detailed steps on how to label objects in the video can be found at https://github.com/microsoft/VoTT/tree/v2.2.0#labeling-a-video.
Download link for VoTT can be found at https://github.com/Microsoft/VoTT/releases, where you can choose your respective OS version. I used v2.2.0 for this project.

Key things to remember while labelling:
1.	Keep the bounding boxes for the objects tightly fitted.
2.	Always tag all objects in given frame.
3.	If the object is hard-to-distinguish/unclassified class, then put it into unknown tag.
4.	Label occluded objects entirely.

## Model Training and Testing:
### Setting Up Environment for Object Tracking Task:

We have to make sure that the notebook’s runtime is set to GPU as it speeds up our execution and performance before running the cells.
Then we install tensorflow version 2.3.1 and tensorboard version 2.4.1 by using pip commands (1st cell). Afterwards, we install torch as YOLOv5 runs on top of the PyTorch library.
Now, we have to clone the YOLOv5 from ultralytics repository (https://github.com/ultralytics/yolov5) and install all the packages mentioned in the requirements.txt file in yolov5 folder.

### Downloading our Dataset:
You should go to your home directory by using ‘%cd /content/’ command. And then using !curl -L command like !curl -L "https://app.roboflow.com/ds/FarhbE41Wp?key=GHJSwix4bV" > roboflow.zip; unzip roboflow.zip; rm roboflow.zip, where “content” will be your unique dataset link from Roboflow.
Now, make sure that the train and val paths written in ‘data.yaml’ file are correct as we will needing this file for Training our model.

### Training the model:

We are going to use train.py script file which is present in yolov5 folder to train our model.
As my dataset only contained 629 Images in total, I have used Yolov5s model version where ‘s’ stands for small. You can find more information on which model version you should choose for your dataset, on ultralytics repository’s readme file.
The arguments which I have passed in my notebook file were:
--img 416 --batch 16 --epochs 100 --data '../data.yaml' --cfg ./models/yolov5s.yaml --weights yolov5s.pt --name yolov5s_results --nosave –cache
But if you want to save detected objects frames in the form of images, omit ‘—nosave’ argument and ‘–cache’ argument was used to fasten up our training process. You can try to experiment with different number of epochs as well as batch size.
After successful batch of training is completed, you will either get ‘best.pt’ or ‘last.pt’ weights file along with the path where the weights file has been saved (for my case, it was saved in runs/train/yolov5s_results).
 
### Model evaluation:

You can use tensorboard to see the logs generated for the training in an interactive manner or you can plot the ‘result.png’ as depicted below: (also in cell 11 in our notebook)
 
Now as my Roboflow Dataset contained collection of images and labels, which we later on used for training our model, we will now need to create videos subfolder inside the test folder, in which the video you want to test or generate the inference should be included.

### Testing the Model:

We will be using detect.py script file, present inside yolov5 folder for the testing purpose. But before we run that script file (cell  22), we are to make the following changes in the detect.py file:
1.	Replace line 212 in detect.py, original line would be ‘s += f"{n} {names[int(c)]}{'s' * (n > 1)}, " ’ with 's += '%g %ss, ' % (n, names[int(c)]) '. (replace without single quotes)
2.	Insert a new sentence on line 213 as 'cv2.putText(im0,s, (20,100), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 1, cv2.LINE_AA)

#### Keynotes:
The above mentioned changes are to count the number of objects (Live object counter) in playing/current frames and print them inside the video using cv2.putText() method.
Now you can test the model by running the detect.py script where arguments were as follows:
-weights runs/train/yolov5s_results/weights/last.pt --img 416 --conf 0.4 --source ../test/videos --save-txt
For above arguments, weights path location may or may not differ for you, if the objects are not getting detected for your custom object problem, you can try to minimize the confidence threshold (--conf 0.2 or 0.1), smyce will act as ymy test location path and lastly, --save-txt will save the labels for the videos contained in the smyce path.
If my testing code cell was successful, then the results will be saved in ‘yolov5/runs/detect/exp’ and we can download the generated inference video by going to the above path and downloading it manually or we can download the whole folder by zip method as shown in code 23 of our notebook. The zip file will contain the inference videos and their related labels.

### Dashboard Visualization
Post-processing for labels:
Combining all .txt files into a single .csv file was done in ‘post_processing.ipynb’ file.
As we downloaded the ‘inference’ folder shown in cell 23 notebook, all the labels were individual for their respective video frames and they were in .txt format, so I had to combine them into .csv files (available on GitHub, with notebook name : post_processing.ipynb) while also replacing the encoded label numbers with their actual names and adding two additional columns which were ‘distance’ and ‘frame’. Distance formula =(SQRT(POWER(x_center2-x_center1,2)+POWER(y_center2-y_center1, 2)) was calculated in the Excel software, Where x_center and y_center were my bounding box coordinates. As the number of objects detected in each .txt file were different, I had to manually enter the frame numbers in csv file while inspecting each .txt files individually, which took around 2-3 hours approximately. Afterwards, I just had to replace encoded label numbers with their respective label names ('0:altissio', '1:alto_dolce', '2:bianco_leggero', '3:chiaro', '4:diavolito', '5:lid', '6:scuro', '7:voltesso') which was also done in Excel using find and replace function. Best resulting frame was frame 208 so I copied all data belonging to frame 208 from ‘05_labels.csv’ file and saved it in ‘frame208.csv’. These .csv files are available in my GitHub repo.

Dashboard was created in Google Data Studio where the elements includes: embedded inference video and it's related class distribution in the form of pie chart, most optimal live object counter for frame 208 shown as a picture and it's related details (label, distance, record count) in tabular format.
Link for the Dashboard can be found at https://datastudio.google.com/s/lytisJX5Tis
