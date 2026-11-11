# Waste Sorting Model — Technical Report

In this project, I developed a Python application that uses object detection to identify different types of waste in images and videos. I wanted to see how a trained computer vision model could be used in a simple application that gives visual feedback on waste sorting.

The project has three main parts: preparing and training the model, evaluating the results, and building the application.

## 1. Model training

### Dataset preparation

I worked with a dataset containing three classes:

- **Dryckeskartong** — beverage carton
- **Konservburk** — food can
- **Pantburk** — deposit can

We collected images as part of the course project, with each participant contributing at least 30 images. I used **Label Studio** to annotate objects with bounding boxes, and the labels were exported in YOLO format.

I then moved the data into **Google Colab** and wrote Python code to organize the dataset. My goal was to split it into **80% training, 10% validation and 10% testing** so I could evaluate the model on images outside the training set.

In the saved notebook, there are 753 image files and 721 annotation files before matching filenames. The split step reports **528 training, 66 validation and 67 test images**. The notebook uses a random shuffle without a fixed seed, so I would use a fixed seed and an empty output directory if I repeated the experiment.

### Training

For training, I used the **Ultralytics YOLO** library. The training notebook loads **YOLO26n** (`yolo26n.pt`) and trains it using:

| Setting | Value |
|---|---|
| Environment | Google Colab, T4 GPU |
| Epochs | 50 |
| Image size | 640 pixels |
| Batch size | 8 |
| Model | YOLO26n |

Using the GPU made training much faster than running it on a CPU. I followed the loss values and detection metrics during training to see how well the model was learning.

## 2. Evaluation and analysis

After training, I evaluated the model on the test set. My recorded results were:

| Metric | Result |
|---|---:|
| Test images | 67 |
| Object instances | 205 |
| Precision | 0.954 |
| Recall | 0.967 |
| mAP50 | **0.988** |
| mAP50–95 | **0.891** |

The loss curves improved during training, and the detection metrics reached a good level. The confusion matrix showed that most objects were classified correctly. The main mistakes were between **food cans and deposit cans**, which makes sense because their shapes can be quite similar.

These numbers describe the performance of the **object detector**. They do not measure the final sorting decisions made by my application.

### Stress test

I also wanted to see how the model would work outside the normal test images, so I tried it on **10 new images** with more difficult conditions, including darker lighting, different backgrounds and partially hidden objects.

The model recognized objects in most of the images, but in **2 very dark images** it did not detect any objects. That made it clear that lighting has a big effect on the result, even when the overall test metrics are high.

## 3. Application implementation

Once the model was trained, I started building the sorting application in Python.

My first version worked with still images. It detected the objects, read their classes and bounding boxes, and calculated their center points. I then used the distance between objects to decide whether they belonged to the same group.

I later developed a version that processes **video**. In that version, I used YOLO's tracking functionality to follow objects between frames. I also added smoothing so the bounding boxes and sorting labels would not jump around as much.

The basic logic was:

1. Detect and track waste objects in the current frame.
2. Calculate the center point and size of each detected object.
3. Group objects that are close to each other.
4. Check whether objects in each group belong to the same class.
5. Draw the results on the video.

I used **green boxes** for groups with the same type of waste, **red boxes** for mixed groups and **yellow boxes** for individual objects. This made it easy to see how the logic behaved in the output video.

The grouping method is a simple position-based rule, rather than an independent assessment of whether the waste has been disposed of correctly.

## 4. Challenges and possible improvements

One of the biggest challenges was getting the model to perform well in different conditions. It worked well in the standard tests, but the stress test showed that low lighting could cause missed detections.

Another challenge was turning detections into a useful application. Recognizing an object is one thing, but deciding how it relates to other objects requires additional logic. I experimented with grouping by distance and smoothing the results over video frames.

If I developed this further, I would:

- Collect more images in low-light and unusual environments.
- Test more object types and different packaging designs.
- Make the dataset split reproducible with a fixed random seed.
- Measure the **sorting logic's accuracy** separately from object detection.

## 5. Conclusion

This project gave me practical experience with several parts of computer vision, from dataset annotation and model training to evaluation and video processing.

I also learned that strong model metrics are only one part of building an application. The detection model needs to work together with additional logic, and that logic has to be tested in realistic conditions.

The [GitHub repository](https://github.com/william-skogberg/waste-sorting-object-detection) contains my Colab notebooks, code and project documentation.

---

**William Skogberg**  
Arcada University of Applied Sciences
