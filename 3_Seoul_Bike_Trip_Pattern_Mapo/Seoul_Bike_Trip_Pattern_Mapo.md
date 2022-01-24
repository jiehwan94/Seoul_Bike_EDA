## Intro

In our last episode, we explored the trips taken in February of 2021.

In this post, let's carry out a deeper analysis of Mapo county in specific. 

## Data


```python
import pandas as pd
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns
import folium
from folium import plugins
import missingno as mnso
import warnings
warnings.filterwarnings('ignore')
mpl.rcParams['axes.unicode_minus'] = False
# plt.rcParams["font.family"] = 'NanumGothic'
plt.rc('font', family='NanumGothic')
```


```python
import matplotlib
import matplotlib.font_manager as fm
fm.get_fontconfig_fonts()
font_location = 'C:/Users/82104/Desktop/Side Projects/Seoul_Bike/NanumGothic.ttf'
font_name = fm.FontProperties(fname=font_location).get_name()
matplotlib.pyplot.rc('font', family=font_name)


%matplotlib inline
```


```python
mpl.rcParams['axes.unicode_minus'] = False
plt.rcParams["font.family"] = 'NanumGothic'
```


```python
print(mpl.rcParams['font.family'])
print(plt.rcParams['font.family'])
```

    ['NanumGothic']
    ['NanumGothic']
    


```python
import json
geo_path = 'C:/Users/82104/Desktop/Side Projects/Seoul_Bike/New_Seoul_Bike/Data/Station/seoul_municipalities_geo_simple.json'
geo_str = json.load(open(geo_path, encoding='utf-8'))
```


```python
trips = pd.read_csv('C:/Users/82104/Desktop/Side Projects/Seoul_Bike/New_Seoul_Bike/Data/Trips/공공자전거 대여이력 정보_2021.02.csv', encoding = 'euc-kr')
trips.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2/1/2021 0:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2/1/2021 0:03</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>736.02</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2/1/2021 0:01</td>
      <td>1977</td>
      <td>천왕역 1번 출입구 앞</td>
      <td>2/1/2021 0:04</td>
      <td>1981</td>
      <td>천왕이펜하우스5단지 앞</td>
      <td>3.0</td>
      <td>856.09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2/1/2021 0:02</td>
      <td>1358</td>
      <td>정릉도서관 앞</td>
      <td>2/1/2021 0:05</td>
      <td>1360</td>
      <td>정릉역</td>
      <td>2.0</td>
      <td>397.92</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2/1/2021 0:00</td>
      <td>1211</td>
      <td>방이삼거리</td>
      <td>2/1/2021 0:05</td>
      <td>2639</td>
      <td>석촌역 8번출구</td>
      <td>4.0</td>
      <td>1015.50</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2/1/2021 0:01</td>
      <td>1721</td>
      <td>창동역 2번출구</td>
      <td>2/1/2021 0:06</td>
      <td>1708</td>
      <td>보건소사거리(다비치안경창동점)</td>
      <td>4.0</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
trips['start_time'] = pd.to_datetime(trips['start_time'])
trips['end_time'] = pd.to_datetime(trips['end_time'], errors = 'coerce')
```


```python
trips.shape
```




    (1048575, 8)




```python
trips = trips.dropna()
```


```python
trips.shape
```




    (1048558, 8)




```python
trips['start_hour'] = trips['start_time'].dt.hour
trips['start_day'] = trips['start_time'].dt.day
trips['start_dayofweek'] = trips['start_time'].dt.dayofweek
trips['end_hour'] = trips['end_time'].dt.hour
```


```python
trips.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1048570</th>
      <td>2021-02-26 15:21:00</td>
      <td>1108</td>
      <td>공항시장역 2번출구 뒤</td>
      <td>2021-02-26 15:36:00</td>
      <td>1153</td>
      <td>발산역 1번, 9번 인근 대여소</td>
      <td>15.0</td>
      <td>2524.64</td>
      <td>15</td>
      <td>26</td>
      <td>4</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1048571</th>
      <td>2021-02-26 15:29:00</td>
      <td>2610</td>
      <td>여흥레이크빌 앞 (석촌호수 까페거리)</td>
      <td>2021-02-26 15:36:00</td>
      <td>1210</td>
      <td>롯데월드타워(잠실역2번출구 쪽)</td>
      <td>7.0</td>
      <td>1086.82</td>
      <td>15</td>
      <td>26</td>
      <td>4</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1048572</th>
      <td>2021-02-26 14:50:00</td>
      <td>2000</td>
      <td>신도림4차 e편한세상 아파트 1109동 앞</td>
      <td>2021-02-26 15:36:00</td>
      <td>3789</td>
      <td>염창나들목</td>
      <td>46.0</td>
      <td>6310.07</td>
      <td>14</td>
      <td>26</td>
      <td>4</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1048573</th>
      <td>2021-02-26 14:41:00</td>
      <td>2648</td>
      <td>헬리오시티 112동 앞</td>
      <td>2021-02-26 15:36:00</td>
      <td>2625</td>
      <td>가락1동 주민센터</td>
      <td>54.0</td>
      <td>11315.31</td>
      <td>14</td>
      <td>26</td>
      <td>4</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1048574</th>
      <td>2021-02-26 15:29:00</td>
      <td>1946</td>
      <td>구로역 광장</td>
      <td>2021-02-26 15:36:00</td>
      <td>1956</td>
      <td>도야미리숯불갈비 앞</td>
      <td>7.0</td>
      <td>1063.86</td>
      <td>15</td>
      <td>26</td>
      <td>4</td>
      <td>15</td>
    </tr>
  </tbody>
</table>
</div>




```python
trips.dtypes
```




    start_time            datetime64[ns]
    start_station_id               int64
    start_station_name            object
    end_time              datetime64[ns]
    end_station_id                object
    end_station_name              object
    duration                     float64
    distance                     float64
    start_hour                     int64
    start_day                      int64
    start_dayofweek                int64
    end_hour                       int64
    dtype: object




```python
import missingno as msno
msno.matrix(trips)
```




    <AxesSubplot:>



    findfont: Font family ['NanumGothic'] not found. Falling back to DejaVu Sans.
    findfont: Font family ['NanumGothic'] not found. Falling back to DejaVu Sans.
    findfont: Font family ['NanumGothic'] not found. Falling back to DejaVu Sans.
    


    
![png](output_15_2.png)
    



```python
s_eng = pd.read_csv('C:/Users/82104/Desktop/Side Projects/Seoul_Bike/New_Seoul_Bike/Data/Station/Seoul_Distict_ID_English.csv', encoding = 'euc-kr')
s_eng = s_eng.drop(columns=['county_id','lat','long'])
s_kr = pd.read_csv('C:/Users/82104/Desktop/Side Projects/Seoul_Bike/New_Seoul_Bike/Data/Station/station_df_facilities_info.csv', encoding = 'cp949')
station = pd.merge(s_eng, s_kr, how = "inner", on = "county")
station.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county</th>
      <th>county_eng</th>
      <th>station_id</th>
      <th>station_name</th>
      <th>lat</th>
      <th>long</th>
      <th>station_install_date</th>
      <th>capa</th>
      <th>rent_type</th>
      <th>dong</th>
      <th>bus_min_dist</th>
      <th>num_bus_within_150m</th>
      <th>subway_500m</th>
      <th>culture_500m</th>
      <th>school_500m</th>
      <th>market_500m</th>
      <th>tour_500m</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1702</td>
      <td>녹천역 1번출구 앞</td>
      <td>37.646172</td>
      <td>127.050560</td>
      <td>5/8/2017</td>
      <td>10</td>
      <td>LCD</td>
      <td>번1동</td>
      <td>65.854878</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1703</td>
      <td>도봉산광역환승센터앞</td>
      <td>37.689720</td>
      <td>127.045197</td>
      <td>5/11/2017</td>
      <td>15</td>
      <td>LCD</td>
      <td>창1동</td>
      <td>62.015531</td>
      <td>6</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1705</td>
      <td>도봉구청 정문앞</td>
      <td>37.669224</td>
      <td>127.046516</td>
      <td>5/8/2017</td>
      <td>8</td>
      <td>LCD</td>
      <td>도봉1동</td>
      <td>30.788507</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1706</td>
      <td>기업은행 앞</td>
      <td>37.665665</td>
      <td>127.042671</td>
      <td>6/27/2017</td>
      <td>10</td>
      <td>LCD</td>
      <td>방학1동</td>
      <td>33.531927</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1707</td>
      <td>도봉구민회관</td>
      <td>37.654461</td>
      <td>127.038513</td>
      <td>5/8/2017</td>
      <td>15</td>
      <td>LCD</td>
      <td>방학1동</td>
      <td>31.866318</td>
      <td>5</td>
      <td>0</td>
      <td>6</td>
      <td>5</td>
      <td>0</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
