---
layout: post
title: waymo open dataset visualization
date: 2022-08-10 19:20:23 +0900
category: autonomous_dataset
---
# Visualize manual code

### [2D bbox visualize]

camera labels 

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled.png)

label.box.center_x, label.box.center_y, label.box.length, lable.box.width를 이용해서 2d bbox를 visualize 할 수 있다. 

+) domain adaptation 이미지는 label 정보가 없다.

```python
import matplotlib.pyplot as plt
import matplotlib.patches as patches
plt.figure(figsize=(25, 20))

def image_show(data, name, layout, cmap=None):
  ax = plt.subplot(*layout)
  for camera_labels in frame.camera_labels:
    # Ignore camera labels that do not correspond to this camera.
    if camera_labels.name != data.name:
      continue

    # Iterate over the individual labels.
    for label in camera_labels.labels:
      # Draw the object bounding box.
      ax.add_patch(patches.Rectangle(
        xy=(label.box.center_x - 0.5 * label.box.length,
            label.box.center_y - 0.5 * label.box.width),
        width=label.box.length,
        height=label.box.width,
        linewidth=1,
        edgecolor='red',
        facecolor='none'))
      print(label.box.length)

  # Show the camera image.
  plt.imshow(tf.image.decode_jpeg(data.image), cmap=cmap)
  plt.title(open_dataset.CameraName.Name.Name(data.name))
  plt.grid(False)
  plt.axis('off')

for index, image in enumerate(frame.images):
  image_show(image, frame.camera_labels, [3, 3, index+1])
```

### [Camera only 3d labels]

- camera_synced_boxes:  자율 주행 차량에 장착된 카메라는 시계 방향으로 순차적으로 트리거되며, 각 카메라는 [롤링 셔터](https://en.wikipedia.org/wiki/Rolling_shutter) 를 사용하여 장면을 옆으로 빠르게 스캔합니다 . 사용자가 이 롤링 셔터 타이밍의 영향을 줄이는 데 도움이 되도록 카메라와 동기화되는 데이터 세트에 추가 3D 경계 상자를 포함합니다. 롤링 셔터, 자율주행 차량의 움직임, 물체의 움직임을 고려한 최적화 문제를 해결하기 위해 만들어진 label
    
    lidar로 물체를 관찰한 시간과 다르다.   
    
    camera synced box에는 lidar synced box가 있지만 lidar bbox에는  camera bbox가 없을 수 있따. ⇒  camera의 수평시야는 230도이기 때문..
    
- (frame.)projected_lidar_labels: 카메라 이미지에 투영된 기본 3D LiDAR 레이블
- Visualize projected_lidar_labels

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled%201.png)

- Visualize camera_synced_box Projections
    
    (3D Wireframe boxes)
    

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled%202.png)

(2D Boxes)

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled%203.png)

### [2D panoramic video panoptic segmentation]

![스크린샷 2022-08-09 오전 10.31.52.png](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2022-08-09_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB_10.31.52.png)

Waymo Open Dataset 에서 ‘2022: Added 2D Video panoptic segmentation labels’ 되어 있는 데이터셋을 다운받는다. (그 외 데이터는 annotation 되어 있지 않다.)

하나의 *.tfrecord 당 sequence id 를 가지고 있다. 

1. Proto로 된 Frame 읽어들이기 
    
    

할 수 있는거 

: 파놉틱 파노라마 띄우기, 원본도 파노라마로 띄우기, 

라이다 포인트 띄워서 파노라마로 띄우기, 2D 바운딩 박스 띄우기, 박스의 신비 

### [panorama로 image 변환]

