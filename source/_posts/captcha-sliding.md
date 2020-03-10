---
title: 反爬必修课之----(3)极验滑动验证码识别
tags:
  - 爬虫
  - 验证码识别
categories:
  - 爬虫
date: 2019-06-01 10:42:09
---
>&#160; &#160; &#160; &#160;验证码识别成为了对抗反爬虫的必修课之一，看了崔庆才著的《python3网络爬虫开发实战》后受益匪浅，本专题将着重学习记录不同的验证码识别方式：图像验证码、宫格验证码、极验滑动验证码、点触验证码。
<!--more-->
***
&#160; &#160; &#160; &#160;[2019.3.6更新] 测试了下发现以下的版本识别率比较低，稍微改动了下代码。
&#160; &#160; &#160; &#160;放到了[GitHub](https://github.com/lxherman/SlidingCaptcha)上
***

## <center>极验滑动验证码识别</center>

先看看效果：

![](https://upload-images.jianshu.io/upload_images/16325133-9cc840f151f9b830.gif?imageMogr2/auto-orient/strip)
### 识别思路
1. 模拟点击切换为滑动验证、并显示验证界面。
2. 识别滑动缺口的位置，计算位移
3. 模拟拖动滑块
4. 若认证失败，重复调用

### 详细过程及代码

#### 初始化
```python
from selenium.webdriver.support import expected_conditions as EC    
from selenium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver import ActionChains
from PIL import Image
from io import BytesIO
import time

BORDER = 6

class CrackGeetest():
    def __init__(self):
        self.url = 'https://www.geetest.com/type/'
        self.browser = webdriver.Chrome()
        self.wait = WebDriverWait(self.browser,10)

    def open(self):
        '''
        打开网页
        :return None 
        '''
        self.browser.get(self.url)

    def close(self):
        '''
        关闭网页
        :return None
        '''
        self.browser.close()
        self.browser.quit()

```
&#160; &#160; &#160; &#160;定义了一个 CrackGeetest 类，初始化selenium对象和一些参数配置，网址是极验的验证码测试页面。
#### 模拟点击
&#160; &#160; &#160; &#160;首先模拟点击切换为滑动验证，然后模拟点击弹出验证图片。
```python
    def change_to_slide(self):
        '''
        切换为滑动认证
        :return 滑动选项对象
        '''
        huadong = self.wait.until(
            EC.element_to_be_clickable((By.CSS_SELECTOR,'.products-content ul > li:nth-child(2)'))
        )
        return huadong

    def get_geetest_button(self):
        '''
        获取初始认证按钮
        :return 按钮对象
        '''
        button = self.wait.until(
            EC.element_to_be_clickable((By.CSS_SELECTOR,'.geetest_radar_tip'))
        )
        return button

```
&#160; &#160; &#160; &#160;该步骤定义了两个方法，均利用显示等待的方法实现。并返回按钮对象，后用click()方法模拟点击。
效果如下：

![](https://upload-images.jianshu.io/upload_images/16325133-9a598c31d166652a.gif?imageMogr2/auto-orient/strip)

#### 获取背景图
&#160; &#160; &#160; &#160;首先等待验证码加载完成(wait_pic)，获取网页截图(get_screenshot)，然后获取验证背景图所在的位置及大小参数(get_position)和滑块对象(get_slider)。
```python
    def wait_pic(self):
        '''
        等待验证图片加载完成
        :return None
        '''
        self.wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR,'.geetest_popup_wrap'))
        ) 

    def get_screenshot(self):
        """
        获取网页截图
        :return: 截图对象
        """
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        return screenshot

    def get_position(self):
        '''
        获取验证码位置
        :return: 位置元组
        '''
        img = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME,'geetest_canvas_img')))
        time.sleep(2)
        location = img.location
        size = img.size
        top, bottom = location['y'], location['y'] + size['height']
        left, right = location['x'], location['x'] + size['width'] 
        return (top, bottom, left, right) 

    def get_slider(self):
        '''
        获取滑块
        :return: 滑块对象
        '''
        slider = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME,'geetest_slider_button')))
        return slider

```
&#160; &#160; &#160; &#160;再通过上述返回的背景图位置和大小参数，对网页截图进行切片(get_geetest_image)，最后获取背景图。
```python
    def get_geetest_image(self,name='captcha.png'):
        '''
        获取验证码图片
        :return: 图片对象
        '''      
        top, bottom, left, right = self.get_position()
        print('验证码位置',top, bottom, left, right)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop((left, top, right, bottom))
        captcha.save(name)
        return captcha
```
 &#160; &#160; &#160; &#160;到这里，已经获取了带缺口的背景图，那怎么样才可以获取不带缺口滑块的原图呢？

&#160; &#160; &#160; &#160;网上提供的方法中，我筛选出了两种实测可行的方法，一种是通过改变CSS样式获得原图，另一种是将源码返回的乱序图还原，这里着重介绍第一种，另一种在日后的文章中将补充。

&#160; &#160; &#160; &#160;在《python3网络爬虫开发实战》中，由于写书的时间距今有一段时间，所以极验的验证码也存在更新，截至到今天（2018-11-30）无法直接先获取无缺口的原图，在通过点击获得带缺口背景图了。观察一下验证码页面的源代码，可以发现：

![](https://upload-images.jianshu.io/upload_images/16325133-12f84f029c3b03e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
&#160; &#160; &#160; &#160;将该canvas标签的style删除之后，滑块和阴影果然不见了，这样我们只需要通过js操作css样式属性，便可以继续分析了。这里用到了execute_script()方法，基本上Selenium API没有提供的功能都可以通过它执行JavaScript的方式来实现。不得不说这个功能还是挺好用的，可以类比Splash执行Lua脚本。
&#160; &#160; &#160; &#160;执行js脚本之后(delete_style)获得了无缺口的原图，再调用之前的截图方法，就可以获取同大小的背景图了。
```python
    def delete_style(self):
        '''
        执行js脚本，获取无滑块图
        :return None
        '''
        js = 'document.querySelectorAll("canvas")[2].style=""'
        self.browser.execute_script(js)

```
![不带缺口的背景图](https://upload-images.jianshu.io/upload_images/16325133-8965f91f7fc07e13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![带缺口和阴影的背景图](https://upload-images.jianshu.io/upload_images/16325133-66d6dc1de3aafd4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 识别缺口
&#160; &#160; &#160; &#160;我们得到了两张图，接下来就要对比他们来获取缺口位置。
&#160; &#160; &#160; &#160;遍历图片的每个坐标点，获取两张图片的RGB数据，若差距在一定范围内，则认为两个像素相同，继续往下比对。若超过一定范围，则代表像素点不同，当且位置即为缺口位置。&#160; &#160; &#160; &#160;is_pixel_equal()中定义了一个阈值范围threshold，为60，原因是缺口图中不仅有缺口部分的像素不同，其中还设置了一个干扰阴影块，和缺口大小类似，所以我们需要将范围适当提高。
```python
    def is_pixel_equal(self, img1, img2, x, y):
        '''
        判断两个像素是否相同
        :param img1: 不带缺口图片
        :param img2: 带缺口图
        :param x: 位置x
        :param y: 位置y
        :return: 像素是否相同
        '''
        # 取两个图片的像素点
        pix1 = img1.load()[x, y]
        pix2 = img2.load()[x, y]
        threshold = 60
        if abs(pix1[0] - pix2[0]) < threshold \
        and abs(pix1[1] - pix2[1]) < threshold \
        and abs(pix1[2] - pix2[2]) < threshold:
            return True
        else:
            return False

```
&#160; &#160; &#160; &#160;get_gap()方法遍历两张图片的每个像素，再利用is_pixel_equal()方法判断两张图片同一位置的像素。get_gap()中，left为起始横坐标，即是从滑块的右边开始寻找缺口位置。
```python
    def get_gap(self, img1, img2):
        '''
        获取缺口偏移量
        :param img1: 不带缺口图片
        :param img2: 带缺口图
        :return 缺口位置
        '''
        left = 60 
        for i in range(left, img1.size[0]):
            for j in range(img1.size[1]):
                if not self.is_pixel_equal(img1, img2, i, j):
                    left = i
                    return left
        return left

```
#### 模拟拖动
&#160; &#160; &#160; &#160;上面我们获得了滑块的位置，现在只需要计算距离并且模拟拖动即可。
&#160; &#160; &#160; &#160;不难想到，将滑块匀速运动或者直接闪到缺口位置是肯定不行的，我们要尽量模拟人手拖动鼠标的情况。所以我在崔大给的方案的基础上做了一点改进。

大致过程：
&#160; &#160; &#160; &#160;首先加速拖动滑块，当快接近缺口时，开始减速拖动，超过缺口一点距离后再往回拖拽对齐，由于人手可能不能对得非常整齐，所以我设置了1到2个像素的误差，并且再最后加入了3个像素距离的左右滑动来模拟释放鼠标时的抖动情况。

效果如下：

![](https://upload-images.jianshu.io/upload_images/16325133-95a2c6c341ffd0f9.gif?imageMogr2/auto-orient/strip)

利用中学时期的物理公式，即可构造轨迹移动算法。
&#160; &#160; &#160; &#160;`x = v0*t + 1/2*a*t^2`
&#160; &#160; &#160; &#160;`v = v0 + a*t`
```python
    def get_track(self, distance):
        '''
        根据偏移量获取移动轨迹
        :param distance: 偏移量
        :return: 移动轨迹
        '''
        #移动轨迹
        track = []
        #当前位移
        current = 0
        #减速阈值
        mid = distance * 3 / 5
        #计算间隔
        t = 0.2
        #初速度
        v = 0
        #滑超过过一段距离
        distance += 14
        while current < distance:
            if current < mid:
                #加速度为正
                a = 2
            else:
                #加速度为负
                a = -1.5
            #初速度 v0
            v0 = v
            #当前速度 v
            v = v0 + a * t
            #移动距离 move-->x
            move = v0 * t + 1 / 2 * a * t * t
            #当前位移
            current += move
            #加入轨迹
            track.append(round(move))
        return track

```
&#160; &#160; &#160; &#160;前3/5路程加速，后面减速，track返回的是一个列表，其中每个元素代表的是每次移动的距离。然后模拟释放鼠标时的人手抖动(shake_mouse)。
&#160; &#160; &#160; &#160;最后根据之前所得到的运动轨迹拖动滑块(move_to_gap)即可。
```python
    def shake_mouse(self):
        '''
        模拟人手释放鼠标时的抖动
        :return: None
        '''
        ActionChains(self.browser).move_by_offset(xoffset=-3, yoffset=0).perform()
        ActionChains(self.browser).move_by_offset(xoffset=2, yoffset=0).perform()

    def move_to_gap(self, slider, tracks):
        '''
        拖动滑块到缺口处
        :param slider: 滑块
        :param tracks: 轨迹
        :return
        '''
        back_tracks = [-1, -1, -2, -2, -3, -2, -2, -1, -1]
        ActionChains(self.browser).click_and_hold(slider).perform()
        #正向
        for x in tracks:
            ActionChains(self.browser).move_by_offset(xoffset=x, yoffset=0).perform()
        #逆向
        for x in back_tracks:
            ActionChains(self.browser).move_by_offset(xoffset=x, yoffset=0).perform()
        #模拟抖动
        self.shake_mouse()
        time.sleep(0.5)
        ActionChains(self.browser).release().perform()

```
其整个控制流程，如下：
&#160; &#160; &#160; &#160;执行主体流程，若验证失败， 则再次调用crack()进行识别，直至成功。
```python
    def crack(self):
        try:
            #打开网页
            self.open()
            #转换验证方式，点击认证按钮
            s_button = self.change_to_slide()
            s_button.click()
            g_button = self.get_geetest_button()
            g_button.click()
            #确认图片加载完成
            self.wait_pic()
            #获取滑块
            slider = self.get_slider()
            #获取带缺口的验证码图片
            image1 = self.get_geetest_image('captcha1.png')
            self.delete_style()
            image2 = self.get_geetest_image('captcha2.png')
            gap = self.get_gap(image1,image2)
            print('缺口位置',gap)
            gap -= BORDER
            track = self.get_track(gap)
            self.move_to_gap(slider, track)
            success =  self.wait.until(
                EC.text_to_be_present_in_element((By.CLASS_NAME,'geetest_success_radar_tip_content'),'验证成功')
            )
            print(success)
            time.sleep(5)
            self.close()
        except:
            print('Failed-Retry')
            self.crack()


    
if __name__ == '__main__':
    crack = CrackGeetest()
    crack.crack()

```
&#160; &#160; &#160; &#160;至此，极验滑动验证码识别——网页截图对比方法已经记录完毕。
___
## 关键字总结
1. webdriver():
&#160; &#160; &#160; &#160;support.expected_conditions as EC
&#160; &#160; &#160; &#160;support.wait.WebDriverWait
&#160; &#160; &#160; &#160;ActionChains
&#160; &#160; &#160; &#160;common.by.By
2. BytesIO
3. 代码风格
4. 验证码分析思路