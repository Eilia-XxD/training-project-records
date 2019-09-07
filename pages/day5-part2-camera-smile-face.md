
### 1. 使用微信jssdk
根据微信的开发文档，想要使用微信的jssdk，需要5个步骤
#### 1.1. 绑定域名:先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”
#### 1.2. 引入JS文件
在需要调用JS接口的页面引入如下JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.2.0.js  支持使用 AMD/CMD 标准模块加载方法加载
#### 1.3. 通过config接口注入权限验证配置
```
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名
    jsApiList: [] // 必填，需要使用的JS接口列表
});
```
#### 1.4. 通过ready接口处理成功验证
```
wx.ready(function(){
    // dosomething
});
```
 * 拍照或从手机相册中选图接口
```
wx.chooseImage({
    count: 1, // 默认9
    sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
    sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
    success: function (res) {
        var localIds = res.localIds; // 返回选定照片的本地ID列表，localId可以作为img标签的src属性显示图片
    }
});
```
#### 1.5. 通过error接口处理失败验证
```
wx.error(function(res){
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
});
```

### 2. 微笑检测的实现
#### 2.1. 微笑检测大概流程
* 用深度学习技术做分类
* 用OpenCV的级联分类器做人脸识别
#### 2.2. 数据介绍
* 数据地址：https://github.com/hromi/SMILEsmileD
* 数据包含13165张灰度图片，每张图片的尺寸是64*64。这个数据集并不算平衡，13165张图片中，有9475张图片不是笑脸图片，有3690张图片是笑脸图片。数据差异很大。
#### 2.3. 数据预处理
*  导入相应的包
*  解决样本不平衡问题
#### 2.4. 使用LeNet实现笑脸检测分类
* 模型实现
* 输出结果
#### 2.5. 人脸检测实现
```
from keras.preprocessing.image import img_to_array
from keras.models import load_model
import numpy as np
import imutils
import cv2
import os

haarcascade_face_dir = os.path.abspath(r"./model/haarcascade_frontalface_default.xml")
model_dir = os.path.abspath(r"./model/lenet.hdf5")
video_dir = os.path.abspath(r"./src.mp4")

detector = cv2.CascadeClassifier(haarcascade_face_dir)
model = load_model(model_dir)
camera = cv2.VideoCapture(video_dir)


while True:
    (grabbed, frame) = camera.read()

    
# grabbed和frame是read的两个返回值，grabbed是布尔类型的返回值，如果读取帧是正确的返回True，当文件读到结尾的时候返回False
# frame是每一帧的图像，是一个三维矩阵   

    frame = imutils.resize(frame, width = 300) #把图像宽度重新指定为300像素
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #因为模型训练是对灰度图像处理，所以这里要转换成灰度图像
    frameClone = frame.copy()    #重新克隆frame，用于接下来绘制边界框

     # scaleFactor用来控制图像金字塔， minNeighbors表示的最小特征存在数量，比如等于5表示的是有5个特征的时候才算这个人脸匹配成功
    rects = detector.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30),
                                     flags=cv2.CASCADE_SCALE_IMAGE)

 
 # .detectMultiScale方法返回一个四元组列表，来表示视频流的脸部矩形框
 # 刚开始的两个变量表示的是候选框x和y的位置，接下来两个变量的值表示的是候选框的宽度和高度  
    for (fX, fY, fW, fH) in rects:
        roi = gray[fY:fY + fH, fX:fX + fW]  # 提取灰度图像中的候选区域
        roi = cv2.resize(roi, (28, 28))   # 将候选区域变成28*28像素，接下来的操作都是便于roi用于CNN学习
        roi = roi.astype("float") / 255.0
        roi = img_to_array(roi)
        roi = np.expand_dims(roi, axis = 0)   # 扩展数组，在最开始的位置新加一个维度，便于Keras识别
        
        (notSmiling, smiling) = model.predict(roi)[0]    # 看概率，哪个概率大，就说明是笑还是不笑
        label = "Smiling" if smiling > notSmiling else "Not Smiling"
        
        cv2.putText(frameClone, label, (fX, fY - 10), cv2.FONT_HERSHEY_SIMPLEX, 
                   0.45, (0, 0, 255), 2)
        cv2.rectangle(frameClone, (fX, fY), (fX + fW, fY + fH),
                     (0, 0, 255), 2)
        
        
    cv2.imshow("Face", frameClone)  # 展示包含label的微笑判断
    
    if cv2.waitKey(1) & 0xFF == ord("q"):  # 如果 ‘q’被按下了，停止循环
        break
        
camera.release()  # 清空相机
cv2.destroyAllWindows()  # 关闭所有window
```

