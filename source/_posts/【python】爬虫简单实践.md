---
title: 【python】爬虫简单实践
date: 2017-10-24 16:35:22
tags:
- python
---

### 豆瓣

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding('utf8')
from bs4 import BeautifulSoup
import re
import urllib2
import xlwt

#得到页面全部内容
def askURL(url):
    request = urllib2.Request(url)#发送请求
    try:
        response = urllib2.urlopen(request)#取得响应
        html= response.read()#获取网页内容
        #print html
    except urllib2.URLError, e:
        if hasattr(e,"code"):
            print e.code
        if hasattr(e,"reason"):
            print e.reason
    return html

#获取相关内容
def getData(baseurl):
    #找到评论标题
    pattern_title = re.compile(r'<a class="".+title="(.+)"')
    #找到评论全文链接
    pattern_link = re.compile(r'<a class="".+href="(.+review.*?)"')
    #找到作者
    pattern_author = re.compile(r'<a.+people.+">(.+)</a>')
    #找到评论的影片和影评详情链接
    pattern_subject_link = re.compile(r'<a href="(.+subject.+)" title="(.+)">')
    #找到推荐等级
    pattern_star=re.compile(r'<span class="allstar\d+" title="(.+)"></span>')
    #找到回应数
    pattern_response=re.compile(r'<span class="">\((\d+)回应\)</span>')
    #找到有用数
    pattern_use=re.compile(r'<em id="ucount\d+u">(\d+)</em>')
    remove=re.compile(r'<.+?>')#去除标签
    datalist=[]
    for i in range(0,5):#总共5页
        url=baseurl+str(i*10)#更新url
        html=askURL(url)
        soup = BeautifulSoup(html)
        #找到每一个影评项
        for item in soup.find_all('ul',class_='tlst clearfix'):
            data=[]
            item=str(item)#转换成字符串
            #print item
            title = re.findall(pattern_title,item)[0]
            #print title
            reviewlink=re.findall(pattern_link,item)[0]
            data.append(title)#添加标题
            author=re.findall(pattern_author, item)[0]
            data.append(author)#添加作者
            list_subject_link = re.findall(pattern_subject_link, item)[0]
            moviename=list_subject_link[1]
            movielink=list_subject_link[0]
            data.append(moviename)#添加片名
            data.append(movielink)#添加影片链接
            star=re.findall(pattern_star,item)[0]
            data.append(star)#添加推荐等级
            response=re.findall(pattern_response,item)
            #回应数可能为0，就找不到
            if(len(response)!=0):
                response=response[0]
            else:
                response=0
            data.append(response)#添加回应数
            data.append(reviewlink)#添加评论正文链接
            content=askURL(reviewlink)
            use=re.findall(pattern_use,content)
            #有用数可能为0，就找不到
            if len(use)!=0:
                use=use[0]
            else:
                use=0
            content=BeautifulSoup(content)
            desc=content.find_all('div',id='link-report')[0]
            desc=re.sub(remove,'',str(desc))#去掉标签
            data.append(desc)#添加评论正文
            data.append(use)#添加有用数
            datalist.append(data)
    return datalist

#将相关数据写入excel中
def saveData(datalist,savepath):
    book=xlwt.Workbook(encoding='utf-8',style_compression=0)
    sheet=book.add_sheet('豆瓣最受欢迎影评',cell_overwrite_ok=True)
    col=('标题','作者','影片名','影片详情链接','推荐级','回应数','影评链接','影评','有用数')
    for i in range(0,9):
        sheet.write(0,i,col[i])#列名
    for i in range(0,50):#总共50条影评
        data=datalist[i]
        for j in range(0,9):
            sheet.write(i+1,j,data[j])#数据
    book.save(savepath)#保存

def main():
    baseurl='http://movie.douban.com/review/best/?start='
    datalist=getData(baseurl)
    savapath=u'豆瓣最受欢迎影评.xlsx'
    saveData(datalist,savapath)

main()

