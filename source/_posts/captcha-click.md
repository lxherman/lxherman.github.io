---
title: 反爬必修课之----(4)点触验证码识别
tags:
  - 爬虫
  - 验证码
categories:
  - 爬虫
top: 89
date: 2019-08-15 10:42:38
---

>&#160; &#160; &#160; &#160;验证码识别成为了对抗反爬虫的必修课之一，看了崔庆才著的《python3网络爬虫开发实战》后受益匪浅，本专题将着重学习记录不同的验证码识别方式：图像验证码、宫格验证码、极验滑动验证码、点触验证码。
<!--more-->
___ 

## <center>点触验证码识别</center>

先看看效果：

![点触.gif](https://upload-images.jianshu.io/upload_images/16325133-4f30f869602c7514.gif?imageMogr2/auto-orient/strip)


下图为12306的点触验证码，识别难点有两个:

1. 文字识别，即首先要识别出“盘子”二字，但往往该文字经过了一系列的变换，如果借助第一节提到的OCR技术，基本上无法识别。

2. 将图片转化为相应的文字。

![](https://upload-images.jianshu.io/upload_images/16325133-76722036277145e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
另一种验证码：

![](https://upload-images.jianshu.io/upload_images/16325133-6830ff513c192548.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（ 和上述验证码相比较而言，会简单一些，但是识别思路相同，本文就着重记录一下第一种验证码的识别过程。）
### 识别思路
&#160; &#160; &#160; &#160;点触验证码难度较大，我们可以接触互联网上一些验证码服务平台，他们提供7×24小时的服务，并且识别准确率在90%以上。
&#160; &#160; &#160; &#160;大致流程是将验证码图片按照字节流的形式传递给服务平台，平台几秒钟后将会返回一个带有坐标信息的字段，该坐标信息就是我们需要点击的位置。按照得到的坐标信息进行点击，便可以完成识别。
&#160; &#160; &#160; &#160;验证码服务平台往往提供了多种类型的验证码识别，例如“超级鹰”网址：https://www.chaojiying.com/，其提供了数字、中英文、特殊字符、数学计算和坐标选择识别等多种验证码类型的识别。点触验证码识别就是坐标选择识别的一种。下面将利用该平台进行识别。
### 详细过程及代码
#### 获取超级鹰python API
```python
import requests
from hashlib import md5

class Chaojiying_Client(object):

    def __init__(self, username, password, soft_id):
        self.username = username
        self.password = md5(password.encode('utf-8')).hexdigest()
        self.soft_id = soft_id
        self.base_params = {
            'user': self.username,
            'pass2': self.password,
            'softid': self.soft_id,
        }
        self.headers = {
            'Connection': 'Keep-Alive',
            'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
        }

    def PostPic(self, im, codetype):
        """
        im: 图片字节
        codetype: 题目类型 参考 http://www.chaojiying.com/price.html
        """
        params = {
            'codetype': codetype,
        }
        params.update(self.base_params)
        files = {'userfile': ('ccc.jpg', im)}
        r = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, files=files, headers=self.headers)
        return r.json()

    def ReportError(self, im_id):
        """
        im_id:报错题目的图片ID
        """
        params = {
            'id': im_id,
        }
        params.update(self.base_params)
        r = requests.post('http://upload.chaojiying.net/Upload/ReportError.php', data=params, headers=self.headers)
        return r.json()


if __name__ == '__main__':
	chaojiying = Chaojiying_Client('超级鹰用户名', '超级鹰用户名的密码', '96001')	#用户中心>>软件ID 生成一个替换 96001
	im = open('a.jpg', 'rb').read()											#本地图片文件路径 来替换 a.jpg 有时WIN系统须要//
	print(chaojiying.PostPic(im, 1902))										#1902 验证码类型  官方网站>>价格体系 3.4+版 print 后要加()

```
&#160; &#160; &#160; &#160;代码如上，其定义了一个Chaojiying_Client的类，内有三个参数，分别是超级鹰的用户名、密码、软件ID。内含两个函数 PostPic()、ReportError()。

&#160; &#160; &#160; &#160;*PostPic()需要传入图片对象和验证码的代号（验证码类型）。该方法会将图片对象发送给给超级鹰的后台进行识别，然后返回一个JSON。*

&#160; &#160; &#160; &#160;*ReportError()发生错误时的回调。*

#### 验证过程
##### 初始化、登录
```python
import time
from io import BytesIO
from PIL import Image
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from chaojiying import Chaojiying_Client
#账号密码自行设定
EMAIL = '12306账号'
PASSWORD = '12306密码'
CHAOJIYING_USERNAME = '超级鹰账号'
CHAOJIYING_PASSWORD = '超级鹰密码'

CHAOJIYING_SOFT_ID = 898443
CHAOJIYING_KIND = 9202

class Crackclick():
    def __init__(self):
        self.url = 'https://kyfw.12306.cn/otn/resources/login.html'
        self.browser = webdriver.Chrome()
        self.wait = WebDriverWait(self.browser,20)
        self.email = EMAIL
        self.password = PASSWORD
        self.chaojiying = Chaojiying_Client(CHAOJIYING_USERNAME,CHAOJIYING_PASSWORD,CHAOJIYING_SOFT_ID)

    def open(self):
        '''
        打开网页,切换验证模式
        '''
        self.browser.get(self.url)
        account = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME,'login-hd-account')))
        account.click()
        email = self.wait.until(EC.presence_of_element_located((By.ID,'J-userName')))
        password = self.wait.until(EC.presence_of_element_located((By.ID,'J-password')))
        email.send_keys(self.email)
        password.send_keys(self.password)

    def refresh_code(self):
        '''
        刷新验证码
        '''
        button = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME,'lgcode-refresh')))
        return button

```
##### 获取验证码图片
&#160; &#160; &#160; &#160;**该截图操作在各类验证码识别中会经常使用**
```python
    def get_element(self):
        '''获取验证图片对象'''
        element = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME,'loginImg')))
        return element

    def get_position(self):
        '''获取验证码位置'''
        element = self.get_element()
        time.sleep(2)
        location = element.location
        size = element.size
        top,bottom,left,right = location['y'],location['y']+size['height'],location['x'],location['x']+size['width']
        return (top,bottom,left,right)

    def get_screenshot(self):
        '''获取网页截图'''
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        return screenshot

    def get_image(self,name='captcha.png'):
        '''获取验证码图片'''
        top,bottom,left,right = self.get_position()
        print('验证码位置',top,bottom,left,right)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop((left,top,right,bottom))
        return captcha

```
##### 解析识别信息
```python
    def get_points(self, captcha_result):
        '''
        解析识别信息
        '''
        groups = captcha_result.get('pic_str').split('|')
        locations = [[int(number) for number in group.split(',')] for group in groups]
        return locations

```
平台返回的JSON格式信息如下：
```
{
'err_no': 0, 
'err_str': 'OK', 
'pic_id': '8056217061965800013', 
'pic_str': '29,72|257,71',
'md5': '8de76ac4036ebab2b1e05375c1bf84b8'
}
```
&#160; &#160; &#160; &#160;get_points()方法即是将'pic_str'中的坐标信息格式化为：`[29, 72]   [257, 71]`
##### 点击验证图片
```python
    def touch_click_words(self,locations):
        '''
        点击验证码
        '''
        for location in locations:
            print(location)
            ActionChains(self.browser).move_to_element_with_offset(self.get_element(),location[0],
                                                                     location[1]).click().perform()
        time.sleep(1)

```
&#160; &#160; &#160; &#160;其中move_to_element_with_offset函数的参数及解释：
![](https://upload-images.jianshu.io/upload_images/16325133-7122e5bae02808db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 点击登录
```python
    def login(self):
        '''
        点击登录按钮
        '''
        button = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME,'login-btn')))
        button.click()
        time.sleep(10)

```
##### 控制流程
&#160; &#160; &#160; &#160;首先打开12306网站，并刷新验证码(button)，然后将验证码图片对象按照字节流的格式传递给超级鹰平台，并将返回的locations信息传递给touch_click_words()函数，最后完成验证码识别。
```python
    def crack(self):
        '''入口'''
        self.open()
        button = self.refresh_code()
        button.click()
        image = self.get_image()
        bytes_array = BytesIO()
        image.save(bytes_array,format='PNG')
        #识别验证码
        result = self.chaojiying.PostPic(bytes_array.getvalue(),CHAOJIYING_KIND)
        print(result)
        locations = self.get_points(result)
        self.touch_click_words(locations)
        self.login()
    

if __name__ == "__main__":
    crack = Crackclick()
    crack.crack()

```
___
## 关键字总结
1. 获取图片位置、截取图片
2. 调用超级鹰API
3. ActionChains
4. 字节流处理


