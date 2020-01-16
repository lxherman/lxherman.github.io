---
title: 反爬必修课之----(2)宫格验证码识别
tags:
  - 爬虫
  - 验证码
categories:
  - 爬虫
date: 2019-05-25 10:39:14
---
>&#160; &#160; &#160; &#160;验证码识别成为了对抗反爬虫的必修课之一，看了崔庆才著的《python3网络爬虫开发实战》后受益匪浅，本专题将着重学习记录不同的验证码识别方式：图像验证码、宫格验证码、极验滑动验证码、点触验证码。
<!--more-->
***
&#160; &#160; &#160; &#160;[2019.3.7更新]似乎宫格验证码被弃用了，换成了滑动验证码，那本文就当作日后遇到此类问题的一种思路吧~
***

## <center>微博宫格验证码识别</center>

先看看效果：

![](https://upload-images.jianshu.io/upload_images/16325133-28af7443ce1db823.gif?imageMogr2/auto-orient/strip)
### 识别思路
&#160; &#160; &#160; &#160;验证码包含四个宫格，每个宫格有都有一条连线经过，并且起点到终点的各个线段有箭头标识，不难计算出其一共有4\*3\*2\*1=24种组合方式。所以把每种验证码模板保存下来，再和待识别验证码比对，就可以判断其属于哪种路径。

模板如下图，按照路径将其重命名。
![](https://upload-images.jianshu.io/upload_images/16325133-3457ff9d513ab68f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 获取模板
&#160; &#160; &#160; &#160;先初始化微博账号密码和图片的保存路径（账号可以通过淘宝购买）。
```python
import os
import time
from io import BytesIO
from PIL import Image
from selenium import webdriver
from selenium.common.exceptions import TimeoutException
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from os import listdir

USERNAME = '15688215917'  #微博账号
PASSWORD = 'xgpf0g4w8'  #微博密码

TEMPLATES_FOLDER = 'templates/'

class CrackWeiboSlide():
    def __init__(self):
        self.url = 'https://passport.weibo.cn/signin/login?entry=mweibo&r=https://m.weibo.cn/'
        self.browser = webdriver.Chrome()
        self.wait = WebDriverWait(self.browser, 20)
        self.username = USERNAME
        self.password = PASSWORD
    
    def __del__(self):
        self.browser.close()
    
    def open(self):
        """
        打开网页输入用户名密码并点击
        :return: None
        """
        self.browser.get(self.url)
        username = self.wait.until(EC.presence_of_element_located((By.ID, 'loginName')))
        password = self.wait.until(EC.presence_of_element_located((By.ID, 'loginPassword')))
        submit = self.wait.until(EC.element_to_be_clickable((By.ID, 'loginAction')))
        username.send_keys(self.username)
        password.send_keys(self.password)
        submit.click()

    def get_position(self):
        """
        获取验证码位置
        :return: 验证码位置元组
        """
        try:
            img = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'patt-shadow')))
        except TimeoutException:
            print('未出现验证码')
            self.open()
        time.sleep(2)
        location = img.location
        size = img.size
        top, bottom, left, right = location['y'], location['y'] + size['height'], location['x'], location['x'] + size[
            'width']
        return (top, bottom, left, right)

    def get_image(self, name='captcha.png'):
        """
        获取验证码图片
        :return: 图片对象
        """
        top, bottom, left, right = self.get_position()
        print('验证码位置', top, bottom, left, right)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop((left, top, right, bottom))
        captcha.save(name)
        return captcha

    def get_all_captcha(self):
        '''
        获取批量验证码
        :return: 图片对象
        '''
        count = 0
        while True:
            self.open()
            self.get_image(str(count) + '.png')
            count += 1

if __name__ == '__main__':
    crack = CrackWeiboSlide()
    crack.get_all_captcha()

```
&#160; &#160; &#160; &#160;保存所有截图并进行整理过后，调用get_image()方法，得到验证码图片对象，和模板进行一一比对，若匹配成功，则将模板文件名转换为列表，如： 4231.png - - > [4,2,3,1]

### 比对方法
&#160; &#160; &#160; &#160;获取截图，与每张模板进行比对(detect_image)，遍历像素(is_pixe_equal)，如果相同，计数加 1 ，最后计算相同像素点占总像素的比例(same_image)，若为0.99以上，代表匹配成功，则返回拖动宫格的顺序(detect_image)。
```python
    def is_pixel_equal(self, image1, image2, x, y):
        """
        判断两个像素是否相同
        :param image1: 图片1
        :param image2: 图片2
        :param x: 位置x
        :param y: 位置y
        :return: 像素是否相同
        """
        # 取两个图片的像素点
        pixel1 = image1.load()[x, y]
        pixel2 = image2.load()[x, y]
        threshold = 30
        if abs(pixel1[0] - pixel2[0]) < threshold and abs(pixel1[1] - pixel2[1]) < threshold and abs(
                pixel1[2] - pixel2[2]) < threshold:
            return True
        else:
            return False
    
    def same_image(self, image, template):
        """
        识别相似验证码
        :param image: 待识别验证码
        :param template: 模板
        :return:
        """
        # 相似度阈值
        threshold = 0.99
        count = 0
        for x in range(image.width):
            for y in range(image.height):
                # 判断像素是否相同
                if self.is_pixel_equal(image, template, x, y):
                    count += 1
        result = float(count) / (image.width * image.height)
        if result > threshold:
            print('成功匹配')
            return True
        return False
    
    def detect_image(self, image):
        """
        匹配图片
        :param image: 图片
        :return: 拖动顺序
        """
        for template_name in listdir(TEMPLATES_FOLDER):
            print('正在匹配', template_name)
            template = Image.open(TEMPLATES_FOLDER + template_name)
            if self.same_image(image, template):
                # 返回顺序
                numbers = [int(number) for number in list(template_name.split('.')[0])]
                print('拖动顺序', numbers)
                return numbers
    
```
### 获取拖动顺序，据此拖动鼠标连接宫格
```python
    def move(self, numbers):
        """
        根据顺序拖动
        :param numbers: 拖动顺序
        :return:
        """
        # 获得四个按点
        circles = self.browser.find_elements_by_css_selector('.patt-wrap .patt-circ')
        dx = dy = 0
        for index in range(4):
            circle = circles[numbers[index] - 1]
            # 如果是第一次循环
            if index == 0:
                # 点击第一个按点
                ActionChains(self.browser) \
                    .move_to_element_with_offset(circle, circle.size['width'] / 2, circle.size['height'] / 2) \
                    .click_and_hold().perform()
            else:
                # 小幅移动次数
                times = 30
                # 拖动
                for i in range(times):
                    ActionChains(self.browser).move_by_offset(dx / times, dy / times).perform()
                    time.sleep(1 / times)
            # 如果是最后一次循环
            if index == 3:
                # 松开鼠标
                ActionChains(self.browser).release().perform()
            else:
                # 计算下一次偏移
                dx = circles[numbers[index + 1] - 1].location['x'] - circle.location['x']
                dy = circles[numbers[index + 1] - 1].location['y'] - circle.location['y']
    
    def crack(self):
        """
        破解入口
        :return:
        """
        self.open()
        # 获取验证码图片
        image = self.get_image('captcha.png')
        numbers = self.detect_image(image)
        self.move(numbers)
        time.sleep(10)
        print('识别结束')

if __name__ == '__main__':
    crack = CrackWeiboSlide()
    # crack.get_all_captcha()
    crack.crack()

```
输出结果如下：

![](https://upload-images.jianshu.io/upload_images/16325133-da9563589657c546.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
***
## 关键字总结
1. 模板匹配
2. os.listdir
3. 模拟鼠标（ActionChains类）