```

### 新闻

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-
import os
import re
import urllib2
import sys
reload(sys)
sys.setdefaultencoding('utf8')

#得到页面全部内容
def askURL(url):
    request = urllib2.Request(url)#发送请求
    try:
        response = urllib2.urlopen(request)#取得响应
        html= response.read()#获取网页内容
        #print html
        html=html.decode('gbk','ignore')#将gbk编码转为unicode编码
        html=html.encode('utf-8','ignore')#将unicode编码转为utf-8编码
    except urllib2.URLError, e:
        if hasattr(e,"code"):
            print e.code
        if hasattr(e,"reason"):
            print e.reason
    return html

#得到正文
def getContent(url):
    html=askURL(url)
    text=''
    #找到新闻主体所在的标签
    findDiv=re.compile(r'<div class="left_zw" style="position:relative">'
                       r'(.*)<div id="function_code_page">',re.S)
    div=re.findall(findDiv,html)
    if len(div)!=0:
        content=div[0]
        labels=re.compile(r'[/]*<.*?>',re.S)
        text=re.sub(labels,'',content)#去掉各种标签
        text=re.sub(r'\s|中新社.*?电|\(完\)|\(记者.*?\)','',text)#去掉空行和换行符,无关内容
        text=re.sub(r'　　','\n',text)#将缩进符替换成换行符
        #print text
    return text

#根据类别按顺序命名文件
def saveFile(labelName,date,fileNum):
    dirname="news"
    #若目录不存在，就新建
    if(not os.path.exists(dirname)):
        os.mkdir(dirname)
    labelName=labelName.encode('gbk','ignore')
    labelpath=dirname+'\\'+labelName
    if(not os.path.exists(labelpath)):
        os.mkdir(labelpath)
    path=labelpath+"\\"+date+"-0"+str(fileNum)+".txt"#w文本保存路径
    print "正在下载"+path
    #path=path.encode('gbk','utf-8')#转换编码
    f=open(path,'w+')
    return f

#得到正文的URL，读取正文，并保存
def getURL(nurl,labelName):
    html=askURL(nurl)
    findDiv=re.compile(r'<div class="dd_lm">.*</div>')
    findTime=re.compile(r'<div class="dd_time">(.*)</div>')
    findTitle=re.compile(r'<div class="dd_bt"><a href="http://www.chinanews.com/.*\.shtml">(.*)</a></div>')
    findURL=re.compile(r'<div class="dd_bt"><a href="(http://www.chinanews.com/.*\.shtml)">.*</a></div>')
    findLabel=re.compile(r'<div class="dd_lm">\[<a href=http://www.chinanews.com/.*\.shtml>(.*)</a>\]</div>')
    fileNum=0
    for info in re.findall(findDiv,html):
        #print info
        time=re.findall(findTime,info)[0]
        date=re.findall(r'\d?-\d*',time)[0]#获取新闻发布日期
        title=re.findall(findTitle,info)[0]
        url=re.findall(findURL,info)[0]#获取新闻正文的链接
        label=re.findall(findLabel,info)[0]#获取新闻所属类别
        if(label=="I&nbsp;&nbsp;T"):#网页为I&nbsp;&nbsp;T
            label="IT"
        if(labelName==label):
            text=getContent(url)
            #如果新闻内容长度大于1000，保存新闻标题和正文
            if(len(text)>1000):
                fileNum=fileNum+1
                f=saveFile(labelName,date,fileNum)
                f.write(title)
                f.write(text)
                f.close()

#抓取新闻标题、类别、发布时间、url，并建立相应的文件，存到相应的类别文件夹中
def getNews(url,begin_page,end_page,labelName):
    for i in range(begin_page, end_page+1):
        nurl=url+str(i)+".html"
        #print nurl
        #获取网页内容
        getURL(nurl,labelName)

#接收输入类别、起始页数、终止页数
def main():
    url='http://www.chinanews.com/scroll-news/news'
    ch=int(raw_input(u'请输入类别的对应的数字（IT=1、财经=2、地方=3、国际=4、国内=5、健康=6、军事=7、'
                 u'社会=8、体育=9、文化=10），输入-1退出，输入0表示全选：\n'))
    labels=('IT','财经','地方',"国际","国内","健康","军事","社会","体育","文化")
    while(ch!=-1):
        begin_page = int(raw_input(u'请输入开始的页数(1,)：\n'))
        end_page = int(raw_input(u'请输入终点的页数(1,)：\n'))
        if(ch>=1 and ch<=10):
            getNews(url,begin_page,end_page,labels[ch-1])
        elif(ch==0):
            for label in labels:
                getNews(url,begin_page,end_page,label)
        else:
            print "输入错误，请重新输入！"
        ch=int(raw_input(u'请输入类别的对应的数字（IT=1、财经=2、地方=3、国际=4、国内=5、健康=6、军事=7、'
                         u'社会=8、体育=9、文化=10），输入-1退出，输入0表示全选：\n'))

#调用主函数
#一页有125条新闻
main()

```