station.shape
```




    (2154, 17)




```python
station = station.astype({
    "county": "category",
    "county_eng": "category",
    "station_id": "category",
    "lat": "category",
    "long": "category"
})
```

Let's join Rent station first


```python
trips.shape
```




    (1048558, 12)




```python
station.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county</th>
      <th>county_eng</th>
      <th>station_id</th>
      <th>station_name</th>
      <th>lat</th>
      <th>long</th>
      <th>station_install_date</th>
      <th>capa</th>
      <th>rent_type</th>
      <th>dong</th>
      <th>bus_min_dist</th>
      <th>num_bus_within_150m</th>
      <th>subway_500m</th>
      <th>culture_500m</th>
      <th>school_500m</th>
      <th>market_500m</th>
      <th>tour_500m</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1702</td>
      <td>녹천역 1번출구 앞</td>
      <td>37.646172</td>
      <td>127.050560</td>
      <td>5/8/2017</td>
      <td>10</td>
      <td>LCD</td>
      <td>번1동</td>
      <td>65.854878</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1703</td>
      <td>도봉산광역환승센터앞</td>
      <td>37.689720</td>
      <td>127.045197</td>
      <td>5/11/2017</td>
      <td>15</td>
      <td>LCD</td>
      <td>창1동</td>
      <td>62.015531</td>
      <td>6</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1705</td>
      <td>도봉구청 정문앞</td>
      <td>37.669224</td>
      <td>127.046516</td>
      <td>5/8/2017</td>
      <td>8</td>
      <td>LCD</td>
      <td>도봉1동</td>
      <td>30.788507</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1706</td>
      <td>기업은행 앞</td>
      <td>37.665665</td>
      <td>127.042671</td>
      <td>6/27/2017</td>
      <td>10</td>
      <td>LCD</td>
      <td>방학1동</td>
      <td>33.531927</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>1707</td>
      <td>도봉구민회관</td>
      <td>37.654461</td>
      <td>127.038513</td>
      <td>5/8/2017</td>
      <td>15</td>
      <td>LCD</td>
      <td>방학1동</td>
      <td>31.866318</td>
      <td>5</td>
      <td>0</td>
      <td>6</td>
      <td>5</td>
      <td>0</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
before_merged = len(trips)
df = pd.merge(trips, station[['county','county_eng','station_id','lat','long']],
              left_on="start_station_id",
              right_on="station_id").drop(columns='station_id')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
      <th>county</th>
      <th>county_eng</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-02-01 00:00:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 00:03:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>736.02</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-02-01 00:01:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 00:11:00</td>
      <td>1536</td>
      <td>번동 두산위브 101동 옆</td>
      <td>9.0</td>
      <td>597.75</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-02-01 00:51:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 00:53:00</td>
      <td>1552</td>
      <td>강북구청사거리(던킨도너츠 앞)</td>
      <td>1.0</td>
      <td>140.00</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-02-01 09:34:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 09:36:00</td>
      <td>1569</td>
      <td>수유역2번출구</td>
      <td>2.0</td>
      <td>0.00</td>
      <td>9</td>
      <td>1</td>
      <td>0</td>
      <td>9</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-02-01 09:39:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 10:00:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>21.0</td>
      <td>2505.19</td>
      <td>9</td>
      <td>1</td>
      <td>0</td>
      <td>10</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.shape
```




    (1035715, 16)



Now let's join Return station


```python
df = pd.merge(df, station[['county','county_eng','station_id','lat','long']], 
              left_on="end_station_id", 
              right_on="station_id").drop(columns='station_id')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
      <th>county_x</th>
      <th>county_eng_x</th>
      <th>lat_x</th>
      <th>long_x</th>
      <th>county_y</th>
      <th>county_eng_y</th>
      <th>lat_y</th>
      <th>long_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-02-01 00:00:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 00:03:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>736.02</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-02-01 18:01:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 18:08:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>7.0</td>
      <td>779.26</td>
      <td>18</td>
      <td>1</td>
      <td>0</td>
      <td>18</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-02-01 20:21:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 20:27:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>5.0</td>
      <td>0.00</td>
      <td>20</td>
      <td>1</td>
      <td>0</td>
      <td>20</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-02-01 20:32:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 20:38:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>5.0</td>
      <td>736.02</td>
      <td>20</td>
      <td>1</td>
      <td>0</td>
      <td>20</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-02-03 00:12:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-03 00:14:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>628.99</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(df.info())
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 578407 entries, 0 to 578406
    Data columns (total 20 columns):
     #   Column              Non-Null Count   Dtype         
    ---  ------              --------------   -----         
     0   start_time          578407 non-null  datetime64[ns]
     1   start_station_id    578407 non-null  object        
     2   start_station_name  578407 non-null  object        
     3   end_time            578407 non-null  datetime64[ns]
     4   end_station_id      578407 non-null  object        
     5   end_station_name    578407 non-null  object        
     6   duration            578407 non-null  float64       
     7   distance            578407 non-null  float64       
     8   start_hour          578407 non-null  int64         
     9   start_day           578407 non-null  int64         
     10  start_dayofweek     578407 non-null  int64         
     11  end_hour            578407 non-null  int64         
     12  county_x            578407 non-null  category      
     13  county_eng_x        578407 non-null  category      
     14  lat_x               578407 non-null  category      
     15  long_x              578407 non-null  category      
     16  county_y            578407 non-null  category      
     17  county_eng_y        578407 non-null  category      
     18  lat_y               578407 non-null  category      
     19  long_y              578407 non-null  category      
    dtypes: category(8), datetime64[ns](2), float64(2), int64(4), object(4)
    memory usage: 64.3+ MB
    None
    


```python
df.shape
```




    (578407, 20)




