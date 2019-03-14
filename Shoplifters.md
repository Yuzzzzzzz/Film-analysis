### 引言
最近在学习用python做数据分析，所以想着把自己觉得不错一些电影拿来做做练习，爬取这些电影的评论做一下简单的分析。

### 环境
本文用到python3.6+pycharm，用到了requests、re、BeautifulSoup、jieba、matplotlib、wordcloud等库。



第一次就拿是枝裕和导演的《小偷家族》来试试手，是枝裕和是我最喜欢的日本导演，该片也荣获第71届戛纳电影节金棕榈，自己看完这部影片后感觉意犹未尽，接下来来看看豆瓣的网友对这部的感受是什么样的。
![](https://github.com/hzy-xiaoyuzhou/imag/blob/master/Shoplifters_image/1.png?raw=true)
本文只是简单的对评论进行分析，所以只用爬取豆瓣网友对该片的评论即可。

### 一、数据获取
先打开豆瓣关于小偷家族评论的详情页（https://movie.douban.com/subject/27622447/）
可以看到豆瓣的影评分为短评和长评，长评带有大量的剧情分析而我更感兴趣的是观众们看完电影的感受，所以这里我们只爬取短评。


观察短评页的url，往后一页翻可以发现url只有箭头指的值在改变，所以爬取时只用改变这个值就可以爬取所有页面的评论了
![](https://github.com/hzy-xiaoyuzhou/imag/blob/master/Shoplifters_image/2.png?raw=true)



接着对页面进行分析，按F12打开开发者模式，可以找到评论文本字段的class为short
![](https://github.com/hzy-xiaoyuzhou/imag/blob/master/Shoplifters_image/3.png?raw=true)

```python
import requests
import re
from bs4 import BeautifulSoup

#返回html页面
def get_one_page(url):
    response = requests.get(url)
    if response.status_code == 200:
        return response.text
    return None

#抓取评论字段
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
        with open('review.txt','a',encoding='utf-8') as f:
            f.write(li+'\n\n')
            f.close()


if __name__ == '__main__':
    for i in range(0,200,20):
        s=str(i)
        douban = 'https://movie.douban.com/subject/27622447/comments?start='+s+'&limit=20&sort=new_score&status=P'
        html = get_one_page(douban)
        print('正在抓取第'+str(int(i/20+1))+'页评论！')
        parser_one_page(html)


```

点开review.txt发现爬虫已经把短评保存到里面了
![](https://github.com/hzy-xiaoyuzhou/imag/blob/master/Shoplifters_image/4.png?raw=true)

### 二、数据分析
将刚刚爬到的评论用jieba分词，然后用wordcloud做一个词云图来展示网友们对影片的评论。
```python
import pickle
import jieba
import matplotlib.pyplot as plt
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

comment = []

#载入数据
with open('review.txt',mode='r',encoding='utf-8') as f:
    rows=f.readlines()
    #print(rows)
    for row in rows:
        comment.append(row.replace('\n',''))


#分词
comment_after_split = jieba.cut(str(comment),cut_all=False)

wl_space_split=' '.join(comment_after_split)

#导入背景图
background_Imag = plt.imread('E:\\Imag\\1.jpg')

#停用词
stopword = STOPWORDS.copy()
stopword.add("电影")
stopword.add("是枝裕和")
stopword.add ("最")
stopword.add("你")
stopword.add("我")
stopword.add("感觉")
stopword.add("觉得")
stopword.add("一部")
stopword.add("一个")
stopword.add("枝裕")
stopword.add("没有")
stopword.add("他们")

#设置词云参数
wc = WordCloud(width=2048,height=1536,background_color='white',
 mask=background_Imag,font_path="E:\\font\\msyhbd.ttc",stopwords=stopword,max_font_size=400,
 random_state=50)
wc.generate_from_text(wl_space_split)
img_colors=ImageColorGenerator(background_Imag)
wc.recolor(color_func=img_colors)
plt.imshow(wc)
plt.axis('off')     #不显示坐标
plt.show()
```
词云图
![](https://github.com/hzy-xiaoyuzhou/imag/blob/master/Shoplifters_image/%E5%B0%8F%E5%81%B7%E5%AE%B6%E6%97%8F.png?raw=true)