### RPM源下载

```python
#!/usr/bin/python
#coding:utf-8

import re
import urllib

print "The Most Basic Download\n"

def getHtml(url):
    page = urllib.urlopen(url)
    html = page.read()
    print html
    return html

def getImg(html):
    reg = r'href="(.*?\.rpm)"' #匹配的正则表达式
    imgre = re.compile(reg)
    imglist = re.findall(imgre,html)

    for imgurl in imglist:
        name = imgurl[:-4]
        print name
        httpurl = 'http://mirrors.163.com/centos/7/os/x86_64/Packages/' + imgurl
        print  httpurl
        urllib.urlretrieve(httpurl,'%s.jpg' % name) #下载的内容，并顺序命名


url ="http://mirrors.163.com/centos/7/os/x86_64/Packages/"
html = getHtml(url)
getImg(html)

#http://www.nvsheng.com/mm/




#!/usr/bin/python
#coding:utf-8
'''
多网址下载改进
'''
import re
import urllib

def getHtml(url):
    page = urllib.urlopen(url)
    html = page.read()
    getImg(html)

def getImg(html):
    reg = r'href="(.*?\.rpm)"' #匹配的正则表达式
    imgre = re.compile(reg)
    imglist = re.findall(imgre,html)
    file1 = open("rpm.txt", "w")
    x = 0
    for imgurl in imglist:
        print imgurl
        x=x+1
        file1.write('http://mirrors.163.com/centos/7/os/x86_64/Packages/' + imgurl + '\n')
    print x
    file1.close()

url ="http://mirrors.163.com/centos/7/os/x86_64/Packages/"
html = getHtml(url)
getImg(html)

```

### 静态网页

```python
#!/usr/bin/python
#coding:utf-8

import re
import urllib


def getHtml(url):
    page = urllib.urlopen(url)
    html = page.read()
    return html

def getImg(html):
    reg = r'src="(.*?\.jpg)"'
    imgre = re.compile(reg)
    imglist = re.findall(imgre,html)
    x = 0
    for imgurl in imglist:
        print imgurl
        urllib.urlretrieve(imgurl,'%s.jpg' % x)
        x += 1


url=""
html = getHtml(url)
getImg(html)

--------------------------------------------------------

def getImg(html):
    reg = r'src="(.*?\.jpg)"' #匹配的正则表达式
    imgre = re.compile(reg)
    imglist = re.findall(imgre,html)

    for imgurl in imglist:
        name1 = imgurl[:-4]
        print name1
        name2 = name1.split('/')[-1]
        print name2
        urllib.urlretrieve(imgurl,'%s.jpg' % name2)

--------------------------------------------------------

import urllib2

def getHtml(url):
    try:
        respHtml = urllib2.urlopen(url).read();
    except urllib2.URLError, e:
        if hasattr(e, "reason"):
            print "Failed to reach the server"
            print "The reason:", e.reason
        elif hasattr(e, "code"):
            print "The server couldn't fulfill the request"
            print "Error code:", e.code
            print "Return content:", e.read()
        else:
            pass
    return respHtml

```

### 图片获取

