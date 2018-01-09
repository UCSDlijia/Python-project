# Python-project
from urllib import request
#import pip    
#def install(package):
#   pip.main(['install', package])

#install('BeautifulSoup4')
#先爬电影信息
def getNowPlayingMovie_list():  
    resp = request.urlopen("https://movie.douban.com/cinema/nowplaying/hangzhou/")
    html_data = resp.read().decode('utf-8')
    from bs4 import BeautifulSoup as bs
    soup=bs(html_data,'html.parser')
    nowplaying_movie = soup.find_all('div', id='nowplaying')
    nowplaying_movie_list= nowplaying_movie[0].find_all('li',class_='list-item')
    #print(nowplaying_movie_list[0])
    nowplaying_list = []
    for item in nowplaying_movie_list:
        nowplaying_dict={} 
        nowplaying_dict['id']=item['data-subject']
        for tag_img_item in item.find_all('img'):
            nowplaying_dict['name']=tag_img_item['alt']
            nowplaying_list.append(nowplaying_dict)
    return nowplaying_list
#print(nowplaying_list[0])
#要爬评论了
def getCommentsById(movieId,pageNum):
    eachCommentList=[];
    if pageNum>0:
        start=(pageNum-1)*20
    else:
        return False
    requrl= 'https://movie.douban.com/subject/' + movieId +'/comments'+'?'+'start='+ str(start)  +'&limits=20'
    resp=request.urlopen(requrl)
    html_data = resp.read().decode('utf-8')
    soup=bs(html_data,'html.parser')
    comment_div_lits=soup.find_all('div',class_="comment")
    for item in comment_div_lits:
        if item.find_all('p')[0].string is not None:
            eachCommentList.append(item.find_all('p')[0].string)
    return eachCommentList   
#print(eachCommentList)
commentList=[]
NowPlayingMovie_List= getNowPlayingMovie_list()
for i in range(10):
    num=i+1
    commentList_temp=getCommentsById(NowPlayingMovie_List[0]['id'],num)
    commentList.append(commentList_temp)
comments = ''
for k in range(len(commentList)):
    comments = comments+(str(commentList[k])).strip()
# print(comments)
#清理标点
import re
pattern=re.compile(r'[\u4e00-\u9fa5]+')#[\u4e00-\u9fa5]用来匹配所有中文
filterdata = re.findall(pattern,comments)
cleaned_comments=''.join(filterdata)
#print(cleaned_comments)
import jieba
import pandas as pd
segment = jieba.lcut(cleaned_comments)
words_df=pd.DataFrame({'segment':segment})
words_df.head()
stopwords=pd.read_csv("stopwords.txt",index_col=False,quoting=3,names=['stopword'],encoding='utf-8')
words_df=words_df[~words_df.segment.isin(stopwords.stopword)]
import numpy
words_stat=words_df.groupby(by=['segment'])['segment'].agg({"计数":numpy.size})
words_stat=words_stat.reset_index().sort_values(by=["计数"],ascending=False)
words_stat.head()

import matplotlib.pyplot as plt
import matplotlib
%matplotlib inline
matplotlib.rcParams['figure.figsize']=(10.0,5.0)

from wordcloud import WordCloud
wordcloud=WordCloud(font_path="simhei.ttf",background_color="white",max_font_size=80)
word_frequence= {x[0]:x[1] for x in words_stat.head(1000).values}
word_frequence_list=[]
for key in word_frequence:
    temp=(key,word_frequence[key])
    word_frequence_list.append(temp)
wordcloud=wordcloud.fit_words(dict(word_frequence_list))
plt.imshow(wordcloud)
plt.axis('off')
plt.show()
