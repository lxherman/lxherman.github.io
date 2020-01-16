---
title: 反爬必修课之----(1)图像验证码识别
tags:
  - 爬虫
  - 验证码识别
categories:
  - 爬虫
date: 2019-05-16 10:30:57
---
>&#160; &#160; &#160; &#160;验证码识别成为了对抗反爬虫的必修课之一，看了崔庆才著的《python3网络爬虫开发实战》后受益匪浅，本专题将着重学习记录不同的验证码识别方式：图像验证码、宫格验证码、极验滑动验证码、点触验证码。
<!--more-->
___
&#160; &#160; &#160; &#160;[3.20更新]如今的图像验证码日益复杂，混淆的参数太多。故OCR技术基本上已经无法满足现在的验证码识别需求，但是其在爬虫中仍然有用武之地。例如某些网站的价格信息是图片的形式，就可以利用tesserocr库方便的将其转化为文本。
&#160; &#160; &#160; &#160;[2019.12.30注]开通博客以后，逐一把简书上的内容迁移到本博客中。
___

## <center>图像验证码识别</center>

![](https://upload-images.jianshu.io/upload_images/16325133-7c5c5b1c3077dcc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&#160; &#160; &#160; &#160;图形验证码，一般由字母或数字组成。对于某些简单图形，利用OCR（Optical Character Recognition光学字符识别）技术就可以比较放方便的识别。OCR通过扫描字符，将其形状翻译成电子文本。所需要用到的是tesserocr。其是Python的一个OCR库，但其实是对tesseract做的一层Python APIde 封装，所以他的核心是tesseract。因此，在安装tesserocr之前，我们需要先安装tesseract。

### 识别测试
![code.jpg](https://upload-images.jianshu.io/upload_images/16325133-0067fb9c0c25ca2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码如下：
```python
import tesserocr
from PIL import Image

image = Image.open('code.jpg')
result = tesserocr.image_to_text(image)
print(result)
```
输出 : `M 8192‘、`

&#160; &#160; &#160; &#160;显然和图片内容不符，主要是因为多余的线条和点干扰了识别。所以还需对这种情况加以处理：转灰度、二值化等。

>&#160; &#160; &#160; &#160;二值图像(Binary Image)，按名字来理解只有两个值，0和1，0代表黑，1代表白，或者说0表示背景，而1表示前景。*

>&#160; &#160; &#160; &#160;灰度图只包含一个通道的信息，而彩色图通常包含三个通道的信息，这类图像通常显示为从最暗黑色到最亮的白色的灰度，与黑白图像不同，灰度图像在黑色与白色之间还有许多级的颜色深度。*

代码如下：
```python
import tesserocr
from PIL import Image

image = Image.open('code.jpg')
image = image.convert('L')  #转为灰度图像
image = image.convert('1'）#二值化
result = tesserocr.image_to_text(image)
print(result)
```
输出：`M8k2`,已经和符合图片上的信息了。

但若对于干扰较大的图片，如下：
![code2.jpg](https://upload-images.jianshu.io/upload_images/16325133-546fe58078eb62cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&#160; &#160; &#160; &#160;同样的操作，并未得到输出，我们可以指定二值化的阈值。（上述方法采用的是默认阈值127）

代码如下:
```python
import tesserocr
from PIL import Image

image = Image.open('code2.jpg')
image = image.convert('L')  #首先转化为灰度图像
threshold = 100   #二值化阈值
table = []
for i in range(256):
    if i < threshold:
        table.append(0)
    else:
        table.append(1)
image = image.point(table,'1')
image.show()
result = tesserocr.image_to_text(image)
print(result)
```
输出结果为：`1717`
得到的样式如下：
![](https://upload-images.jianshu.io/upload_images/16325133-553f4a45ca939eda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&#160; &#160; &#160; &#160;当阈值设置为100的时候，可以得到正确输出，而设置其他相差过大的阈值时（如80、180无输出，127时输出结果为：`，7`），可见阈值的选择也是很重要的。
***
## 总结
- python的tesserocr库可以通过扫描图片上的字符转化为文字输出，但有一定的局限性。

- 通过转化为灰度图像再进行适当的二值化处理，可以提高识别正确率。





