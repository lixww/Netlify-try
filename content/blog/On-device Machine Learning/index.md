+++
title = "On-device Machine Learning"
date = 2024-03-20
[taxonomies]
  tags = ["On-device ML", "ML"]

[extra]
  toc = true
+++

# Update 0618

On-device models deployment.

- For android: ML Kit, or TensorFlow Lite.
See here [*Use ML Kit on Android.*](https://developers.google.com/ml-kit)
See here [*Use TensorFlow Lite on Android.*](https://developer.android.com/ai/custom#androids-custom-ml-stack-high-performance-ml)
- For iOS: Core ML.
See here [*Core ML Tools.*](https://apple.github.io/coremltools/docs-guides/)

# TensorFlow Lite

A library for mobile device. The workflow is to Choose or train a model → Convert it into a compressed flat buffer with Converter (xxx.tflite) → Load the tflite file into a mobile device → Quantize by converting 32-bit floats to more efficient 8-bit integers or run on GPU to optimize. [Android guide is available here.](https://www.tensorflow.org/lite/android)

## Optimize models

The objects of optimizing models are to 

a) reduce the size, b) reduce the latency, c) compatible with accelerator.

But surely there are trade-off between accuracy and optimizations.

Here are some types of optimization available for TensorFlow Lite:

1) Quantization.

2) Pruning.

3) Clustering.

[See more in reference: Model optimization .](https://www.tensorflow.org/lite/performance/model_optimization)

Also, optimize operators by customizing operators could be an option for a faster model. [See more here.](https://www.tensorflow.org/lite/performance/best_practices#profile_and_optimize_operators_in_the_graph)

# Google MediaPipe

Offers both prepared and customizable on-device machine learning pipelines. Models are task-oriented as well. Low-code.

Take the task of pose landmark detection as example. MediaPipe offers models (TensorFlow Lite models actually) encapsulated in xxx.task (downloaded in assets/). In Android, there is a `PoseLandmarker` class provides methods to detect pose landmarks. Pretty neat call. The main code focus more on the application building, instead of code about machine learning. `PoseLandmarker` is in package `com.google.mediapipe.tasks.vision.poselandmarker`. [See more in here.](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker) 🍇 [GitHub demo is here.](https://github.com/googlesamples/mediapipe/tree/main/examples/pose_landmarker/android) 

There are three models available on the task of pose landmark detection, which are _full, _heavy, _lite.

![Reference: [Pose landmark detection Overview](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker)](On-device%20Machine%20Learning%20f53caffdf8fe48a3980e09e55c8d3fa5/Untitled.png)

*Reference: [Pose landmark detection Overview](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker).*

The `PoseLandmarker` accepts still images, decoded video frames, and live video feed as input.

Model customization is possible on some tasks, such as image classification. [See guide here](https://developers.google.com/mediapipe/solutions/customization/image_classifier#model_tuning).

## Retraining model using Model Maker tool

[See more here about MediaPipe Model Maker](https://developers.google.com/mediapipe/solutions/model_maker).

![Hyper-parameters to optimize during retraining.](On-device%20Machine%20Learning%20f53caffdf8fe48a3980e09e55c8d3fa5/%25E6%2588%25AA%25E5%25B1%258F2024-03-21_14.42.04.png)

*Hyper-parameters to optimize during retraining.*

Post-training **model quantization** can help reducing the model size, able to config when exporting the model. See more in [this TensorFlow Lite guide](https://www.tensorflow.org/lite/performance/post_training_quantization).

# Google ML Kit (Older)

Offers prepared task-oriented on-device machine learning models and APIs. 

Take the task of pose landmark detection as example. ML Kit utilizes TensorFlow Lite models directly (xxx.tflite, downloaded in assets/). In Android, there is a `PoseDetector` class (in the package `com.google.mlkit.vision.pose`), similar with the `PoseLandmarker` of MediaPipe. [See more in here.](https://developers.google.com/ml-kit/vision/pose-detection/android?hl=zh-cn)  🍇 [GitHub demo is here.](https://github.com/googlesamples/mlkit/tree/master/android/vision-quickstart)


```txt
💡 The pose landmarker model offered by MediaPipe and ML Kit are the same, both tracks 33 body landmark locations. [They train the model in the same way.](https://blog.research.google/2020/08/on-device-real-time-body-pose-tracking.html) *BlazePose* is now MediaPipe.

```

Only one model (model.tflite) is pre-downloaded under assets/ in the demo project of ML Kit.