```python
#!/usr/bin/python
#coding:gbk

import re
import urllib
import ctypes

STD_INPUT_HANDLE = -10
STD_OUTPUT_HANDLE = -11
STD_ERROR_HANDLE = -12

FOREGROUND_BLACK = 0x0
FOREGROUND_BLUE = 0x01  # text color contains blue.
FOREGROUND_GREEN = 0x02  # text color contains green.
FOREGROUND_RED = 0x04  # text color contains red.
FOREGROUND_INTENSITY = 0x08  # text color is intensified.

BACKGROUND_BLUE = 0x10  # background color contains blue.
BACKGROUND_GREEN = 0x20  # background color contains green.
BACKGROUND_RED = 0x40  # background color contains red.
BACKGROUND_INTENSITY = 0x80  # background color is intensified.


# 上面这一大段都是在设置前景色和背景色，其实可以用数字直接设置，我的代码直接用数字设置颜色


class Color:
    std_out_handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)

    def set_cmd_color(self, color, handle=std_out_handle):
        bool = ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
        return bool

    def reset_color(self):
        self.set_cmd_color(FOREGROUND_RED | FOREGROUND_GREEN | FOREGROUND_BLUE | FOREGROUND_INTENSITY)
        # 初始化颜色为黑色背景，纯白色字，CMD默认是灰色字体的

    def print_red_text(self, print_text):
        self.set_cmd_color(4 | 8)
        print print_text
        self.reset_color()
        # 红色字体

    def print_green_text(self, print_text):
        self.set_cmd_color(FOREGROUND_GREEN | FOREGROUND_INTENSITY)
        c = raw_input(print_text)
        self.reset_color()
        return c
        # 绿色字体。实现的是，让用户输入的字体是绿色的，记得返回函数值。

    def print_yellow_text(self, print_text):
        self.set_cmd_color(6 | 8)
        print print_text
        self.reset_color()
        # 黄色字体

    def print_blue_text(self, print_text):
        self.set_cmd_color(1 | 10)
        print print_text
        self.reset_color()
        # 蓝色字体


clr = Color()
clr.set_cmd_color(FOREGROUND_RED | FOREGROUND_GREEN | FOREGROUND_BLUE | FOREGROUND_INTENSITY)
clr.print_yellow_text('*'*66)
clr.print_yellow_text("*m.mm131.com")
clr.print_yellow_text("*Site examples:\thttp://img1.mm131.com/pic/2721/1.jpg")
clr.print_yellow_text("*You can enter multiple IDs and Pages,Separated by ' '.")
clr.print_yellow_text("If you enter nothing, the program will end.")
clr.print_yellow_text('*'*66+'\n')

modeUrl = "http://img1.mm131.com/pic/"

def getUrl(modeUrl,id,end):
    file1 = open("resUrl.txt","a")
    for i in range(1,int(end)+1):
        url = modeUrl + id + '/' + str(i) +".jpg\n"
        print url
        file1.write(url)
    file1.write('\n')
    file1.close()
    clr.print_yellow_text("Successful!\n")

def main():
    ids = clr.print_green_text("Please enter the ID of the page you want to download:\n")
    ends = clr.print_green_text("Please enter the total number of pages:\n")
    while(ids!=''):
        idlist = ids.split(' ')
        endlist = ends.split(' ')
        for (id,end) in zip(idlist,endlist):
            if(int(id)>0):
                getUrl(modeUrl,id,end)
            else:
                clr.print_red_text("\nInput error,please enter again!\n")
        ids = clr.print_green_text("Please enter the ID of the page you want to download:\n")
        ends = clr.print_green_text("Please enter the total number of pages:\n")

main()

#最新地址到：http://img1.mm131.com/pic/2721/0.jpg  2016-10-29

```



```python
#!/bin/python
#coding=utf-8

from selenium import webdriver
import time
import json
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )


# 获取导航栏
def getNavUrls(url):
    browser.get(url)
    nav_xpath = browser.find_element_by_xpath("/html/body/div[4]")
    nav_atags = nav_xpath.find_elements_by_tag_name("a")
    navs = []
    for nav_atag in nav_atags:
        nav = nav_atag.get_attribute("href")
        navs.append(nav)
    return navs


# 获取总页数
def getAllPages(navurl):
    browser.get(navurl)
    endPageUrl = browser.find_element_by_link_text("末页").get_attribute("href")
    browser.get(endPageUrl)
    page =  browser.find_element_by_class_name("page_now").text
    return int(page)


# 获取模块具体内容
def getModeUrls(navs):
    modeUrls = {}
    for nav in navs:
        print nav
        nums = range(getAllPages(nav))
        firstPage = nav
        pics = {}
        pages = {}
        for nextPage in nums:
            print nextPage
            browser.get(firstPage)
            mode_xpath = browser.find_element_by_xpath("/html/body/div[5]/dl")
            mode_imgs = mode_xpath.find_elements_by_tag_name("img")
            for mode_img in mode_imgs:
                pics[mode_img.get_attribute("alt")] = mode_img.get_attribute("src")
            pages[nextPage + 1] = pics
            if nextPage < len(nums)-1:
                firstPage = browser.find_element_by_link_text("下一页").get_attribute("href")
        modeUrls[nav] = pages
    with open('mm.json', 'w') as fp:
        fp.write(json.dumps(modeUrls, encoding='UTF-8', ensure_ascii=False))
    return json.dumps(modeUrls, encoding='UTF-8', ensure_ascii=False)


print time.ctime()
browser = webdriver.PhantomJS()
url = "http://www.mm131.com/"
navs = getNavUrls(url)[1:]
print navs
print getModeUrls(navs)
browser.close()
print time.ctime()

```