```python
mapo = df[(df['county_eng_x'] == 'Mapo') | (df['county_eng_y'] == 'Mapo')]
print(mapo.shape)
mapo.head()
```

    (44811, 20)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
      <th>county_x</th>
      <th>county_eng_x</th>
      <th>lat_x</th>
      <th>long_x</th>
      <th>county_y</th>
      <th>county_eng_y</th>
      <th>lat_y</th>
      <th>long_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5342</th>
      <td>2021-02-13 15:16:00</td>
      <td>114</td>
      <td>홍대입구역 8번출구 앞</td>
      <td>2021-02-13 17:17:00</td>
      <td>322</td>
      <td>명동성당 앞</td>
      <td>120.0</td>
      <td>11989.94</td>
      <td>15</td>
      <td>13</td>
      <td>5</td>
      <td>17</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.557060</td>
      <td>126.924423</td>
      <td>중구</td>
      <td>Jung</td>
      <td>37.564476</td>
      <td>126.986969</td>
    </tr>
    <tr>
      <th>12612</th>
      <td>2021-02-01 17:43:00</td>
      <td>112</td>
      <td>극동방송국 앞</td>
      <td>2021-02-01 18:41:00</td>
      <td>3600</td>
      <td>사근빗물펌프장 건너편</td>
      <td>57.0</td>
      <td>15975.13</td>
      <td>17</td>
      <td>1</td>
      <td>0</td>
      <td>18</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.549202</td>
      <td>126.923203</td>
      <td>성동구</td>
      <td>Seongdong</td>
      <td>37.560980</td>
      <td>127.049300</td>
    </tr>
    <tr>
      <th>12679</th>
      <td>2021-02-02 14:10:00</td>
      <td>118</td>
      <td>광흥창역 2번출구 앞</td>
      <td>2021-02-02 15:13:00</td>
      <td>3600</td>
      <td>사근빗물펌프장 건너편</td>
      <td>62.0</td>
      <td>15612.86</td>
      <td>14</td>
      <td>2</td>
      <td>1</td>
      <td>15</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.547733</td>
      <td>126.931763</td>
      <td>성동구</td>
      <td>Seongdong</td>
      <td>37.560980</td>
      <td>127.049300</td>
    </tr>
    <tr>
      <th>18645</th>
      <td>2021-02-20 17:43:00</td>
      <td>495</td>
      <td>염리초등학교 앞</td>
      <td>2021-02-20 19:22:00</td>
      <td>1382</td>
      <td>래미안월곡아파트 입구</td>
      <td>98.0</td>
      <td>19559.57</td>
      <td>17</td>
      <td>20</td>
      <td>5</td>
      <td>19</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.542431</td>
      <td>126.946999</td>
      <td>성북구</td>
      <td>Seongbuk</td>
      <td>37.610440</td>
      <td>127.036900</td>
    </tr>
    <tr>
      <th>19013</th>
      <td>2021-02-13 13:58:00</td>
      <td>147</td>
      <td>마포역 4번출구 뒤</td>
      <td>2021-02-13 15:21:00</td>
      <td>1684</td>
      <td>태릉입구역 5번출구</td>
      <td>82.0</td>
      <td>22454.48</td>
      <td>13</td>
      <td>13</td>
      <td>5</td>
      <td>15</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.539272</td>
      <td>126.945915</td>
      <td>노원구</td>
      <td>Nowon</td>
      <td>37.618320</td>
      <td>127.075630</td>
    </tr>
  </tbody>
</table>
</div>



## 1. # of Rents

### 1.1. # of Rents by Hour


```python
def draw_usage(df, region):
    
    fig, axes = plt.subplots(1, 2, figsize=(24, 5))
    for i, dayofweek in enumerate(["Weekday", "Weekend"]):
        if dayofweek == "Weekday":
            _dayofweek = set(range(0,5))
        elif dayofweek == "Weekend":
            _dayofweek = set(range(5,7))
        
        by_hour_rental = df[(df['start_dayofweek'].isin(_dayofweek)) & (df['county_eng_x'] == region)].groupby('start_hour').size() / len(_dayofweek)
        by_hour_return = df[(df['start_dayofweek'].isin(_dayofweek)) & (df['county_eng_y'] == region)].groupby('end_hour').size() / len(_dayofweek)
        by_hour = pd.DataFrame(data={
            "Rent": by_hour_rental,
            "Return": by_hour_return
        }).plot(style='.-', rot=0, title="%s, # of Rents by Hour (Rent, Return)" %dayofweek, ax=axes[i])
        plt.sca(axes[i])
        plt.xticks(range(0, 24, 1))
        plt.xlabel("Hour")
        plt.box(False)
        plt.legend(frameon=False)
    plt.show()
```


```python
draw_usage(mapo, "Mapo")
```

    findfont: Font family ['NanumGothic'] not found. Falling back to DejaVu Sans.
    findfont: Font family ['NanumGothic'] not found. Falling back to DejaVu Sans.
    


    
![png](output_31_1.png)
    


- During weekdays, 
    - Rents/Returns hit the peak at 8am and 6pm which is similar to the pattern of Seoul as a whole.  
    - Given that there are more returns at 8am and more rents between 16pm and 18pm, we can infer that many users are commuting to Mapo county from other counties.
    
- During weekends,
    - \# of Rents keeps rising early afternoon (until 3pm)
    - \# of Returns shows a similar pattern with 1 hour lag and hits the peak at 4pm
    - The pattern may be different throughout the year depending on the temperature and sunset time.

### 1.2 Distance & Trip Duration by Hour


```python
def draw_dist_time(df, region):
    
    fig, axes = plt.subplots(1, 2, figsize=(24, 5))
    
    for i, (var, unit) in enumerate(zip(["distance", "duration"], ["m", "Minute"])):
        by_hour_rental_weekday = df[(df['start_dayofweek'].isin(set(range(0,5)))) & (df['county_eng_x'] == region)].groupby('start_hour')[var].median()
        by_hour_rental_weekend = df[(df['start_dayofweek'].isin(set(range(5,7)))) & (df['county_eng_x'] == region)].groupby('start_hour')[var].median()
        by_hour = pd.DataFrame(data={
            "Weekday": by_hour_rental_weekday,
            "Weekend": by_hour_rental_weekend
        }).plot(style='.-', rot=0, title="Median %s by Hour (%s)" %(var, unit), ax=axes[i], color=["C3", "C4"])
        plt.sca(axes[i])
        plt.xticks(range(0, 24, 1))
        plt.xlabel("Hour")
        plt.box(False)
        plt.legend(frameon=False)
        
        print("[%s]" %var)
        print("Weekday Median %s: %d%s" %(var, by_hour_rental_weekday.mean(), unit))
        print("Weekend Median %s: %d%s" %(var, by_hour_rental_weekend.mean(), unit))
        
    plt.show()
```


```python
draw_dist_time(mapo, "Mapo")
```

    [distance]
    Weekday Median distance: 1809m
    Weekend Median distance: 2686m
    [duration]
    Weekday Median duration: 16Minute
    Weekend Median duration: 25Minute
    


    
![png](output_35_1.png)
    


- The distance and trip duration have similar pattern in both weekdays and weekends
- The overall # of is higher during weekends.
- \# of Rents/Returns is highest between 12pm and 4pm. This might be because that's the warmest time of the day in February.

## 2. User Analysis


