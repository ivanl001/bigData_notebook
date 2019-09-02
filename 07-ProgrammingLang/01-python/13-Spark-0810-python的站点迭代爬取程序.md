```python

#
# author      : ivanl001
# creator     : 2018-12-08 13:59
# description : 爬虫基础, 这里简单写下代码， 在spark的0810小节有更多的内容，这里基本实现了，但是还是有点小问题，如果网页没有回应，就会一直等待，这个明显是不行的，之后再用到会再解决
# 

import urllib3
import re
import time
import os


# 1，这里是创建连接池
http = urllib3.PoolManager()


# 获取一个页面到url
def get_url_from_page(page_str):
    # 1, 解析
    pattern = u"<a\\s*href=\"([\u0000-\uffff]*?)\".*?>"
    # pattern = u'<a[\u0000-\uffff&&^[href]]*href="([\u0000-\uffff&&^"]*?)"'
    urls = re.finditer(pattern, page_str)
    for url in urls:
        addr = url.group(1)
        if str(addr).startswith("http"): # 这里做的比较简单，只判断http开头的，其他的一概不要了
            print(addr)
            # 这里要先判断一下这个文件是否已经存在，如果已经存在就不能再下载了
            name = addr.replace(":", "").replace("//","").replace("/","")
            outputPath = "/Users/ivanl001/Desktop/zhang/" + name + ".html"
            if os.path.exists(outputPath):
                continue
            else:
                # 这里就是相当于循环，一直在不停的循环迭代进行获取网页
                download_page(addr)
        else:
            continue


# 保存到文件
def download_page(urlStr):

    # 1, 进行请求, 这里应该设置一个超时时间什么的， 因为现在有一个问题，如果其中一个网页无响应，就会卡在那里一直等，这个后续
    result = http.request("GET", urlStr)

    # 2，获取(打印)结果
    resultByte = result.data
    # print(result.data)

    # 3, 对获取结果进行解码，并进行保存到文件，这里命名按照时间先来吧
    # 这里命名还是不能用时间，要不没法判断是否已经存在
    # current_time = int(time.time()*100000)
    name = urlStr.replace(":", "").replace("//","").replace("/","")
    outputPath = "/Users/ivanl001/Desktop/zhang/" + name + ".html"
    print(outputPath)
    output = open(outputPath ,"wb")
    output.write(resultByte)

    # 5, 对获取页面进行解码，并获取其中对url
    pageStr = resultByte.decode("utf-8")
    # print(pageStr)
    get_url_from_page(pageStr)





url = "http://focus.tianya.cn/"
# download_page("http://focus.tianya.cn/")
download_page(url)


```