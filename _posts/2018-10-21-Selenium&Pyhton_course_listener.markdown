---
layout:     post
title:      Python+Selenium实现自动听取慕课课程
subtitle:   ""
date:       2018-10-21 14:44:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Crawler
---
*这里实现的是超星在线学习平台上课程的自动听取，虽然最终成功了但是由于并不了解网站后台的监控机制，还是乖乖肉身听课吧（逃

------------


### 一、主体结构
由于实现的功能并不是十分复杂，这里只构造了一个AutoStudent类来完成自动听课，主要分为四个模块：

1. 用户登录
2. 选择课程
3. 检测尚未完成的小节
4. 自动听课

### 二、详细步骤
##### 1.__init__()
首先创建一个类并定义初始化函数，在此函数中定义类变量self.browser用于访问页面，self.courseList用于存储之后检测到的尚未完成的小节，self.db用于访问数据库。

为了避免Chromedriver的元素定位问题，要将浏览器窗口最大化

```python
        self.browser.maximize_window()
```
之后建立与数据库的连接并检测是否已创建表StudentInfo

```python
        self.db = pymysql.connect(host='localhost', user='root', password='Zwp0816...', db='ChaoXingStuInfo')
        self.cursor = self.db.cursor()
        self.cursor.execute('CREATE TABLE IF NOT EXISTS StudentInfo(账号 VARCHAR(30), 密码 VARCHAR(20));')
        self.db.commit()
```
至此，__init__()函数结束，以下为完整代码

```python
from selenium import webdriver
import pymysql

class AutoStudent():
    def __init__(self):
        self.browser = webdriver.Chrome()
        self.browser.maximize_window()
        self.courseList = list()

        self.db = pymysql.connect(host='localhost', user='root', password='*******', db='ChaoXingStuInfo')
        self.cursor = self.db.cursor()
        self.cursor.execute('CREATE TABLE IF NOT EXISTS StudentInfo(账号 VARCHAR(30), 密码 VARCHAR(20));')
        self.db.commit()
```
##### 2.login()
基本准备工作完成以后就可以用浏览器登录课程界面了。定义login()函数用以实现这个功能。

a).首先读取数据库中已存储的信息并显示，并让用户选择，如果不使用已有信息则提示输入，输入后询问是否将新的信息存入数据库

```python
        choice1 = ''
        while True:
            self.cursor.execute('SELECT * FROM StudentInfo')
            result = self.cursor.fetchall()
            if result:
                print("\n===================数据库中已有学生信息===================")
                for i in range(0, len(result)):
                    print(
                        "\n编号：" + str(i + 1) + "\t账号：" + result[i][0] + '\t密码：' + result[i][1])
                print('\n')
                choice1 = input("是否使用数据库中已有学生信息？(y/n) >>>")
                if choice1 == 'y':
                    index = int(input("输入要使用的信息编号：")) - 1
                    try:
                        account = result[index][0]
                        password = result[index][1]
                        break
                    except:
                        print("输入错误！")
                        continue
                else:
                    account = input("输入账号：")
                    password = input("输入密码：")
                    break
            else:
                account = input("输入账号：")
                password = input("输入密码：")
                break
        if choice1 != 'y':
            choice2 = input("\n是否将学生信息存入数据库？(y/n)")
            if choice2 == 'y':
                if choice1 != 'y':
                    sql = 'INSERT INTO StudentInfo VALUES(%s, %s)'
                    self.cursor.execute(sql, (account, password))
                self.db.commit()
```

b).访问登录页面并定位账号输入框、密码输入框、验证码输入框、登录按钮这四个元素

```python
        print("\n正在登陆...")
        loginUrl = "http://passport2.chaoxing.com/login?fid=185"
        self.browser.get(loginUrl)
        #定位元素位置
        inputAccount = self.browser.find_element_by_xpath('//*[@id="unameId"]')
        inputPassword = self.browser.find_element_by_xpath('//*[@id="passwordId"]')
        inputCAPCHA = self.browser.find_element_by_xpath('//*[@id="numcode"]')
        loginBtn = self.browser.find_element_by_xpath('//*[@id="form"]/table/tbody/tr[7]/td[2]/label/input')
```