```python
users = pd.read_csv('C:/Users/82104/Desktop/Side Projects/Seoul_Bike/New_Seoul_Bike/Data/Trips/공공자전거 이용정보(시간대별)_21.2.csv', 
                    encoding = 'euc-kr')

users.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_hour</th>
      <th>station_id</th>
      <th>start_station_name</th>
      <th>sex</th>
      <th>age_group</th>
      <th>rent_cnt</th>
      <th>kcal</th>
      <th>CO2</th>
      <th>distance</th>
      <th>duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>1534</td>
      <td>북서울 꿈의 숲 동문</td>
      <td>NaN</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>136.17</td>
      <td>1.23</td>
      <td>5290.11</td>
      <td>51.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>1171</td>
      <td>염창동 새마을금고 건너편 (모닝글로리)</td>
      <td>NaN</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>20.69</td>
      <td>0.16</td>
      <td>669.97</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>3421</td>
      <td>혜화역 1번출구</td>
      <td>NaN</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>2.86</td>
      <td>0.03</td>
      <td>111.20</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>791</td>
      <td>현대하이페리온</td>
      <td>NaN</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>8.2</td>
      <td>0.07</td>
      <td>318.39</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>2741</td>
      <td>마곡수명산파크5-6단지</td>
      <td>NaN</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>66.57</td>
      <td>0.63</td>
      <td>2711.37</td>
      <td>34.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
mapo_users = pd.merge(users, station[['county','county_eng','station_id','lat','long']],
              left_on="station_id",
              right_on="station_id").drop(columns='station_id')
mapo_users = mapo_users[mapo_users['county_eng'] == 'Mapo']
mapo_users.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_hour</th>
      <th>start_station_name</th>
      <th>sex</th>
      <th>age_group</th>
      <th>rent_cnt</th>
      <th>kcal</th>
      <th>CO2</th>
      <th>distance</th>
      <th>duration</th>
      <th>county</th>
      <th>county_eng</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19739</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>성산2교 사거리</td>
      <td>NaN</td>
      <td>20s</td>
      <td>1.0</td>
      <td>13.08</td>
      <td>0.13</td>
      <td>560.00</td>
      <td>4.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19740</th>
      <td>2/1/2021</td>
      <td>9</td>
      <td>성산2교 사거리</td>
      <td>NaN</td>
      <td>20s</td>
      <td>1.0</td>
      <td>32.02</td>
      <td>0.25</td>
      <td>1078.04</td>
      <td>5.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19741</th>
      <td>2/1/2021</td>
      <td>9</td>
      <td>성산2교 사거리</td>
      <td>F</td>
      <td>20s</td>
      <td>1.0</td>
      <td>63.77</td>
      <td>0.62</td>
      <td>2683.98</td>
      <td>17.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19742</th>
      <td>2/1/2021</td>
      <td>9</td>
      <td>성산2교 사거리</td>
      <td>M</td>
      <td>20s</td>
      <td>1.0</td>
      <td>85.64</td>
      <td>0.77</td>
      <td>3327.11</td>
      <td>20.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19743</th>
      <td>2/1/2021</td>
      <td>10</td>
      <td>성산2교 사거리</td>
      <td>F</td>
      <td>30s</td>
      <td>1.0</td>
      <td>92.81</td>
      <td>1.11</td>
      <td>4782.97</td>
      <td>31.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
  </tbody>
</table>
</div>



### 2.1. Usage Comparison by Sex


```python
sex_colors = ['crimson', 'royalblue']
sex_colors_r = list(reversed(sex_colors))
```


```python
users['rent_cnt'] = pd.to_numeric(users['rent_cnt'])
mapo_users['rent_cnt'] = pd.to_numeric(mapo_users['rent_cnt'])
```


```python
users = users[(users['sex'] == "F") | (users['sex'] == "M")]
mapo_users = mapo_users[(mapo_users['sex'] == "F") | (mapo_users['sex'] == "M")]
```


```python
users.groupby('sex').size()
```




    sex
    F    221785
    M    366067
    dtype: int64




```python
mapo_users.groupby('sex').size()
```




    sex
    F    14695
    M    21887
    dtype: int64




```python
use_per_sex = users.groupby('sex')['rent_cnt'].sum()
print(use_per_sex)

fig, axes = plt.subplots(1, 2, figsize=(18, 5))
ax = use_per_sex.plot(kind='bar', title="Men vs. Women (All Counties in Seoul)", rot=0, ax=axes[0], color=sex_colors)
for p in ax.patches:
    left, bottom, width, height = p.get_bbox().bounds
    ax.annotate("%d"%(height), (left+width/2, height+1000), ha='center')
plt.sca(ax)
plt.box(False)
ax = use_per_sex.div(use_per_sex.sum()).plot(kind='pie', title="Men vs. Women (All Counties in Seoul)", autopct='%.1f%%', ax=axes[1], colors=sex_colors)
plt.sca(ax)
plt.box(False)
plt.show()
```

    sex
    F    240891.0
    M    391872.0
    Name: rent_cnt, dtype: float64
    


    
![png](output_46_1.png)
    



```python
use_per_sex = mapo_users.groupby('sex')['rent_cnt'].sum()
print(use_per_sex)

fig, axes = plt.subplots(1, 2, figsize=(18, 5))
ax = use_per_sex.plot(kind='bar', title="Men vs. Women (Mapo County)", rot=0, ax=axes[0], color=sex_colors)
for p in ax.patches:
    left, bottom, width, height = p.get_bbox().bounds
    ax.annotate("%d"%(height), (left+width/2, height+1000), ha='center')
plt.sca(ax)
plt.box(False)
ax = use_per_sex.div(use_per_sex.sum()).plot(kind='pie', title="Men vs. Women (Mapo County)", autopct='%.1f%%', ax=axes[1], colors=sex_colors)
plt.sca(ax)
plt.box(False)
plt.show()
```

    sex
    F    16478.0
    M    23726.0
    Name: rent_cnt, dtype: float64
    


    
![png](output_47_1.png)
    


- Men's usage is greater than women's usage in both Mapo and all counties in Seoul
- The ratio of Men to Women is 59:41 for Mapo compared to 62:38 for all counties in Seoul. **Ratio of women's usage is slightly greater (3%)**

### 2.2. Usage Comparison by Age


```python
age_order = ['~10s', '20s', '30s', '40s', '50s', '60s', '70s~']

def create_CN(n):
    return ["C%d"%i for i in range(n)]
```


```python
users.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_hour</th>
      <th>station_id</th>
      <th>start_station_name</th>
      <th>sex</th>
      <th>age_group</th>
      <th>rent_cnt</th>
      <th>kcal</th>
      <th>CO2</th>
      <th>distance</th>
      <th>duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>116</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>549</td>
      <td>아차산역 3번출구</td>
      <td>F</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>14.06</td>
      <td>0.18</td>
      <td>789.00</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>117</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>1149</td>
      <td>신방화역환승주차장</td>
      <td>F</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>35.15</td>
      <td>0.4</td>
      <td>1707.19</td>
      <td>12.0</td>
    </tr>
    <tr>
      <th>118</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>1256</td>
      <td>문정현대아파트 교차로</td>
      <td>F</td>
      <td>20s</td>
      <td>1.0</td>
      <td>66.64</td>
      <td>0.46</td>
      <td>1979.71</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>119</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>1457</td>
      <td>동원사거리</td>
      <td>F</td>
      <td>20s</td>
      <td>1.0</td>
      <td>32.83</td>
      <td>0.3</td>
      <td>1275.42</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>120</th>
      <td>2/1/2021</td>
      <td>0</td>
      <td>397</td>
      <td>종묘공영주차장 건너편</td>
      <td>F</td>
      <td>20s</td>
      <td>1.0</td>
      <td>50.35</td>
      <td>0.59</td>
      <td>2543.15</td>
      <td>14.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
use_per_age = users.groupby('age_group').size()
# use_per_age = use_per_age.reindex(age_order)
print(use_per_age)

fig, axes = plt.subplots(1, 2, figsize=(18, 5))
ax = use_per_age.plot(kind='bar', title="Rent by age_group (All Counties in Seoul)", rot=0, ax=axes[0])#colors=['blue', 'orange', 'green', 'red', 'purple', 'brown', 'pink'])
for p in ax.patches:
    left, bottom, width, height = p.get_bbox().bounds
    ax.annotate("%d"%(height), (left+width/2, height+700), ha='center')
plt.sca(ax)
plt.box(False)
ax = use_per_age.div(use_per_age.sum()).plot(kind='pie', title="Rent by age_group (All Counties in Seoul)", autopct='%.1f%%', ax=axes[1])
plt.show()
```

    age_group
    20s     206759
    30s     157462
    40s     105165
    50s      65172
    60s      20588
    70s~      3485
    ~10s     29221
    dtype: int64
    


    
![png](output_52_1.png)
    



```python
mapo_use_per_age = mapo_users.groupby('age_group').size()
# use_per_age = use_per_age.reindex(age_order)
print(mapo_use_per_age)

fig, axes = plt.subplots(1, 2, figsize=(18, 5))
ax = mapo_use_per_age.plot(kind='bar', title="Rent by age_group (Mapo)", rot=0, ax=axes[0])#colors=['blue', 'orange', 'green', 'red', 'purple', 'brown', 'pink'])
for p in ax.patches:
    left, bottom, width, height = p.get_bbox().bounds
    ax.annotate("%d"%(height), (left+width/2, height+700), ha='center')
