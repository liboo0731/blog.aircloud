---
title: 【python】Selenium框架简单实践
date: 2018-04-02 16:42:56
tags:
- python
- selenium
---

### selenium+Python（一）基本操作

```python
#!/usr/bin/python
# -*- coding: utf8-*-


# （一）首先是文件开头上要写
from selenium import webdriver      #引入selenium模块。
from [selenium.webdriver.common.keys](http://selenium.webdriver.common.keys/) import Keys  #模拟键盘输入。
import random,time  #经常要用到，一个是产生随机数，一个是时间操作的功能


#（二）最简单的一段功能：
browser = webdriver.Firefox()   #启动chrome浏览器
time.sleep(3)  #停顿3秒
browser.maximize_window() #浏览器窗口最大化
#OutputLogin = Login(browser,username, password) #登录网页的函数，后续讲解
time.sleep(int(random.uniform(1, 10)))#随机产生一个1到9秒的随机整数，然后等待这个时间
browser.quit() #退出浏览器


#（三）定义一个登录系统的函数
def Login(browser,username, password): #要有冒号
    browser.get('网页的URL')  #浏览器登录网页的URL
    time.sleep(3)
    try:
        # find user login input box
        elem_user=browser.find_element_by_id("username")
        #这个是通过find_element_by_id函数来寻找定位网页上的id为username的控件
        elem_user.clear()
        elem_user.send_keys(username)
        #然后向这个控件发送username的值
        time.sleep(1)
        # find pwd input box
        elem_pwd=browser.find_element_by_id("password")
        elem_pwd.clear()
        elem_pwd.send_keys(password)
        time.sleep(1)
        # enter RETURN in pwd box to activate
        elem_pwd.send_keys(Keys.RETURN)
        #然后向这个控件发送回车键，注意，如果是键盘上的回车，SHIFT，CONTROL键之类的，要用Keys.控制键的名称作为输入。
        return username + "  login successfully \n"
    except:
        return username + "  login failed \n"
        pass
```

### selenium+Python（二）定位元素

自动化测试中常用的功能是通过各种元素，例如id，class，xpath，css等内容来寻找定位元素，而且不光可以定位一个元素，还可以定位一队元素，然后逐个操作。

```python
#定义一个定位操作单个元素的函数
def Signup_Click(browser):
    input3 = browser.find_element_by_class_name("checkbtn")
    try:
        input3.click()
        return "Signup successfully \n"
    except:
        return "Signup failed \n"
        pass

#定义一个定位操作多个元素的函数
def Love_Clicks(browser,k):
    inputs2 = browser.find_elements_by_class_name("love")
    #注意，是elements，不是element，复数形式
    for input2 in inputs2:
        try:
            input2.click()
        except:
            pass
    return str(i)+" Love successfully \n"
```

### selenium+Python（三）键盘和鼠标操作

Python也可以模拟鼠标和键盘的操作，不过要注意的是键盘带来的屏幕游标位置的挪动和鼠标在屏幕上的挪动位置，两个是不同的。

```python
#首先要在文件头引入
from [selenium.webdriver.common.action_chains](http://selenium.webdriver.common.action_chains/) import ActionChains

#定义一个函数
def Transfer_Clicks(browser):
    browser.execute_script("window.scrollBy(0,-document.body.scrollHeight)","")
    #这个是执行一段Javascript函数，将网页滚到到网页顶部。
        try:
            inputs1 = browser.find_elements_by_class_name("feedAttr_transfer")
            for input1 in inputs1:
                try:
                    ActionChains(browser).click(input1).perform()
                    #模拟鼠标点击控件input1，此时的鼠标位置在input1处
                    browser.execute_script("window.scrollBy(0,200)","")
                    #向下滚动200个像素，鼠标位置也跟着变了
                    ActionChains(browser).move_by_offset(0,-80).perform()
                    #向上移动鼠标80个像素，水平方向不同
                    ActionChains(browser).click().perform()
                    #鼠标左键点击
                    ActionChains(browser).key_down(Keys.TAB).perform()
                    #模拟tab键的输入
                    ActionChains(browser).send_keys(Keys.ENTER).perform()
                    #模拟输入ENTER键
                except:
                    pass
        except:
            pass
        return "Transfer successfully \n"
```

### 杰云测试快捷登陆

```python
#coding=utf8
from selenium import webdriver
import time
def Login(browser,url,username, password):
    browser.get(url)
    time.sleep(1)
    try:
        # find user login input box
        elem_user=browser.find_element_by_id("j_username")
        elem_user.clear()
        elem_user.send_keys(username)
        time.sleep(1)
        # find pwd input box
        elem_pwd=browser.find_element_by_id("j_password")
        elem_pwd.clear()
        elem_pwd.send_keys(password)
        time.sleep(1)
        # submit
        elem_sub = browser.find_element_by_id("logoutBtn")
        elem_sub.click()
        # action
        # print browser.current_url
        time.sleep(3)
        # get rdp file (current session)
        # print browser.find_element_by_xpath('/html/body/iframe').get_attribute('src')
        return username + "login successfully \n"
    except:
        return username + "login failed \n"
if __name__ == '__main__':
    browser = webdriver.Chrome()
    browser.maximize_window()
    url = 'https://yangtzi.vicp.net:843'
    usrname = 'user3'
    password = '123456'
    Login(browser,url,usrname,password)
    browser.close()
```