c).向页面输入账号、密码、验证码

这里验证码的识别使用了最简单粗暴的肉眼识别，保存登录页面截图并显示后提示用户输入

```python
        from PIL import Image
     
        inputAccount.send_keys(account)
        inputPassword.send_keys(password)
        self.browser.save_screenshot('screenshot.png')
        Image.open('screenshot.png').show()
        CAPCHA = input("输入验证码 >>>")
        inputCAPCHA.send_keys(CAPCHA)
```
d).点击登录按钮登录，若登录失败提示重新登录

```python
        loginBtn.click()
        while self.browser.current_url == "http://passport2.chaoxing.com/login?refer=http%3A%2F%2Fi.mooc.chaoxing.com":
            print("登陆出错，请重新登陆...")
            self.login()
        print("登陆成功")
```
##### 3.chooseCourse()
登录后进入课程界面，定位课程元素位置，注意课程元素在一个frame中

![](/media/editor/20181021220038420_20190219134426691272.png)

切换到该frame后才可以对该元素进行点击操作，点击课程后会打开新的标签页，需要切换到新的标签页以进入课程界面

```python
    def chooseCourse(self):
        self.browser.switch_to.frame('frame_content')
        img = self.browser.find_element_by_xpath('//div[@class="Mcon1img httpsClass"]/a[1]')
        img.click()
        self.browser.switch_to.window(self.browser.window_handles[1])
        print("已进入课程界面")
```
##### 4.getNextCourse()
首先在页面中定位课程列表

![](/media/editor/20181021220925621_20190219134453946809.png)

展开单个课程，会发现未完成的课程的一个节点的class属性值为orange，可以以此来判断课程是否已经完成学习

![](/media/editor/20181021221239237_20190219134536054138.png)

因此获取到所有课程列表后遍历每一个课程，检测该节点的class值是否为orange。注意到综合测试题也符合该规律，因此用if语句排除掉。最后返回第一个未完成的课程的元素对象。

```python
    def getNextCourse(self):
        classes = self.browser.find_elements_by_css_selector('h3.clearfix')
        count = 0
        for i in range(len(classes)):
            if "orange" in classes[i].find_element_by_css_selector('.icon em').get_attribute('class'):
                if "综合测试题" not in classes[i].find_element_by_css_selector('.articlename').text:
                    course = classes[i].find_element_by_css_selector('.articlename')
                    count += 1
                    return course
        if count == 0:
            return False
```
##### 5.watchVideo()
这个函数用来观看视频。首先调用self.getNextCourse()获取要观看的课程元素在页面中的位置，然后点击进入观看课程的界面

```python
        course = self.getNextCourse()
        if not course:
            return False
        course.click()
```
进入页面，定位视频位置

![](/media/editor/20181021222131579_20190219134605792230.png)

定位到目标元素后点击播放，注意视频还是在一个frame中

```python
        self.browser.switch_to.frame('iframe')
        playBtn = self.browser.find_element_by_css_selector('.ans-attach-ct')
        webdriver.ActionChains(self.browser).move_to_element(playBtn).perform()
        playBtn.click()
```
至此，视频就可以成功播放了，但还需检测视频是否播放完成，如果播放完成，返回之前的界面，否则继续播放。

```python
    while True:
            time.sleep(5)
            if self.ifFinished():
               break
        self.browser.back()
        self.browser.refresh()
        return True
```
这里调用的self.ifFinished()函数将在之后实现

##### 6.ifFinished()
该函数用于检测视频是否播放完成，这里实现的检测思路是定时截取视频左上角任务点，检测其主要颜色，如果是绿色则播放完成。

首先截取整个页面，然后转化便于之后的颜色识别

```python
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        screenshot.convert('RGB')
```
然后定位该图标位置

![](/media/editor/20181021223131709_20190219134641801851.png)

然后根据该元素大小裁剪截图（由于该元素在frame中，用location获取的位置是在frame中的相对位置，这里就直接粗暴的用绝对坐标来定位截取图片的位置）

```python
        target = self.browser.find_element_by_css_selector('div.ans-job-icon')
        left = 456
        top = 340
        elementWidth = left + target.size['width']
        elementHeight = top + target.size['height']
        screenshot = screenshot.crop((left, top, elementWidth, elementHeight))
```
图片截取完后进行颜色识别，判断主要颜色的色域范围，如果为绿色则返回True即视频结束，否则返回False