plt.sca(ax)
plt.box(False)
ax = mapo_use_per_age.div(mapo_use_per_age.sum()).plot(kind='pie', title="Rent by age_group (Mapo)", autopct='%.1f%%', ax=axes[1])
plt.show()
```

    age_group
    20s     12663
    30s     10462
    40s      6779
    50s      3631
    60s      1518
    70s~      161
    ~10s     1368
    dtype: int64
    


    
![png](output_53_1.png)
    



```python
comp = pd.DataFrame({'All Counties in Seoul' : [29221, 206759, 157462, 105165, 65172, 20588, 3485],
              'Mapo' : mapo_use_per_age},
              index = age_order)
comp = comp.div(comp.sum()) * 100
comp['diff'] = abs(comp['All Counties in Seoul'] - comp['Mapo'])
comp = comp.sort_values('diff')
```


```python
ax = comp[['All Counties in Seoul', 'Mapo']].plot(kind='barh', figsize=(10, 5), color=['lightgrey', 'grey'], 
                                                  title="Ratio of Age Group (All Counties in Seoul vs. Mapo County)")
for p in ax.patches: 
    x, y, width, height = p.get_bbox().bounds 
    ax.text(width+0.5, y+height/2, "%.1f%%"%(width), va='center')

plt.box(False)
plt.legend(frameon=False)
plt.tight_layout()
plt.show()
```


    
![png](output_55_0.png)
    


- Compared to all counties in Seoul, Mapo has similar usuage rates across all age groups.

## 3. User Groups by Station

In this part, let's try to find stations that answer the following questions:
- **Are there stations with a big difference in usage ratio between men and women?**
- **Are there stations that a certain age group is more frequently using?**


### 3.1. Men-Women Ratio by Station
If we could find some patterns of a specific group, such information can be utilized for many promotionl events. For example, if one is running for a mayoral or governor position and wants to attract voters from a specific group, he/she can go to the stations where that specific group of users are using bikes most frequently.


```python
df_pivot = mapo_users.pivot_table(index="start_station_name", columns='sex', values='rent_cnt', aggfunc='sum', fill_value=0)
df_pivot = df_pivot.div(df_pivot.sum(axis=1), axis=0)
print(df_pivot.shape)
df_pivot.head()
```

    (101, 2)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>sex</th>
      <th>F</th>
      <th>M</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(구)합정동 주민센터</th>
      <td>0.487395</td>
      <td>0.512605</td>
    </tr>
    <tr>
      <th>DMC빌 앞</th>
      <td>0.282828</td>
      <td>0.717172</td>
    </tr>
    <tr>
      <th>DMC산학협력연구센터 앞</th>
      <td>0.326761</td>
      <td>0.673239</td>
    </tr>
    <tr>
      <th>DMC역 2번출구 옆</th>
      <td>0.290000</td>
      <td>0.710000</td>
    </tr>
    <tr>
      <th>DMC역 9번출구 앞</th>
      <td>0.240545</td>
      <td>0.759455</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_pivot.sort_values('F', inplace=True)

df_pivot.plot(kind='barh', stacked=True, figsize=(24, 15), title="Ratio of Men vs Women (Mapo county)", rot=0, color=sex_colors)
plt.box(False)
plt.legend(frameon=False, loc='center left', bbox_to_anchor=(0.96, 0.5))
plt.ylabel("")
plt.axvline(x=0.41, color='grey')
plt.show()
```


    
![png](output_59_0.png)
    


- In "2.1. Usage Comparison by Age" section, we observed that the men to women ratio is 59:41. The grey line in the middle is at 0.41, which is the standard that determines whether a station has more women or men.
    
- Stations with red bars greater than the grey line represent stations with higher women usage ratio and vice versa.


```python
favor_f = df_pivot.loc[df_pivot['F'] - 0.41 > 0][['F']]
favor_f = favor_f-0.41
favor_f.rename(columns={'F': 'ratio'}, inplace=True)
favor_f['favor'] = 'F'
favor_m = df_pivot.loc[df_pivot['M'] - 0.59 > 0][['M']]
favor_m = -favor_m+0.59
favor_m.rename(columns={'M': 'ratio'}, inplace=True)
favor_m['favor'] = 'M'

favor = favor_f.append(favor_m)
```


```python
plt.figure(figsize=(12, 5))
sns.distplot(favor['ratio'], hist=False)
plt.box(False)
plt.show()
```


    
![png](output_62_0.png)
    


It's an overstatement to say that stations with slightly higher ratio than 0.41 have a difference in usage ratio between men and women. It'd be more reasonable to say that stations with 1.8 standard deviations away from 0.41 are the ones with a statistically significant difference in usage by sex.


```python
std = favor['ratio'].std()
favor[favor['ratio'] < (-1.5)*std]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>sex</th>
      <th>ratio</th>
      <th>favor</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>래미안신공덕3차아파트</th>
      <td>-0.319091</td>
      <td>M</td>
    </tr>
    <tr>
      <th>대흥동 주민센터</th>
      <td>-0.247209</td>
      <td>M</td>
    </tr>
    <tr>
      <th>현대벤처빌 앞</th>
      <td>-0.239932</td>
      <td>M</td>
    </tr>
    <tr>
      <th>서강대 후문 옆</th>
      <td>-0.234798</td>
      <td>M</td>
    </tr>
    <tr>
      <th>서강대 정문 건너편</th>
      <td>-0.194897</td>
      <td>M</td>
    </tr>
    <tr>
      <th>DMC역 9번출구 앞</th>
      <td>-0.169455</td>
      <td>M</td>
    </tr>
    <tr>
      <th>신촌역(2호선) 6번출구 옆</th>
      <td>-0.165459</td>
      <td>M</td>
    </tr>
    <tr>
      <th>LG CNS앞</th>
      <td>-0.165365</td>
      <td>M</td>
    </tr>
    <tr>
      <th>DMC역7번출구</th>
      <td>-0.160000</td>
      <td>M</td>
    </tr>
    <tr>
      <th>월드컵파크 4단지</th>
      <td>-0.152647</td>
      <td>M</td>
    </tr>
  </tbody>
</table>
</div>




```python
favor[favor['ratio'] > 1.5*std]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>sex</th>
      <th>ratio</th>
      <th>favor</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>망원2빗물펌프장 앞</th>
      <td>0.154885</td>
      <td>F</td>
    </tr>
    <tr>
      <th>벽산상암스마트큐브</th>
      <td>0.172456</td>
      <td>F</td>
    </tr>
  </tbody>
</table>
</div>