```python
dataset = tf.data.TFRecordDataset(FILE_NAME, compression_type='')

frames_unordered = []
frames_with_seg = []

sequence_id = None

for data in dataset:
  frame = open_dataset.Frame()
  frame.ParseFromString(bytearray(data.numpy()))
  if frame.images[0].camera_segmentation_label.panoptic_label: # frame.images[i] i = 0..4  
    frames_with_seg.append(frame)
    frames_unordered.append(frame)
    if sequence_id is None:
      sequence_id = frame.images[0].camera_segmentation_label.sequence_id
      print(sequence_id)
    if frame.images[0].camera_segmentation_label.sequence_id != sequence_id or len(frames_with_seg) > 0:
      break
camera_left_to_right_order = [open_dataset.CameraName.SIDE_LEFT, # 3 
                              open_dataset.CameraName.FRONT_LEFT,
                              open_dataset.CameraName.FRONT,  # 1 
                              open_dataset.CameraName.FRONT_RIGHT,
                              open_dataset.CameraName.SIDE_RIGHT] 
                              
frames_ordered = []

for frame in frames_unordered:
  image_proto_dict = {image.name : image.image for image in frame.images}
  frames_ordered.append([image_proto_dict[name] for name in camera_left_to_right_order])

def _pad_to_common_shape(image):
  return np.pad(image, [[1280 - image.shape[0], 0], [0, 0], [0, 0]])

images_decode = [[tf.image.decode_jpeg(frame) for frame in frames ] for frames in frames_ordered]

print(np.shape(images_decode))
padding_images = [[_pad_to_common_shape(image) for image in images ] for images in images_decode]

panorama_image_no_concat = [np.concatenate(image, axis=1) for image in padding_images]
panorama_image = np.concatenate(panorama_image_no_concat, axis=0)

plt.figure(figsize=(64, 60))
plt.imshow(panorama_image)
plt.grid(False)
plt.axis('off')
plt.show()

```

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled%204.png)

Panoptic label 이 198개의 Frames 중 전부 있는 것이 아니고 일부 구간만 존재한다. 

### Image distortion correction

```python
w = frame.context.camera_calibrations[0].width
h =  frame.context.camera_calibrations[0].height
f_x, f_y, c_u, c_v, k_1, k_2, p_1, p_2, k_3= frame.context.camera_calibrations[0].intrinsic 

mtx = np.array([[f_x, 0, c_u],[0, f_y, c_v],[0, 0, 1]])
dist = np.array([[k_1, k_2, p_1, p_2, k_3]])
print('1-1. Calibration: intrinsic matrix')
print (mtx)
print('1-2. Calibration: distortion coefficients')
print (dist)

plt.figure(figsize=(18, 10))

frame_jpg = tf.image.decode_jpeg(frame.images[1].image)

plt.subplot(1, 2, 1)
plt.imshow (frame_jpg)
plt.title('distortion')
plt.axis = ('off')

frame_jpg = np.array(frame_jpg)
frame_gray = cv2.cvtColor(frame_jpg, cv2.COLOR_BGR2GRAY)
undist, roi = cv2.getOptimalNewCameraMatrix(mtx, dist, (w, h), 1, (w, h))
img_undist = cv2.undistort(frame_jpg, mtx, dist, None, undist)

plt.subplot(1, 2, 2)
plt.imshow(img_undist)
plt.title('undistortion')
plt.axis = ('off')
```

![Untitled](Visualize%20manual%20code%2016385ee154564bf1aa1b6cfeaf265290/Untitled%205.png)

### [3D floating]

matplotlib

```python

points, cp_points = frame_utils.convert_range_image_to_point_cloud(frame,
                                                       range_images,
                                                       camera_projections,
                                                       range_image_top_pose)

# 3d points in vehicle frame.
points_all = np.concatenate(points, axis=0)

fig = plt.figure(figsize=(20,20))
ax = fig.gca(projection='3d')

data = points_all
X = data[:,0]
Y = data[:,1]
Z = data[:,2]

ax.scatter3D(X,Y,Z, s = 0.15)
ax.set_xlabel('X label')
ax.set_ylabel('Y label')
ax.set_zlabel('Z label')
axes_limit = 50
ax.set_xlim(-axes_limit, axes_limit)
ax.set_ylim(-axes_limit, axes_limit)
ax.set_zlim(-axes_limit, axes_limit)
plt.show()
```

plotly

```python
import plotly.graph_objects as go

x = points_all[:,0]
y = points_all[:,1]
z = points_all[:,2] 

axis = 50
fig = go.Figure(data=[go.Scatter3d(x=x, y=y, z=z, 
mode = 'markers',
marker=dict(size = 0.2), )], 
layout=go.Layout(
width=1024,
height=1024,
scene = dict(
    xaxis = dict( range = [-axis,axis]),
    yaxis = dict( range = [-axis,axis]),
    zaxis = dict( range = [-axis,axis]))))

fig.show()
```