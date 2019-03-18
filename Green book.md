最近奥斯卡获名单公布，《绿皮书》凭借的最佳影片奖引起人们的热议，最近国内也引进的这部影片，这次就就爬取这部影片的评论看看《绿皮书》到底好在哪里。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/1.png?raw=true)
### 页面分析
打开豆瓣关于《绿皮书》的详情页，可以看到《绿皮书》的短评多达13w多条。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/2.png?raw=true)
在上次《小偷家族》的爬虫中，实际上爬虫只爬取了10页的短评，而往后的评论就需要登陆才能查看了，所以这次就试着模拟登陆豆瓣来爬取更多的评论。

怎么爬取评论信息上次已经写过了，这次重点来看如何模拟登陆豆瓣，这次我用的方法是保存cookie的方法来获取登陆后的网页，cookie是指某些网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。

在豆瓣的登录页面按f12打开开发者模式，输入自己的账号密码可以发现这个cookie值发生了变化，登录后的cookie值就是你账号的cookie值。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/3.png?raw=true)
用这个cookie就可以获取登录后的网页了。但是豆瓣实在太小气了，总共只展示500条评论，多一条都不给，所有最多只能爬到这么多评论。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/4.png?raw=true)

### code
代码是之前的代码差不多，只是获取网页的方式稍微有点改变。
```python
import requests
import re
from bs4 import BeautifulSoup


def get_one_page(url):
    response = requests.get(url)
    if response.status_code == 200:
        return response.text
    return None


def parser_one_page(html):
    soup = BeautifulSoup(html, 'html.parser')
    # print(soup)
    data = soup.select('.short')
    # print(data)
    for li in data:
        li=str(li)
        li = li.replace('<span class=\"short\">', '')
        li = li.replace('</span>', '')
        #print(li+'\n\n')
        with open('Green Book.txt','a',encoding='utf-8') as f:
            f.write(li+'\n\n')
            f.close()


if __name__ == '__main__':
    cookie = {
        "Cookie": " XXX"}
    r = requests.session()
    #html = requests.get(url, cookies=cookie).content

    for i in range(0,500,20):
        url = 'https://movie.douban.com/subject/27060077/comments?start='+str(i)+'&limit=20&sort=new_score&status=P'
        html = requests.get(url, cookies=cookie).content
        print('正在抓取第'+str(int(i/20+1))+'页评论！')
        parser_one_page(html)
```
从爬出来的影评行数可以看到，这次爬到的数量比之前的多得多。模拟登录网页的方法还有能多种，之后会一一试试其它的方法。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/5.png?raw=true)


生成词云的方法和之前一样，只是用到的停用次需要有些改变。
![](https://github.com/Yuzzzzzzz/imag/blob/master/Green%20book_image/%E7%BB%BF%E7%9A%AE%E4%B9%A6.png?raw=true)

通过词云的可以看到大部分的网友都觉得这部电影还不错，但是也还很多人认为它能获得奥斯卡最佳影片是因为种族平等的政治正确。
