---
layout: post
title: Waymo Dataset Preview
date: 2022-07-23 21:34:23 +0900
category: autonomous_dataset
---

# Dataset Structure
![Untitled](https://user-images.githubusercontent.com/65657711/185524283-ada1f911-0748-4c92-a2b2-2be62ef588e0.png)



### message **Frame**

- extensions 1000 to max
- optional **Context** context
- optional int64 timestamp_micros
- optional Transform pose
- repeated CameraImage images
- repeated Laser lasers
- repeated Label laser_labels
- repeated CameraLabels projected_lidar_labels
- repeated CameraLabels camera_labels
- repeated Polygon2dProto no_label_zones

### message Label

- message Box
    - optional double center_x
    - optional double center_y
    - optional double center_z
    - optional double length
    - optional double width
    - optional double height
    - optional double heading
    - enum Type
        - TYPE_UNKNOWN
        - TYPE_3D
        - TYPE_2D
        - TYPE_AA_2D
- optional Box box
- message Metadata
    - optional double speed_x
    - optional double speed_y
    - optional double speed_z
    - optional double accel_x
    - optionaldouble accel_y
    - optional double accel_z
- optional Metadata metadata
- enum Type
    - TYPE_UNKNOWN
    - TYPE_VEHICLE
    - TYPE_PEDESTRIAN
    - TYPE_SIGN
    - TYPE_CYCLIST
- optional Type type
- optional string id
- enum DifficultyLevel
    - UNKNOWN
    - LEVEL_1
    - LEVEL_2
- optional DifficultyLevel detection_difficulty_level
- optional DifficultyLevel tracking_difficulty_level
- optional int32 num_lidar_points_in_box
- optional int32 num_top_lidar_points_in_box
- oneof keypoints_oneof
    - keypoints.LaserKeyPoints laser_keypoints
    - keypoints.CameraKeyPoints camera_keypoints
- message Association
    - optional string laser_object_id
- optional Association association
- optional string most_visible_camera_name
- optional Box camera_synced_box

- message MatrixShape → 차원의 모양을 선언
    - repeated int32 dims
- message MatrixFloat
    - repeated float data
    - optional MatrixShape shape
- message MatrixInt32
    - repeated int32 data
    - optional MatrixShape
- message CameraName
    - enum Name
        - UNKNOWN
        - FRONT
        - FRONT_LEFT
        - FRONT_RIGHT
        - SIDE_LEFT
        - SIDE_RIGHT
- message LaserName
    - enum Name
        - UNKNOWN
        - TOP
        - FRONT
        - SIDE_LEFT
        - SIDE_RIGHT
        - REAR
- message Transform  → 3d points를 한 프레임에서 다른 프레임으로 변환할 때 쓰이는 행렬
    - repeated double transform
- message Velocity
    - optional float v_x
    - optional float v_y
    - optional float v_z
    - optional double w_x
    - optional double w_y
    - optional double w_z
    

### message CameraCalibration

- optional CameraName.Name name
- repeated **double intrinsic**
    
    : 1d Array of [f_u, f_v, c_u, c_v, k{1, 2}, p{1, 2}, k{3}]
    
    Note that this intrinsic corresponds to the images after scaling → 스케일링 후의 이미지의 intrinsic parameter
    
    Radial distortion coefficients: k1, k2, k3
    
    Tangential distortion coefficients: p1, p2
    
- optional **Transform extrinsic**
    
    : Camera frame to vehicle frame
    
- optional int32 width
- optional int32 height
    
    : Camera image size.
    
- enum RollingShutterReadOutDirection
    - UNKNOWN
    - TOP_TO_BOTTOM
    - LEFT_TO_RIGHT
    - BOTTOM_TO_TOP
    - RIGHT_TO_LEFT
    - GLOBAL_SHUTTER
- optional RollingShutterReadOutDirection rolling_shutter_direction

### message LaserCalibration

- optional [LaserName.Name](http://LaserName.Name) name
- repeated double beam_inclinations
- optional double beam_inclination_min
- optional double beam_inclination_max
- optional Transform extrinsic

### message **Context**

- optional string name
- repeated CameraCalibration camera_calibrations
- repeated LaserCalibration laser_calibrations
- message Stats
    - message ObjectCount
        - optional Label.Type type
        - optional int32 count
    - repeated ObjectCount laser_object_counts
    - repeated ObjectCount camera_object_counts
    - optional string time_of_day
    - optional string location
    - optional string weather
- optional Stats stats

### message **RangeImage**

: RangeImage is a 2d tensor. The first dim (row) represents pitch, The second dim represents yaw.

![Untitled 1](https://user-images.githubusercontent.com/65657711/185524386-33b05ce8-c2d4-4618-b778-b8d0e38f98df.png)


- optional bytes **range_image_compressed** → [H, W, 4]

: Zlib compressed [H,W,4] serialized version of matrixFloat

→ 추후 tf.io.decode_compressed(””, ‘ZLIB’)을 통해 압축해제 해서 사용. // 아래 compressed 형태도 모두 적용

channel 0: range / channel 1: intensity / channel 2: elongation / channel 3: is in any no label zone

- optional bytes **camera_projection_compressed** → [H, W, 6]

: Lidar point to camera image projections, A point can be projected to multiple camera images. We pick the first two at the following order: [FRONT, FRONT_LEFT, FRONT_RIGHT, SIDE_LEFT, SIDE_RIGHT]

하나의 3d 포인트가 여러 카메라 이미지로 부터 투영된 점일 수 있다. 카메라 전부를 고려하진 않고 2개의 카메라에서 3d 포인트를 투영한다.(각도상)

channel 0: [CameraName.Name](http://CameraName.Name) of 1st projection. Set to UNKNOWN if no projection.

channel 1: x (axis along image width) / channel 2: y (axis along image height)

channel 3:  [CameraName.Name](http://CameraName.Name) of 2nd projection. Set to UNKNOWN if no projection.

channel 4: x (axis along image width) / channel 5: y (axis along image height)

 

- optional bytes **range_image_pose_compressed** → [H, W, 6]

: Inner dimensions are [roll, pitch, yaw, x, y, z]

represents a transform from vehicle frame to global frame for every range image pixel.

The roll, pitch and yaw are specified as 3-2-1 Euler angle rotations → roll rotation about the z, y and x axes.

- optional bytes **range_image_flow_compressed** → [H, W, 5]

: Inner dimensions are [vx, vy, vz, pointwise class]

(vx, vy, vz) are velocity along (x, y, z) -axis

class is set to one of the following value:

-1: no-flow-label(the point has no flow information) / 0: unlabeled or “background”

1: vehicle / 2: pedestrian / 3: sign / 4: cyclist

- optional bytes **segmentation_label_compressed** → [H, W, 2]

: Inner dimensions are [instance_id, semantic_class]

+) 주의사항

1. Only TOP LiDAR has segmentation labels.
2. Not every frame has segmentation labels. This field is not set if a frame is not labeled

3.There can be points missing segmentation labels within a labeled frame. Their label are set to TYPE_NOT_LABELED when that happens.

- optional bytes **MatrixFloat range_image**

### message CameraSegmentationLabel

- optional int32 panoptic_label_divisor
- optional bytes panoptic_label → (semantic_class_id * panoptic_label_divisor + instance_id)
- message InstanceIDToGlobalIDMapping
    - optional int32 local_instance_id
    - optional int32 global_instance_id
    - optional bool is_tracked
- repeated InstanceIDToGlobalIDMapping instance_id_to_global_id_mapping
- optional string sequence_id

### message CameraImage

- optional [CameraName.Name](http://CameraName.Name) name
- optional bytes image
    
    : JPEG  image,  tf.image.decode_jpeg(frame.images)를 통해 사용
    
- optional Transform pose
    
    :SDC(Self Driving Car) pose / **4 x 4 matrix (like extrinsic parameters)**
    
- optional Velocity velocity
    
    :SDC velocity at ‘pose_timestamp’, The velocity value is represented at ‘global’ frame
    
- optional double pose_timestamp
    
    : Timestamp of the ‘pose’ above
    
- optional double shutter
- optional double camera_trigger_time
- optional double camera_readout_done_time
- optional CameraSegmentationLabel camera_segmentation_label

### message CameraLabels

- optional [CameraName.Name](http://CameraName.Name) name
- repeated Label labels

### message Laser

- optional [LaserName.Name](http://LaserName.Name) name
- optional RangeImage ri_return1\\
- optional Rangeimage ri_return2