These are the stations with 1.5 standard deviation away from 0.41 (value inside parentheses is the difference from 0.41 basis).
- Stations with statistically higher usage by Men:
    - 래미안신공덕3차아파트 (0.31)
    - 대흥동 주민센터 (0.24
    - LG CNS앞 (0.16)
- Stations with statistically higher usage by Women:
    - 망원2빗물펌프장 앞 (0.16)
    - 벽산상암스마트큐브 (0.17)

## 3.2. Ratio of Age Group by Station

Next, let's see if there are stations with statistically significant differences by age group.


```python
mapo_users.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_hour</th>
      <th>start_station_name</th>
      <th>sex</th>
      <th>age_group</th>
      <th>rent_cnt</th>
      <th>kcal</th>
      <th>CO2</th>
      <th>distance</th>
      <th>duration</th>
      <th>county</th>
      <th>county_eng</th>
      <th>lat</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19741</th>
      <td>2/1/2021</td>
      <td>9</td>
      <td>성산2교 사거리</td>
      <td>F</td>
      <td>20s</td>
      <td>1.0</td>
      <td>63.77</td>
      <td>0.62</td>
      <td>2683.98</td>
      <td>17.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19742</th>
      <td>2/1/2021</td>
      <td>9</td>
      <td>성산2교 사거리</td>
      <td>M</td>
      <td>20s</td>
      <td>1.0</td>
      <td>85.64</td>
      <td>0.77</td>
      <td>3327.11</td>
      <td>20.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19743</th>
      <td>2/1/2021</td>
      <td>10</td>
      <td>성산2교 사거리</td>
      <td>F</td>
      <td>30s</td>
      <td>1.0</td>
      <td>92.81</td>
      <td>1.11</td>
      <td>4782.97</td>
      <td>31.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19744</th>
      <td>2/1/2021</td>
      <td>10</td>
      <td>성산2교 사거리</td>
      <td>F</td>
      <td>~10s</td>
      <td>1.0</td>
      <td>27.85</td>
      <td>0.18</td>
      <td>781.30</td>
      <td>6.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
    <tr>
      <th>19745</th>
      <td>2/1/2021</td>
      <td>10</td>
      <td>성산2교 사거리</td>
      <td>M</td>
      <td>40s</td>
      <td>1.0</td>
      <td>215.77</td>
      <td>1.5</td>
      <td>6486.55</td>
      <td>29.0</td>
      <td>마포구</td>
      <td>Mapo</td>
      <td>37.564697</td>
      <td>126.912613</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_pivot = mapo_users.pivot_table(index="start_station_name", columns='age_group', values='rent_cnt', aggfunc='sum', fill_value=0)
df_pivot = df_pivot.div(df_pivot.sum(axis=1), axis=0)
print(df_pivot.shape)
df_pivot.head()
```

    (101, 7)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>age_group</th>
      <th>20s</th>
      <th>30s</th>
      <th>40s</th>
      <th>50s</th>
      <th>60s</th>
      <th>70s~</th>
      <th>~10s</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(구)합정동 주민센터</th>
      <td>0.357143</td>
      <td>0.411765</td>
      <td>0.147059</td>
      <td>0.063025</td>
      <td>0.008403</td>
      <td>0.000000</td>
      <td>0.012605</td>
    </tr>
    <tr>
      <th>DMC빌 앞</th>
      <td>0.333333</td>
      <td>0.292929</td>
      <td>0.313131</td>
      <td>0.040404</td>
      <td>0.020202</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>DMC산학협력연구센터 앞</th>
      <td>0.194366</td>
      <td>0.326761</td>
      <td>0.264789</td>
      <td>0.129577</td>
      <td>0.039437</td>
      <td>0.019718</td>
      <td>0.025352</td>
    </tr>
    <tr>
      <th>DMC역 2번출구 옆</th>
      <td>0.250000</td>
      <td>0.275000</td>
      <td>0.260000</td>
      <td>0.075000</td>
      <td>0.060000</td>
      <td>0.055000</td>
      <td>0.025000</td>
    </tr>
    <tr>
      <th>DMC역 9번출구 앞</th>
      <td>0.307110</td>
      <td>0.257186</td>
      <td>0.228442</td>
      <td>0.148260</td>
      <td>0.042360</td>
      <td>0.010590</td>
      <td>0.006051</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_pivot.sort_values('20s', inplace=True)

df_pivot[age_order].plot(kind='barh', stacked=True, figsize=(24, 15), title="Age Group Ratio by Station (Mapo county)", rot=0)
plt.box(False)
plt.legend(frameon=False, loc='center left', bbox_to_anchor=(0.96, 0.5))
plt.ylabel("")
plt.show()
```


    
![png](output_70_0.png)
    


Stations are sorted in the descending order of age group 20s.
There seem to be differences in usage by age group.


```python
plt.figure(figsize=(18, 5))
for age in age_order:
    df_age = df_pivot[age]
    df_age = (df_age - df_age.mean()) / df_age.std()
    sns.distplot(df_age, hist=False, label=age, rug=True)
plt.legend(frameon=False)
plt.box(False)
plt.tight_layout()
plt.xlabel("")
plt.show()
```


    
![png](output_72_0.png)
    


The distribution plot above demonstrates the normalized distribution of all stations by age group.

- Age group 20s and 30s are most normally distributed. In other words, there exists deviations in these groups.
- Age group 10s and 60s have similar distribution.
- Age group 70s~ have the least deviation in ratio.
- Age group 70s~ have one value that exteremly high which indicate that there is a station that's more frequently used by age group 70s~ than others.

However, it's quiet difficult to find which stations are frequently used by which age group. Let's draw a map of stations used by each age group.

1. Each point represents a station.
2. The darker the color is, the more frequently the station is used.


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>start_time</th>
      <th>start_station_id</th>
      <th>start_station_name</th>
      <th>end_time</th>
      <th>end_station_id</th>
      <th>end_station_name</th>
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
      <th>county_x</th>
      <th>county_eng_x</th>
      <th>lat_x</th>
      <th>long_x</th>
      <th>county_y</th>
      <th>county_eng_y</th>
      <th>lat_y</th>
      <th>long_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-02-01 00:00:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 00:03:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>736.02</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-02-01 18:01:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 18:08:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>7.0</td>
      <td>779.26</td>
      <td>18</td>
      <td>1</td>
      <td>0</td>
      <td>18</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-02-01 20:21:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 20:27:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>5.0</td>
      <td>0.00</td>
      <td>20</td>
      <td>1</td>
      <td>0</td>
      <td>20</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-02-01 20:32:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-01 20:38:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>5.0</td>
      <td>736.02</td>
      <td>20</td>
      <td>1</td>
      <td>0</td>
      <td>20</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-02-03 00:12:00</td>
      <td>1514</td>
      <td>강북구청 사거리 버스정류소 앞</td>
      <td>2021-02-03 00:14:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>2.0</td>
      <td>628.99</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.638805</td>
      <td>127.028358</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
  </tbody>
</table>
</div>




```python
rental_latlngs = df[['start_station_name', 'lat_x', 'long_x']].drop_duplicates().set_index('start_station_name')
rental_latlngs.index = rental_latlngs.index.str.strip()
df_pivot.index = df_pivot.index.str.strip()
df_pivot.columns = df_pivot.columns.astype('object')
df_pivot = df_pivot.join(rental_latlngs)
```


```python
df_pivot.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>20s</th>
      <th>30s</th>
      <th>40s</th>
      <th>50s</th>
      <th>60s</th>
      <th>70s~</th>
      <th>~10s</th>
      <th>lat_x</th>
      <th>long_x</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(구)합정동 주민센터</th>
      <td>0.357143</td>
      <td>0.411765</td>
      <td>0.147059</td>
      <td>0.063025</td>
      <td>0.008403</td>
      <td>0.000000</td>
      <td>0.012605</td>
      <td>37.549561</td>
      <td>126.905754</td>
    </tr>
    <tr>
      <th>DMC빌 앞</th>
      <td>0.333333</td>
      <td>0.292929</td>
      <td>0.313131</td>
      <td>0.040404</td>
      <td>0.020202</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>37.582657</td>
      <td>126.885788</td>
    </tr>
    <tr>
      <th>DMC산학협력연구센터 앞</th>
      <td>0.194366</td>
      <td>0.326761</td>
      <td>0.264789</td>
      <td>0.129577</td>
      <td>0.039437</td>
      <td>0.019718</td>
      <td>0.025352</td>
      <td>37.575802</td>
      <td>126.890739</td>
    </tr>
    <tr>
      <th>DMC역 2번출구 옆</th>
      <td>0.250000</td>
      <td>0.275000</td>
      <td>0.260000</td>
      <td>0.075000</td>
      <td>0.060000</td>
      <td>0.055000</td>
      <td>0.025000</td>
      <td>37.575069</td>
      <td>126.899918</td>
    </tr>
    <tr>
      <th>DMC역 9번출구 앞</th>
      <td>0.307110</td>
      <td>0.257186</td>
      <td>0.228442</td>
      <td>0.148260</td>
      <td>0.042360</td>
      <td>0.010590</td>
      <td>0.006051</td>
      <td>37.577469</td>
      <td>126.897362</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_pivot.index.name = "start_station_name"
```


```python
df_pivot.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>20s</th>
      <th>30s</th>
      <th>40s</th>
      <th>50s</th>
      <th>60s</th>
      <th>70s~</th>
      <th>~10s</th>
      <th>lat_x</th>
      <th>long_x</th>
    </tr>
    <tr>
      <th>start_station_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(구)합정동 주민센터</th>
      <td>0.357143</td>
      <td>0.411765</td>
      <td>0.147059</td>
      <td>0.063025</td>
      <td>0.008403</td>
      <td>0.000000</td>
      <td>0.012605</td>
      <td>37.549561</td>
      <td>126.905754</td>
    </tr>
    <tr>
      <th>DMC빌 앞</th>
      <td>0.333333</td>
      <td>0.292929</td>
      <td>0.313131</td>
      <td>0.040404</td>
      <td>0.020202</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>37.582657</td>
      <td>126.885788</td>
    </tr>
    <tr>
      <th>DMC산학협력연구센터 앞</th>
      <td>0.194366</td>
      <td>0.326761</td>
      <td>0.264789</td>
      <td>0.129577</td>
      <td>0.039437</td>
      <td>0.019718</td>
      <td>0.025352</td>
      <td>37.575802</td>
      <td>126.890739</td>
    </tr>
    <tr>
      <th>DMC역 2번출구 옆</th>
      <td>0.250000</td>
      <td>0.275000</td>
      <td>0.260000</td>
      <td>0.075000</td>
      <td>0.060000</td>
      <td>0.055000</td>
      <td>0.025000</td>
      <td>37.575069</td>
      <td>126.899918</td>
    </tr>
    <tr>
      <th>DMC역 9번출구 앞</th>
      <td>0.307110</td>
      <td>0.257186</td>
      <td>0.228442</td>
      <td>0.148260</td>
      <td>0.042360</td>
      <td>0.010590</td>
      <td>0.006051</td>
      <td>37.577469</td>
      <td>126.897362</td>
    </tr>
  </tbody>
</table>
</div>




```python
bike_map = folium.Map(location=[37.56261, 126.90761], zoom_start=14, zoom_control=False, height=800)
folium.TileLayer(tiles='cartodbpositron', overlay=True).add_to(bike_map)

# Add regional poligon
region = '마포구'
for region_data in geo_str['features']:
    if region_data['properties']['SIG_KOR_NM'] == region:
        polygon = np.array(region_data['geometry']['coordinates']).reshape(-1, 2).tolist()
        for coord in polygon:
            coord[0], coord[1] = coord[1], coord[0]
        folium.Polygon(polygon, color='darkgrey').add_to(bike_map)
        break

age_color = {age: color for age, color in zip(age_order, ['blue', 'darkorange', 'green', 'red', 'purple', 'brown', 'magenta'])}
dict_df_age = {}

for age in age_order:
    df_latlng_values = df_pivot.reset_index()[['lat_x', 'long_x', age, 'start_station_name']]
    
    # normalize data between 0 to 1.
    df_latlng_values[age] = (df_latlng_values[age] - df_latlng_values[age].min()) / (df_latlng_values[age].max() - df_latlng_values[age].min())
    
    # draw circle on each rental position.
    feature_group = folium.FeatureGroup(name=age, overlay=False)
    for idx, row in df_latlng_values.iterrows():
        folium.CircleMarker(row[['lat_x', 'long_x']],
                    fill=True, fill_color=age_color[age], fill_opacity=row[age],
                    tooltip="[%s]\n%.2f"%(row['start_station_name'], row[age]),
                    radius=5, opacity=0).add_to(feature_group)
    feature_group.add_to(bike_map)
    dict_df_age[age] = df_latlng_values[['start_station_name', age]].sort_values(age, ascending=False)

folium.LayerControl(collapsed=False).add_to(bike_map)

bike_map
```

### ~10s


```python
from IPython.display import Image
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "1.png", width=600, height=600)
```




    
![png](output_81_0.png)
    



For users in 10s age group, these are the stations with high usage rate:

1. 공덕역 2번 출구 (1.00)
2. 마포구청역 (0.90)
3. 마포 신수 공원 앞 (0.87)
4. 상암 월드컵파크 1단지 교차로 (0.84)

It does not seem to have a distinctive pattern.

### 20s


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "2.png", width=600, height=600)
```




    
![png](output_84_0.png)
    



For users in their 20s,they are using stations nearby college towns.

### 30s


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "3.png", width=600, height=600)
```




    
![png](output_87_0.png)
    



