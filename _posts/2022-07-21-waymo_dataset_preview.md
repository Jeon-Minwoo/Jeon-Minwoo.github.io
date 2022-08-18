---
layout: post
title: Waymo Dataset Preview
date: 2022-07-21 21:34:23 +0900
category: Autonomous Dataset
---
# 개요

machine perception, autonomous driving technology 분야를 발전시키는데 도움을 주기 위해 Waymo Open Dataset 공개

- Dataset
    - Motion Dataset
        - 103,354개 주행 구간 대한 Object trajectories 및 corresponding 3D maps (behavior prediction research)
    - **Perception Dataset**
        - 2,030개 주행 구간에 대한 고해상도 센서 데이터 및 레이블
- Sensor
    - Mid-range LiDAR 1대
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/906d3439-ea92-48ca-abd6-10bdca2f6c00/Untitled.png)
    
    - Short-range LiDAR 4대
    - 전,측면 카메라 5대
    

## Perception Dataset

2,030 segments of 20s each, collected at 10Hz (390,000 frames) in diverse geographies and conditions

- Sensor data
- Bounding box label
    - Labels for 4 object classes - Vehicles, Pedestrians, Cyclists, Signs
    - High-quality labels for lidar data in 1,200 segments
    - 12.6M 3D bounding box labels with tracking IDs on lidar data
    - High-quality labels for camera data in 1,000 segments
    - 11.8M 2D bounding box labels with tracking IDs on camera data
- 2D video panoptic segmentation labels
    - For a subset of 100k camera images.
    - Labels for 28 classes
    - Including: Car, Bus, Truck, Other Large Vehicle, Trailer, Ego Vehicle, Motorcycle, Bicycle, Pedestrian, Cyclist, Motorcyclist, Ground Animal, Bird, Pole, Sign, Traffic Light, Construction Cone, Pedestrian Object, Building, Road, Sidewalk, Road Marker, Lane Marker, Vegetation, Sky, Ground, Static, Dynamic.
    - Instance segmentation labels are provided for the Vehicle, Pedestrian and Cyclist classes and are consistent both across cameras and over time.
- Key point labels
    - Labels for 2 object classes - Pedestrians and Cyclists
    - 14 key points from nose to ankle
    - 200k object frames with 2D key point labels
    - 10k object frames with 3D key point labels
- 3D semantic segmentation labels
    - Segmentation labels for 1,150 segments
    - Labels for 23 classes
    - Including: Car, Truck, Bus, Motorcyclist, Bicyclist, Pedestrian, Sign, Traffic Light, Pole, Construction Cone, Bicycle, Motorcycle, Building, Vegetation, Tree Trunk, Curb, Road, Lane Marker, Walkable, Sidewalk, Other Ground, Other Vehicle, Undefined
- Association of 2D and 3D bounding boxes
    - Corresponding object IDs are provided for 2 object classes - Pedestrians and Cyclists
- Data for 3D Camera-Only Detection Challenge, with 80 segments of 20s camera imagery

**[Labeling Specifications]**

