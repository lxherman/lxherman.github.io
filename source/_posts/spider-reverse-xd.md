---
title: 逆向记录-小电充电微信小程序sign参数破解
tags:
  - 爬虫
  - 逆向
categories:
  - 爬虫
date: 2020-01-07 21:48:09
top: 101
---
>&#160; &#160; &#160; &#160;app逆向需要脱壳、反编译等操作从而获取源码。同理，微信小程序的也需要经过一系列处理才能获取源码，再通过源码寻找加密逻辑，利用python复现该接口的加密、防篡改方式，即可完成逆向。
>&#160; &#160; &#160; &#160;本文主要分为两个部分：1、如何找到获取小程序源码 &#160; &#160;2、寻找加密函数以及复现过程
<!--more-->

{% note danger %}
郑重声明：
本项目的所有代码和相关文章， 仅用于经验技术交流分享，禁止将相关技术应用到不正当途径，因为滥用技术产生的风险与本人无关。如有侵权，请联系作者删除。
{% endnote %}

# 逆向背景
## 缘起

出于对能打脸王校长的行业的好奇和职业本能，我决定看一看充电宝小程序的数据和接口长什么样。本案例就选择小电充电宝作为分析对象。

## 分析经过
通过mitmproxy + 夜神模拟器抓包以后。我把请求提出来在postman上测试，效果如下
<center>
<img src="https://s2.ax1x.com/2020/01/08/l2CuxH.png" width=80% />
</center>

但是我发现一个问题，一旦我隔一段时间再去访问这个接口，或者我稍微改变了请求的参数（如定位的经纬度），服务器就会返回：`{"success":false,"code":1,"msg":"服务器开了小差请稍后重试"}`

接口具体是长什么样的呢？
```
GET https://b.dian.so/lhc/2.0/h5/shop/gets?sign=5d8260aeee65754610ee5528f6a21bcd&t=1578466598885&appType=0&data={"keywords":"","deviceType":0,"latitude":22.553314,"longitude":114.115462,"offset":0,"pageSize":100,"userLatitude":22.542807074652778,"userLongitude":114.05800754123264,"priShopType":0,"secShopType":0}
```
把他拆开，连接分为三个部分：
- 接口部分：该接口是用于返回附近商家的列表。
  `https://b.dian.so/lhc/2.0/h5/shop/gets`
- 请求参数1：包含了sign（签名）参数、请求发生的时间戳（毫秒）、appType
  `sign=5d8260aeee65754610ee5528f6a21bcd&t=1578466598885&appType=0`
- 请求参数2：可以理解为请求体，内有待返回的设备型号、坐标、用户坐标等。
   `data={"keywords":"","deviceType":0,"latitude":22.553314,"longitude":114.115462,"offset":0,"pageSize":100,"userLatitude":22.542807074652778,"userLongitude":114.05800754123264,"priShopType":0,"secShopType":0}`

经过推测，原因就是接口加上了sign防止篡改请求。为了验证我的假设，我决定看看小程序的代码到底是怎么写的。

# 解析wxapkg文件获取源码
*这里要用到adb工具，它可以在命令行与客户端进行通信。没配置的建议百度一下。*
## 首先连接夜神模拟器
```linux
adb connect 127.0.0.1:62001
adb shell
```
**注意，每个模拟器、客户端连接的初始端口号不一样**
>夜神模拟器：62001
>MUMU模拟器：7555
>逍遥模拟器：21503
成功连接如图
<center>
<img src="https://s2.ax1x.com/2020/01/08/l29hUf.png" width=80% />
</center>

## 找到小程序所在的文件目录
一般来说，微信小程序的目录都存放在下面这个路径，cd进去就好了
```linux
cd /data/data/com.tencent.mm/MicroMsg/326687790a68a9f2b4338b778052b9a2/appbrand/pkg 
```

但是这里有个问题：`326687790a68a9f2b4338b778052b9a2` 是用户的文件夹，如果登录过多个微信号，我们怎么才能判断进入到哪个文件夹下面呢？MicroMsg文件夹下大概长这样
<center>
<img src="https://s2.ax1x.com/2020/01/08/l297vj.png" width=80% />
</center>
解决方法：我个人是通过重新登录这个账号，然后对比文件夹的修改时间与我登录的时间，找到时间最接近的那个，自然就对应一个微信号了。

进入到pkg文件夹后可以看到

<center>
<img src="https://s2.ax1x.com/2020/01/08/l29bKs.png" width=80% />
</center>

问题又来了，哪个才是小电小程序的包？
类比上一个问题的解决方法，我在模拟器上重新删除小程序，再添加回来，再从pkg文件夹下找到与添加小程序的时间最接近的那个，就肯定是我们要拿到的小程序包了，这里是 `_-2089721779_438.wxapkg`

<center>
<img src="https://s2.ax1x.com/2020/01/08/l2eDeg.png" width=80% />
</center>

## 解wxapkg包
夜神模拟器和电脑之间有一个共享文件夹
PC端路径：`C:\Users\Administrator\Nox_share`
模拟器端路径：`/mnt/shared`
使用cat命令吧wxapkg包复制一份在这个文件夹内，就可以在电脑上看到它了。

```linux
cat _-2089721779_438.wxapkg >/mnt/shared/xd.wxapkg
```

下面需要用工具解析这个包的内容，我这里使用的是WxApkgUnpacker解包器。
我把他放在[**百度网盘**](https://pan.baidu.com/s/1ej92Ioq3tQ9tCwkr0-cSWQ)里了（提取码：`0chj`）

<center>
<img src="https://s2.ax1x.com/2020/01/08/l29IPS.png" width=50% />
</center>

把wxapkg包解析后放入文件夹下，就大工告成了。

# 寻找加密函数并用python重写

我们来看看解析后的文件夹里有什么

<center>
<img src="https://s2.ax1x.com/2020/01/08/l29fVP.png" width=80% />
</center>

下面我们验证一下，能不能在这些文件内找到之前提到的接口信息。
进入到`app-service.js`中，以`h5/shop/gets`为关键字检索一下这个文件，看我们发现了什么

<center>
<img src="https://s2.ax1x.com/2020/01/08/l2QnWd.png" width=100% />
</center>

如上所示，sign、时间戳、appType都在这个函数里了。

>*BTW，刚开始打开`app-service.js`时，文件内容很乱，在vscode中我们可以使用`Beautify`这个插件美化一下代码，更易于阅读。*

<center>
<img src="https://s2.ax1x.com/2020/01/08/l29458.png" width=50% />
</center>


剩下的就是**耐心**阅读js代码。这里就不贴python复现后的代码了，大概说下签名的思路。（如有完整复现过程的需要，大家可以联系我补充：wx : `lxh_Herman`）

小电充电小程序的sign加密还是比较简单常规的
- 把各个参数进行一个sort排序
- 序列化，二进制编码
- md5加密
最后在请求中带上md5加密过后的字符串放到sign参数中就可以了。

用postman测试下我们构造出来的url，发现可以返回正确的商户数据，至此，小电小程序sign参数逆向全部完成。