### 3. 封装"微笑检测"为后端API

#### 3.1. 编写脚本文件

本案例使用 python 编写脚本文件

>首先识别并框住人脸所在部位，然后在此基础上继续识别框住嘴唇部位 
>
>通过判断嘴角上扬的高度来确定是否微笑
>
>先设定一个全局变量 flag = 0，假设没有微笑
>
>若微笑，则在识别嘴唇的代码块后设 flag 为 1

**实现代码：**

```
# -*- coding=utf-8 -*-
import cv2
import sys
import json

flag = 0

# 人脸检测器
facePath = "lbpcascade_frontalface.xml"
faceCascade = cv2.CascadeClassifier(facePath)

# 笑脸检测器
smilePath = "haarcascade_smile.xml"
smileCascade = cv2.CascadeClassifier(smilePath)

img = cv2.imread(sys.argv[1])
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# 首先检测人脸，返回的是框住人脸的矩形框
faces = faceCascade.detectMultiScale(
    gray,
    scaleFactor=1.1,
    minNeighbors=8,
    minSize=(55, 55),
    flags=cv2.CASCADE_SCALE_IMAGE
)

# 画出每一个人脸，提取出人脸所在区域
for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 0, 255), 2)
    roi_gray = gray[y:y+h, x:x+w]
    roi_color = img[y:y+h, x:x+w]

    # 对人脸进行笑脸检测
    smile = smileCascade.detectMultiScale(
        roi_gray,
        scaleFactor=1.16,
        minNeighbors=35,
        minSize=(25, 25),
        flags=cv2.CASCADE_SCALE_IMAGE
    )

    # 框出上扬的嘴角并对笑脸打上Smile标签
    for (x2, y2, w2, h2) in smile:
        cv2.rectangle(roi_color, (x2, y2), (x2+w2, y2+h2), (255, 0, 0), 2)
        cv2.putText(img,'Smile',(x,y-7), 3, 1.2, (0, 255, 0), 2, cv2.LINE_AA)
        flag = 1

# cv2.imshow('Smile?', img)     # 展示图片
#cv2.imwrite("smile.jpg",img)  
# c = cv2.waitKey(0)            # 等待按键关闭图片
print(flag)

```

[参考代码](https://github.com/LiuXiaolong19920720/smile-detection-Python)

#### 3.2. 前端图片传给后端

**实现方法：**

前端 `index.js` 中在上传图片功能 `wx.chooseImage` 的 `success` 回调函数内

使用 `wx.uploadFile` 方法上传图片

```
//上传图片
wx.uploadFile({
    url: 'http://127.0.0.1:8360/smile/upload',
    filePath: that.data.smileImage,
    name: 'image',
    formData: {
        'userId': 10001
    },
    success(res) {
        const data = res.data
        console.log(res);
    }
})
```

Thinkjs后端在控制器中添加自己的控制文件，接收前端传来的图片

```
// src/controller/smile.js

const Base = require("./base.js");
const fs = require('fs');
const path = require('path');
const rename = think.promisify(fs.rename, fs);
module.exports = class extends think.Controller {
    async uploadAction(){
        const file = this.file('image');
        // 如果上传的是 png 格式的图片文件，则移动到其他目录
        if (file) {
            const filepath = path.join(think.ROOT_PATH, `www/static/upload/fake_smile.png`);
            think.mkdir(path.dirname(filepath));
            await rename(file.path, filepath);
        }
    }

    async downloadAction(){
        const filepath = path.join(think.ROOT_PATH, 'smile.png');
        ctx.download(filepath);
    }
}
```

保存到 `www/static/upload` 目录中，作为参数被后续脚本文件调用。

#### 3.3. 后端将图片作为参数传递给 python 脚本执行

在以上 `smile.js` 的`if`判断后面加上如下代码

**传参并调用**python脚本文件

```
// smile.js

const spawn = require('child_process').spawnSync;
const ls = spawn('python', ['smile.py', filepath]);
```

`python`脚本文件导入`sys`模块获取传递的参数

```
// smile.py

import sys
img = cv2.imread(sys.argv[1])
```

#### 3.4. 后端返回结果给前端显示

在 `smile.js` 的 `if` 判断后加上如下代码

获取`python`脚本的输出并回传给前端

```
// ls.stdout获取的是一个buffer格式的数值，需转换成字符串
var smile_value = ls.stdout.toString();   

// 这里返回一个json格式的数值
return this.json({smile_value});
```

此时，前端应该可以收到一个json格式的数值

```
data: "{"smile_value": "1"}"    //微笑为1，没微笑为0
```

![avatar](../image/return_value.png)