[waymo-open-dataset/labeling_specifications.md at master · waymo-research/waymo-open-dataset](https://github.com/waymo-research/waymo-open-dataset/blob/master/docs/labeling_specifications.md)

- Vehicle Labeling Specifications
    - 차량은 LiDAR 데이터나 카메라 이미지로 검출되면 만들어지며, 자전거나 오토바이의 운전자도 이동수단 주석 처리 대상에 포함된다. (단, 열차나 트램은 주석처리 대상에 포함되지 않는다.)
    - 카메라 이미지로 물체가 차량인지의 여부를 알 수 없는 경우에는 레이블이 지정되지 않는다.
    - 자동차의 사이드미러는 포함되나 안테나 같은 작은 물체는 제외된다.
    - 트레일러는 별도의 이동수단으로 표시되며, 포크레인처럼 움직이는 부분도 별도로 표시된다.
    - 
- Pedestrian Labeling Specifications
    - 차량에 탑승해 있는 경우를 제외한 모든 사람들은 Pedestrian으로 주석처리한다. (킥보드 같은 개인 운송 장치에 탄 사람도 Pedestrian으로 포함)
    - 사람이 아닌 마네킹이나 더미, 포스터 속 인물들은 주석 처리 대상이 아니다.
    - 겹쳐있는 사람은 분리 인식하며, 2m 이상의 물체를 가진 사람을 제외하고 들고 있는 물건도 보행자로 인식한다.
    - 유모차도 보행자로 인식한다.(child로 인식되진 않는다.)
- Cyclist Labeling Specifications
    - 2륜, 3륜 등 자전거로 분류되는 모든 것들을 타고있는 사람들을 포함한다. (단, 자전거에 오를 때까지는 보행자로 인식한다.)
    - Cyclist가 주석 대상이기 때문에 자전거만 있는 경우는 포함하지 않는다.
- Sign Labeling Specifications
    - 2D 이미지 없이 3D bounding box로만 인식한다.
    - 교통 흐름에 영향을 주는 표시판만 인식한다. (사람이 임의로 쓴 표지판이나 길의 이름을 쓴 표지판은 포함되지 않는다.)
    - 그림이 그려진 부분을 앞쪽으로 인식한다.

**[2D Video Panoptic Segmentation]**

- 5대의 카메라에 의해 캡처한 2860개의 시간 시퀀스로 그룹화된 100k 카메라 이미지에 대한 semantic & instance segmentation label을 제공한다.
- 28 classes로 labeling → Car, Bus, Truck, Other Large Vehicle, Trailer, Ego Vehicle, Motorcycle, Bicycle, Pedestrian, Cyclist, Motorcyclist, Ground Animal, Bird, Pole, Sign, Traffic Light, Construction Cone, Pedestrian Object, Building, Road, Sidewalk, Road Marker, Lane Marker, Vegetation, Sky, Ground, Static, Dynamic

**[Key Points]**

![Untitled](Waymo%20Dataset%20Preview%20f8d29d2f29e9414ebfd6f627ee5b631e/Untitled%201.png)

- 인체의 14개 key points의 2D & 3D bounding box labels의 집합을 제공한다. (including nose, right and left shoulders, elbows, wrists, hips, knees, ankle)
- camera objects(2D key points) - # 172.6K object annotations
- laser objects(3D key points) - # 10K object annotations
- 3D human body pose estimation을 위한 semi & weakly supervised models 훈련에 쓰일 수 있다.

**[2D-to-3D correspondence]**

![https://waymo.com/static/images/dataset/format/correspondance@2x.png](https://waymo.com/static/images/dataset/format/correspondance@2x.png)

- Corresponding object IDs are provided for 2 object classes - Pedestrians and Cyclists
- 2D Bounding box가 있는 카메라 이미지에 레이블이 지정된 object들은 또한 LiDAR 포인트 클라우드의 3D Bounding boxes로 레이블이 지정되어 있고, 해당 개체 ID를 포함한다.

**[3D Segmentation]**

[https://waymo.com/static/images/dataset/format/2d-video-panoptic-segmentation.mp4](https://waymo.com/static/images/dataset/format/2d-video-panoptic-segmentation.mp4)

- 모든 LiDAR 포인트에 대한 Dense한 labels
- 레이블은 고해상도 LiDAR 센서로 캡처한 전체 데이터 세트에 대해 2Hz로 제공.
- 23 classes로 labeling → Car, Truck, Bus, Motorcyclist, Bicyclist, Pedestrian, Sign, Traffic Light, Pole, Construction Cone, Bicycle, Motorcycle, Building, Vegetation, Tree Trunk, Curb, Road, Lane Marker, Walkable, Sidewalk, Other Ground, Other Vehicle, Undefined

/