```python
#!/bin/python
#coding=utf-8

from selenium import webdriver
import time
import json
import re
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )


# 获取导航栏
def getNavUrls(url):
    browser.get(url)
    nav_xpath = browser.find_element_by_xpath("/html/body/div[4]")
    nav_atags = nav_xpath.find_elements_by_tag_name("a")
    navs = []
    for nav_atag in nav_atags:
        nav = nav_atag.get_attribute("href")
        navs.append(nav)
    return navs


# 获取概览的总页数
def getAllPages(navurl):
    browser = webdriver.PhantomJS()
    browser.get(navurl)
    endPageUrl = browser.find_element_by_link_text("末页").get_attribute("href")
    browser.get(endPageUrl)
    page =  browser.find_element_by_class_name("page_now").text
    return int(page)


# 获取所有概念图到json文件
def getModeUrls(navs):
    modeUrls = {}
    for nav in navs:
        print nav
        nums = range(getAllPages(nav))
        firstPage = nav
        pics = {}
        pages = {}
        for nextPage in nums:
            print nextPage
            browser.get(firstPage)
            mode_xpath = browser.find_element_by_xpath("/html/body/div[5]/dl")
            mode_imgs = mode_xpath.find_elements_by_tag_name("img")
            for mode_img in mode_imgs:
                pics[mode_img.get_attribute("alt")] = mode_img.get_attribute("src")
            pages[nextPage + 1] = pics
            if nextPage < len(nums)-1:
                firstPage = browser.find_element_by_link_text("下一页").get_attribute("href")
        modeUrls[nav] = pages
    with open('mm.json', 'w') as fp:
        fp.write(json.dumps(modeUrls, encoding='UTF-8', ensure_ascii=False))
    return json.dumps(modeUrls, encoding='UTF-8', ensure_ascii=False)


# 查看所有概览内容
def viewAllModel(navs):
    for nav in  navs:
        nums = getAllPages(nav)
        browser.get(nav)
        time.sleep(3)
        num = 1
        while num < nums:
            browser.find_element_by_link_text("下一页").click()
            time.sleep(3)
            num += 1


# 顺序查看所有图片具体内容
def viewAllDetailPics(navs):
    for nav in navs:
        browser.get(nav)
        durl = browser.find_element_by_xpath("/html/body/div[5]/dl/dd[1]/a").get_attribute("href")
        browser.get(durl)
        time.sleep(3)
        while True:
            # 获取总页数
            allPages = browser.find_element_by_xpath("/html/body/div[6]/div[3]/span[1]").text
            pages = int(re.findall(r"[0-9]{1,9}", allPages)[0])
            page = 1
            while page < pages:
                if browser.find_element_by_link_text("下一页"):
                    browser.find_element_by_link_text("下一页").click()
                    time.sleep(3)
                page += 1
            if browser.find_element_by_xpath("/html/body/div[6]/div[4]/a[1]").text != "没有了":
                browser.find_element_by_xpath("/html/body/div[6]/div[4]/a[1]").click()
            else:
                break


# 查看某个图片的具体信息
def viewSingleDetailPic(url):
    browser.get(url)
    # 获取总页数
    allPages = browser.find_element_by_xpath("/html/body/div[6]/div[3]/span[1]").text
    pages = int(re.findall(r"[0-9]{1,9}", allPages)[0])
    page = 1
    while page < pages:
        if browser.find_element_by_link_text("下一页"):
            browser.find_element_by_link_text("下一页").click()
        page += 1


# 网页截图
def webScreenShotPics(url):
    browser.get(url)
    # 获取总页数
    allPages = browser.find_element_by_xpath("/html/body/div[6]/div[3]/span[1]").text
    pages = int(re.findall(r"[0-9]{1,9}", allPages)[0])
    page = 1
    while page < pages:
        if browser.find_element_by_link_text("下一页"):
            browser.save_screenshot(str(page) + ".png")
            browser.find_element_by_link_text("下一页").click()
        page += 1


# 运行程序
if __name__ == '__main__':
    browser = webdriver.PhantomJS()
    browser.maximize_window()

    # url = "http://www.mm131.com/"
    # navs = getNavUrls(url)[1:]
    # print getModeUrls(navs)
    # viewAllModel(navs)
    # viewAllDetailPics(navs)

    durl = "http://www.mm131.com/xinggan/3455.html"
    # viewSingleDetailPic(durl)
    # webScreenShotPics(durl)

    browser.close()
    
```