For users in their 30s, they are more widely spread compared to those in 20s.

They seem to be using stations where most offices are located.

### 40s


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "4.png", width=600, height=600)
```




    
![png](output_90_0.png)
    



For users in their 40s, they are heavily using stations nearby DMC area (upper left corner) which is quiet different from other age groups.

### 50s


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "5.png", width=600, height=600)
```




    
![png](output_93_0.png)
    



For users in their 50s, they seem to have similar pattern with those in 40s.

### 60s


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "1.png", width=600, height=600)
```




    
![png](output_96_0.png)
    



For users in their 60s, they are using stations nearby Mapo County Office and Changcheon-dong, where 2 story houses are located (Just like city of New York, most people live in apartment in Seoul).

### 70s~


```python
PATH = "C:/jiehwan94.github.io-master/assets/img/project/Seoul_Trip_3/"
Image(filename = PATH + "7.png", width=600, height=600)
```




    
![png](output_99_0.png)
    



For users in their 70s, they are using stations at a specific region in DMC.

## 4. Frequency of Trips between Two Stations

In this part, let's take a closer look at the trip route.

It'd have been better if we have access to the exact path that each trip took place, but we only have origin and destination information in our dataset. 

Given what we have, let's draw lines from origin to destination with the following rules:

1. Each **dot** indicates a station.
2. Each station's usage ratio (Rent + Return) is represented by **transparency**. The greater usage ratio, the thicker the line is.
3. Each trip from origin to destination is reperesented by **a line**.
4. For routes with relatively high frequency, we indicate those routes with **arrows** (because it's hard to interpret the map if we were to draw arrows for every trip).
5. We will looking at weekday trips only. (Excluding trips on weekends)
6. Assuming that trips have different patterns by different time of day, we will split the day into specific times and look for patterns.


```python
from collections import namedtuple

def get_arrows(locations, color='blue', weight=1, size=6, n_arrows=3, some_map=None):
    
    '''
    Get a list of correctly placed and rotated 
    arrows/markers to be plotted
    
    Parameters
    locations : list of lists of lat lons that represent the 
                start and end of the line. 
                eg [[41.1132, -96.1993],[41.3810, -95.8021]]
    arrow_color : default is 'blue'
    size : default is 6
    n_arrows : number of arrows to create.  default is 3
    Return
    list of arrows/markers
    '''
    
    Point = namedtuple('Point', field_names=['lat', 'lon'])
    
    # creating point from our Point named tuple
    p1 = Point(locations[0][0], locations[0][1])
    p2 = Point(locations[1][0], locations[1][1])
    
    # getting the rotation needed for our marker.  
    # Subtracting 90 to account for the marker's orientation
    # of due East(get_bearing returns North)
    rotation = get_bearing(p1, p2) - 90
    
    # get an evenly space list of lats and lons for our arrows
    # note that I'm discarding the first and last for aesthetics
    # as I'm using markers to denote the start and end
    arrow_lats = np.linspace(p1.lat, p2.lat, n_arrows + 2)[1:n_arrows+1]
    arrow_lons = np.linspace(p1.lon, p2.lon, n_arrows + 2)[1:n_arrows+1]
    
    arrows = []
    
    #creating each "arrow" and appending them to our arrows list
    for points in zip(arrow_lats, arrow_lons):
        arrows.append(folium.RegularPolygonMarker(location=points, 
                      fill_color=color, number_of_sides=3, opacity=weight,
                      radius=size, rotation=rotation).add_to(some_map))
    return arrows