```python
        result = self.getDominantColor(screenshot)
        if 230<result[0]<245 and 165<result[1]<180 and 30<result[2]<45:
            print("Watching...")
            return False
        else:
            return True
```
这里使用的颜色识别函数稍后实现

##### 7.getDominantColor(image)
这个函数传入一个PIL.Image对象，返回其主要颜色的rgb值构成的元组

```python
     def getDominantColor(self, image):
        max_score = 0.0001
        dominant_color = None
        for count, (r, g, b, a) in image.getcolors(image.size[0] * image.size[1]):
            # 转为HSV标准
            saturation = colorsys.rgb_to_hsv(r / 255.0, g / 255.0, b / 255.0)[1]
            y = min(abs(r * 2104 + g * 4130 + b * 802 + 4096 + 131072) >> 13, 235)
            y = (y - 16.0) / (235 - 16)

            # 忽略高亮色
            if y > 0.9:
                continue
            score = (saturation + 0.1) * count
            if score > max_score:
                max_score = score
                dominant_color = (r, g, b)
        return dominant_color
```
##### 8.run()
这个函数用于所有类成员函数的调用，模拟整个听课过程

```python
    def run(self):
        self.login()
        self.chooseCourse()    #选择课程
        while True:    
            result = self.watchVideo()    
            if not result:    #如果所有课程全部听完则退出循环
                print("课程全部听取完成！")
                break
```
     至此，AutoStudent类已实现
##### 9.main()
在主函数中实例化一个AuyoStudent对象并调用其run()函数，实现自动听课

```python
def main():
    robot = AutoStudent()
    robot.run()
```
### 三、源代码
这里给出的源码比较混乱，但跑起来还是没问题的



