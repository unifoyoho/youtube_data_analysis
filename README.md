# youtube_data_analysis
youtube视频趋势分析结果
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import json
%matplotlib inline
#读取CA数据

data_CA = pd.read_csv('D:/学习/kaggle/youtube视频分析/CAvideos.csv')
data_DE = pd.read_csv('D:/学习/kaggle/youtube视频分析/DEvideos.csv')
data_FR = pd.read_csv('D:/学习/kaggle/youtube视频分析/FRvideos.csv')
data_GB = pd.read_csv('D:/学习/kaggle/youtube视频分析/GBvideos.csv')
data_IN = pd.read_csv('D:/学习/kaggle/youtube视频分析/INvideos.csv')
data_JP = pd.read_csv('D:/学习/kaggle/youtube视频分析/JPvideos.csv')
data_JP = data_JP.loc[:,~data_JP.columns.str.contains('^Unnamed')]
data_KR = pd.read_csv('D:/学习/kaggle/youtube视频分析/KRvideos.csv')
data_KR = data_KR.loc[:,~data_KR.columns.str.contains('^Unnamed')]
data_MX = pd.read_csv('D:/学习/kaggle/youtube视频分析/MXvideos.csv')
data_RU = pd.read_csv('D:/学习/kaggle/youtube视频分析/RUvideos.csv')
data_RU = data_RU.loc[:,~data_RU.columns.str.contains('^Unnamed')]
data_US = pd.read_csv('D:/学习/kaggle/youtube视频分析/USvideos.csv')
data_US = data_US.loc[:,~data_US.columns.str.contains('^Unnamed')]
#添加国家名称
data_CA['country'] = 'Canada'
data_DE['country'] = 'Germany'
data_FR['country'] = 'France'
data_GB['country'] = 'Great Britain'
data_IN['country'] = 'India'
data_JP['country'] = 'Japan'
data_KR['country'] = 'Korea'
data_MX['country'] = 'Mexico'
data_RU['country'] = 'Russia'
data_US['country'] = 'America'
# 处理数据
# 合并数据集
data_full = pd.concat([data_CA,data_DE,data_FR,data_GB,data_IN,data_JP,data_KR,
                     data_MX,data_RU,data_US],axis=0)
pd.set_option('display.float_format',lambda x:'%.3f'%x)
data_full.info(verbose = True)
data_publish_time = pd.DataFrame(data_full['publish_time'].str.split('T',expand=True,n=1))
data_full = pd.concat([data_full,data_publish_time],axis=1)
#分割publishi_data列并修改列名称
data_full.reset_index(drop=True,inplace=True)
data_full.rename(columns={0:'publish_date'},inplace=True)
data_full.isnull().sum()
# like~description 缺失的数据展示
miss_data = data_full[data_full['likes'].isnull()]
#删除冗余列
data_full.drop(['publish_time',1],axis=1,inplace=True)
#删除冗余行
data_full.drop(index=[320120,321672],axis=0,inplace=True)
data_full['publish_date'] = pd.to_datetime(data_full['publish_date'])
data_full['trending_date'] = pd.to_datetime(data_full['trending_date'],format='%y.%d.%m')
data_full['video_id'].nunique()
data_full[data_full['views'].isin([' а также сборы трав и питания необходимо'])]
data_full[data_full['views'].isin([' при этом'])]
data_full[data_full['views'].isin([' то значит'])]
data_full.drop(index=[320522],inplace=True)
data_full.drop(index=[328509],inplace=True)
data_full.drop(index=[295963],inplace=True)
data_full['views'].astype(int)
data_full['likes'] = data_full['likes'].astype(int)
data_full['dislikes'] = data_full['dislikes'].astype('int')
data_full['comment_count'] = data_full['comment_count'].astype('int')
#裁剪年-月
data_full['trending_date_year_month'] = data_full['trending_date'].dt.to_period('M')
data_json = pd.read_json('D://学习/kaggle/youtube视频分析/CA_category_id.json')
data_json_cag = data_json['items']
print(data_json_cag[0])
category_data = data_json_cag[0]['snippet']['title']

category_id = []
category_id_num = []
snippet = []
for i in data_json_cag:
    category_id.append(i['id'])
    snippet.append(i['snippet'])
title=[]   
data_json_cag['snippet'] = np.array(snippet)
for j in data_json_cag['snippet']:
    title.append(j['title'])
category_id_num = []
for i in category_id:
    i = eval(i)
    category_id_num.append(i)
category_id_num    
data_full['category_id'] = data_full['category_id'].astype(object)
dictionary = dict(zip(category_id_num,title))
data_full['category'] =data_full['category_id'].map(dictionary)
#数据分析：
#遍历每一个月份，按照category分组统计channel_title
CA_cat_cot = data_full[data_full['country']=='Canada'].groupby(['trending_date_year_month','category'])['category'].count()
CA_cat_cot_DF = pd.DataFrame(CA_cat_cot)
CA_cat_cot_DF = CA_cat_cot_DF.rename(columns={'category':'category_count'})
CA_cat_cot_DF = CA_cat_cot_DF.reset_index()
cag_count = CA_cat_cot_DF.groupby(['trending_date_year_month'])['category_count'].sum()
cag_count = pd.DataFrame(cag_count).reset_index()
cag_count = cag_count.rename(columns={'category_count':'month_count'})
CA_cat_cot_DF
#计算每个类别每个月份占比
CA_data_merge = pd.merge(CA_cat_cot_DF,cag_count,on='trending_date_year_month',how='inner')
CA_data_merge
CA_data_merge['percent%'] = CA_data_merge['category_count'] / CA_data_merge['month_count'] * 100
# CA_data_merge['percent'] = CA_data_merge['percent'].apply(lambda x: format(x,'.2%'))
CA_data_merge
a = list(CA_data_merge['trending_date_year_month'].astype(str))
trending_data_year_month_unique = CA_data_merge['trending_date_year_month'].unique()
#使用rcparams添加matplotlib显示中文字体：
plt.rcParams['font.family'] = 'SimHei'
sns.palplot(sns.color_palette('hls',16))# 每个月热门上榜视频的变化情况
plt.figure(figsize=(25,16))
plt.grid()
plt.ylim((0,40))
sns.lineplot(x=a,y='percent%',data=CA_data_merge,hue='category',palette=sns.color_palette('hls',16))
