# Ground Truth

### VOC2012

本文将使用VOC2012来进行训练，所以首先来了解一下这个数据集的结构

```
parent
└── VOCdevkit
    └── VOC2012
		├── Annotations
		├── ImageSets
		├── JPEGImage
		├── SegmentationClass
		└── SegmentationObject
```

将下载的VOCtrainval_11-May-2012.tar进行解压后将得到如上所示的目录结构，可以看到文件主要分5个部分

1、Annotations

Annotations文件夹中存放的是xml格式的标签文件，每一个xml文件都对应于JPEGImages文件夹中的一张图片。

```
VOC2012
├── Annotations
    ├── 2007_000027.xml
    ├── 2007_000032.xml
    └── ...
├── ...
```

任取一个xml文件为例

```xml
<annotation>
	<folder>VOC2012</folder>
	<filename>2007_000042.jpg</filename>	// 文件名
	<source>
		<database>The VOC2007 Database</database>
		<annotation>PASCAL VOC2007</annotation>
		<image>flickr</image>
	</source>
	<size>									// 图片尺寸
		<width>500</width>
		<height>335</height>
		<depth>3</depth>
	</size>
	<segmented>1</segmented>				// 是否用于分割
	<object>								// 存在的物体
		<name>train</name>					// 物体的类别
		<pose>Unspecified</pose>			// 拍摄角度
		<truncated>1</truncated>			// 是否被截断（0表示完整）
		<difficult>0</difficult>			// 目标是否难以识别（0表示容易识别）
		<bndbox>							// bounding box 坐标（左上角、右下角坐标）
			<xmin>263</xmin>			
			<ymin>32</ymin>
			<xmax>500</xmax>
			<ymax>295</ymax>
		</bndbox>
	</object>
	<object>								// 第2个物体
		<name>train</name>
		<pose>Unspecified</pose>
		<truncated>1</truncated>
		<difficult>0</difficult>
		<bndbox>
			<xmin>1</xmin>
			<ymin>36</ymin>
			<xmax>235</xmax>
			<ymax>299</ymax>
		</bndbox>
	</object>
</annotation>
```

2、JPEGImage

JPEGImage文件夹中存放的就是上面Annotations文件夹中xml所对应的图片了。

```
VOC2012
├── Annotations
    ├── 2007_000027.jpg
    ├── 2007_000028.jpg
    ├── 2007_000032.jpg
    └── ...
├── ...
```

并且所有图片都属于下面这20个分类

```
'aeroplane', 'bicycle', 'bird', 'boat', 'bottle', 'bus', 'car', 'cat', 'chair', 'cow', 'diningtable', 'dog', 'horse', 'motorbike', 'person', 'pottedplant', 'sheep', 'sofa', 'train', 'tvmonitor'
```

### 制作标签

因为我们要做目标检测任务，只需要目标所属的类别以及目标的位置这两个信息就可以了，所以下面我们要将这两个关键信息给提取出来。为了方便对后面知识点的理解，我先把关键代码放出来。

```python
def convert_box(size, box):  # ①
    dw, dh = 1. / size[0], 1. / size[1]
    x, y, w, h = (box[0] + box[1]) / 2.0, (box[2] + box[3]) / 2.0,
    			  box[1] - box[0], box[3] - box[2]
    return x * dw, y * dh, w * dw, h * dh

in_file = open(path / f'VOC{year}/Annotations/{image_id}.xml')
out_file = open(lb_path, 'w')
tree = ET.parse(in_file)
root = tree.getroot()
size = root.find('size')
w = int(size.find('width').text)
h = int(size.find('height').text)

for obj in root.iter('object'):  # ②
    cls = obj.find('name').text
    if cls in yaml['names'] and not int(obj.find('difficult').text) == 1:
        xmlbox = obj.find('bndbox')  # ③
        bb = convert_box((w, h), [float(xmlbox.find(x).text) 
                                  for x in ('xmin', 'xmax', 'ymin', 'ymax')])
        cls_id = yaml['names'].index(cls)  # class id
        out_file.write(" ".join([str(a) for a in (cls_id, *bb)]) + '\n')
```

+ ①`convert_box`函数：传入图片大小尺寸、bounding box坐标
+ ②`root.iter('object')`：遍历所有xml中的object元素
+ ③`obj.find('bndbox')`：获取bonding box坐标信息元素

通过上面的代码我们可以知道ground truth的边界框是怎么得来的。

我们已知bbox的左上角和右下角坐标（$x_{min}, y_{min}, x_{max}, y_{max}$），我们先计算它的宽高
$$
w = x_{max} - x_{min} \\
h = y_{max} - y_{min}
$$
那么它的中心坐标自然而然也能确定下来
$$
Center_x = \frac{x_{max} + x_{min}}{2} \\
Center_y = \frac{y_{max} + y_{min}}{2}
$$
最后将$(x, y, w, h)$分别除以图片的宽高将其值限制在0~1之间，我们就能得到图片的边界框标签了