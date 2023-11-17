---
layout:     post
title:      python-requests模块
subtitle:   
date:       2023-11-17
author:     Zeng Liwen
header-img: 
catalog: 	 true
tags:
    - work
---

[toc]

### requests

参考博客：[python中requests库使用方法详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/137649301)、[Python之requests模块-session - 酌三巡 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhuosanxun/p/12679121.html)

官网文档：[Requests: HTTP for Humans™ — Requests 2.31.0 documentation](https://requests.readthedocs.io/en/latest/)

#### 一、定义

​	request是用python语言编写，基于urlilib，采用Apache2 Licensed开源协议的HTTP库。Python实现的简单易用的HTTP库。

#### 二、安装&使用

`pip install request` `import request`

#### 三、背景

项目中需要对对给定地址的网页内容做解析，提取数据，主要用到了request.get方法。

![](/img/2023-11-17/request使用1.jpg)

#### 四、初步了解

```
import request

print(dir(requests))
'''
['ConnectTimeout', 'ConnectionError', 'DependencyWarning', 'FileModeWarning', 'HTTPError', 'NullHandler', 'PreparedRequest', 'ReadTimeout', 'Request', 'RequestException', 'RequestsDependencyWarning', 'Response', 'Session', 'Timeout', 'TooManyRedirects', 'URLRequired', '__author__', '__author_email__', '__build__', '__builtins__', '__cached__', '__cake__', '__copyright__', '__description__', '__doc__', '__file__', '__license__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '__title__', '__url__', '__version__', '_check_cryptography', '_internal_utils', 'adapters', 'api', 'auth', 'certs', 'chardet', 'check_compatibility', 'codes', 'compat', 'cookies', 'delete', 'exceptions', 'get', 'head', 'hooks', 'logging', 'models', 'options', 'packages', 'patch', 'post', 'put', 'request', 'session', 'sessions', 'status_codes', 'structures', 'urllib3', 'utils', 'warnings']
'''
```

#### 五、知识点明晰

- 请求

  ```
  import requests
  requests.post(url)
  requests.put(url)
  requests.delate(url)
  requests.head(url)
  requests.get(url)
  ```

  **get请求**

  ```
  import requests
  
  # 基本请求
  response1 = requests.get('http://httpbin.org/get')
  # 携带参数
  response2 = requests.get("http://httpbin.org/get?name=germey&age=22")
  data = {
   'name': 'germey',
   'age': 22
  }
  response3 = requests.get("http://httpbin.org/get", params=data)
  
  # 添加headers 
  headers = {
   'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
  }
  response = requests.get("发现 - 知乎", headers=headers)
  print(response.text)
  ```

  **post请求**

  ```
  import requests
  data = {'name': 'germey', 'age': '22'}
  response = requests.post("http://httpbin.org/post", data=data)
  print(response.text)
  ```

  **返回值解析**

  ```
  # 解析json
  import requests
  import json
   
  response = requests.get("http://httpbin.org/get")
  print(type(response.text))
  print(response.json())
  print(json.loads(response.text))
  print(type(response.json()))
  
  # 解析二进制数据
  response = requests.get("https://github.com/favicon.ico")
  print(type(response.content))
  print(response.content)
  ```

- 响应

  **responese属性**

  ```
  import requests
  
  response = requests.get(url)
  print(type(response.status_code), response.status_code) # 常用
  print(type(response.headers), response.headers)
  print(type(response.cookies), response.cookies)
  print(type(response.url), response.url)
  print(type(response.history), response.history)
  ```

- 高级操作

  - 文件上传

    ```
    import requests
    
    files = {'file': open('cookie.txt', 'rb')} 
    response = requests.post(url,files=files)
    print(response.text)
    ```

  - 获取cookie

    当需要cookie时，直接调用response.cookie

    ```
    import requests
    
    response = requests.get(url)
    for key, value in response.cookies.items():
    	print(key+"="+value)
    ```

  - 会话维持、模拟登录

    如果某个响应中包含一些Cookie，你可以快速访问它们

    ```
    import requests
    
    response = requests.get(url)
    print(response.cookies['NID'])
    print(tuple(r.cookies))
    ```

    要想发送你的cookies到服务器，可以使用cookies参数：

    ```
    import requests
     
    url = 'http://httpbin.org/cookies'
    cookies = {'testCookies_1': 'Hello_Python3', 'testCookies_2': 'Hello_Requests'}
    # 在Cookie Version 0中规定空格、方括号、圆括号、等于号、逗号、双引号、斜杠、问号、@，冒号，分号等特殊符号都不能作为Cookie的内容。
    r = requests.get(url, cookies=cookies)
    print(r.json())
    ```

  - 证书验证(存疑)

    因为12306有一个错误证书，我们那它的网站做测试会出现下面的情况，证书不是官方证书，浏览器会识别出一个错误

    ```text
    import requests
     
    response = requests.get('中国铁路12306')
    print(response.status_code)
    ```

    返回值：

    怎么正常进入这样的网站了，代码如下：

    ```text
    import requests
    from requests.packages import urllib3
    urllib3.disable_warnings()
    response = requests.get('中国铁路12306', verify=False)
    print(response.status_code)
    将verify设置位False即可，返回的状态码为200
    ```

    urllib3.disable_warnings()这条命令主要用于消除警告信息

  - 代理设置

    在进行爬虫爬取时，有时爬虫会被服务器给屏蔽调，这时采用的方法主要有降低访问时间，通过代理ip访问，如下：

    ```
    import requests
    
    proxies = {
     "http": "http://127.0.0.1:9743",
     "https": "https://127.0.0.1:9743",
    }
     
    response = requests.get("https://www.taobao.com", proxies=proxies)
    print(response.status_code)
    ```

    ip可以从网上抓取，或者某宝购买

    如果代理需要设置账户名和密码,只需要将字典更改为如下：

    ```text
    proxies = {
    "http":"http://user:password@127.0.0.1:9999"
    }
    如果你的代理是通过sokces这种方式则需要pip install "requests[socks]"
    proxies= {
    "http":"socks5://127.0.0.1:9999",
    "https":"sockes5://127.0.0.1:8888"
    }
    ```

  - 超时设置

    访问有些网站时可能会超时，这时设置好timeout就可以解决这个问题

    ```text
    import requests
    from requests.exceptions import ReadTimeout
    try:
     response = requests.get("http://httpbin.org/get", timeout = 0.5)
     print(response.status_code)
    except ReadTimeout:
     print('Timeout')
    ```

    正常访问，状态吗返回200

  - 认证设置

    如果碰到需要认证的网站可以通过requests.auth模块实现

    ```text
    import requests
     
    from requests.auth import HTTPBasicAuth
    response = requests.get("http://120.27.34.24:9001/",auth=HTTPBasicAuth("user","123"))
    print(response.status_code)
    ```

    当然这里还有一种方式

    ```text
    import requests
     
    response = requests.get("http://120.27.34.24:9001/",auth=("user","123"))
    print(response.status_code)
    ```

  - 异常处理

    遇到网络问题（如：DNS查询失败、拒绝连接等）时，Requests会抛出一个ConnectionError 异常。

    遇到罕见的无效HTTP响应时，Requests则会抛出一个 HTTPError 异常。

    若请求超时，则抛出一个 Timeout 异常。

    若请求超过了设定的最大重定向次数，则会抛出一个 TooManyRedirects 异常。

    所有Requests显式抛出的异常都继承自 requests.exceptions.RequestException 。

#### 六、运用

使用requests模拟用户登录登录

```
# -*- coding: utf-8 -*-

import requests

# python2 和 python3的兼容代码
try:
    # python2 中
    import cookielib
    print(f"user cookielib in python2.")
except:
    # python3 中
    import http.cookiejar as cookielib
    print(f"user cookielib in python3.")

# session代表某一次连接
mafengwoSession = requests.session()
# 因为原始的session.cookies 没有save()方法，所以需要用到cookielib中的方法LWPCookieJar，这个类实例化的cookie对象，就可以直接调用save方法。
mafengwoSession.cookies = cookielib.LWPCookieJar(filename = "mafengwoCookies.txt")

userAgent = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36"
header = {
    # "origin": "https://passport.mafengwo.cn",
    "Referer": "https://passport.mafengwo.cn/",
    'User-Agent': userAgent,
}


# 马蜂窝模仿 登录
def mafengwoLogin(account, password):
    print("开始模拟登录马蜂窝")
    postUrl = "https://passport.mafengwo.cn/login/"
    postData = {
        "passport": account,
        "password": password,
    }
    # 使用session直接post请求
    responseRes = mafengwoSession.post(postUrl, data = postData, headers = header)
    # 无论是否登录成功，状态码一般都是 statusCode = 200
    print(f"statusCode = {responseRes.status_code}")
    print(f"text = {responseRes.text}")
    # 登录成功之后，将cookie保存在本地文件中，好处是，以后再去获取马蜂窝首页的时候，就不需要再走mafengwoLogin的流程了，因为已经从文件中拿到cookie了
    mafengwoSession.cookies.save()


# 通过访问个人中心页面的返回状态码来判断是否为登录状态
def isLoginStatus():
    routeUrl = "http://www.mafengwo.cn/plan/route.php"
    # 下面有两个关键点
        # 第一个是header，如果不设置，会返回500的错误
        # 第二个是allow_redirects，如果不设置，session访问时，服务器返回302，
        # 然后session会自动重定向到登录页面，获取到登录页面之后，变成200的状态码
        # allow_redirects = False  就是不允许重定向
    responseRes = mafengwoSession.get(routeUrl, headers = header, allow_redirects = False)
    print(f"isLoginStatus = {responseRes.status_code}")
    if responseRes.status_code != 200:
        return False
    else:
        return True


if __name__ == "__main__":
	# 第一步：尝试使用已有的cookie登录
    mafengwoSession.cookies.load()
    isLogin = isLoginStatus()
    print(f"is login mafengwo = {isLogin}")
    if isLogin == False:
        # 第二步：如果cookie已经失效了，那就尝试用帐号登录
        print(f"cookie失效，用户重新登录...")
        mafengwoLogin("13756567832", "000000001")

    resp = mafengwoSession.get("http://www.mafengwo.cn/plan/fav_type.php", headers = header, allow_redirects = False)
    print(f"resp.status = {resp.status_code}")

```

工作中用的到网页爬虫，登录和非登录网页内容不完全一致

```
import os
import requests
from lxml import etree
from bs4 import BeautifulSoup,NavigableString

class Crawler():

    def __init__(self, base_url):
        self.header = {
            "Content-Type": "application/x-www-form-urlencoded",
            "User-Agent": 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36',
            }
        self.session = requests.Session()
        self.base_url = base_url

    def login(self, username, password):
        resp = self.session.get("{}/koji".format(self.base_url),
                                headers=self.header)
        html = etree.HTML(resp.text)
        obj = html.xpath('//*[@id="kc-form-login"]')

        url = obj[0].attrib['action']
        resp = self.session.post(url,
                                data={"username": username,
                                      "password": password,
                                      "credentialId": ""}, headers=self.header)
        if username in resp.text and "logout" in resp.text:
            return True
        elif "login" in resp.text:
            resp = self.session.get("{}/koji/login".format(self.base_url))
            if username in resp.text and "logout" in resp.text:
                return True
            else:
                return False
        else:
            return False
    
    def search(self, keyword):
        resp = self.session.get("{}/koji/search".format(self.base_url),
                                params={"match": "glob", "type": "package", "terms": keyword})
        return resp.text
        # 处理搜索结果

    def get_content_as_bs4(self,url):
        resp = self.session.get(url)
        htmlStr = BeautifulSoup(resp.text,"html.parser")
        return htmlStr

    def get_content_as_str(self,url):
        resp = self.session.get(url)
        return resp.text

    def close(self):
        self.session.close()

if __name__ == "__main__":
    craw = Crawler("http://172.30.12.238")
    craw.login("zengliwen", "1115@Zlw")
    # craw.search("cairo")

```