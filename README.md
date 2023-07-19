# labelme2labelbee(josn)转换脚本
## 一、为何要写这个转换脚本？
### 1.1 labelme
labelme是使用python写的基于QT的跨平台图像标注工具，可用来标注分类、检测、分割、关键点常见的视觉任务，被广大的CV研究者、学生和自由开发者所使用着。
但labelme是基于PyQt编写的，性能有限。labelme在实例分割场景下，当关键点过多时，会出现严重的卡顿，影响标注性能。
Labeme:https://github.com/wkentaro/labelme

### 1.2 labelbee
labelbee是商汤科技开源的计算机视觉领域的 AI 算法框架OpenMMLab团队推出的开源标注工具。
LabelBee标注工具发源于对商业应用场景的一系列算法标注需求，具备拉框、标签、标点、线条、多边形、文本等常用标注工具，可广泛适用于目标检测、分类、分割、关键点、折线、OCR等算法场景。可以JavaScript SDK的方式灵活的“一键”嵌入到业务系统。也可以直接安装开源的客户端版本，解压开箱即用。Labelbee基于JavaScript标注工具，面对更多关键点的实例分割标注问题有更好的性能和更流畅的交互体验，减少了卡顿提高了标注工作效率。
Labelme:https://github.com/open-mmlab/labelbee/
存储结构：
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/17edfef0-063f-46ff-9532-d3b5fe4dbb52)

labelme的数据由一个.json文件（名称为：图片名+.json）和图片组成。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/771b9f91-ea82-4648-a6ed-bae8e62008b2)

labelbee的数据由一个.json文件（名称为：图片名+后缀+.json）和图片组成。

## 二、如何将labelme的json转为labelbee的json文件?
### 2.1 分析数据结构
labelme和labelbee数据结构组成如下：
labelme的json是一个字典结构，key1:version和key2:flags对于我们的转换并无直接帮助，直接舍弃。
所有的实例分割的多边形以列表的形式储存在shape✔️这个key3对应的值中。列表中每个元素都代表一个多边形，多边形的信息以字典的形式存储在列表中。每个多边形的字典中包裹label、pionts、group_id、shape_type、flags共5个key，对应的值分别代表：label的名称；多边形中的所有关键点的坐标以先x后y的形式存储在一个列表中，来表征一个坐标[x,y]，所有的坐标以标注的顺序存在一个关键点集的列表中；group_id无法映射到labelbee的json中；shape_type用语表述任务类型本脚本用于处理实例分割任务，故都是polygon；flags无法映射到labelbee的json中。
key4:imagePath对应原图的地址
key5:imageData对应原图的地址
key6:imageHeight：图片的高度✔️
key7:imageWidth：图片的宽度✔️
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/bd1cf47f-0b06-4d0e-9fa5-92c555dbaadd)

labelbee的json是一个字典结构，key1:width为图片宽度（转换自labelme-imageWidth）✔️和key2:height为图片高度（转换自labelme-imageHeight）✔️
key3:vaild对应一个布尔值初始为True，表示该标签生效（这个在映射过程中硬编码为True，后续可以通过交互修改）。key4:rotate对应旋转角度，一般为0（这个在映射过程中硬编码为0，后续可以通过交互修改）。
key5:step_1对应一个字典，这个字典内部包括dataSourceStep，对应于、toolName对应于任务类型（因为用于实例分割，所以硬编码为："polygonTool"）；result对应的一个列表，不同序数对应于不同的多边形。
id'(str)8位包括大小写字母，也可以包括部分数字，此处根据规则随记生成一串id；sourcelD一般为空，此处硬编码为空；valid对应一个布尔值，该多边形生效为 True，不生效为False，本脚本硬编码为True；textAttribute，一般为空，此处硬编码为空；pointList对应值为多个字典，每个字典有x、y两个键对应的值为关键点的x、y坐标值（转换自labelme-shapes）；attribute，一般为空，此处硬编码为空；order对应的值是顺序，以1开始，此处根据顺序生成数字的序数。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/698ebc93-6efc-4790-976e-4281ed4a24d3)

## 三、环境配置
Python 3.8
labelme 5.0.5
labelbee 0.1.2

## 四 操作
### 4.1 labelme标注
在cmd输出labelme，启动labelme，点击Open选择一张图片。选择CreatePolygons在图片上点击通过绘制多边形全选目标，并以一种labelname命名这个mask。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/6455c295-731c-4c14-8245-a447786ec490)

保存后，图片路径里面可以看到图片和json文件
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/e4cf6c5b-3a48-437e-8f35-e8c72e738552)

### 4.2 脚本转化
在该路径下创建.py的脚本，粘贴本文脚本进入。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/5bdd3f0d-40a7-42a5-b55b-7fd8905992c5)

填写image name，运行脚本。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/c26185b6-6acc-416b-97fb-bc2335c03c17)

得到一个使用labelbee的json数据文件。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/24c653cd-69a3-4c61-b538-b33a7783a3bf)

### 4.3 重新将json载入labelbee
进入labelbee，点击New project，创建一个新的标注项目。选择Single Step创建一个单步骤项目。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/624ff110-c9f7-4796-925e-a99872261c6d)
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/669eba40-01af-4d4e-8b24-79c7996eebb0)
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/2bf2afe2-981c-4a20-ac66-a210917680d0)

设定开启Attribute，添加多个类型，点击OK创建完成。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/ba6192b0-f3f0-4265-a1ac-43b543fc1ba9)

选择新创建的Project。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/2a7d03a6-bddb-459d-aff7-88834d174fa7)

启动之后发现多个多边形已经存在，只是对应的类型未设定。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/3fa92ee0-0f97-4eaf-a06c-eebf2dbf5976)

通过右键选中多边形，右侧勾选类型，完成整个数据的转换过程。
![image](https://github.com/Woshidawenxi/labelme2labelbee-json/assets/72373043/50ee0f31-4496-430b-bfeb-b0e0dafdc341)
