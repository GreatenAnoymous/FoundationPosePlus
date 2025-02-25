# Enhanced FoundationPose (with New Features)

## ToDo

1. Add support for SAM2
2. Add support for TensorRT
3. Upload the technique report

## Introduction

This repository introduces new features and improvements to **FoundationPose**, focusing on enhancing 6D pose estimation and tracking capabilities. The key updates include:

1. **Pose Estimation and Tracking Using Accurate CAD Models Without Depth**
2. **Pose Estimation and Tracking Using CAD Models With Unknown Scale**
3. **Auto-Recovery Mechanism for Tracking Loss in Pose Tracking**


## Installation

To set up the enhanced FoundationPose environment:

1. Follow the instructions in the [original README](./readme_original.md) to install FoundationPose.
2. Install **[XMem](https://github.com/hkchengrex/XMem)** or **[SAM2](https://github.com/Gy920/segment-anything-2-real-time.git)** for object tracking.
3. Install **[Metric3D](https://github.com/YvanYin/Metric3D)** or **[ZoeDepth](https://github.com/isl-org/ZoeDepth)** for monocular depth estimation.

---

## Demos

### Demo 1: Pose Estimation and Tracking Using Accurate CAD Models Without Depth

The original FoundationPose requires depth input for both pose estimation and tracking. However, if a CAD model with accurate scale is available, depth input becomes unnecessary.

Below is the framework for pose estimation without depth. Download the demo dataset from the original FoundationPose repo then run:

```bash
python run_demo_without_depth.py
```

https://github.com/user-attachments/assets/a8d59141-37e4-4a3f-a822-91e5639904b6

For another demo, download our [custom data](https://drive.google.com/file/d/1RxnoS9QNRRYxJCyXAFPdeTd3UOKXMFAT/view?usp=sharing) captured using D405  and place it in the demo_data folder. Then run
```bash
python run_demo_without_depth.py --mesh_file ./demo_data/d405_demo/mesh/model.obj --test_scene_dir ./demo_data/d405_demo/

```




https://github.com/user-attachments/assets/c3c37b5b-b81a-401a-a665-658aafb5dc44




### Demo 2: Pose Estimation and Tracking Using CAD Models With Unknown Scale

In the following demo, we scale the original CAD model. The original FoundationPose fails when the CAD model is of unkown scale. On the contrary, our scale recovery is able to recover the true scale using the first frame of depth. 

```bash
python run_demo_unkown_scale.py
```
https://github.com/user-attachments/assets/6333f520-9b9c-44ce-8454-55750f9e55fe

Nowadays, many **AIGC (AI-Generated Content)** methods can generate 3D models from single-view or multi-view RGB images. Examples include **Zero-1-to-3**, **One-2-3-45**, **One-2-3-45++**, and others. These approaches can effectively create CAD models. However, the models generated by these methods lack true scale information, making them unsuitable for direct use in **FoundationPose**.

In this demo, we demonstrate how to combine One-2-3-45++, a monocular depth estimation model ZoeDepth, and FoundationPose to perform pose tracking and estimation using RGB images only. 







```bash
python run_demo_colacan.py
```


https://github.com/user-attachments/assets/e7852842-a41d-4a3e-a65b-a8e6d4d05941


### Demo 3: Auto-Recovery Mechanism Guided by XMem for Tracking Loss in Pose Tracking

In the tracking process of **FoundationPose**, a refiner network and the pose from the previous frame are used to predict the object pose for the current frame. This approach is effective because the pose for the current frame is typically close to the previous frame, provided the object is not moving rapidly or remains in view. However, **FoundationPose** can easily lose track of an object when it is briefly obstructed by other objects. A common solution to this issue is to re-register the object's pose once it becomes visible again. This poses certain challenges.

### Challenges and Solutions

1. **Detecting Tracking Loss**
   - We used a pretrained network, **XMem**, to track the object and obtain its mask. 
   - If the mask of the object in the current frame is smaller than a predefined threshold, we consider the object tracking as lost. 
   - When the object reappears and the mask returned by **XMem** exceeds the threshold, we use the **register network** to re-evaluate the object's pose based on the mask.

2. **Handling Pose Inaccuracy**
   - The object's pose output by the register network may still be inaccurate, especially if the object mask is small. 
   - To address this, we continue the re-evaluation process until the object returns to a well-tracked state. 
   - We use a heuristic: if the position predicted by the refiner network is sufficiently close to the coarse position estimated from the mask and depth, we terminate the re-evaluation process and switch back to the tracking state.

3. **Optimizing for Real-Time Performance**
   - To improve real-time performance, we reduce the number of hypothesis poses by 80%, selecting only those close to the pose before tracking was lost.
   - The final tracking frame rate of **FoundationPose + XMem** is approximately **22 FPS** on an **RTX 3090** under a resolution of **640 x 480**, using default refining iteration parameters.

### Running a Demo on the ClearPose Dataset
First download the **[ClearPose](https://github.com/opipari/ClearPose)** dataset.
 
Below is the command line to run a demo on set 8 scene 2 in the ClearPose dataset:
```bash
python run_demo_clearpose.py
```




https://github.com/user-attachments/assets/f9200409-62ee-4f8d-8699-78edfc2415e1





---

## References
1. Wen, B., Yang, W., Kautz, J., & Birchfield, S. (2024). Foundationpose: Unified 6d pose estimation and tracking of novel objects. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (pp. 17868-17879).
2. Cheng, H. K., & Schwing, A. G. (2022, October). Xmem: Long-term video object segmentation with an atkinson-shiffrin memory model. In European Conference on Computer Vision (pp. 640-658). Cham: Springer Nature Switzerland.
   