def get_bearing(p1, p2):
    
    '''
    Returns compass bearing from p1 to p2
    
    Parameters
    p1 : namedtuple with lat lon
    p2 : namedtuple with lat lon
    
    Return
    compass bearing of type float
    
    Notes
    Based on https://gist.github.com/jeromer/2005586
    '''
    
    long_diff = np.radians(p2.lon - p1.lon)
    
    lat1 = np.radians(p1.lat)
    lat2 = np.radians(p2.lat)
    
    x = np.sin(long_diff) * np.cos(lat2)
    y = (np.cos(lat1) * np.sin(lat2) 
        - (np.sin(lat1) * np.cos(lat2) 
        * np.cos(long_diff)))
    bearing = np.degrees(np.arctan2(x, y))
    
    # adjusting for compass bearing
    if bearing < 0:
        return bearing + 360
    return bearing
```


```python
def draw_traffic(df, dayofweek, time, region, normalize=False, marker_on=False, heatmap=False, arrow=True):
    if dayofweek == "Weekday":
        _dayofweek = set(range(0,5))
    elif dayofweek == "Weekend":
        _dayofweek = set(range(5,7))
    else:
        print("dayofweek must be either Weekdays or Weekends")
        return -1
    
    df = df[df['start_dayofweek'].isin(_dayofweek)]
    df = df[df['start_hour'].isin(time)]
    
    # Traffic line
    by_rental = df.groupby(['lat_x', 'long_x', 'lat_y', 'long_y']).size()
    weights = by_rental.values.tolist()
    lat_lngs = by_rental.index.values.tolist()
    
    # Route Usage Distribution
    plt.figure(figsize=(24, 5))
    sns.distplot(np.array(weights).flatten(), rug=True, kde=False, bins=140)
    plt.title("Route Usage Distribution")
    plt.xlabel("# of Usage")
    plt.show()
    
    # - weight
    if normalize:
        weights = np.log(np.array(weights).flatten()) / np.log(np.array(weights).flatten()).max()
        weights = weights
        
        # - Specific Route Usage Distribution
        plt.figure(figsize=(24, 5))
        sns.distplot(np.array(weights).flatten(), rug=True, kde=False, bins=140)
        plt.title("Specific Route Usage Distribution")
        plt.xlabel("Usage Count (logged)")
        plt.show()
    else:
        weights = [weight / max(weights) for weight in weights]
        
    
    # - plot map first
    center = np.array(lat_lngs).mean(axis=0)[:2]
    bike_map = folium.Map(location=center, zoom_start=14, tiles='cartodbpositron', zoom_control=False)
    
    for line, weight in zip(lat_lngs, weights):
        # Add arrow
        if arrow and weight >= 0.3:
            # only >= 0.3
            arrows = get_arrows([line[:2], line[2:4]], weight=weight, size=5, n_arrows=3, some_map=bike_map)
            for arrow in arrows:
                arrow.add_to(bike_map)
        
        line = [(line[0], line[1]), (line[2], line[3])]
        folium.PolyLine(line, opacity=weight+0.05, weight=1.5).add_to(bike_map)
        
    
    # heatmap 
    if heatmap:
        for use, color in zip(["Rent", "Return"], ["red", "green"]):
            _df = df.groupby(['%sStation Latitude'%use, '%sStation Longitude'%use]).size()

            # Set weight to 0~1
            _df = _df / _df.max()

            points = _df.reset_index().values.tolist()

            folium.HeatMap(points, radius=15, ).add_to(bike_map)
            break
    
    #marker
    if marker_on:
        rentals = pd.DataFrame(data={
            "Rent": df.groupby(["lat_x", "long_x"]).size(),
            "Return": df.groupby(["lat_y", "long_y"]).size()
        }).fillna(0)
        
        rentals.index.names = ['lat_x','long_x']
        rentals.reset_index(inplace=True)
        rentals = rentals.merge(df.groupby(['lat_x', 'long_x', 'start_station_name']).size().reset_index().drop(columns=0),
                left_on=['lat_x', 'long_x'],
                right_on=['lat_x', 'long_x'])[['start_station_name', 'lat_x', 'long_x', 'Rent', 'Return']]
        rentals['Usage'] = rentals['Rent'] + rentals['Return']
        rentals['Ratio'] = rentals['Ratio'] / rentals['Ratio'].max()
        
        for idx, rental in rentals.iterrows():
            tooltip_text = "[%s]\nRent: %d\nReturn: %d" %(rental['start_station_name'], rental['Rent'], rental['Return'])
            folium.CircleMarker(rental[['lat_x', 'long_x']], 
                                fill=True, fill_color='blue', fill_opacity=rental['Ratio'],
                                radius=5, opacity=0, tooltip=tooltip_text).add_to(bike_map)
    
    # region poligon
    for region_data in geo_str['features']:
        if region_data['properties']['SIG_KOR_NM'] == region:
            polygon = np.array(region_data['geometry']['coordinates']).reshape(-1, 2).tolist()
            for coord in polygon:
                coord[0], coord[1] = coord[1], coord[0]
            folium.Polygon(polygon, color='darkgrey').add_to(bike_map)
            break
    
    # html save.
    if normalize:
        bike_map.save('map/%s-%s-%s-%s(normalized).html' %(region, dayofweek, time[0], time[-1]+1))
    else:
        bike_map.save('map/%s-%s-%s-%s.html' %(region, dayofweek, time[0], time[-1]+1))
    return bike_map
```

### 4.1. Commute to work (7am ~ 10am)


```python
draw_traffic(mapo, "Weekday", list(range(7, 10)), "마포구", marker_on=True)
```




    
![png](output_106_0.png)
    



- **Most traffic at DMC and Hongik University 2 Exit"**
- These are the stations that people commute to work or school.

### 4.2. Afternoon (10am ~ 5pm)


```python
draw_traffic(mapo, "Weekday", list(range(10, 17)), "마포구", marker_on=True)
```




    
![png](output_109_0.png)
    



- More distributed traffic than 7am ~ 10am
- Stations at Hongik University still have a lot of traffic coming in which means that the bike operation management team has to move bikes from here to other stations to avoid shortage of bikes in certain stations. However, when it gets closer to 5pm when people start going home, they might need to have extra bikes at Hongik Univ. stations.

### 4.3. Commute back home (5pm ~ 8pm)


```python
draw_traffic(mapo, "Weekday", list(range(17, 20)), "마포구", marker_on=True)
```




    
![png](output_112_0.png)
    



- As shown in the arrows, the very opposite happens when they are commuting back home.
- People are riding bikes to the Han River.

### 4.4. Night (8pm ~ 0 am)


```python
draw_traffic(mapo, "Weekday", list(range(20, 24)), "마포구", marker_on=True)
```




    
![png](output_115_0.png)
    



- Similar to 5pm ~ 8pm with relatively less traffic

## 5. Inflow/Outflow: Trips to and from Mapo County

With which county does Mapo county has most traffic?


```python
inout = pd.DataFrame([mapo['county_eng_y'].value_counts().drop("Mapo"),
                      mapo['county_eng_x'].value_counts().drop("Mapo")],
                    index=['Outflow', 'Inflow']).T
inout = inout.sort_values('Outflow', ascending=False)
inout.plot(kind='bar', figsize=(24, 5), title="Outflow, Inflow County", rot=0, color=['C3', 'C2'])
plt.box(False)
plt.legend(frameon=False)
plt.xticks(rotation=45, ha='right')
plt.show()
```


    
![png](output_119_0.png)
    


- Inflow-Outflow are in linear relationship. In other words, counties with high outflow have high inflow as well.
- As you might have guessed, there are many traffics to counties close to Mapo.
- Seodaemun > Eunpyeong > Yeongdeungpo > Yongsan

## Summary

- Mapo has distinctive patterns in commute time.
- During 7am ~ 10am: inflow is approximately 30~35%
- During 5pm ~ 8pm: outflow is approximately 25~30%
- During commute time, there's highest traffic at stations in DMC and Hongik University.
- Traffics with other counties are mostly with nearby counties.


```python

```
