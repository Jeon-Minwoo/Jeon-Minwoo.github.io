# Waymo Open Dataset 정리_1

Created: 2022년 8월 19일 오전 11:16
Last Edited Time: 2022년 8월 19일 오전 11:36

## *.tfrecord

---

### waymo open dataset 포맷

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled.png)

- *.tfrecord 이란
: Tensorflow 에서 Protocol buffer 를 통해 직렬화된 정보를 저장하기 위한 포맷이다.
- Protocol buffer 이란
: 2008년 Google이 공개한 것으로 데이터를 직렬화 시켜주는 라이브러리 이다. Protocol buffer 는 *.proto 파일을 사용한다.
해당 파일을 protoc 라는 명령으로 각 언어별 binary 직렬화 시켜주는 class 를 생성하는 방법으로 동작하므로
language-neutral 한 것이 특징이다.
- *.tfrecord 저장 순서
    
    Image, label, image metadata. Etc. → Protocol buffer → *.tfrecord
    
- *.tfrecord 읽기
: *.tfrecord 는 tensorflow.data.TFRecordDataset() 를 통해서 읽을 수 있다. 파일 내부의 직렬화 된 데이터는 역직렬화를 통해 읽으며
*.proto 파일을 사용한다. Waymo Open Dataset 의 경우 ‘waymo_open_dataset/dataset.proto’ 를 통해 제공하고 있다.
- *.tfrecord 특징
- 직렬화된 데이터는 읽어들이는 속도가 빠르다.
- 하나의 *.tfrecord 파일에 여러장의 프레임을 넣을 수 있다.
- Waymo Open Dataset의 파일 분할

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%201.png)

하나의 파일 별로 약 1GB 크기로 구성되어 있다.
Camera 정보의 경우 하나의 *.tfrecord 마다 198 frames 장면 정보가 담겨있다.
-> 5대의 카메라 이므로 990 개의 frame 이 있다.

## Sensors

---

### Camera

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%202.png)

**[Image Shape]**

- FRONT: 1920x1280
- FRONT_LEFT: 1920x1280
- FRONT_RIGHT: 1920 x 1280
- SIDE_LEFT: 1920 x 886
- SIDE_RIGHT: 1920 x 886

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%203.png)

|  | F,FL,FR | SL,SR |
| --- | --- | --- |
| Size | 1920x1280 | 1920x1040 |
| HFOV | ±25.2˚ | ±25.2˚ |
| VFOV | ±50˚ | ±50˚ |

**[message CameraImage]**

- **optional bytes image**

: tf.image.decode_jpeg(frame.images)를 통해 JPEG image를 얻을 수 있다.

- **optional Transform pose**

: SDC(Self Driving Car) pose, 4x4 matrix → Vehicle Frame ⇔ Global Frame

- **optional Velocity velocity**

: SDC velocity at ‘pose_timestamp’, The velocity value is represented at ‘global’ frame

- **optional double pose_timestamp**

**[Distortion Handling]**

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%204.png)

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%205.png)

- 1d Array of [f_u, f_v, c_u, c_v, **k{1, 2}, p{1, 2}, k{3}**].
- Lens Distortion:
    
    Radial distortion coefficients: k1, k2, k3
    
    Tangential distortion coefficients: p1, p2
    
- k1, k2, k3, p1, p2 follows the same definition as OpenCV.

### LiDAR

- Mid-range LiDAR:
Top LiDAR
truncated to a maximum of 75 meters
- Short-range LiDAR:
front, side left, side right, rear LiDAR
truncated to a maximum of 20 meters

**[LiDAR FOV]**

|  | TOP | F,SL,SR,R |
| --- | --- | --- |
| VFOV | [-17.6 ˚,+2.4˚] | [-90˚,30˚] |
| Range | 75m | 20m |
| Returns | 2 | 2 |

**[message RangeImage]**

- RangeImage is a 2D tensor. The first dim(row) represents pitch, The second dim represents yaw.

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%206.png)

- FRONT, REAR, SIDE_LEFT, SIDE_RIGHT: the number of points - 200 * 600 = 120000 points with a range of up to 20m
- TOP: the number of points - 64 * 2650 = 169600 ****points with a range of up to 75m
- Multiple return LiDAR: Number of return ⇒ 2

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%207.png)

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%208.png)

⇒  second return이 나무, 물체의 가장자리를 표현하는데 도움을 준다.

- **optional bytes range_image_compressed → [H, W, 4]**

: Zlib compressed [H,W,4] serialized version of matrixFloat

→ 추후 tf.io.decode_compressed(””, ‘ZLIB’)을 통해 압축해제 해서 사용. // 다른 compressed 형태도 모두 적용

channel 0: range (0 ~ 75)

channel 1: intensity (0 ~ 1)

channel 2: elongation (0 ~ 1)

channel 3: is in any no label zone (1.0e+10)

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%209.png)

- **optional bytes camera_projection_compressed → [H, W, 6]**

:  하나의 3d 포인트가 여러 카메라 이미지로 투영될 수 있다. 카메라 전부를 고려하지는 않고, 2개의 카메라로 3d 포인트를 투영한다.

channel 0: CameraName.Name of 1st projection. Set to UNKNOWN if no projection.

channel 1: x (axis along image width) / channel 2: y (axis along image height)

channel 3:  CameraName.Name of 2nd projection. Set to UNKNOWN if no projection.

channel 4: x (axis along image width) / channel 5: y (axis along image height)

- **optional bytes range_image_pose_compressed** → [H, W, 6]

:  Inner dimensions are [roll, pitch, yaw, x, y, z]

represents a transform from vehicle frame to global frame for every range image pixel.

The roll, pitch and yaw are specified as 3-2-1 Euler angle rotations → roll rotation about the z, y and x axes.

### Coordinate Transformation

**[Coordinate Structure]**

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%2010.png)

**[message CameraCalibration]**

- **repeated double intrinsic**: 1d Array of [f_u, f_v, c_u, c_v, k{1, 2}, p{1, 2}, k{3}]
Radial distortion coefficients: k1, k2, k3
Tangential distortion coefficients: p1, p2
- **optional Transform extrinsic**: Camera frame to Vehicle frame

![Untitled](Waymo%20Open%20Dataset%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5_1%20318d43bb2f784b478d2489985f10020e/Untitled%2011.png)

**[message LaserCalibration]**

- **repeated double beam_inclinations**
- **optional double beam_inclination_min**
- **optional double beam_inclination_max**
- **optional Transform extrinsic**

: LiDAR frame to Vehicle frame