```python
from selenium import webdriver
from PIL import Image
import time
from io import BytesIO
import colorsys
import pymysql

class AutoStudent():
    def __init__(self):
        #opt = webdriver.ChromeOptions()
        #opt.add_argument('--headless')
        #self.browser = webdriver.Chrome(chrome_options=opt)
        self.browser = webdriver.Chrome()
        self.browser.maximize_window()
        self.courseList = list()

        self.db = pymysql.connect(host='localhost', user='root', password='*******', db='ChaoXingStuInfo')
        self.cursor = self.db.cursor()
        self.cursor.execute('CREATE TABLE IF NOT EXISTS StudentInfo(账号 VARCHAR(30), 密码 VARCHAR(20));')
        self.db.commit()

    def login(self):
        choice1 = ''
        while True:
            self.cursor.execute('SELECT * FROM StudentInfo')
            result = self.cursor.fetchall()
            if result:
                print("\n===================数据库中已有学生信息===================")
                for i in range(0, len(result)):
                    print(
                        "\n编号：" + str(i + 1) + "\t账号：" + result[i][0] + '\t密码：' + result[i][1])
                print('\n')
                choice1 = input("是否使用数据库中已有学生信息？(y/n) >>>")
                if choice1 == 'y':
                    index = int(input("输入要使用的信息编号：")) - 1
                    try:
                        account = result[index][0]
                        password = result[index][1]
                        break
                    except:
                        print("输入错误！")
                        continue
                else:
                    account = input("输入账号：")
                    password = input("输入密码：")
                    break
            else:
                account = input("输入账号：")
                password = input("输入密码：")
                break
        if choice1 != 'y':
            choice2 = input("\n是否将学生信息存入数据库？(y/n)")
            if choice2 == 'y':
                if choice1 != 'y':
                    sql = 'INSERT INTO StudentInfo VALUES(%s, %s)'
                    self.cursor.execute(sql, (account, password))
                self.db.commit()

        print("\n正在登陆...")
        loginUrl = "http://passport2.chaoxing.com/login?fid=185"
        self.browser.get(loginUrl)
        #定位元素位置
        inputAccount = self.browser.find_element_by_xpath('//*[@id="unameId"]')
        inputPassword = self.browser.find_element_by_xpath('//*[@id="passwordId"]')
        inputCAPCHA = self.browser.find_element_by_xpath('//*[@id="numcode"]')
        loginBtn = self.browser.find_element_by_xpath('//*[@id="form"]/table/tbody/tr[7]/td[2]/label/input')
        '''
        验证码截图
        CAPCHAImage = re.compile('img src="(.*?)".*?id="numVerCode"').findall(self.browser.page_source)[0]
        CAPCHAImage = self.browser.find_element_by_xpath('//img[@id="numVerCode"]')
        left = CAPCHAImage.location['x']
        top = CAPCHAImage.location['y']
        right = left + CAPCHAImage.size['width']
        bottom = top + CAPCHAImage.size['height']
        image = image.crop((left, top, right, bottom))
        '''
        #填入信息
        inputAccount.send_keys(account)
        inputPassword.send_keys(password)
        #Image.open('./CAPCHAImage.jpg').show()
        self.browser.save_screenshot('screenshot.png')
        Image.open('screenshot.png').show()
        CAPCHA = input("输入验证码 >>>")
        inputCAPCHA.send_keys(CAPCHA)
        loginBtn.click()
        while self.browser.current_url == "http://passport2.chaoxing.com/login?refer=http%3A%2F%2Fi.mooc.chaoxing.com":
            print("登陆出错，请重新登陆...")
            self.login()
        print("登陆成功")

    def chooseCourse(self):
        #print(self.browser.page_source)
        #time.sleep(5)
        self.browser.switch_to.frame('frame_content')
        img = self.browser.find_element_by_xpath('//div[@class="Mcon1img httpsClass"]/a[1]')
        img.click()
        self.browser.switch_to.window(self.browser.window_handles[1])
        print("已进入课程界面")

    def getNextCourse(self):
        classes = self.browser.find_elements_by_css_selector('h3.clearfix')
        count = 0
        for i in range(len(classes)):
            if "orange" in classes[i].find_element_by_css_selector('.icon em').get_attribute('class'):
                if "综合测试题" not in classes[i].find_element_by_css_selector('.articlename').text:
                    course = classes[i].find_element_by_css_selector('.articlename')
                    count += 1
                    return course
        if count == 0:
            return False

    def watchVideo(self):
        course = self.getNextCourse()
        if not course:
            return False
        course.click()
        self.browser.switch_to.frame('iframe')
        #self.browser.switch_to.frame('ans-attach-online ans-insertvideo-online')
        playBtn = self.browser.find_element_by_css_selector('.ans-attach-ct')
        webdriver.ActionChains(self.browser).move_to_element(playBtn).perform()
        playBtn.click()
        while True:
            time.sleep(5)
            if self.ifFinished():
               break
        self.browser.back()
        self.browser.refresh()
        return True

    def ifFinished(self):
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        screenshot.convert('RGB')
        target = self.browser.find_element_by_css_selector('div.ans-job-icon')
        left = 456
        top = 340
        elementWidth = left + target.size['width']
        elementHeight = top + target.size['height']

        screenshot = screenshot.crop((left, top, elementWidth, elementHeight))
        #screenshot.show()
        result = self.getDominantColor(screenshot)
        if 230<result[0]<245 and 165<result[1]<180 and 30<result[2]<45:
            print("Watching...")
            return False
        else:
            return True

    def getDominantColor(self, image):
        max_score = 0.0001
        dominant_color = None
        for count, (r, g, b, a) in image.getcolors(image.size[0] * image.size[1]):
            # 转为HSV标准
            saturation = colorsys.rgb_to_hsv(r / 255.0, g / 255.0, b / 255.0)[1]
            y = min(abs(r * 2104 + g * 4130 + b * 802 + 4096 + 131072) >> 13, 235)
            y = (y - 16.0) / (235 - 16)

            # 忽略高亮色
            if y > 0.9:
                continue
            score = (saturation + 0.1) * count
            if score > max_score:
                max_score = score
                dominant_color = (r, g, b)
        return dominant_color

    def run(self):
        self.login()
        self.chooseCourse()
        while True:
            result = self.watchVideo()
            if not result:
                print("课程全部听取完成！")
                break

def main():
    robot = AutoStudent()
    robot.run()

if __name__ == '__main__':
    main()
```