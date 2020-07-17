# Keyword-analysis-and-collocation
This part will use keyword analysis to figure out the most frequent words and its collocation

# the first part is about the keyword analysis
#import the package
```
import pandas as pd
import jieba
from collections import Counter
import re
```
#import the stopword list
```
def stopwordslist(filepath):  
    stopwords = [line.strip() for line in open(filepath, 'r', encoding='utf-8').readlines()]  
    return stopwords  
stopwords = stopwordslist('stopwords-zh.txt')
```
#import the data
```
data1 = pd.read_csv('dup_com_id.csv')
```
#clean the data, drop duplicates and rearrange
```
data1 = data1[data1.duplicated()]
data1.index = range(len(data1))
```
#use re to delete the non-Chinese text
```
pattern = re.compile(r'[^\u4e00-\u9fa5]')
data1['comment_content'] = data1['comment_content'].apply(lambda x: re.sub(pattern, '', x))
```
#use jieba to clean the data
```
data1['content_cut'] = data1['comment_content'].apply(lambda x:list(jieba.cut(x,cut_all=False)))
data1['content_cut'] = data1['content_cut'].apply(lambda x: [item for item in x if item not in stopwords and item != ' '])
```
#use counter to count the word
```
cyw_word=Counter()
for word in biaoji2['content_cut']:
    for word2 in word:
        cyw_word[word2]+=1
```
#Select the first 200 words with the most occurrences
```
word_2 = cyw_word.most_common(200)
word2=[]
num2=[]
for i,j in word_2:
    word2.append(i)
    num2.append(j)
w_22 = {
    'Words':word2,
    'num':num2
}
w_22 = pd.DataFrame(w_22)
```
 #save it
```
biaoji2.to_csv("duplicated_ID.csv",index=False,sep=',',mode='a')
```
# The second part is about collocations
#import packages
```
import pandas as pd
import jieba
from collections import Counter
import re
import jieba.posseg as pseg
import numpy as np
```
#import the stopword list
```
def stopwordslist(filepath):  
    stopwords = [line.strip() for line in open(filepath, 'r', encoding='utf-8').readlines()]  
    return stopwords  
stopwords = stopwordslist('stopwords-zh.txt')
```
#import the data
```
data = pd.read_csv('dup_com_id.csv')
```
#clean it
```
data = data.dropna(axis=0,how='any')
data = data.drop(columns = ["Unnamed: 0"])
```
#to check the content with the specific word
```
data[data['comment_content'].str.contains("尴尬")]
```
#if you want to check the content with two or more words together
```
xh = data[data['comment_content'].str.contains("美国")]
xh.index = range(len(xh))

rb = data[data['comment_content'].str.contains("中国")]
rb.index = range(len(rb))

xh_rb = pd.merge(rb, xh, how='inner', on='comment_content')
```
#if you want to get the abject about this subject
```
#first clean it
pattern = re.compile(r'[^\u4e00-\u9fa5]')
data['comment_content'] = data['comment_content'].apply(lambda x: re.sub(pattern, '', x))

#use package form jieba---pseg.cut(x)
data['content_cut'] = data['comment_content'].apply(lambda x:list(pseg.cut(x)))

#put them together
word_list=[]
for w in data['content_cut']:
    for ww in w:
        if ww.word not in stopwords and ww.word != ' ':
            word_list.append(ww)
```
#set a def about adj
```
def distant_count_a(word,text):
    window_size = 9
    ww0 = []
    ww1 = []
    dis = []
    cc = []
    
    word_pair_distance_counts_before = Counter()
    
    for i in range(len(text)-1):
        for distance in range(1,window_size):
            if i + distance < len(text):
                (w0,w1) = (text[i-distance].word,text[i].word)
                if w1 == word and text[i-distance].flag == 'a' and len(w0)>1: 
                    word_pair_distance_counts_before[(w0,w1,distance)] += 1


    for (w0,w1,distance),c in word_pair_distance_counts_before.most_common(100):
        #print("%s\t%s\t%d\t%d"%(w0,w1,distance,c))
        ww0.append(w0)
        ww1.append(w1)
        dis.append(distance)
        cc.append(c)
     
    data_adj = {
        'w0':ww0,
        #'w1':ww1,
        "distance":dis,
        'number':cc
    }
    return data_adj
```
#set a def about verb
```
def distant_count_v(word,text):
    window_size = 9
    ww00 = []
    ww11 = []
    dis1 = []
    cc1 = []
    
    word_pair_distance_counts_before = Counter()
    
    for i in range(len(text)-1):
        for distance in range(1,window_size):
            if i + distance < len(text):
                (w0,w1) = (text[i-distance].word,text[i].word)
                if w1 == word and text[i-distance].flag == 'v' and len(w0)>1: 
                    word_pair_distance_counts_before[(w0,w1,distance)] += 1


    for (w0,w1,distance),c in word_pair_distance_counts_before.most_common(100):
        #print("%s\t%s\t%d\t%d"%(w0,w1,distance,c))
        ww00.append(w0)
        ww11.append(w1)
        dis1.append(distance)
        cc1.append(c)
     
    data_v = {
        'w0':ww00,
        #'w1':ww1,
        "distance":dis1,
        'number':cc1
    }
    return data_v
```
#practice
```
tt_a = distant_count_a('中国',word_list)
```
#conclusion
```
tt_a = pd.DataFrame(tt_a)
tt_a = tt_a.groupby('w0')
agg = tt_a['number'].agg([np.sum])
agg = agg.sort_values('sum',ascending=False)
```


 
