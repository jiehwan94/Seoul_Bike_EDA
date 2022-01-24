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


```python
bike_map = folium.plugins.DualMap(location=[37.541, 126.986], zoom_start=10, tiles='cartodbpositron', zoom_control=False)
folium.Choropleth(geo_data=geo_str,
                  data=mapo['county_y'].value_counts().drop("마포구"),
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='Reds').add_to(bike_map.m1)
folium.Choropleth(geo_data=geo_str,
                  data=mapo['county_x'].value_counts().drop("마포구"),
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='Greens').add_to(bike_map.m2)
bike_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_d6fb1ba880dc49459feac09a2f6f713d%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20absolute%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js%22%3E%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_1fa756f29afb4313accf910c33f13947%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20absolute%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/gh/jieter/Leaflet.Sync/L.Map.Sync.min.js%22%3E%3C/script%3E%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_d6fb1ba880dc49459feac09a2f6f713d%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_1fa756f29afb4313accf910c33f13947%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_d6fb1ba880dc49459feac09a2f6f713d%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_d6fb1ba880dc49459feac09a2f6f713d%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.541%2C%20126.986%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_006465f744c14d7397d1c1452445e096%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//cartodb-basemaps-%7Bs%7D.global.ssl.fastly.net/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%20contributors%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eCartoDB%5Cu003c/a%5Cu003e%2C%20CartoDB%20%5Cu003ca%20href%20%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eattributions%5Cu003c/a%5Cu003e%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_d6fb1ba880dc49459feac09a2f6f713d%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20choropleth_0dff84495dce49bc8237c5530d93bd43%20%3D%20L.featureGroup%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_d6fb1ba880dc49459feac09a2f6f713d%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.properties.SIG_CD%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211380%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23fb6a4a%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211440%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22black%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211410%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23a50f15%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211560%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23fc9272%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211170%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23fcbba1%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23fee5d9%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_eb03cf63e8a94996b56ba997ca531f7f%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_styler%2C%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_eb03cf63e8a94996b56ba997ca531f7f%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28choropleth_0dff84495dce49bc8237c5530d93bd43%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_eb03cf63e8a94996b56ba997ca531f7f_add%28%7B%22features%22%3A%20%5B%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00831558599998%2C%2037.69001680000002%5D%2C%20%5B127.00764424700003%2C%2037.69150638999997%5D%2C%20%5B127.00973504700005%2C%2037.69324682000001%5D%2C%20%5B127.00969102500005%2C%2037.696558160999984%5D%2C%20%5B127.01214887599997%2C%2037.69724089499999%5D%2C%20%5B127.01390304300003%2C%2037.698660491%5D%2C%20%5B127.01544040299996%2C%2037.70130154100002%5D%2C%20%5B127.019851357%2C%2037.700884901999984%5D%2C%20%5B127.02217147700003%2C%2037.69960736799999%5D%2C%20%5B127.02341184299996%2C%2037.69995983299998%5D%2C%20%5B127.02533791899998%2C%2037.69948105499998%5D%2C%20%5B127.02771004600004%2C%2037.700837447000026%5D%2C%20%5B127.02930740500005%2C%2037.699210467%5D%2C%20%5B127.02975002699998%2C%2037.69621560600001%5D%2C%20%5B127.03104551199999%2C%2037.69304682699999%5D%2C%20%5B127.032188658%2C%2037.69279409400002%5D%2C%20%5B127.03243270899998%2C%2037.69182616099999%5D%2C%20%5B127.03678096299996%2C%2037.692638805%5D%2C%20%5B127.038515888%2C%2037.69328696000002%5D%2C%20%5B127.03896796200002%2C%2037.69397974100002%5D%2C%20%5B127.04112667799996%2C%2037.695296503%5D%2C%20%5B127.04309027199997%2C%2037.69523619%5D%2C%20%5B127.04376597600003%2C%2037.693511828%5D%2C%20%5B127.04566611500002%2C%2037.692384920999984%5D%2C%20%5B127.04783268899996%2C%2037.69329455000002%5D%2C%20%5B127.04864899799998%2C%2037.69406779399998%5D%2C%20%5B127.04938379999999%2C%2037.691693921000024%5D%2C%20%5B127.04998367500002%2C%2037.690875163999976%5D%2C%20%5B127.04983654099999%2C%2037.687983056%5D%2C%20%5B127.05093367799998%2C%2037.686160674%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%200%2C%20%22SHAPE_AREA%22%3A%200.00211%2C%20%22SHAPE_LEN%22%3A%200.239901%2C%20%22SIG_CD%22%3A%20%2211320%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dobong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88359471499996%2C%2037.591855451000015%5D%2C%20%5B126.88502350199997%2C%2037.593642095%5D%2C%20%5B126.88720157499995%2C%2037.59368140999999%5D%2C%20%5B126.88561580199996%2C%2037.591502085%5D%2C%20%5B126.88577464100001%2C%2037.589569513000015%5D%2C%20%5B126.88717970799996%2C%2037.58853613399998%5D%2C%20%5B126.88927454400005%2C%2037.58883090799998%5D%2C%20%5B126.89225666100003%2C%2037.58852532399999%5D%2C%20%5B126.89334847199996%2C%2037.589070849%5D%2C%20%5B126.89687879200005%2C%2037.588570065%5D%2C%20%5B126.89810143800003%2C%2037.58954265199998%5D%2C%20%5B126.89966320099995%2C%2037.58982035399998%5D%2C%20%5B126.899585279%2C%2037.591694511000014%5D%2C%20%5B126.89898042499999%2C%2037.592683197999975%5D%2C%20%5B126.90037465800003%2C%2037.59421526400001%5D%2C%20%5B126.90184962900003%2C%2037.59514499199997%5D%2C%20%5B126.90103258399995%2C%2037.597339986%5D%2C%20%5B126.901343312%2C%2037.59936941699999%5D%2C%20%5B126.90000258099997%2C%2037.602042943000015%5D%2C%20%5B126.90017282600002%2C%2037.60311939799999%5D%2C%20%5B126.90212880000001%2C%2037.60374458000001%5D%2C%20%5B126.90062064300002%2C%2037.60932366100002%5D%2C%20%5B126.90032356100005%2C%2037.611195898%5D%2C%20%5B126.90175330700004%2C%2037.61407416499998%5D%2C%20%5B126.901843654%2C%2037.615756572%5D%2C%20%5B126.90314656400005%2C%2037.61730752099999%5D%2C%20%5B126.90338372099995%2C%2037.61884112299998%5D%2C%20%5B126.90525097600005%2C%2037.61907907300002%5D%2C%20%5B126.90563169100005%2C%2037.620708664%5D%2C%20%5B126.90713704200004%2C%2037.62194588400001%5D%2C%20%5B126.90716203600005%2C%2037.62331746299998%5D%2C%20%5B126.90664762899996%2C%2037.62449994799999%5D%2C%20%5B126.908630388%2C%2037.62627061799998%5D%2C%20%5B126.90879064700005%2C%2037.629332764000026%5D%2C%20%5B126.90731509700004%2C%2037.63087173100001%5D%2C%20%5B126.90677806899998%2C%2037.63274096499998%5D%2C%20%5B126.90711683699999%2C%2037.63340278800001%5D%2C%20%5B126.910875079%2C%2037.635322556%5D%2C%20%5B126.91123094800002%2C%2037.635911695%5D%2C%20%5B126.91011226199998%2C%2037.638515595%5D%2C%20%5B126.91081456899997%2C%2037.64003255799997%5D%2C%20%5B126.91230017800001%2C%2037.64432203199999%5D%2C%20%5B126.91016072699995%2C%2037.645083612%5D%2C%20%5B126.90772442000002%2C%2037.64629642699998%5D%2C%20%5B126.90843309399997%2C%2037.647262366%5D%2C%20%5B126.90974709700004%2C%2037.646882914%5D%2C%20%5B126.91234747099998%2C%2037.645182541%5D%2C%20%5B126.913740469%2C%2037.64476066499998%5D%2C%20%5B126.92414365900004%2C%2037.64611043100001%5D%2C%20%5B126.92715483799998%2C%2037.64810585499998%5D%2C%20%5B126.92972022900005%2C%2037.65004959800001%5D%2C%20%5B126.93224238100004%2C%2037.65038895999999%5D%2C%20%5B126.935727004%2C%2037.65120514099999%5D%2C%20%5B126.93718405799996%2C%2037.652317728000014%5D%2C%20%5B126.93847852299996%2C%2037.654773405000014%5D%2C%20%5B126.94014671100001%2C%2037.65663874799998%5D%2C%20%5B126.94211837499995%2C%2037.65772510800002%5D%2C%20%5B126.94441488899997%2C%2037.65816625799999%5D%2C%20%5B126.94758363699998%2C%2037.659217403000014%5D%2C%20%5B126.94790109500002%2C%2037.65712764900002%5D%2C%20%5B126.94922168000005%2C%2037.656772183999976%5D%2C%20%5B126.94980975299995%2C%2037.655913363000025%5D%2C%20%5B126.95139494700004%2C%2037.654951117999985%5D%2C%20%5B126.95437039%2C%2037.654596949999984%5D%2C%20%5B126.95712830900004%2C%2037.65283925900002%5D%2C%20%5B126.95781039600001%2C%2037.64981070699997%5D%2C%20%5B126.95786031099999%2C%2037.64804963099999%5D%2C%20%5B126.95904808800003%2C%2037.64678533599999%5D%2C%20%5B126.95892784399996%2C%2037.64476220400002%5D%2C%20%5B126.95928919999994%2C%2037.642134535000025%5D%2C%20%5B126.959992629%2C%2037.64146349599997%5D%2C%20%5B126.96162582500006%2C%2037.63711909400001%5D%2C%20%5B126.96256224599995%2C%2037.63582057799999%5D%2C%20%5B126.96335032800005%2C%2037.633246824000025%5D%2C%20%5B126.96164091799994%2C%2037.631889602%5D%2C%20%5B126.95984451699996%2C%2037.629762777999986%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%201%2C%20%22SHAPE_AREA%22%3A%200.003041%2C%20%22SHAPE_LEN%22%3A%200.327143%2C%20%22SIG_CD%22%3A%20%2211380%22%2C%20%22SIG_ENG_NM%22%3A%20%22Eunpyeong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%202%2C%20%22SHAPE_AREA%22%3A%200.001453%2C%20%22SHAPE_LEN%22%3A%200.182837%2C%20%22SIG_CD%22%3A%20%2211230%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongdaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%203%2C%20%22SHAPE_AREA%22%3A%200.00167%2C%20%22SHAPE_LEN%22%3A%200.237796%2C%20%22SIG_CD%22%3A%20%2211590%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongjak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92829901899995%2C%2037.449360259%5D%2C%20%5B126.926538273%2C%2037.44839745899998%5D%2C%20%5B126.92500225699996%2C%2037.44678811799997%5D%2C%20%5B126.92327265100005%2C%2037.44577942699999%5D%2C%20%5B126.92278057800002%2C%2037.444047814999976%5D%2C%20%5B126.92118598900004%2C%2037.442552917%5D%2C%20%5B126.92028939299996%2C%2037.44047332299999%5D%2C%20%5B126.91927808399998%2C%2037.43985946599997%5D%2C%20%5B126.91613959300003%2C%2037.440060542000026%5D%2C%20%5B126.91232267199996%2C%2037.438587175%5D%2C%20%5B126.911215782%2C%2037.43720903600001%5D%2C%20%5B126.91134523699998%2C%2037.436179159%5D%2C%20%5B126.90940698500003%2C%2037.433868523%5D%2C%20%5B126.90726602899997%2C%2037.43352405000002%5D%2C%20%5B126.90611662000003%2C%2037.43400280399999%5D%2C%20%5B126.90299885399997%2C%2037.43407347499999%5D%2C%20%5B126.902763%2C%2037.43586855199999%5D%2C%20%5B126.90086236399998%2C%2037.43699089%5D%2C%20%5B126.89877013700004%2C%2037.439283811999985%5D%2C%20%5B126.89991895399999%2C%2037.439562146000014%5D%2C%20%5B126.897287549%2C%2037.445443447%5D%2C%20%5B126.89580121999995%2C%2037.445640562999984%5D%2C%20%5B126.89466890899996%2C%2037.446706108%5D%2C%20%5B126.89476993300002%2C%2037.447710443%5D%2C%20%5B126.89592769599994%2C%2037.44842231299998%5D%2C%20%5B126.89400040400005%2C%2037.452723881%5D%2C%20%5B126.89277619200004%2C%2037.452046575%5D%2C%20%5B126.88951816999997%2C%2037.45270753599999%5D%2C%20%5B126.889280761%2C%2037.45448889199997%5D%2C%20%5B126.88641922299996%2C%2037.456180116999974%5D%2C%20%5B126.88589653600002%2C%2037.457514893999985%5D%2C%20%5B126.88630108799998%2C%2037.459120703%5D%2C%20%5B126.885364977%2C%2037.459868728%5D%2C%20%5B126.886087029%2C%2037.460865333000015%5D%2C%20%5B126.88889275600002%2C%2037.460953249%5D%2C%20%5B126.88713625100002%2C%2037.462909712%5D%2C%20%5B126.88515609299998%2C%2037.462473671%5D%2C%20%5B126.88433205199999%2C%2037.46272970799998%5D%2C%20%5B126.88276046399994%2C%2037.464396598%5D%2C%20%5B126.88463173000002%2C%2037.46601745999999%5D%2C%20%5B126.88302833499995%2C%2037.467069503%5D%2C%20%5B126.881647767%2C%2037.468466234%5D%2C%20%5B126.88161172399998%2C%2037.469425546000025%5D%2C%20%5B126.878031942%2C%2037.47383573500002%5D%2C%20%5B126.87600453499999%2C%2037.477111283%5D%2C%20%5B126.87178398799995%2C%2037.48527755399999%5D%2C%20%5B126.87315863100002%2C%2037.486160427000016%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%204%2C%20%22SHAPE_AREA%22%3A%200.001325%2C%20%22SHAPE_LEN%22%3A%200.211649%2C%20%22SIG_CD%22%3A%20%2211545%22%2C%20%22SIG_ENG_NM%22%3A%20%22Geumcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.88272419199996%2C%2037.515768666999975%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.87599162499998%2C%2037.48691906800002%5D%2C%20%5B126.87680700600004%2C%2037.488578991%5D%2C%20%5B126.87583373899997%2C%2037.489035374000025%5D%2C%20%5B126.87309748400003%2C%2037.488261317000024%5D%2C%20%5B126.87245908399996%2C%2037.489543242000025%5D%2C%20%5B126.87024621900002%2C%2037.48960327200001%5D%2C%20%5B126.86936249799999%2C%2037.49164208000002%5D%2C%20%5B126.86972553299995%2C%2037.49410186699998%5D%2C%20%5B126.86836486200002%2C%2037.49511503000002%5D%2C%20%5B126.86738599800003%2C%2037.49308047%5D%2C%20%5B126.8663199%2C%2037.492637301%5D%2C%20%5B126.86541991000001%2C%2037.49151324600001%5D%2C%20%5B126.86260271100002%2C%2037.49076571799998%5D%2C%20%5B126.86139963599999%2C%2037.489972641%5D%2C%20%5B126.85798566000005%2C%2037.48609330599999%5D%2C%20%5B126.855521178%2C%2037.485415111%5D%2C%20%5B126.853961895%2C%2037.48297475200002%5D%2C%20%5B126.85271384700002%2C%2037.48182609899999%5D%2C%20%5B126.85043164800004%2C%2037.48157525800002%5D%2C%20%5B126.84848405499997%2C%2037.482095259%5D%2C%20%5B126.84642145600003%2C%2037.48150571299999%5D%2C%20%5B126.84594900499997%2C%2037.480323644%5D%2C%20%5B126.84537853999996%2C%2037.473821137000016%5D%2C%20%5B126.842799474%2C%2037.47498250699999%5D%2C%20%5B126.84104901299997%2C%2037.474656347%5D%2C%20%5B126.83831728799998%2C%2037.47540135999998%5D%2C%20%5B126.83484912899996%2C%2037.47443089%5D%2C%20%5B126.83488769799999%2C%2037.47575841499997%5D%2C%20%5B126.83349606700006%2C%2037.47716231800001%5D%2C%20%5B126.83176399199999%2C%2037.47765735500002%5D%2C%20%5B126.82788973599997%2C%2037.47599654700002%5D%2C%20%5B126.82435334800005%2C%2037.476259733%5D%2C%20%5B126.82212405999996%2C%2037.47575425399998%5D%2C%20%5B126.82116790299995%2C%2037.476307167000016%5D%2C%20%5B126.81909562600003%2C%2037.476138418%5D%2C%20%5B126.818336391%2C%2037.47530395899997%5D%2C%20%5B126.818647371%2C%2037.474085166%5D%2C%20%5B126.81708964200004%2C%2037.47329169699998%5D%2C%20%5B126.81657626699996%2C%2037.473998096%5D%2C%20%5B126.81464318099995%2C%2037.47465330900002%5D%2C%20%5B126.81531048900001%2C%2037.47636645199998%5D%2C%20%5B126.81722552199994%2C%2037.47814126600002%5D%2C%20%5B126.81827151499999%2C%2037.478170568%5D%2C%20%5B126.81901268700005%2C%2037.47916943500002%5D%2C%20%5B126.81999807199998%2C%2037.48164884%5D%2C%20%5B126.81959813599997%2C%2037.48244711299998%5D%2C%20%5B126.81934165300004%2C%2037.48551259499999%5D%2C%20%5B126.82130494499995%2C%2037.48618739800003%5D%2C%20%5B126.82351963899998%2C%2037.487744236000026%5D%2C%20%5B126.82277597899997%2C%2037.48998772700003%5D%2C%20%5B126.81790518800005%2C%2037.49154342000003%5D%2C%20%5B126.81459113400001%2C%2037.49320034200002%5D%2C%20%5B126.81436892199997%2C%2037.49443227400002%5D%2C%20%5B126.81302860799997%2C%2037.496407777%5D%2C%20%5B126.81415555599995%2C%2037.49813429599999%5D%2C%20%5B126.81611235499997%2C%2037.49765224499998%5D%2C%20%5B126.818224268%2C%2037.49886467499999%5D%2C%20%5B126.81965596999999%2C%2037.499219118999974%5D%2C%20%5B126.81943123600001%2C%2037.50079738%5D%2C%20%5B126.82158116200003%2C%2037.50216396000002%5D%2C%20%5B126.82184366299998%2C%2037.504545139000015%5D%2C%20%5B126.82256759899997%2C%2037.505692083999975%5D%2C%20%5B126.82223955400002%2C%2037.50769455599999%5D%2C%20%5B126.82615287800002%2C%2037.50872472999998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.837508791%2C%2037.50328697200001%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.84076167%2C%2037.50604941900002%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84873119999997%2C%2037.508888773000024%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85562926600005%2C%2037.50916079199999%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.858345407%2C%2037.509923406999974%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%205%2C%20%22SHAPE_AREA%22%3A%200.002047%2C%20%22SHAPE_LEN%22%3A%200.347568%2C%20%22SIG_CD%22%3A%20%2211530%22%2C%20%22SIG_ENG_NM%22%3A%20%22Guro-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.96146158%2C%2037.62996095%5D%2C%20%5B126.96378891899997%2C%2037.62979475499998%5D%2C%20%5B126.96524997699998%2C%2037.63076970399999%5D%2C%20%5B126.968256763%2C%2037.63073174200002%5D%2C%20%5B126.97134811%2C%2037.63162062700002%5D%2C%20%5B126.973049887%2C%2037.632376954999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%206%2C%20%22SHAPE_AREA%22%3A%200.002448%2C%20%22SHAPE_LEN%22%3A%200.2901%2C%20%22SIG_CD%22%3A%20%2211110%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jongno-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98421633700002%2C%2037.63634100199999%5D%2C%20%5B126.98510166300002%2C%2037.637318127000015%5D%2C%20%5B126.98569312400002%2C%2037.640048433%5D%2C%20%5B126.98528469200005%2C%2037.641354295999975%5D%2C%20%5B126.98427374300002%2C%2037.64164886999998%5D%2C%20%5B126.98326407499997%2C%2037.643687516%5D%2C%20%5B126.98512211000002%2C%2037.64573277%5D%2C%20%5B126.98462050299997%2C%2037.647616578%5D%2C%20%5B126.98399871900006%2C%2037.64818593799998%5D%2C%20%5B126.98395619999997%2C%2037.649613708%5D%2C%20%5B126.98251481800003%2C%2037.650544008%5D%2C%20%5B126.98169600799997%2C%2037.652320175%5D%2C%20%5B126.98106737800003%2C%2037.652716521%5D%2C%20%5B126.97996370600004%2C%2037.65481924699998%5D%2C%20%5B126.97967724299997%2C%2037.65604285799998%5D%2C%20%5B126.98302857199997%2C%2037.65700805%5D%2C%20%5B126.98603858499996%2C%2037.65894778400002%5D%2C%20%5B126.987294798%2C%2037.660604260000014%5D%2C%20%5B126.98782001899997%2C%2037.66359695900002%5D%2C%20%5B126.98828632200002%2C%2037.66439418800002%5D%2C%20%5B126.99001520599995%2C%2037.66450267%5D%2C%20%5B126.99224251800001%2C%2037.66523496299999%5D%2C%20%5B126.99414712600003%2C%2037.667033076%5D%2C%20%5B126.99357858400003%2C%2037.66780672099998%5D%2C%20%5B126.99437535000004%2C%2037.66956914000002%5D%2C%20%5B126.99382043900005%2C%2037.670199244%5D%2C%20%5B126.99409198599994%2C%2037.672763257999975%5D%2C%20%5B126.99356165699999%2C%2037.67410523500001%5D%2C%20%5B126.99399960799997%2C%2037.674909437999986%5D%2C%20%5B126.99321780000002%2C%2037.675722317%5D%2C%20%5B126.99326569200002%2C%2037.67766322599999%5D%2C%20%5B126.99221614400005%2C%2037.67962970100001%5D%2C%20%5B126.99409428599995%2C%2037.68032697899997%5D%2C%20%5B126.99475084200003%2C%2037.681246918%5D%2C%20%5B126.99675604100003%2C%2037.68239214%5D%2C%20%5B126.997333905%2C%2037.68337292299998%5D%2C%20%5B127.00170893100005%2C%2037.68426214499999%5D%2C%20%5B127.003605311%2C%2037.684165339%5D%2C%20%5B127.00454835300002%2C%2037.68501282099999%5D%2C%20%5B127.00607232000004%2C%2037.684984327%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%207%2C%20%22SHAPE_AREA%22%3A%200.002412%2C%20%22SHAPE_LEN%22%3A%200.267441%2C%20%22SIG_CD%22%3A%20%2211305%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10914571%2C%2037.62052080400002%5D%2C%20%5B127.11053916100002%2C%2037.62105968399999%5D%2C%20%5B127.11216023500003%2C%2037.620311775%5D%2C%20%5B127.11503737099997%2C%2037.61954149500002%5D%2C%20%5B127.11716289399999%2C%2037.61789223%5D%2C%20%5B127.11668740899995%2C%2037.61601964499999%5D%2C%20%5B127.11749802600002%2C%2037.61177613000001%5D%2C%20%5B127.11677896900005%2C%2037.610134802%5D%2C%20%5B127.116712333%2C%2037.60885401899998%5D%2C%20%5B127.11849163500005%2C%2037.60761747%5D%2C%20%5B127.118075341%2C%2037.606595825%5D%2C%20%5B127.118061087%2C%2037.604606131000025%5D%2C%20%5B127.115739211%2C%2037.60211296%5D%2C%20%5B127.11408091199996%2C%2037.599527276%5D%2C%20%5B127.11689451500001%2C%2037.595503880000024%5D%2C%20%5B127.11668170400003%2C%2037.594023413%5D%2C%20%5B127.11335498100004%2C%2037.59326654199998%5D%2C%20%5B127.110693577%2C%2037.58916251699998%5D%2C%20%5B127.11000566899997%2C%2037.586023006%5D%2C%20%5B127.10898535199999%2C%2037.58338535199999%5D%2C%20%5B127.10345938600005%2C%2037.58060086099999%5D%2C%20%5B127.10304415999997%2C%2037.578912213000024%5D%2C%20%5B127.10116216200004%2C%2037.57607578599999%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%208%2C%20%22SHAPE_AREA%22%3A%200.001893%2C%20%22SHAPE_LEN%22%3A%200.184716%2C%20%22SIG_CD%22%3A%20%2211260%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jungnang-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.12187312100002%2C%2037.46514826399999%5D%2C%20%5B127.12145320599996%2C%2037.46438168499998%5D%2C%20%5B127.11748896200004%2C%2037.462210077%5D%2C%20%5B127.11700353900005%2C%2037.46149384900002%5D%2C%20%5B127.116910318%2C%2037.458650388000024%5D%2C%20%5B127.11590693999995%2C%2037.45860154000002%5D%2C%20%5B127.11328498199998%2C%2037.460064267%5D%2C%20%5B127.11298953899995%2C%2037.46114412899999%5D%2C%20%5B127.11183504999997%2C%2037.461654001%5D%2C%20%5B127.10648246400001%2C%2037.46243379200001%5D%2C%20%5B127.10435658899996%2C%2037.46218425799998%5D%2C%20%5B127.10479563499996%2C%2037.46116971399999%5D%2C%20%5B127.10395681800003%2C%2037.46006054700001%5D%2C%20%5B127.10139603200003%2C%2037.45898237099999%5D%2C%20%5B127.10129219299995%2C%2037.45825120400002%5D%2C%20%5B127.09975283000006%2C%2037.457364144%5D%2C%20%5B127.099109139%2C%2037.45622823999997%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%209%2C%20%22SHAPE_AREA%22%3A%200.004027%2C%20%22SHAPE_LEN%22%3A%200.348412%2C%20%22SIG_CD%22%3A%20%2211680%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangnam-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.81658266%2C%2037.540568317%5D%2C%20%5B126.81234887999994%2C%2037.54074469400001%5D%2C%20%5B126.81060934799996%2C%2037.54150444200002%5D%2C%20%5B126.80725610699994%2C%2037.54362225199998%5D%2C%20%5B126.80186598399996%2C%2037.542723611999975%5D%2C%20%5B126.80048554400003%2C%2037.54177814600001%5D%2C%20%5B126.80040658300004%2C%2037.540804252999976%5D%2C%20%5B126.79887833%2C%2037.5402999%5D%2C%20%5B126.79875436099996%2C%2037.539406192%5D%2C%20%5B126.79949804199998%2C%2037.537775685999975%5D%2C%20%5B126.79782849100002%2C%2037.537056888999984%5D%2C%20%5B126.79605918200002%2C%2037.53675834400002%5D%2C%20%5B126.79481063799994%2C%2037.535972963%5D%2C%20%5B126.79397960699998%2C%2037.537369168%5D%2C%20%5B126.79390197299995%2C%2037.539558371%5D%2C%20%5B126.79496077700003%2C%2037.541381135999984%5D%2C%20%5B126.79182670499995%2C%2037.54191169%5D%2C%20%5B126.79159995700002%2C%2037.54307308199998%5D%2C%20%5B126.79077654100001%2C%2037.543847862%5D%2C%20%5B126.78872746499997%2C%2037.54482803399998%5D%2C%20%5B126.78727682700003%2C%2037.54608495799999%5D%2C%20%5B126.78332478000004%2C%2037.54603042999997%5D%2C%20%5B126.77875616300003%2C%2037.546729674%5D%2C%20%5B126.77757101300006%2C%2037.54672980700002%5D%2C%20%5B126.77675212199995%2C%2037.548221075000015%5D%2C%20%5B126.775301227%2C%2037.548988871%5D%2C%20%5B126.77258571799996%2C%2037.548773638%5D%2C%20%5B126.77169496199997%2C%2037.548329146000015%5D%2C%20%5B126.76992633199995%2C%2037.550196289999974%5D%2C%20%5B126.769793875%2C%2037.551086956%5D%2C%20%5B126.767468484%2C%2037.551972233000015%5D%2C%20%5B126.76742527299996%2C%2037.554237899999976%5D%2C%20%5B126.76458060899995%2C%2037.555466594999984%5D%2C%20%5B126.76720647299999%2C%2037.55666916000001%5D%2C%20%5B126.76988573200003%2C%2037.55724517599998%5D%2C%20%5B126.772397404%2C%2037.55699389%5D%2C%20%5B126.773200245%2C%2037.557536051%5D%2C%20%5B126.77183969700002%2C%2037.55861259400001%5D%2C%20%5B126.77320943200004%2C%2037.55929739800001%5D%2C%20%5B126.77612545800002%2C%2037.56187228599998%5D%2C%20%5B126.77795148799999%2C%2037.56016005399999%5D%2C%20%5B126.7771186%2C%2037.563728853999976%5D%2C%20%5B126.77515191199996%2C%2037.565895781999984%5D%2C%20%5B126.77478348299996%2C%2037.56772791100002%5D%2C%20%5B126.77635320499996%2C%2037.567107521000025%5D%2C%20%5B126.777826946%2C%2037.567006467%5D%2C%20%5B126.78025671600005%2C%2037.567453419%5D%2C%20%5B126.780491568%2C%2037.56829614100002%5D%2C%20%5B126.781642583%2C%2037.568909259%5D%2C%20%5B126.78133354800002%2C%2037.570185116%5D%2C%20%5B126.782647215%2C%2037.57055544799999%5D%2C%20%5B126.78191796099998%2C%2037.57181951000001%5D%2C%20%5B126.78242742400005%2C%2037.57361885900002%5D%2C%20%5B126.78526689%2C%2037.57487061799998%5D%2C%20%5B126.787630605%2C%2037.57558866%5D%2C%20%5B126.78935921899995%2C%2037.57772807700002%5D%2C%20%5B126.79058199500003%2C%2037.577776819%5D%2C%20%5B126.79074497399995%2C%2037.58061836799999%5D%2C%20%5B126.79228461599996%2C%2037.580056910999986%5D%2C%20%5B126.79275533099997%2C%2037.57684457%5D%2C%20%5B126.79323752200003%2C%2037.57689948500001%5D%2C%20%5B126.79288111400001%2C%2037.58022635499998%5D%2C%20%5B126.79390501399996%2C%2037.582257949%5D%2C%20%5B126.79411019999998%2C%2037.58425737099998%5D%2C%20%5B126.79555674699998%2C%2037.58515790000001%5D%2C%20%5B126.79569588599998%2C%2037.583367872%5D%2C%20%5B126.79711367100003%2C%2037.58427787900001%5D%2C%20%5B126.79670218499996%2C%2037.58491212500002%5D%2C%20%5B126.79878484200003%2C%2037.58807839000002%5D%2C%20%5B126.80075747499995%2C%2037.587841243000014%5D%2C%20%5B126.80053959500003%2C%2037.590315551%5D%2C%20%5B126.798817848%2C%2037.59132633199999%5D%2C%20%5B126.79867010299995%2C%2037.59366814100002%5D%2C%20%5B126.79745919100003%2C%2037.595248595999976%5D%2C%20%5B126.79708180299997%2C%2037.59773291099998%5D%2C%20%5B126.79756259099997%2C%2037.59793496100002%5D%2C%20%5B126.79787158299996%2C%2037.599898203%5D%2C%20%5B126.799935054%2C%2037.601445969%5D%2C%20%5B126.79994633199999%2C%2037.602546226000015%5D%2C%20%5B126.80258612299997%2C%2037.605033999%5D%2C%20%5B126.80587186000002%2C%2037.60258296500001%5D%2C%20%5B126.808249756%2C%2037.601211834000026%5D%2C%20%5B126.81138204800004%2C%2037.59896582800002%5D%2C%20%5B126.81396850600004%2C%2037.59762536400001%5D%2C%20%5B126.816573747%2C%2037.595688145999986%5D%2C%20%5B126.81754378000005%2C%2037.59533660400001%5D%2C%20%5B126.81932148199996%2C%2037.592855004%5D%2C%20%5B126.82783387699999%2C%2037.58748727300002%5D%2C%20%5B126.83178168200004%2C%2037.585846134%5D%2C%20%5B126.837170002%2C%2037.582568493%5D%2C%20%5B126.83811851799999%2C%2037.581798564999986%5D%2C%20%5B126.84326033599996%2C%2037.578886398%5D%2C%20%5B126.84840263399997%2C%2037.57516434500002%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2010%2C%20%22SHAPE_AREA%22%3A%200.004227%2C%20%22SHAPE_LEN%22%3A%200.435694%2C%20%22SIG_CD%22%3A%20%2211500%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangseo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2011%2C%20%22SHAPE_AREA%22%3A%200.001017%2C%20%22SHAPE_LEN%22%3A%200.191242%2C%20%22SIG_CD%22%3A%20%2211140%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jung-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.11428417299999%2C%2037.554386276%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11566284000003%2C%2037.557702898%5D%2C%20%5B127.11735874600004%2C%2037.559467202%5D%2C%20%5B127.12011629799997%2C%2037.56120324699998%5D%2C%20%5B127.12297571600004%2C%2037.56349386%5D%2C%20%5B127.12862746099995%2C%2037.56616545000003%5D%2C%20%5B127.134283364%2C%2037.567961086000025%5D%2C%20%5B127.13768321700002%2C%2037.56843244200002%5D%2C%20%5B127.14896618800003%2C%2037.568444026%5D%2C%20%5B127.154998245%2C%2037.57204888699999%5D%2C%20%5B127.15691686699995%2C%2037.572958835%5D%2C%20%5B127.16257241200003%2C%2037.576732698%5D%2C%20%5B127.16676596299999%2C%2037.57897662%5D%2C%20%5B127.17026691299998%2C%2037.57901745999999%5D%2C%20%5B127.17185117300005%2C%2037.57926277399997%5D%2C%20%5B127.17521472800001%2C%2037.58046697200001%5D%2C%20%5B127.17734763199996%2C%2037.581575015%5D%2C%20%5B127.17701410100005%2C%2037.579511809%5D%2C%20%5B127.17550670900005%2C%2037.578472404000024%5D%2C%20%5B127.17534214%2C%2037.57723767800002%5D%2C%20%5B127.17569917399999%2C%2037.57490180399998%5D%2C%20%5B127.17789114499999%2C%2037.57187252799997%5D%2C%20%5B127.17922984100005%2C%2037.56892072800002%5D%2C%20%5B127.17959913300001%2C%2037.56549991899999%5D%2C%20%5B127.181200053%2C%2037.56301996299999%5D%2C%20%5B127.18200949499999%2C%2037.56099529800002%5D%2C%20%5B127.181566868%2C%2037.557559535%5D%2C%20%5B127.18169641500003%2C%2037.55600353400001%5D%2C%20%5B127.18135382599996%2C%2037.552973123000015%5D%2C%20%5B127.18294495500004%2C%2037.55177914199999%5D%2C%20%5B127.183131151%2C%2037.551111615000025%5D%2C%20%5B127.18268913400004%2C%2037.54791615900001%5D%2C%20%5B127.182837425%2C%2037.54639986199999%5D%2C%20%5B127.17934642900002%2C%2037.54657307399998%5D%2C%20%5B127.17573322400006%2C%2037.545213538999974%5D%2C%20%5B127.17392236499995%2C%2037.545588527%5D%2C%20%5B127.17212391600003%2C%2037.545541722999985%5D%2C%20%5B127.16978076600003%2C%2037.544813007000016%5D%2C%20%5B127.16699588400002%2C%2037.54521138699999%5D%2C%20%5B127.16665500399995%2C%2037.54426145999997%5D%2C%20%5B127.16317850999997%2C%2037.544997455999976%5D%2C%20%5B127.16032596900004%2C%2037.541629215%5D%2C%20%5B127.15766934199996%2C%2037.53927536100002%5D%2C%20%5B127.15696225299996%2C%2037.537974744999985%5D%2C%20%5B127.15527845999998%2C%2037.53622368600003%5D%2C%20%5B127.15358375799997%2C%2037.533672017000015%5D%2C%20%5B127.15395052600002%2C%2037.531926252%5D%2C%20%5B127.15270367899996%2C%2037.528535456999975%5D%2C%20%5B127.15041863099998%2C%2037.52637816499998%5D%2C%20%5B127.14781481299997%2C%2037.52215484599998%5D%2C%20%5B127.145682889%2C%2037.52193569399998%5D%2C%20%5B127.144801873%2C%2037.519630431999985%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2012%2C%20%22SHAPE_AREA%22%3A%200.002504%2C%20%22SHAPE_LEN%22%3A%200.242596%2C%20%22SIG_CD%22%3A%20%2211740%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11411142500003%2C%2037.55409376599999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.10168325400002%2C%2037.57240515400002%5D%2C%20%5B127.103148411%2C%2037.57228406899998%5D%2C%20%5B127.10424898199994%2C%2037.571392263%5D%2C%20%5B127.10347415499996%2C%2037.57005447199998%5D%2C%20%5B127.10240523599998%2C%2037.56564081%5D%2C%20%5B127.10225269499995%2C%2037.564314869999976%5D%2C%20%5B127.10122363599999%2C%2037.56158417300003%5D%2C%20%5B127.10187808800003%2C%2037.559498245999976%5D%2C%20%5B127.10429532800003%2C%2037.55756740800001%5D%2C%20%5B127.10494775899997%2C%2037.55642552299997%5D%2C%20%5B127.10641446099999%2C%2037.55646542699998%5D%2C%20%5B127.10753783500002%2C%2037.55761021400002%5D%2C%20%5B127.10954748999995%2C%2037.558557784000016%5D%2C%20%5B127.11036889499997%2C%2037.55825685100001%5D%2C%20%5B127.11231650399998%2C%2037.55900405400001%5D%2C%20%5B127.11388262800006%2C%2037.55843289500001%5D%2C%20%5B127.11333075200002%2C%2037.55683318299998%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2013%2C%20%22SHAPE_AREA%22%3A%200.001737%2C%20%22SHAPE_LEN%22%3A%200.186732%2C%20%22SIG_CD%22%3A%20%2211215%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwangjin-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85364710199997%2C%2037.573804395000025%5D%2C%20%5B126.85930276099998%2C%2037.57492123999998%5D%2C%20%5B126.85931123%2C%2037.575246692%5D%2C%20%5B126.86495888599995%2C%2037.57747814700002%5D%2C%20%5B126.868396404%2C%2037.577526395%5D%2C%20%5B126.870619272%2C%2037.578013113%5D%2C%20%5B126.87393583999994%2C%2037.57828489799999%5D%2C%20%5B126.87628222700005%2C%2037.57818935199998%5D%2C%20%5B126.87765407400002%2C%2037.57975304299998%5D%2C%20%5B126.87668444099995%2C%2037.581263981%5D%2C%20%5B126.87707186%2C%2037.58221029200001%5D%2C%20%5B126.87710903799996%2C%2037.58483216500002%5D%2C%20%5B126.877564388%2C%2037.586252527%5D%2C%20%5B126.87915565399999%2C%2037.58677640399998%5D%2C%20%5B126.88036942999997%2C%2037.58950964899998%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2014%2C%20%22SHAPE_AREA%22%3A%200.002415%2C%20%22SHAPE_LEN%22%3A%200.289336%2C%20%22SIG_CD%22%3A%20%2211440%22%2C%20%22SIG_ENG_NM%22%3A%20%22Mapo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98662702599995%2C%2037.457216786%5D%2C%20%5B126.98290313899997%2C%2037.456831314%5D%2C%20%5B126.98241282200001%2C%2037.455895618%5D%2C%20%5B126.97849284100005%2C%2037.455739153000025%5D%2C%20%5B126.97777249%2C%2037.455203476%5D%2C%20%5B126.97459526800003%2C%2037.45441720500003%5D%2C%20%5B126.97285906000002%2C%2037.45238675000002%5D%2C%20%5B126.97176838200005%2C%2037.45174170500002%5D%2C%20%5B126.97058548899997%2C%2037.449457128%5D%2C%20%5B126.96767973600004%2C%2037.44838496%5D%2C%20%5B126.964307298%2C%2037.446271943%5D%2C%20%5B126.96395069300002%2C%2037.44521969200002%5D%2C%20%5B126.96443595200003%2C%2037.44431798300002%5D%2C%20%5B126.96464967600002%2C%2037.44204566000002%5D%2C%20%5B126.96295770799998%2C%2037.44028699400002%5D%2C%20%5B126.96007066799996%2C%2037.44040704499997%5D%2C%20%5B126.95898017000002%2C%2037.43907666199999%5D%2C%20%5B126.95512268899995%2C%2037.43869914800001%5D%2C%20%5B126.95242507900002%2C%2037.43919711699999%5D%2C%20%5B126.95148094199999%2C%2037.43858365%5D%2C%20%5B126.94929766799999%2C%2037.43825732%5D%2C%20%5B126.94837678299996%2C%2037.43871966299997%5D%2C%20%5B126.94680271000004%2C%2037.43817427499999%5D%2C%20%5B126.94510345499998%2C%2037.43709568600002%5D%2C%20%5B126.94143729300004%2C%2037.437411664000024%5D%2C%20%5B126.940232358%2C%2037.435720186000026%5D%2C%20%5B126.93858713099996%2C%2037.43642808300001%5D%2C%20%5B126.93775941000001%2C%2037.437393122%5D%2C%20%5B126.93726096700004%2C%2037.439219657000024%5D%2C%20%5B126.93787838100002%2C%2037.44020405700002%5D%2C%20%5B126.93666192499995%2C%2037.44175181600002%5D%2C%20%5B126.93176733099995%2C%2037.444982075999974%5D%2C%20%5B126.93057246299998%2C%2037.445474494%5D%2C%20%5B126.93018248700002%2C%2037.44638402800001%5D%2C%20%5B126.93018641599997%2C%2037.448390587%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2015%2C%20%22SHAPE_AREA%22%3A%200.003012%2C%20%22SHAPE_LEN%22%3A%200.280092%2C%20%22SIG_CD%22%3A%20%2211620%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwanak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.093554412%2C%2037.45589802400002%5D%2C%20%5B127.09273821199997%2C%2037.453643511999985%5D%2C%20%5B127.09097172%2C%2037.45279201300002%5D%2C%20%5B127.09031546300002%2C%2037.45127626300001%5D%2C%20%5B127.088339345%2C%2037.44863720799998%5D%2C%20%5B127.08817389900003%2C%2037.44539478899998%5D%2C%20%5B127.08643383499998%2C%2037.444272048000016%5D%2C%20%5B127.08387430300002%2C%2037.44393121899998%5D%2C%20%5B127.08244417699996%2C%2037.441589282%5D%2C%20%5B127.08018384599995%2C%2037.441060924%5D%2C%20%5B127.07602324899995%2C%2037.4421491%5D%2C%20%5B127.07300687899999%2C%2037.442027784%5D%2C%20%5B127.07163946599997%2C%2037.441534323999974%5D%2C%20%5B127.07202035099999%2C%2037.43886650600001%5D%2C%20%5B127.07377263199999%2C%2037.4377991%5D%2C%20%5B127.07303654400005%2C%2037.43641643799998%5D%2C%20%5B127.07145178799999%2C%2037.43587310999999%5D%2C%20%5B127.07153192800001%2C%2037.434126424%5D%2C%20%5B127.07064098%2C%2037.432054365%5D%2C%20%5B127.07121871100003%2C%2037.430827679%5D%2C%20%5B127.06828150800004%2C%2037.43068911%5D%2C%20%5B127.06673287599995%2C%2037.43013986099999%5D%2C%20%5B127.06569412299996%2C%2037.42899537400001%5D%2C%20%5B127.06332888700001%2C%2037.429761353%5D%2C%20%5B127.06115775299997%2C%2037.429994996%5D%2C%20%5B127.05997391899996%2C%2037.429578751%5D%2C%20%5B127.05808265300004%2C%2037.430020472000024%5D%2C%20%5B127.05369652000002%2C%2037.428984660000026%5D%2C%20%5B127.05218338899999%2C%2037.42834757000003%5D%2C%20%5B127.04959235199999%2C%2037.430279794%5D%2C%20%5B127.04738197699999%2C%2037.43071133900003%5D%2C%20%5B127.04723810300004%2C%2037.43204956%5D%2C%20%5B127.04632535799999%2C%2037.43345562899998%5D%2C%20%5B127.04496506500004%2C%2037.43385551799997%5D%2C%20%5B127.044199833%2C%2037.435063092%5D%2C%20%5B127.04006758800006%2C%2037.438243274%5D%2C%20%5B127.03708267800005%2C%2037.43828114399997%5D%2C%20%5B127.03558860400005%2C%2037.43901011600002%5D%2C%20%5B127.03518600799998%2C%2037.440947884000025%5D%2C%20%5B127.03644468300001%2C%2037.44183865799999%5D%2C%20%5B127.03749597800004%2C%2037.443272594%5D%2C%20%5B127.038243717%2C%2037.44518611900003%5D%2C%20%5B127.03726765%2C%2037.44645425300001%5D%2C%20%5B127.03717336600005%2C%2037.44801560799999%5D%2C%20%5B127.03782636200003%2C%2037.44892774700003%5D%2C%20%5B127.03692812600002%2C%2037.45081401300001%5D%2C%20%5B127.03579206200004%2C%2037.452201682%5D%2C%20%5B127.03477066100004%2C%2037.45262298599999%5D%2C%20%5B127.03714478699999%2C%2037.45521278799998%5D%2C%20%5B127.03675713500002%2C%2037.45644861599999%5D%2C%20%5B127.03503165799998%2C%2037.45789780400003%5D%2C%20%5B127.03494060599996%2C%2037.46018196900002%5D%2C%20%5B127.033746501%2C%2037.461242513%5D%2C%20%5B127.03471186900003%2C%2037.46416056200002%5D%2C%20%5B127.03315839200002%2C%2037.465024089999986%5D%2C%20%5B127.03121260499995%2C%2037.46563217200003%5D%2C%20%5B127.02954249100003%2C%2037.46537973%5D%2C%20%5B127.02970909099997%2C%2037.463910751000014%5D%2C%20%5B127.02933072500002%2C%2037.46270655%5D%2C%20%5B127.02805724300003%2C%2037.46125024600002%5D%2C%20%5B127.02837573700003%2C%2037.459898619%5D%2C%20%5B127.02658642100005%2C%2037.459646512%5D%2C%20%5B127.02645936199997%2C%2037.45834107600001%5D%2C%20%5B127.02529273699997%2C%2037.457498327%5D%2C%20%5B127.02280887500001%2C%2037.457254868%5D%2C%20%5B127.02152010400005%2C%2037.456229913000016%5D%2C%20%5B127.01970823600004%2C%2037.45579098899998%5D%2C%20%5B127.01756149699997%2C%2037.45613975200001%5D%2C%20%5B127.01630291799995%2C%2037.455234355000016%5D%2C%20%5B127.01452250900002%2C%2037.45486602300002%5D%2C%20%5B127.01066817699996%2C%2037.456042489000026%5D%2C%20%5B127.008544057%2C%2037.45805628699998%5D%2C%20%5B127.00822678700001%2C%2037.459104508%5D%2C%20%5B127.00705377999998%2C%2037.460220048999986%5D%2C%20%5B127.00578293399997%2C%2037.46255271299998%5D%2C%20%5B127.00448335800002%2C%2037.464077615%5D%2C%20%5B127.00492006399998%2C%2037.46584683899999%5D%2C%20%5B127.00369035799997%2C%2037.46772444499999%5D%2C%20%5B127.00274926199995%2C%2037.467128762000016%5D%2C%20%5B126.99879405800004%2C%2037.46723823899998%5D%2C%20%5B126.99676674199998%2C%2037.467076564000024%5D%2C%20%5B126.996245907%2C%2037.46661254999998%5D%2C%20%5B126.99663361800003%2C%2037.46478279199999%5D%2C%20%5B126.997384828%2C%2037.46369182699999%5D%2C%20%5B126.996782414%2C%2037.461877791%5D%2C%20%5B126.993368691%2C%2037.46150029400002%5D%2C%20%5B126.99266931099999%2C%2037.46034838899999%5D%2C%20%5B126.99171279500001%2C%2037.46052342799999%5D%2C%20%5B126.99000889599995%2C%2037.45946741%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2016%2C%20%22SHAPE_AREA%22%3A%200.004776%2C%20%22SHAPE_LEN%22%3A%200.437399%2C%20%22SIG_CD%22%3A%20%2211650%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seocho-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.97782373300004%2C%2037.63391334300002%5D%2C%20%5B126.981433809%2C%2037.634832856%5D%2C%20%5B126.983724113%2C%2037.636460397%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2017%2C%20%22SHAPE_AREA%22%3A%200.002511%2C%20%22SHAPE_LEN%22%3A%200.318673%2C%20%22SIG_CD%22%3A%20%2211290%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05182101699995%2C%2037.687069296%5D%2C%20%5B127.05519755800003%2C%2037.689209908%5D%2C%20%5B127.05840713099997%2C%2037.689689869%5D%2C%20%5B127.05969812599994%2C%2037.690342471%5D%2C%20%5B127.06130232099997%2C%2037.69219614100001%5D%2C%20%5B127.06249161400001%2C%2037.69305951199999%5D%2C%20%5B127.06274278700005%2C%2037.69467682200002%5D%2C%20%5B127.06356049299995%2C%2037.69497166299999%5D%2C%20%5B127.06650615199999%2C%2037.694498565%5D%2C%20%5B127.068049664%2C%2037.694837018999976%5D%2C%20%5B127.069366791%2C%2037.69375161099998%5D%2C%20%5B127.07291609799995%2C%2037.69387412100002%5D%2C%20%5B127.07522005500005%2C%2037.695310147999976%5D%2C%20%5B127.07856303100004%2C%2037.696092556999986%5D%2C%20%5B127.08150813999998%2C%2037.69625423399998%5D%2C%20%5B127.084240346%2C%2037.694384093%5D%2C%20%5B127.08425235100003%2C%2037.69188345499998%5D%2C%20%5B127.08551096600002%2C%2037.69048708600002%5D%2C%20%5B127.08687032%2C%2037.69001125599999%5D%2C%20%5B127.09093103999999%2C%2037.68968092699998%5D%2C%20%5B127.09345258600001%2C%2037.690087849%5D%2C%20%5B127.096066758%2C%2037.687916323000024%5D%2C%20%5B127.09641572400005%2C%2037.685647404%5D%2C%20%5B127.09580503999996%2C%2037.684650288%5D%2C%20%5B127.09428565300004%2C%2037.683427962999986%5D%2C%20%5B127.09292189500002%2C%2037.68156068000002%5D%2C%20%5B127.09282796000002%2C%2037.68005578999998%5D%2C%20%5B127.09197763199995%2C%2037.67920024699998%5D%2C%20%5B127.09221964100004%2C%2037.678039738%5D%2C%20%5B127.09388436100005%2C%2037.676669027%5D%2C%20%5B127.09464265999998%2C%2037.67354516199998%5D%2C%20%5B127.09577928399995%2C%2037.672681931%5D%2C%20%5B127.09577485800003%2C%2037.67062784799998%5D%2C%20%5B127.09626208899999%2C%2037.668837161%5D%2C%20%5B127.09500097800003%2C%2037.667606956999975%5D%2C%20%5B127.09469822699998%2C%2037.664927189000025%5D%2C%20%5B127.09542861800003%2C%2037.663895431000014%5D%2C%20%5B127.094157774%2C%2037.66330400700002%5D%2C%20%5B127.091221335%2C%2037.65934019600002%5D%2C%20%5B127.09114721900005%2C%2037.658248809999975%5D%2C%20%5B127.09190627600003%2C%2037.65744077099998%5D%2C%20%5B127.09236725000005%2C%2037.65549788499999%5D%2C%20%5B127.09401619699997%2C%2037.65230150399998%5D%2C%20%5B127.09248088799995%2C%2037.64971631999998%5D%2C%20%5B127.09364724600005%2C%2037.646632361%5D%2C%20%5B127.09448968499999%2C%2037.645856236999975%5D%2C%20%5B127.09458566599994%2C%2037.644576943%5D%2C%20%5B127.09757475499998%2C%2037.643971025999974%5D%2C%20%5B127.10160801200004%2C%2037.64477886399999%5D%2C%20%5B127.103936524%2C%2037.645499323000024%5D%2C%20%5B127.10658921100003%2C%2037.645384454%5D%2C%20%5B127.10770375100003%2C%2037.64497346799999%5D%2C%20%5B127.10930262900001%2C%2037.64286426299998%5D%2C%20%5B127.11076511800002%2C%2037.64273476300002%5D%2C%20%5B127.11223881%2C%2037.64037708500001%5D%2C%20%5B127.110580357%2C%2037.639364204%5D%2C%20%5B127.11249787%2C%2037.63648892100002%5D%2C%20%5B127.11242501499999%2C%2037.63425247999999%5D%2C%20%5B127.11162094999997%2C%2037.633851106%5D%2C%20%5B127.11222175299997%2C%2037.632645615%5D%2C%20%5B127.111649757%2C%2037.63150772300003%5D%2C%20%5B127.108900973%2C%2037.62930749999998%5D%2C%20%5B127.10597855699996%2C%2037.627337269%5D%2C%20%5B127.10434078499998%2C%2037.62327850299999%5D%2C%20%5B127.10407893299998%2C%2037.62166095499998%5D%2C%20%5B127.10492130800003%2C%2037.62157580500002%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2018%2C%20%22SHAPE_AREA%22%3A%200.00364%2C%20%22SHAPE_LEN%22%3A%200.309723%2C%20%22SIG_CD%22%3A%20%2211350%22%2C%20%22SIG_ENG_NM%22%3A%20%22Nowon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.144621648%2C%2037.51561223599998%5D%2C%20%5B127.14215399099999%2C%2037.51566003099998%5D%2C%20%5B127.14138931900004%2C%2037.514975155%5D%2C%20%5B127.14341256199998%2C%2037.51387651099998%5D%2C%20%5B127.14354620699999%2C%2037.512663953000015%5D%2C%20%5B127.1414575%2C%2037.51240795199999%5D%2C%20%5B127.14096457300002%2C%2037.51040582000002%5D%2C%20%5B127.13998328499997%2C%2037.50864916199998%5D%2C%20%5B127.141534557%2C%2037.506717287000015%5D%2C%20%5B127.14102941299996%2C%2037.505493221999984%5D%2C%20%5B127.144121606%2C%2037.50440674700002%5D%2C%20%5B127.14597136099997%2C%2037.503236346999984%5D%2C%20%5B127.14771286899997%2C%2037.503223030000015%5D%2C%20%5B127.15022088800004%2C%2037.50474927699997%5D%2C%20%5B127.15212420600005%2C%2037.50297788%5D%2C%20%5B127.15285967099999%2C%2037.503062426999975%5D%2C%20%5B127.15656939099995%2C%2037.50190080300001%5D%2C%20%5B127.157743278%2C%2037.50318546800003%5D%2C%20%5B127.15887991900001%2C%2037.50239690400002%5D%2C%20%5B127.159410547%2C%2037.501369312%5D%2C%20%5B127.16140357300003%2C%2037.50020737699998%5D%2C%20%5B127.15977913799998%2C%2037.49679815600001%5D%2C%20%5B127.16001896600005%2C%2037.49433240399998%5D%2C%20%5B127.15848990500001%2C%2037.492042157000014%5D%2C%20%5B127.158182699%2C%2037.49051667399999%5D%2C%20%5B127.156300945%2C%2037.48949331900002%5D%2C%20%5B127.154795626%2C%2037.48731379999998%5D%2C%20%5B127.15219320300002%2C%2037.48624278199998%5D%2C%20%5B127.150769406%2C%2037.48474927500001%5D%2C%20%5B127.15104542899996%2C%2037.483961959%5D%2C%20%5B127.15068947400005%2C%2037.48202662699998%5D%2C%20%5B127.14915395599996%2C%2037.48074899300002%5D%2C%20%5B127.14783019699996%2C%2037.478664200000026%5D%2C%20%5B127.14439008099998%2C%2037.476456739000014%5D%2C%20%5B127.14354172599997%2C%2037.475545689%5D%2C%20%5B127.139692276%2C%2037.47419605099998%5D%2C%20%5B127.13937783300003%2C%2037.473447066%5D%2C%20%5B127.13558350400001%2C%2037.474578011%5D%2C%20%5B127.13368163799998%2C%2037.47568719499998%5D%2C%20%5B127.13267746400004%2C%2037.475808433%5D%2C%20%5B127.13024850199997%2C%2037.47510375899998%5D%2C%20%5B127.13042151800005%2C%2037.47189767899999%5D%2C%20%5B127.13538943799995%2C%2037.46932071200001%5D%2C%20%5B127.13404767700001%2C%2037.46859963000003%5D%2C%20%5B127.13014299300005%2C%2037.467765722000024%5D%2C%20%5B127.12704911399999%2C%2037.46842315399999%5D%2C%20%5B127.12521919400001%2C%2037.46962967899998%5D%2C%20%5B127.125063372%2C%2037.46727479200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2019%2C%20%22SHAPE_AREA%22%3A%200.003449%2C%20%22SHAPE_LEN%22%3A%200.316773%2C%20%22SIG_CD%22%3A%20%2211710%22%2C%20%22SIG_ENG_NM%22%3A%20%22Songpa-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2020%2C%20%22SHAPE_AREA%22%3A%200.001714%2C%20%22SHAPE_LEN%22%3A%200.193793%2C%20%22SIG_CD%22%3A%20%2211200%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2021%2C%20%22SHAPE_AREA%22%3A%200.001811%2C%20%22SHAPE_LEN%22%3A%200.231131%2C%20%22SIG_CD%22%3A%20%2211410%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seodaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.858345407%2C%2037.50992339800001%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.85562926600005%2C%2037.50916078300003%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.84873119999997%2C%2037.508888764%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84076167%2C%2037.50604941%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.837508791%2C%2037.50328696299999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.826851927%2C%2037.510431685000015%5D%2C%20%5B126.82420556800002%2C%2037.51053149099999%5D%2C%20%5B126.82387418500002%2C%2037.51088144200003%5D%2C%20%5B126.824425776%2C%2037.513182637%5D%2C%20%5B126.824273167%2C%2037.514473764%5D%2C%20%5B126.82341871799997%2C%2037.51495970899998%5D%2C%20%5B126.82312208200005%2C%2037.51622650500002%5D%2C%20%5B126.82451390100005%2C%2037.51772160199999%5D%2C%20%5B126.82563535199995%2C%2037.520085435%5D%2C%20%5B126.82515520799996%2C%2037.523010279%5D%2C%20%5B126.82676624299995%2C%2037.525130822999984%5D%2C%20%5B126.82816515299999%2C%2037.525836518%5D%2C%20%5B126.82844406000004%2C%2037.52670751199997%5D%2C%20%5B126.827086602%2C%2037.529715297%5D%2C%20%5B126.825575308%2C%2037.529853316000015%5D%2C%20%5B126.82344723300002%2C%2037.532623817%5D%2C%20%5B126.82356902000004%2C%2037.533770715%5D%2C%20%5B126.82193790199995%2C%2037.534835165%5D%2C%20%5B126.82244956399995%2C%2037.536550956999974%5D%2C%20%5B126.82233701300004%2C%2037.53793206300003%5D%2C%20%5B126.82160576399997%2C%2037.539720295999985%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2022%2C%20%22SHAPE_AREA%22%3A%200.001775%2C%20%22SHAPE_LEN%22%3A%200.294094%2C%20%22SIG_CD%22%3A%20%2211470%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yangcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88272419199996%2C%2037.51576865800001%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2023%2C%20%22SHAPE_AREA%22%3A%200.002512%2C%20%22SHAPE_LEN%22%3A%200.262953%2C%20%22SIG_CD%22%3A%20%2211560%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yeongdeungpo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2024%2C%20%22SHAPE_AREA%22%3A%200.002234%2C%20%22SHAPE_LEN%22%3A%200.214496%2C%20%22SIG_CD%22%3A%20%2211170%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yongsan-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20var%20color_map_801479b7173b4c5b83daafec6ff5f357%20%3D%20%7B%7D%3B%0A%0A%20%20%20%20%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.color%20%3D%20d3.scale.threshold%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B0.0%2C%207.38877755511022%2C%2014.77755511022044%2C%2022.16633266533066%2C%2029.55511022044088%2C%2036.943887775551104%2C%2044.33266533066132%2C%2051.721442885771545%2C%2059.11022044088176%2C%2066.49899799599199%2C%2073.88777555110221%2C%2081.27655310621242%2C%2088.66533066132264%2C%2096.05410821643287%2C%20103.44288577154309%2C%20110.83166332665331%2C%20118.22044088176352%2C%20125.60921843687375%2C%20132.99799599198397%2C%20140.38677354709418%2C%20147.77555110220442%2C%20155.16432865731463%2C%20162.55310621242484%2C%20169.94188376753507%2C%20177.33066132264528%2C%20184.71943887775552%2C%20192.10821643286573%2C%20199.49699398797594%2C%20206.88577154308618%2C%20214.2745490981964%2C%20221.66332665330663%2C%20229.05210420841684%2C%20236.44088176352705%2C%20243.82965931863728%2C%20251.2184368737475%2C%20258.60721442885773%2C%20265.99599198396794%2C%20273.38476953907815%2C%20280.77354709418836%2C%20288.1623246492986%2C%20295.55110220440883%2C%20302.93987975951904%2C%20310.32865731462925%2C%20317.71743486973946%2C%20325.1062124248497%2C%20332.49498997995994%2C%20339.88376753507015%2C%20347.27254509018036%2C%20354.66132264529057%2C%20362.0501002004008%2C%20369.43887775551104%2C%20376.82765531062125%2C%20384.21643286573146%2C%20391.6052104208417%2C%20398.9939879759519%2C%20406.38276553106215%2C%20413.77154308617236%2C%20421.16032064128257%2C%20428.5490981963928%2C%20435.937875751503%2C%20443.32665330661325%2C%20450.71543086172346%2C%20458.10420841683367%2C%20465.4929859719439%2C%20472.8817635270541%2C%20480.27054108216436%2C%20487.65931863727457%2C%20495.0480961923848%2C%20502.436873747495%2C%20509.8256513026052%2C%20517.2144288577155%2C%20524.6032064128257%2C%20531.9919839679359%2C%20539.3807615230461%2C%20546.7695390781563%2C%20554.1583166332665%2C%20561.5470941883767%2C%20568.9358717434869%2C%20576.3246492985973%2C%20583.7134268537075%2C%20591.1022044088177%2C%20598.4909819639279%2C%20605.8797595190381%2C%20613.2685370741483%2C%20620.6573146292585%2C%20628.0460921843687%2C%20635.4348697394789%2C%20642.8236472945891%2C%20650.2124248496993%2C%20657.6012024048097%2C%20664.9899799599199%2C%20672.3787575150301%2C%20679.7675350701403%2C%20687.1563126252505%2C%20694.5450901803607%2C%20701.9338677354709%2C%20709.3226452905811%2C%20716.7114228456913%2C%20724.1002004008016%2C%20731.4889779559119%2C%20738.8777555110221%2C%20746.2665330661323%2C%20753.6553106212425%2C%20761.0440881763527%2C%20768.4328657314629%2C%20775.8216432865731%2C%20783.2104208416833%2C%20790.5991983967936%2C%20797.9879759519038%2C%20805.376753507014%2C%20812.7655310621243%2C%20820.1543086172345%2C%20827.5430861723447%2C%20834.9318637274549%2C%20842.3206412825651%2C%20849.7094188376753%2C%20857.0981963927856%2C%20864.4869739478958%2C%20871.875751503006%2C%20879.2645290581162%2C%20886.6533066132265%2C%20894.0420841683367%2C%20901.4308617234469%2C%20908.8196392785571%2C%20916.2084168336673%2C%20923.5971943887776%2C%20930.9859719438878%2C%20938.374749498998%2C%20945.7635270541082%2C%20953.1523046092184%2C%20960.5410821643287%2C%20967.9298597194389%2C%20975.3186372745491%2C%20982.7074148296593%2C%20990.0961923847696%2C%20997.4849699398798%2C%201004.87374749499%2C%201012.2625250501002%2C%201019.6513026052104%2C%201027.0400801603207%2C%201034.428857715431%2C%201041.8176352705411%2C%201049.2064128256513%2C%201056.5951903807616%2C%201063.9839679358718%2C%201071.372745490982%2C%201078.7615230460922%2C%201086.1503006012024%2C%201093.5390781563126%2C%201100.9278557114228%2C%201108.316633266533%2C%201115.7054108216432%2C%201123.0941883767534%2C%201130.4829659318636%2C%201137.8717434869739%2C%201145.260521042084%2C%201152.6492985971945%2C%201160.0380761523047%2C%201167.426853707415%2C%201174.8156312625251%2C%201182.2044088176353%2C%201189.5931863727455%2C%201196.9819639278558%2C%201204.370741482966%2C%201211.7595190380762%2C%201219.1482965931864%2C%201226.5370741482966%2C%201233.9258517034068%2C%201241.314629258517%2C%201248.7034068136272%2C%201256.0921843687374%2C%201263.4809619238476%2C%201270.8697394789579%2C%201278.258517034068%2C%201285.6472945891783%2C%201293.0360721442885%2C%201300.4248496993987%2C%201307.8136272545091%2C%201315.2024048096193%2C%201322.5911823647295%2C%201329.9799599198398%2C%201337.36873747495%2C%201344.7575150300602%2C%201352.1462925851704%2C%201359.5350701402806%2C%201366.9238476953908%2C%201374.312625250501%2C%201381.7014028056112%2C%201389.0901803607214%2C%201396.4789579158316%2C%201403.8677354709419%2C%201411.256513026052%2C%201418.6452905811623%2C%201426.0340681362725%2C%201433.4228456913827%2C%201440.811623246493%2C%201448.200400801603%2C%201455.5891783567133%2C%201462.9779559118238%2C%201470.366733466934%2C%201477.7555110220442%2C%201485.1442885771544%2C%201492.5330661322646%2C%201499.9218436873748%2C%201507.310621242485%2C%201514.6993987975952%2C%201522.0881763527054%2C%201529.4769539078156%2C%201536.8657314629259%2C%201544.254509018036%2C%201551.6432865731463%2C%201559.0320641282565%2C%201566.4208416833667%2C%201573.809619238477%2C%201581.198396793587%2C%201588.5871743486973%2C%201595.9759519038075%2C%201603.3647294589177%2C%201610.753507014028%2C%201618.1422845691384%2C%201625.5310621242486%2C%201632.9198396793588%2C%201640.308617234469%2C%201647.6973947895792%2C%201655.0861723446894%2C%201662.4749498997996%2C%201669.8637274549098%2C%201677.25250501002%2C%201684.6412825651303%2C%201692.0300601202405%2C%201699.4188376753507%2C%201706.807615230461%2C%201714.196392785571%2C%201721.5851703406813%2C%201728.9739478957915%2C%201736.3627254509017%2C%201743.751503006012%2C%201751.1402805611222%2C%201758.5290581162324%2C%201765.9178356713426%2C%201773.306613226453%2C%201780.6953907815632%2C%201788.0841683366734%2C%201795.4729458917836%2C%201802.8617234468938%2C%201810.250501002004%2C%201817.6392785571143%2C%201825.0280561122245%2C%201832.4168336673347%2C%201839.805611222445%2C%201847.194388777555%2C%201854.5831663326653%2C%201861.9719438877755%2C%201869.3607214428857%2C%201876.749498997996%2C%201884.1382765531062%2C%201891.5270541082164%2C%201898.9158316633266%2C%201906.3046092184368%2C%201913.693386773547%2C%201921.0821643286574%2C%201928.4709418837676%2C%201935.8597194388778%2C%201943.248496993988%2C%201950.6372745490983%2C%201958.0260521042085%2C%201965.4148296593187%2C%201972.803607214429%2C%201980.192384769539%2C%201987.5811623246493%2C%201994.9699398797595%2C%202002.3587174348697%2C%202009.74749498998%2C%202017.1362725450902%2C%202024.5250501002004%2C%202031.9138276553106%2C%202039.3026052104208%2C%202046.691382765531%2C%202054.0801603206414%2C%202061.4689378757516%2C%202068.857715430862%2C%202076.246492985972%2C%202083.6352705410823%2C%202091.0240480961925%2C%202098.4128256513027%2C%202105.801603206413%2C%202113.190380761523%2C%202120.5791583166333%2C%202127.9679358717435%2C%202135.3567134268537%2C%202142.745490981964%2C%202150.134268537074%2C%202157.5230460921844%2C%202164.9118236472946%2C%202172.300601202405%2C%202179.689378757515%2C%202187.078156312625%2C%202194.4669338677354%2C%202201.8557114228456%2C%202209.244488977956%2C%202216.633266533066%2C%202224.0220440881762%2C%202231.4108216432865%2C%202238.7995991983967%2C%202246.188376753507%2C%202253.577154308617%2C%202260.9659318637273%2C%202268.3547094188375%2C%202275.7434869739477%2C%202283.132264529058%2C%202290.521042084168%2C%202297.9098196392783%2C%202305.298597194389%2C%202312.687374749499%2C%202320.0761523046094%2C%202327.4649298597196%2C%202334.85370741483%2C%202342.24248496994%2C%202349.6312625250503%2C%202357.0200400801605%2C%202364.4088176352707%2C%202371.797595190381%2C%202379.186372745491%2C%202386.5751503006013%2C%202393.9639278557115%2C%202401.3527054108217%2C%202408.741482965932%2C%202416.130260521042%2C%202423.5190380761524%2C%202430.9078156312626%2C%202438.296593186373%2C%202445.685370741483%2C%202453.074148296593%2C%202460.4629258517034%2C%202467.8517034068136%2C%202475.240480961924%2C%202482.629258517034%2C%202490.0180360721442%2C%202497.4068136272545%2C%202504.7955911823647%2C%202512.184368737475%2C%202519.573146292585%2C%202526.9619238476953%2C%202534.3507014028055%2C%202541.7394789579157%2C%202549.128256513026%2C%202556.517034068136%2C%202563.9058116232463%2C%202571.2945891783565%2C%202578.6833667334668%2C%202586.072144288577%2C%202593.460921843687%2C%202600.8496993987974%2C%202608.2384769539076%2C%202615.6272545090183%2C%202623.0160320641285%2C%202630.4048096192387%2C%202637.793587174349%2C%202645.182364729459%2C%202652.5711422845693%2C%202659.9599198396795%2C%202667.3486973947897%2C%202674.7374749499%2C%202682.12625250501%2C%202689.5150300601204%2C%202696.9038076152306%2C%202704.2925851703408%2C%202711.681362725451%2C%202719.070140280561%2C%202726.4589178356714%2C%202733.8476953907816%2C%202741.236472945892%2C%202748.625250501002%2C%202756.0140280561122%2C%202763.4028056112224%2C%202770.7915831663327%2C%202778.180360721443%2C%202785.569138276553%2C%202792.9579158316633%2C%202800.3466933867735%2C%202807.7354709418837%2C%202815.124248496994%2C%202822.513026052104%2C%202829.9018036072143%2C%202837.2905811623245%2C%202844.6793587174348%2C%202852.068136272545%2C%202859.456913827655%2C%202866.8456913827654%2C%202874.2344689378756%2C%202881.623246492986%2C%202889.012024048096%2C%202896.400801603206%2C%202903.7895791583164%2C%202911.1783567134266%2C%202918.567134268537%2C%202925.9559118236475%2C%202933.3446893787577%2C%202940.733466933868%2C%202948.122244488978%2C%202955.5110220440883%2C%202962.8997995991986%2C%202970.2885771543088%2C%202977.677354709419%2C%202985.066132264529%2C%202992.4549098196394%2C%202999.8436873747496%2C%203007.23246492986%2C%203014.62124248497%2C%203022.0100200400802%2C%203029.3987975951904%2C%203036.7875751503007%2C%203044.176352705411%2C%203051.565130260521%2C%203058.9539078156313%2C%203066.3426853707415%2C%203073.7314629258517%2C%203081.120240480962%2C%203088.509018036072%2C%203095.8977955911823%2C%203103.2865731462925%2C%203110.6753507014027%2C%203118.064128256513%2C%203125.452905811623%2C%203132.8416833667334%2C%203140.2304609218436%2C%203147.619238476954%2C%203155.008016032064%2C%203162.396793587174%2C%203169.7855711422844%2C%203177.1743486973946%2C%203184.563126252505%2C%203191.951903807615%2C%203199.3406813627253%2C%203206.7294589178355%2C%203214.1182364729457%2C%203221.507014028056%2C%203228.8957915831666%2C%203236.2845691382768%2C%203243.673346693387%2C%203251.062124248497%2C%203258.4509018036074%2C%203265.8396793587176%2C%203273.228456913828%2C%203280.617234468938%2C%203288.0060120240482%2C%203295.3947895791584%2C%203302.7835671342687%2C%203310.172344689379%2C%203317.561122244489%2C%203324.9498997995993%2C%203332.3386773547095%2C%203339.7274549098197%2C%203347.11623246493%2C%203354.50501002004%2C%203361.8937875751503%2C%203369.2825651302605%2C%203376.6713426853707%2C%203384.060120240481%2C%203391.448897795591%2C%203398.8376753507014%2C%203406.2264529058116%2C%203413.615230460922%2C%203421.004008016032%2C%203428.392785571142%2C%203435.7815631262524%2C%203443.1703406813626%2C%203450.559118236473%2C%203457.947895791583%2C%203465.3366733466933%2C%203472.7254509018035%2C%203480.1142284569137%2C%203487.503006012024%2C%203494.891783567134%2C%203502.2805611222443%2C%203509.6693386773545%2C%203517.0581162324647%2C%203524.446893787575%2C%203531.835671342685%2C%203539.224448897796%2C%203546.613226452906%2C%203554.0020040080162%2C%203561.3907815631264%2C%203568.7795591182366%2C%203576.168336673347%2C%203583.557114228457%2C%203590.9458917835673%2C%203598.3346693386775%2C%203605.7234468937877%2C%203613.112224448898%2C%203620.501002004008%2C%203627.8897795591183%2C%203635.2785571142285%2C%203642.6673346693387%2C%203650.056112224449%2C%203657.444889779559%2C%203664.8336673346694%2C%203672.2224448897796%2C%203679.61122244489%2C%203687.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fee5d9ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fcbba1ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fc9272ff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23fb6a4aff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23de2d26ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%2C%20%27%23a50f15ff%27%5D%29%3B%0A%20%20%20%20%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.x%20%3D%20d3.scale.linear%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B0.0%2C%203687.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B0%2C%20400%5D%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.legend%20%3D%20L.control%28%7Bposition%3A%20%27topright%27%7D%29%3B%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.legend.onAdd%20%3D%20function%20%28map%29%20%7Bvar%20div%20%3D%20L.DomUtil.create%28%27div%27%2C%20%27legend%27%29%3B%20return%20div%7D%3B%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.legend.addTo%28map_d6fb1ba880dc49459feac09a2f6f713d%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.xAxis%20%3D%20d3.svg.axis%28%29%0A%20%20%20%20%20%20%20%20.scale%28color_map_801479b7173b4c5b83daafec6ff5f357.x%29%0A%20%20%20%20%20%20%20%20.orient%28%22top%22%29%0A%20%20%20%20%20%20%20%20.tickSize%281%29%0A%20%20%20%20%20%20%20%20.tickValues%28%5B0.0%2C%20614.5%2C%201229.0%2C%201843.5%2C%202458.0%2C%203072.5%2C%203687.0%5D%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.svg%20%3D%20d3.select%28%22.legend.leaflet-control%22%29.append%28%22svg%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22id%22%2C%20%27legend%27%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20450%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2040%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.g%20%3D%20color_map_801479b7173b4c5b83daafec6ff5f357.svg.append%28%22g%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22key%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22transform%22%2C%20%22translate%2825%2C16%29%22%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.g.selectAll%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.data%28color_map_801479b7173b4c5b83daafec6ff5f357.color.range%28%29.map%28function%28d%2C%20i%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20return%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20x0%3A%20i%20%3F%20color_map_801479b7173b4c5b83daafec6ff5f357.x%28color_map_801479b7173b4c5b83daafec6ff5f357.color.domain%28%29%5Bi%20-%201%5D%29%20%3A%20color_map_801479b7173b4c5b83daafec6ff5f357.x.range%28%29%5B0%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x1%3A%20i%20%3C%20color_map_801479b7173b4c5b83daafec6ff5f357.color.domain%28%29.length%20%3F%20color_map_801479b7173b4c5b83daafec6ff5f357.x%28color_map_801479b7173b4c5b83daafec6ff5f357.color.domain%28%29%5Bi%5D%29%20%3A%20color_map_801479b7173b4c5b83daafec6ff5f357.x.range%28%29%5B1%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20z%3A%20d%0A%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%7D%29%29%0A%20%20%20%20%20%20.enter%28%29.append%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2010%29%0A%20%20%20%20%20%20%20%20.attr%28%22x%22%2C%20function%28d%29%20%7B%20return%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20function%28d%29%20%7B%20return%20d.x1%20-%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.style%28%22fill%22%2C%20function%28d%29%20%7B%20return%20d.z%3B%20%7D%29%3B%0A%0A%20%20%20%20color_map_801479b7173b4c5b83daafec6ff5f357.g.call%28color_map_801479b7173b4c5b83daafec6ff5f357.xAxis%29.append%28%22text%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22caption%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22y%22%2C%2021%29%0A%20%20%20%20%20%20%20%20.text%28%27%27%29%3B%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_1fa756f29afb4313accf910c33f13947%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_1fa756f29afb4313accf910c33f13947%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.541%2C%20126.986%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_8551c574699e4f5f9f85e0a63c06a8bc%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//cartodb-basemaps-%7Bs%7D.global.ssl.fastly.net/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%20contributors%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eCartoDB%5Cu003c/a%5Cu003e%2C%20CartoDB%20%5Cu003ca%20href%20%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eattributions%5Cu003c/a%5Cu003e%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_1fa756f29afb4313accf910c33f13947%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20choropleth_563acdee5ef54555ad1f4cec43320e32%20%3D%20L.featureGroup%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_1fa756f29afb4313accf910c33f13947%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_484805e91b8a43f5844a6a982750f3c8_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.properties.SIG_CD%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211380%22%3A%20case%20%2211560%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23a1d99b%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211440%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22black%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211410%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23006d2c%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211170%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23c7e9c0%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22black%22%2C%20%22fillColor%22%3A%20%22%23edf8e9%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%201%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_484805e91b8a43f5844a6a982750f3c8_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_484805e91b8a43f5844a6a982750f3c8%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_484805e91b8a43f5844a6a982750f3c8_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_484805e91b8a43f5844a6a982750f3c8_styler%2C%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_484805e91b8a43f5844a6a982750f3c8_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_484805e91b8a43f5844a6a982750f3c8%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28choropleth_563acdee5ef54555ad1f4cec43320e32%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_484805e91b8a43f5844a6a982750f3c8_add%28%7B%22features%22%3A%20%5B%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00831558599998%2C%2037.69001680000002%5D%2C%20%5B127.00764424700003%2C%2037.69150638999997%5D%2C%20%5B127.00973504700005%2C%2037.69324682000001%5D%2C%20%5B127.00969102500005%2C%2037.696558160999984%5D%2C%20%5B127.01214887599997%2C%2037.69724089499999%5D%2C%20%5B127.01390304300003%2C%2037.698660491%5D%2C%20%5B127.01544040299996%2C%2037.70130154100002%5D%2C%20%5B127.019851357%2C%2037.700884901999984%5D%2C%20%5B127.02217147700003%2C%2037.69960736799999%5D%2C%20%5B127.02341184299996%2C%2037.69995983299998%5D%2C%20%5B127.02533791899998%2C%2037.69948105499998%5D%2C%20%5B127.02771004600004%2C%2037.700837447000026%5D%2C%20%5B127.02930740500005%2C%2037.699210467%5D%2C%20%5B127.02975002699998%2C%2037.69621560600001%5D%2C%20%5B127.03104551199999%2C%2037.69304682699999%5D%2C%20%5B127.032188658%2C%2037.69279409400002%5D%2C%20%5B127.03243270899998%2C%2037.69182616099999%5D%2C%20%5B127.03678096299996%2C%2037.692638805%5D%2C%20%5B127.038515888%2C%2037.69328696000002%5D%2C%20%5B127.03896796200002%2C%2037.69397974100002%5D%2C%20%5B127.04112667799996%2C%2037.695296503%5D%2C%20%5B127.04309027199997%2C%2037.69523619%5D%2C%20%5B127.04376597600003%2C%2037.693511828%5D%2C%20%5B127.04566611500002%2C%2037.692384920999984%5D%2C%20%5B127.04783268899996%2C%2037.69329455000002%5D%2C%20%5B127.04864899799998%2C%2037.69406779399998%5D%2C%20%5B127.04938379999999%2C%2037.691693921000024%5D%2C%20%5B127.04998367500002%2C%2037.690875163999976%5D%2C%20%5B127.04983654099999%2C%2037.687983056%5D%2C%20%5B127.05093367799998%2C%2037.686160674%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%200%2C%20%22SHAPE_AREA%22%3A%200.00211%2C%20%22SHAPE_LEN%22%3A%200.239901%2C%20%22SIG_CD%22%3A%20%2211320%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dobong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88359471499996%2C%2037.591855451000015%5D%2C%20%5B126.88502350199997%2C%2037.593642095%5D%2C%20%5B126.88720157499995%2C%2037.59368140999999%5D%2C%20%5B126.88561580199996%2C%2037.591502085%5D%2C%20%5B126.88577464100001%2C%2037.589569513000015%5D%2C%20%5B126.88717970799996%2C%2037.58853613399998%5D%2C%20%5B126.88927454400005%2C%2037.58883090799998%5D%2C%20%5B126.89225666100003%2C%2037.58852532399999%5D%2C%20%5B126.89334847199996%2C%2037.589070849%5D%2C%20%5B126.89687879200005%2C%2037.588570065%5D%2C%20%5B126.89810143800003%2C%2037.58954265199998%5D%2C%20%5B126.89966320099995%2C%2037.58982035399998%5D%2C%20%5B126.899585279%2C%2037.591694511000014%5D%2C%20%5B126.89898042499999%2C%2037.592683197999975%5D%2C%20%5B126.90037465800003%2C%2037.59421526400001%5D%2C%20%5B126.90184962900003%2C%2037.59514499199997%5D%2C%20%5B126.90103258399995%2C%2037.597339986%5D%2C%20%5B126.901343312%2C%2037.59936941699999%5D%2C%20%5B126.90000258099997%2C%2037.602042943000015%5D%2C%20%5B126.90017282600002%2C%2037.60311939799999%5D%2C%20%5B126.90212880000001%2C%2037.60374458000001%5D%2C%20%5B126.90062064300002%2C%2037.60932366100002%5D%2C%20%5B126.90032356100005%2C%2037.611195898%5D%2C%20%5B126.90175330700004%2C%2037.61407416499998%5D%2C%20%5B126.901843654%2C%2037.615756572%5D%2C%20%5B126.90314656400005%2C%2037.61730752099999%5D%2C%20%5B126.90338372099995%2C%2037.61884112299998%5D%2C%20%5B126.90525097600005%2C%2037.61907907300002%5D%2C%20%5B126.90563169100005%2C%2037.620708664%5D%2C%20%5B126.90713704200004%2C%2037.62194588400001%5D%2C%20%5B126.90716203600005%2C%2037.62331746299998%5D%2C%20%5B126.90664762899996%2C%2037.62449994799999%5D%2C%20%5B126.908630388%2C%2037.62627061799998%5D%2C%20%5B126.90879064700005%2C%2037.629332764000026%5D%2C%20%5B126.90731509700004%2C%2037.63087173100001%5D%2C%20%5B126.90677806899998%2C%2037.63274096499998%5D%2C%20%5B126.90711683699999%2C%2037.63340278800001%5D%2C%20%5B126.910875079%2C%2037.635322556%5D%2C%20%5B126.91123094800002%2C%2037.635911695%5D%2C%20%5B126.91011226199998%2C%2037.638515595%5D%2C%20%5B126.91081456899997%2C%2037.64003255799997%5D%2C%20%5B126.91230017800001%2C%2037.64432203199999%5D%2C%20%5B126.91016072699995%2C%2037.645083612%5D%2C%20%5B126.90772442000002%2C%2037.64629642699998%5D%2C%20%5B126.90843309399997%2C%2037.647262366%5D%2C%20%5B126.90974709700004%2C%2037.646882914%5D%2C%20%5B126.91234747099998%2C%2037.645182541%5D%2C%20%5B126.913740469%2C%2037.64476066499998%5D%2C%20%5B126.92414365900004%2C%2037.64611043100001%5D%2C%20%5B126.92715483799998%2C%2037.64810585499998%5D%2C%20%5B126.92972022900005%2C%2037.65004959800001%5D%2C%20%5B126.93224238100004%2C%2037.65038895999999%5D%2C%20%5B126.935727004%2C%2037.65120514099999%5D%2C%20%5B126.93718405799996%2C%2037.652317728000014%5D%2C%20%5B126.93847852299996%2C%2037.654773405000014%5D%2C%20%5B126.94014671100001%2C%2037.65663874799998%5D%2C%20%5B126.94211837499995%2C%2037.65772510800002%5D%2C%20%5B126.94441488899997%2C%2037.65816625799999%5D%2C%20%5B126.94758363699998%2C%2037.659217403000014%5D%2C%20%5B126.94790109500002%2C%2037.65712764900002%5D%2C%20%5B126.94922168000005%2C%2037.656772183999976%5D%2C%20%5B126.94980975299995%2C%2037.655913363000025%5D%2C%20%5B126.95139494700004%2C%2037.654951117999985%5D%2C%20%5B126.95437039%2C%2037.654596949999984%5D%2C%20%5B126.95712830900004%2C%2037.65283925900002%5D%2C%20%5B126.95781039600001%2C%2037.64981070699997%5D%2C%20%5B126.95786031099999%2C%2037.64804963099999%5D%2C%20%5B126.95904808800003%2C%2037.64678533599999%5D%2C%20%5B126.95892784399996%2C%2037.64476220400002%5D%2C%20%5B126.95928919999994%2C%2037.642134535000025%5D%2C%20%5B126.959992629%2C%2037.64146349599997%5D%2C%20%5B126.96162582500006%2C%2037.63711909400001%5D%2C%20%5B126.96256224599995%2C%2037.63582057799999%5D%2C%20%5B126.96335032800005%2C%2037.633246824000025%5D%2C%20%5B126.96164091799994%2C%2037.631889602%5D%2C%20%5B126.95984451699996%2C%2037.629762777999986%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%201%2C%20%22SHAPE_AREA%22%3A%200.003041%2C%20%22SHAPE_LEN%22%3A%200.327143%2C%20%22SIG_CD%22%3A%20%2211380%22%2C%20%22SIG_ENG_NM%22%3A%20%22Eunpyeong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%202%2C%20%22SHAPE_AREA%22%3A%200.001453%2C%20%22SHAPE_LEN%22%3A%200.182837%2C%20%22SIG_CD%22%3A%20%2211230%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongdaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%203%2C%20%22SHAPE_AREA%22%3A%200.00167%2C%20%22SHAPE_LEN%22%3A%200.237796%2C%20%22SIG_CD%22%3A%20%2211590%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongjak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92829901899995%2C%2037.449360259%5D%2C%20%5B126.926538273%2C%2037.44839745899998%5D%2C%20%5B126.92500225699996%2C%2037.44678811799997%5D%2C%20%5B126.92327265100005%2C%2037.44577942699999%5D%2C%20%5B126.92278057800002%2C%2037.444047814999976%5D%2C%20%5B126.92118598900004%2C%2037.442552917%5D%2C%20%5B126.92028939299996%2C%2037.44047332299999%5D%2C%20%5B126.91927808399998%2C%2037.43985946599997%5D%2C%20%5B126.91613959300003%2C%2037.440060542000026%5D%2C%20%5B126.91232267199996%2C%2037.438587175%5D%2C%20%5B126.911215782%2C%2037.43720903600001%5D%2C%20%5B126.91134523699998%2C%2037.436179159%5D%2C%20%5B126.90940698500003%2C%2037.433868523%5D%2C%20%5B126.90726602899997%2C%2037.43352405000002%5D%2C%20%5B126.90611662000003%2C%2037.43400280399999%5D%2C%20%5B126.90299885399997%2C%2037.43407347499999%5D%2C%20%5B126.902763%2C%2037.43586855199999%5D%2C%20%5B126.90086236399998%2C%2037.43699089%5D%2C%20%5B126.89877013700004%2C%2037.439283811999985%5D%2C%20%5B126.89991895399999%2C%2037.439562146000014%5D%2C%20%5B126.897287549%2C%2037.445443447%5D%2C%20%5B126.89580121999995%2C%2037.445640562999984%5D%2C%20%5B126.89466890899996%2C%2037.446706108%5D%2C%20%5B126.89476993300002%2C%2037.447710443%5D%2C%20%5B126.89592769599994%2C%2037.44842231299998%5D%2C%20%5B126.89400040400005%2C%2037.452723881%5D%2C%20%5B126.89277619200004%2C%2037.452046575%5D%2C%20%5B126.88951816999997%2C%2037.45270753599999%5D%2C%20%5B126.889280761%2C%2037.45448889199997%5D%2C%20%5B126.88641922299996%2C%2037.456180116999974%5D%2C%20%5B126.88589653600002%2C%2037.457514893999985%5D%2C%20%5B126.88630108799998%2C%2037.459120703%5D%2C%20%5B126.885364977%2C%2037.459868728%5D%2C%20%5B126.886087029%2C%2037.460865333000015%5D%2C%20%5B126.88889275600002%2C%2037.460953249%5D%2C%20%5B126.88713625100002%2C%2037.462909712%5D%2C%20%5B126.88515609299998%2C%2037.462473671%5D%2C%20%5B126.88433205199999%2C%2037.46272970799998%5D%2C%20%5B126.88276046399994%2C%2037.464396598%5D%2C%20%5B126.88463173000002%2C%2037.46601745999999%5D%2C%20%5B126.88302833499995%2C%2037.467069503%5D%2C%20%5B126.881647767%2C%2037.468466234%5D%2C%20%5B126.88161172399998%2C%2037.469425546000025%5D%2C%20%5B126.878031942%2C%2037.47383573500002%5D%2C%20%5B126.87600453499999%2C%2037.477111283%5D%2C%20%5B126.87178398799995%2C%2037.48527755399999%5D%2C%20%5B126.87315863100002%2C%2037.486160427000016%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%204%2C%20%22SHAPE_AREA%22%3A%200.001325%2C%20%22SHAPE_LEN%22%3A%200.211649%2C%20%22SIG_CD%22%3A%20%2211545%22%2C%20%22SIG_ENG_NM%22%3A%20%22Geumcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.88272419199996%2C%2037.515768666999975%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.87599162499998%2C%2037.48691906800002%5D%2C%20%5B126.87680700600004%2C%2037.488578991%5D%2C%20%5B126.87583373899997%2C%2037.489035374000025%5D%2C%20%5B126.87309748400003%2C%2037.488261317000024%5D%2C%20%5B126.87245908399996%2C%2037.489543242000025%5D%2C%20%5B126.87024621900002%2C%2037.48960327200001%5D%2C%20%5B126.86936249799999%2C%2037.49164208000002%5D%2C%20%5B126.86972553299995%2C%2037.49410186699998%5D%2C%20%5B126.86836486200002%2C%2037.49511503000002%5D%2C%20%5B126.86738599800003%2C%2037.49308047%5D%2C%20%5B126.8663199%2C%2037.492637301%5D%2C%20%5B126.86541991000001%2C%2037.49151324600001%5D%2C%20%5B126.86260271100002%2C%2037.49076571799998%5D%2C%20%5B126.86139963599999%2C%2037.489972641%5D%2C%20%5B126.85798566000005%2C%2037.48609330599999%5D%2C%20%5B126.855521178%2C%2037.485415111%5D%2C%20%5B126.853961895%2C%2037.48297475200002%5D%2C%20%5B126.85271384700002%2C%2037.48182609899999%5D%2C%20%5B126.85043164800004%2C%2037.48157525800002%5D%2C%20%5B126.84848405499997%2C%2037.482095259%5D%2C%20%5B126.84642145600003%2C%2037.48150571299999%5D%2C%20%5B126.84594900499997%2C%2037.480323644%5D%2C%20%5B126.84537853999996%2C%2037.473821137000016%5D%2C%20%5B126.842799474%2C%2037.47498250699999%5D%2C%20%5B126.84104901299997%2C%2037.474656347%5D%2C%20%5B126.83831728799998%2C%2037.47540135999998%5D%2C%20%5B126.83484912899996%2C%2037.47443089%5D%2C%20%5B126.83488769799999%2C%2037.47575841499997%5D%2C%20%5B126.83349606700006%2C%2037.47716231800001%5D%2C%20%5B126.83176399199999%2C%2037.47765735500002%5D%2C%20%5B126.82788973599997%2C%2037.47599654700002%5D%2C%20%5B126.82435334800005%2C%2037.476259733%5D%2C%20%5B126.82212405999996%2C%2037.47575425399998%5D%2C%20%5B126.82116790299995%2C%2037.476307167000016%5D%2C%20%5B126.81909562600003%2C%2037.476138418%5D%2C%20%5B126.818336391%2C%2037.47530395899997%5D%2C%20%5B126.818647371%2C%2037.474085166%5D%2C%20%5B126.81708964200004%2C%2037.47329169699998%5D%2C%20%5B126.81657626699996%2C%2037.473998096%5D%2C%20%5B126.81464318099995%2C%2037.47465330900002%5D%2C%20%5B126.81531048900001%2C%2037.47636645199998%5D%2C%20%5B126.81722552199994%2C%2037.47814126600002%5D%2C%20%5B126.81827151499999%2C%2037.478170568%5D%2C%20%5B126.81901268700005%2C%2037.47916943500002%5D%2C%20%5B126.81999807199998%2C%2037.48164884%5D%2C%20%5B126.81959813599997%2C%2037.48244711299998%5D%2C%20%5B126.81934165300004%2C%2037.48551259499999%5D%2C%20%5B126.82130494499995%2C%2037.48618739800003%5D%2C%20%5B126.82351963899998%2C%2037.487744236000026%5D%2C%20%5B126.82277597899997%2C%2037.48998772700003%5D%2C%20%5B126.81790518800005%2C%2037.49154342000003%5D%2C%20%5B126.81459113400001%2C%2037.49320034200002%5D%2C%20%5B126.81436892199997%2C%2037.49443227400002%5D%2C%20%5B126.81302860799997%2C%2037.496407777%5D%2C%20%5B126.81415555599995%2C%2037.49813429599999%5D%2C%20%5B126.81611235499997%2C%2037.49765224499998%5D%2C%20%5B126.818224268%2C%2037.49886467499999%5D%2C%20%5B126.81965596999999%2C%2037.499219118999974%5D%2C%20%5B126.81943123600001%2C%2037.50079738%5D%2C%20%5B126.82158116200003%2C%2037.50216396000002%5D%2C%20%5B126.82184366299998%2C%2037.504545139000015%5D%2C%20%5B126.82256759899997%2C%2037.505692083999975%5D%2C%20%5B126.82223955400002%2C%2037.50769455599999%5D%2C%20%5B126.82615287800002%2C%2037.50872472999998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.837508791%2C%2037.50328697200001%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.84076167%2C%2037.50604941900002%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84873119999997%2C%2037.508888773000024%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85562926600005%2C%2037.50916079199999%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.858345407%2C%2037.509923406999974%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%205%2C%20%22SHAPE_AREA%22%3A%200.002047%2C%20%22SHAPE_LEN%22%3A%200.347568%2C%20%22SIG_CD%22%3A%20%2211530%22%2C%20%22SIG_ENG_NM%22%3A%20%22Guro-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.96146158%2C%2037.62996095%5D%2C%20%5B126.96378891899997%2C%2037.62979475499998%5D%2C%20%5B126.96524997699998%2C%2037.63076970399999%5D%2C%20%5B126.968256763%2C%2037.63073174200002%5D%2C%20%5B126.97134811%2C%2037.63162062700002%5D%2C%20%5B126.973049887%2C%2037.632376954999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%206%2C%20%22SHAPE_AREA%22%3A%200.002448%2C%20%22SHAPE_LEN%22%3A%200.2901%2C%20%22SIG_CD%22%3A%20%2211110%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jongno-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98421633700002%2C%2037.63634100199999%5D%2C%20%5B126.98510166300002%2C%2037.637318127000015%5D%2C%20%5B126.98569312400002%2C%2037.640048433%5D%2C%20%5B126.98528469200005%2C%2037.641354295999975%5D%2C%20%5B126.98427374300002%2C%2037.64164886999998%5D%2C%20%5B126.98326407499997%2C%2037.643687516%5D%2C%20%5B126.98512211000002%2C%2037.64573277%5D%2C%20%5B126.98462050299997%2C%2037.647616578%5D%2C%20%5B126.98399871900006%2C%2037.64818593799998%5D%2C%20%5B126.98395619999997%2C%2037.649613708%5D%2C%20%5B126.98251481800003%2C%2037.650544008%5D%2C%20%5B126.98169600799997%2C%2037.652320175%5D%2C%20%5B126.98106737800003%2C%2037.652716521%5D%2C%20%5B126.97996370600004%2C%2037.65481924699998%5D%2C%20%5B126.97967724299997%2C%2037.65604285799998%5D%2C%20%5B126.98302857199997%2C%2037.65700805%5D%2C%20%5B126.98603858499996%2C%2037.65894778400002%5D%2C%20%5B126.987294798%2C%2037.660604260000014%5D%2C%20%5B126.98782001899997%2C%2037.66359695900002%5D%2C%20%5B126.98828632200002%2C%2037.66439418800002%5D%2C%20%5B126.99001520599995%2C%2037.66450267%5D%2C%20%5B126.99224251800001%2C%2037.66523496299999%5D%2C%20%5B126.99414712600003%2C%2037.667033076%5D%2C%20%5B126.99357858400003%2C%2037.66780672099998%5D%2C%20%5B126.99437535000004%2C%2037.66956914000002%5D%2C%20%5B126.99382043900005%2C%2037.670199244%5D%2C%20%5B126.99409198599994%2C%2037.672763257999975%5D%2C%20%5B126.99356165699999%2C%2037.67410523500001%5D%2C%20%5B126.99399960799997%2C%2037.674909437999986%5D%2C%20%5B126.99321780000002%2C%2037.675722317%5D%2C%20%5B126.99326569200002%2C%2037.67766322599999%5D%2C%20%5B126.99221614400005%2C%2037.67962970100001%5D%2C%20%5B126.99409428599995%2C%2037.68032697899997%5D%2C%20%5B126.99475084200003%2C%2037.681246918%5D%2C%20%5B126.99675604100003%2C%2037.68239214%5D%2C%20%5B126.997333905%2C%2037.68337292299998%5D%2C%20%5B127.00170893100005%2C%2037.68426214499999%5D%2C%20%5B127.003605311%2C%2037.684165339%5D%2C%20%5B127.00454835300002%2C%2037.68501282099999%5D%2C%20%5B127.00607232000004%2C%2037.684984327%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%207%2C%20%22SHAPE_AREA%22%3A%200.002412%2C%20%22SHAPE_LEN%22%3A%200.267441%2C%20%22SIG_CD%22%3A%20%2211305%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10914571%2C%2037.62052080400002%5D%2C%20%5B127.11053916100002%2C%2037.62105968399999%5D%2C%20%5B127.11216023500003%2C%2037.620311775%5D%2C%20%5B127.11503737099997%2C%2037.61954149500002%5D%2C%20%5B127.11716289399999%2C%2037.61789223%5D%2C%20%5B127.11668740899995%2C%2037.61601964499999%5D%2C%20%5B127.11749802600002%2C%2037.61177613000001%5D%2C%20%5B127.11677896900005%2C%2037.610134802%5D%2C%20%5B127.116712333%2C%2037.60885401899998%5D%2C%20%5B127.11849163500005%2C%2037.60761747%5D%2C%20%5B127.118075341%2C%2037.606595825%5D%2C%20%5B127.118061087%2C%2037.604606131000025%5D%2C%20%5B127.115739211%2C%2037.60211296%5D%2C%20%5B127.11408091199996%2C%2037.599527276%5D%2C%20%5B127.11689451500001%2C%2037.595503880000024%5D%2C%20%5B127.11668170400003%2C%2037.594023413%5D%2C%20%5B127.11335498100004%2C%2037.59326654199998%5D%2C%20%5B127.110693577%2C%2037.58916251699998%5D%2C%20%5B127.11000566899997%2C%2037.586023006%5D%2C%20%5B127.10898535199999%2C%2037.58338535199999%5D%2C%20%5B127.10345938600005%2C%2037.58060086099999%5D%2C%20%5B127.10304415999997%2C%2037.578912213000024%5D%2C%20%5B127.10116216200004%2C%2037.57607578599999%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%208%2C%20%22SHAPE_AREA%22%3A%200.001893%2C%20%22SHAPE_LEN%22%3A%200.184716%2C%20%22SIG_CD%22%3A%20%2211260%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jungnang-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.12187312100002%2C%2037.46514826399999%5D%2C%20%5B127.12145320599996%2C%2037.46438168499998%5D%2C%20%5B127.11748896200004%2C%2037.462210077%5D%2C%20%5B127.11700353900005%2C%2037.46149384900002%5D%2C%20%5B127.116910318%2C%2037.458650388000024%5D%2C%20%5B127.11590693999995%2C%2037.45860154000002%5D%2C%20%5B127.11328498199998%2C%2037.460064267%5D%2C%20%5B127.11298953899995%2C%2037.46114412899999%5D%2C%20%5B127.11183504999997%2C%2037.461654001%5D%2C%20%5B127.10648246400001%2C%2037.46243379200001%5D%2C%20%5B127.10435658899996%2C%2037.46218425799998%5D%2C%20%5B127.10479563499996%2C%2037.46116971399999%5D%2C%20%5B127.10395681800003%2C%2037.46006054700001%5D%2C%20%5B127.10139603200003%2C%2037.45898237099999%5D%2C%20%5B127.10129219299995%2C%2037.45825120400002%5D%2C%20%5B127.09975283000006%2C%2037.457364144%5D%2C%20%5B127.099109139%2C%2037.45622823999997%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%209%2C%20%22SHAPE_AREA%22%3A%200.004027%2C%20%22SHAPE_LEN%22%3A%200.348412%2C%20%22SIG_CD%22%3A%20%2211680%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangnam-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.81658266%2C%2037.540568317%5D%2C%20%5B126.81234887999994%2C%2037.54074469400001%5D%2C%20%5B126.81060934799996%2C%2037.54150444200002%5D%2C%20%5B126.80725610699994%2C%2037.54362225199998%5D%2C%20%5B126.80186598399996%2C%2037.542723611999975%5D%2C%20%5B126.80048554400003%2C%2037.54177814600001%5D%2C%20%5B126.80040658300004%2C%2037.540804252999976%5D%2C%20%5B126.79887833%2C%2037.5402999%5D%2C%20%5B126.79875436099996%2C%2037.539406192%5D%2C%20%5B126.79949804199998%2C%2037.537775685999975%5D%2C%20%5B126.79782849100002%2C%2037.537056888999984%5D%2C%20%5B126.79605918200002%2C%2037.53675834400002%5D%2C%20%5B126.79481063799994%2C%2037.535972963%5D%2C%20%5B126.79397960699998%2C%2037.537369168%5D%2C%20%5B126.79390197299995%2C%2037.539558371%5D%2C%20%5B126.79496077700003%2C%2037.541381135999984%5D%2C%20%5B126.79182670499995%2C%2037.54191169%5D%2C%20%5B126.79159995700002%2C%2037.54307308199998%5D%2C%20%5B126.79077654100001%2C%2037.543847862%5D%2C%20%5B126.78872746499997%2C%2037.54482803399998%5D%2C%20%5B126.78727682700003%2C%2037.54608495799999%5D%2C%20%5B126.78332478000004%2C%2037.54603042999997%5D%2C%20%5B126.77875616300003%2C%2037.546729674%5D%2C%20%5B126.77757101300006%2C%2037.54672980700002%5D%2C%20%5B126.77675212199995%2C%2037.548221075000015%5D%2C%20%5B126.775301227%2C%2037.548988871%5D%2C%20%5B126.77258571799996%2C%2037.548773638%5D%2C%20%5B126.77169496199997%2C%2037.548329146000015%5D%2C%20%5B126.76992633199995%2C%2037.550196289999974%5D%2C%20%5B126.769793875%2C%2037.551086956%5D%2C%20%5B126.767468484%2C%2037.551972233000015%5D%2C%20%5B126.76742527299996%2C%2037.554237899999976%5D%2C%20%5B126.76458060899995%2C%2037.555466594999984%5D%2C%20%5B126.76720647299999%2C%2037.55666916000001%5D%2C%20%5B126.76988573200003%2C%2037.55724517599998%5D%2C%20%5B126.772397404%2C%2037.55699389%5D%2C%20%5B126.773200245%2C%2037.557536051%5D%2C%20%5B126.77183969700002%2C%2037.55861259400001%5D%2C%20%5B126.77320943200004%2C%2037.55929739800001%5D%2C%20%5B126.77612545800002%2C%2037.56187228599998%5D%2C%20%5B126.77795148799999%2C%2037.56016005399999%5D%2C%20%5B126.7771186%2C%2037.563728853999976%5D%2C%20%5B126.77515191199996%2C%2037.565895781999984%5D%2C%20%5B126.77478348299996%2C%2037.56772791100002%5D%2C%20%5B126.77635320499996%2C%2037.567107521000025%5D%2C%20%5B126.777826946%2C%2037.567006467%5D%2C%20%5B126.78025671600005%2C%2037.567453419%5D%2C%20%5B126.780491568%2C%2037.56829614100002%5D%2C%20%5B126.781642583%2C%2037.568909259%5D%2C%20%5B126.78133354800002%2C%2037.570185116%5D%2C%20%5B126.782647215%2C%2037.57055544799999%5D%2C%20%5B126.78191796099998%2C%2037.57181951000001%5D%2C%20%5B126.78242742400005%2C%2037.57361885900002%5D%2C%20%5B126.78526689%2C%2037.57487061799998%5D%2C%20%5B126.787630605%2C%2037.57558866%5D%2C%20%5B126.78935921899995%2C%2037.57772807700002%5D%2C%20%5B126.79058199500003%2C%2037.577776819%5D%2C%20%5B126.79074497399995%2C%2037.58061836799999%5D%2C%20%5B126.79228461599996%2C%2037.580056910999986%5D%2C%20%5B126.79275533099997%2C%2037.57684457%5D%2C%20%5B126.79323752200003%2C%2037.57689948500001%5D%2C%20%5B126.79288111400001%2C%2037.58022635499998%5D%2C%20%5B126.79390501399996%2C%2037.582257949%5D%2C%20%5B126.79411019999998%2C%2037.58425737099998%5D%2C%20%5B126.79555674699998%2C%2037.58515790000001%5D%2C%20%5B126.79569588599998%2C%2037.583367872%5D%2C%20%5B126.79711367100003%2C%2037.58427787900001%5D%2C%20%5B126.79670218499996%2C%2037.58491212500002%5D%2C%20%5B126.79878484200003%2C%2037.58807839000002%5D%2C%20%5B126.80075747499995%2C%2037.587841243000014%5D%2C%20%5B126.80053959500003%2C%2037.590315551%5D%2C%20%5B126.798817848%2C%2037.59132633199999%5D%2C%20%5B126.79867010299995%2C%2037.59366814100002%5D%2C%20%5B126.79745919100003%2C%2037.595248595999976%5D%2C%20%5B126.79708180299997%2C%2037.59773291099998%5D%2C%20%5B126.79756259099997%2C%2037.59793496100002%5D%2C%20%5B126.79787158299996%2C%2037.599898203%5D%2C%20%5B126.799935054%2C%2037.601445969%5D%2C%20%5B126.79994633199999%2C%2037.602546226000015%5D%2C%20%5B126.80258612299997%2C%2037.605033999%5D%2C%20%5B126.80587186000002%2C%2037.60258296500001%5D%2C%20%5B126.808249756%2C%2037.601211834000026%5D%2C%20%5B126.81138204800004%2C%2037.59896582800002%5D%2C%20%5B126.81396850600004%2C%2037.59762536400001%5D%2C%20%5B126.816573747%2C%2037.595688145999986%5D%2C%20%5B126.81754378000005%2C%2037.59533660400001%5D%2C%20%5B126.81932148199996%2C%2037.592855004%5D%2C%20%5B126.82783387699999%2C%2037.58748727300002%5D%2C%20%5B126.83178168200004%2C%2037.585846134%5D%2C%20%5B126.837170002%2C%2037.582568493%5D%2C%20%5B126.83811851799999%2C%2037.581798564999986%5D%2C%20%5B126.84326033599996%2C%2037.578886398%5D%2C%20%5B126.84840263399997%2C%2037.57516434500002%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2010%2C%20%22SHAPE_AREA%22%3A%200.004227%2C%20%22SHAPE_LEN%22%3A%200.435694%2C%20%22SIG_CD%22%3A%20%2211500%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangseo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2011%2C%20%22SHAPE_AREA%22%3A%200.001017%2C%20%22SHAPE_LEN%22%3A%200.191242%2C%20%22SIG_CD%22%3A%20%2211140%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jung-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.11428417299999%2C%2037.554386276%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11566284000003%2C%2037.557702898%5D%2C%20%5B127.11735874600004%2C%2037.559467202%5D%2C%20%5B127.12011629799997%2C%2037.56120324699998%5D%2C%20%5B127.12297571600004%2C%2037.56349386%5D%2C%20%5B127.12862746099995%2C%2037.56616545000003%5D%2C%20%5B127.134283364%2C%2037.567961086000025%5D%2C%20%5B127.13768321700002%2C%2037.56843244200002%5D%2C%20%5B127.14896618800003%2C%2037.568444026%5D%2C%20%5B127.154998245%2C%2037.57204888699999%5D%2C%20%5B127.15691686699995%2C%2037.572958835%5D%2C%20%5B127.16257241200003%2C%2037.576732698%5D%2C%20%5B127.16676596299999%2C%2037.57897662%5D%2C%20%5B127.17026691299998%2C%2037.57901745999999%5D%2C%20%5B127.17185117300005%2C%2037.57926277399997%5D%2C%20%5B127.17521472800001%2C%2037.58046697200001%5D%2C%20%5B127.17734763199996%2C%2037.581575015%5D%2C%20%5B127.17701410100005%2C%2037.579511809%5D%2C%20%5B127.17550670900005%2C%2037.578472404000024%5D%2C%20%5B127.17534214%2C%2037.57723767800002%5D%2C%20%5B127.17569917399999%2C%2037.57490180399998%5D%2C%20%5B127.17789114499999%2C%2037.57187252799997%5D%2C%20%5B127.17922984100005%2C%2037.56892072800002%5D%2C%20%5B127.17959913300001%2C%2037.56549991899999%5D%2C%20%5B127.181200053%2C%2037.56301996299999%5D%2C%20%5B127.18200949499999%2C%2037.56099529800002%5D%2C%20%5B127.181566868%2C%2037.557559535%5D%2C%20%5B127.18169641500003%2C%2037.55600353400001%5D%2C%20%5B127.18135382599996%2C%2037.552973123000015%5D%2C%20%5B127.18294495500004%2C%2037.55177914199999%5D%2C%20%5B127.183131151%2C%2037.551111615000025%5D%2C%20%5B127.18268913400004%2C%2037.54791615900001%5D%2C%20%5B127.182837425%2C%2037.54639986199999%5D%2C%20%5B127.17934642900002%2C%2037.54657307399998%5D%2C%20%5B127.17573322400006%2C%2037.545213538999974%5D%2C%20%5B127.17392236499995%2C%2037.545588527%5D%2C%20%5B127.17212391600003%2C%2037.545541722999985%5D%2C%20%5B127.16978076600003%2C%2037.544813007000016%5D%2C%20%5B127.16699588400002%2C%2037.54521138699999%5D%2C%20%5B127.16665500399995%2C%2037.54426145999997%5D%2C%20%5B127.16317850999997%2C%2037.544997455999976%5D%2C%20%5B127.16032596900004%2C%2037.541629215%5D%2C%20%5B127.15766934199996%2C%2037.53927536100002%5D%2C%20%5B127.15696225299996%2C%2037.537974744999985%5D%2C%20%5B127.15527845999998%2C%2037.53622368600003%5D%2C%20%5B127.15358375799997%2C%2037.533672017000015%5D%2C%20%5B127.15395052600002%2C%2037.531926252%5D%2C%20%5B127.15270367899996%2C%2037.528535456999975%5D%2C%20%5B127.15041863099998%2C%2037.52637816499998%5D%2C%20%5B127.14781481299997%2C%2037.52215484599998%5D%2C%20%5B127.145682889%2C%2037.52193569399998%5D%2C%20%5B127.144801873%2C%2037.519630431999985%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2012%2C%20%22SHAPE_AREA%22%3A%200.002504%2C%20%22SHAPE_LEN%22%3A%200.242596%2C%20%22SIG_CD%22%3A%20%2211740%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11411142500003%2C%2037.55409376599999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.10168325400002%2C%2037.57240515400002%5D%2C%20%5B127.103148411%2C%2037.57228406899998%5D%2C%20%5B127.10424898199994%2C%2037.571392263%5D%2C%20%5B127.10347415499996%2C%2037.57005447199998%5D%2C%20%5B127.10240523599998%2C%2037.56564081%5D%2C%20%5B127.10225269499995%2C%2037.564314869999976%5D%2C%20%5B127.10122363599999%2C%2037.56158417300003%5D%2C%20%5B127.10187808800003%2C%2037.559498245999976%5D%2C%20%5B127.10429532800003%2C%2037.55756740800001%5D%2C%20%5B127.10494775899997%2C%2037.55642552299997%5D%2C%20%5B127.10641446099999%2C%2037.55646542699998%5D%2C%20%5B127.10753783500002%2C%2037.55761021400002%5D%2C%20%5B127.10954748999995%2C%2037.558557784000016%5D%2C%20%5B127.11036889499997%2C%2037.55825685100001%5D%2C%20%5B127.11231650399998%2C%2037.55900405400001%5D%2C%20%5B127.11388262800006%2C%2037.55843289500001%5D%2C%20%5B127.11333075200002%2C%2037.55683318299998%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2013%2C%20%22SHAPE_AREA%22%3A%200.001737%2C%20%22SHAPE_LEN%22%3A%200.186732%2C%20%22SIG_CD%22%3A%20%2211215%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwangjin-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85364710199997%2C%2037.573804395000025%5D%2C%20%5B126.85930276099998%2C%2037.57492123999998%5D%2C%20%5B126.85931123%2C%2037.575246692%5D%2C%20%5B126.86495888599995%2C%2037.57747814700002%5D%2C%20%5B126.868396404%2C%2037.577526395%5D%2C%20%5B126.870619272%2C%2037.578013113%5D%2C%20%5B126.87393583999994%2C%2037.57828489799999%5D%2C%20%5B126.87628222700005%2C%2037.57818935199998%5D%2C%20%5B126.87765407400002%2C%2037.57975304299998%5D%2C%20%5B126.87668444099995%2C%2037.581263981%5D%2C%20%5B126.87707186%2C%2037.58221029200001%5D%2C%20%5B126.87710903799996%2C%2037.58483216500002%5D%2C%20%5B126.877564388%2C%2037.586252527%5D%2C%20%5B126.87915565399999%2C%2037.58677640399998%5D%2C%20%5B126.88036942999997%2C%2037.58950964899998%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2014%2C%20%22SHAPE_AREA%22%3A%200.002415%2C%20%22SHAPE_LEN%22%3A%200.289336%2C%20%22SIG_CD%22%3A%20%2211440%22%2C%20%22SIG_ENG_NM%22%3A%20%22Mapo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98662702599995%2C%2037.457216786%5D%2C%20%5B126.98290313899997%2C%2037.456831314%5D%2C%20%5B126.98241282200001%2C%2037.455895618%5D%2C%20%5B126.97849284100005%2C%2037.455739153000025%5D%2C%20%5B126.97777249%2C%2037.455203476%5D%2C%20%5B126.97459526800003%2C%2037.45441720500003%5D%2C%20%5B126.97285906000002%2C%2037.45238675000002%5D%2C%20%5B126.97176838200005%2C%2037.45174170500002%5D%2C%20%5B126.97058548899997%2C%2037.449457128%5D%2C%20%5B126.96767973600004%2C%2037.44838496%5D%2C%20%5B126.964307298%2C%2037.446271943%5D%2C%20%5B126.96395069300002%2C%2037.44521969200002%5D%2C%20%5B126.96443595200003%2C%2037.44431798300002%5D%2C%20%5B126.96464967600002%2C%2037.44204566000002%5D%2C%20%5B126.96295770799998%2C%2037.44028699400002%5D%2C%20%5B126.96007066799996%2C%2037.44040704499997%5D%2C%20%5B126.95898017000002%2C%2037.43907666199999%5D%2C%20%5B126.95512268899995%2C%2037.43869914800001%5D%2C%20%5B126.95242507900002%2C%2037.43919711699999%5D%2C%20%5B126.95148094199999%2C%2037.43858365%5D%2C%20%5B126.94929766799999%2C%2037.43825732%5D%2C%20%5B126.94837678299996%2C%2037.43871966299997%5D%2C%20%5B126.94680271000004%2C%2037.43817427499999%5D%2C%20%5B126.94510345499998%2C%2037.43709568600002%5D%2C%20%5B126.94143729300004%2C%2037.437411664000024%5D%2C%20%5B126.940232358%2C%2037.435720186000026%5D%2C%20%5B126.93858713099996%2C%2037.43642808300001%5D%2C%20%5B126.93775941000001%2C%2037.437393122%5D%2C%20%5B126.93726096700004%2C%2037.439219657000024%5D%2C%20%5B126.93787838100002%2C%2037.44020405700002%5D%2C%20%5B126.93666192499995%2C%2037.44175181600002%5D%2C%20%5B126.93176733099995%2C%2037.444982075999974%5D%2C%20%5B126.93057246299998%2C%2037.445474494%5D%2C%20%5B126.93018248700002%2C%2037.44638402800001%5D%2C%20%5B126.93018641599997%2C%2037.448390587%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2015%2C%20%22SHAPE_AREA%22%3A%200.003012%2C%20%22SHAPE_LEN%22%3A%200.280092%2C%20%22SIG_CD%22%3A%20%2211620%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwanak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.093554412%2C%2037.45589802400002%5D%2C%20%5B127.09273821199997%2C%2037.453643511999985%5D%2C%20%5B127.09097172%2C%2037.45279201300002%5D%2C%20%5B127.09031546300002%2C%2037.45127626300001%5D%2C%20%5B127.088339345%2C%2037.44863720799998%5D%2C%20%5B127.08817389900003%2C%2037.44539478899998%5D%2C%20%5B127.08643383499998%2C%2037.444272048000016%5D%2C%20%5B127.08387430300002%2C%2037.44393121899998%5D%2C%20%5B127.08244417699996%2C%2037.441589282%5D%2C%20%5B127.08018384599995%2C%2037.441060924%5D%2C%20%5B127.07602324899995%2C%2037.4421491%5D%2C%20%5B127.07300687899999%2C%2037.442027784%5D%2C%20%5B127.07163946599997%2C%2037.441534323999974%5D%2C%20%5B127.07202035099999%2C%2037.43886650600001%5D%2C%20%5B127.07377263199999%2C%2037.4377991%5D%2C%20%5B127.07303654400005%2C%2037.43641643799998%5D%2C%20%5B127.07145178799999%2C%2037.43587310999999%5D%2C%20%5B127.07153192800001%2C%2037.434126424%5D%2C%20%5B127.07064098%2C%2037.432054365%5D%2C%20%5B127.07121871100003%2C%2037.430827679%5D%2C%20%5B127.06828150800004%2C%2037.43068911%5D%2C%20%5B127.06673287599995%2C%2037.43013986099999%5D%2C%20%5B127.06569412299996%2C%2037.42899537400001%5D%2C%20%5B127.06332888700001%2C%2037.429761353%5D%2C%20%5B127.06115775299997%2C%2037.429994996%5D%2C%20%5B127.05997391899996%2C%2037.429578751%5D%2C%20%5B127.05808265300004%2C%2037.430020472000024%5D%2C%20%5B127.05369652000002%2C%2037.428984660000026%5D%2C%20%5B127.05218338899999%2C%2037.42834757000003%5D%2C%20%5B127.04959235199999%2C%2037.430279794%5D%2C%20%5B127.04738197699999%2C%2037.43071133900003%5D%2C%20%5B127.04723810300004%2C%2037.43204956%5D%2C%20%5B127.04632535799999%2C%2037.43345562899998%5D%2C%20%5B127.04496506500004%2C%2037.43385551799997%5D%2C%20%5B127.044199833%2C%2037.435063092%5D%2C%20%5B127.04006758800006%2C%2037.438243274%5D%2C%20%5B127.03708267800005%2C%2037.43828114399997%5D%2C%20%5B127.03558860400005%2C%2037.43901011600002%5D%2C%20%5B127.03518600799998%2C%2037.440947884000025%5D%2C%20%5B127.03644468300001%2C%2037.44183865799999%5D%2C%20%5B127.03749597800004%2C%2037.443272594%5D%2C%20%5B127.038243717%2C%2037.44518611900003%5D%2C%20%5B127.03726765%2C%2037.44645425300001%5D%2C%20%5B127.03717336600005%2C%2037.44801560799999%5D%2C%20%5B127.03782636200003%2C%2037.44892774700003%5D%2C%20%5B127.03692812600002%2C%2037.45081401300001%5D%2C%20%5B127.03579206200004%2C%2037.452201682%5D%2C%20%5B127.03477066100004%2C%2037.45262298599999%5D%2C%20%5B127.03714478699999%2C%2037.45521278799998%5D%2C%20%5B127.03675713500002%2C%2037.45644861599999%5D%2C%20%5B127.03503165799998%2C%2037.45789780400003%5D%2C%20%5B127.03494060599996%2C%2037.46018196900002%5D%2C%20%5B127.033746501%2C%2037.461242513%5D%2C%20%5B127.03471186900003%2C%2037.46416056200002%5D%2C%20%5B127.03315839200002%2C%2037.465024089999986%5D%2C%20%5B127.03121260499995%2C%2037.46563217200003%5D%2C%20%5B127.02954249100003%2C%2037.46537973%5D%2C%20%5B127.02970909099997%2C%2037.463910751000014%5D%2C%20%5B127.02933072500002%2C%2037.46270655%5D%2C%20%5B127.02805724300003%2C%2037.46125024600002%5D%2C%20%5B127.02837573700003%2C%2037.459898619%5D%2C%20%5B127.02658642100005%2C%2037.459646512%5D%2C%20%5B127.02645936199997%2C%2037.45834107600001%5D%2C%20%5B127.02529273699997%2C%2037.457498327%5D%2C%20%5B127.02280887500001%2C%2037.457254868%5D%2C%20%5B127.02152010400005%2C%2037.456229913000016%5D%2C%20%5B127.01970823600004%2C%2037.45579098899998%5D%2C%20%5B127.01756149699997%2C%2037.45613975200001%5D%2C%20%5B127.01630291799995%2C%2037.455234355000016%5D%2C%20%5B127.01452250900002%2C%2037.45486602300002%5D%2C%20%5B127.01066817699996%2C%2037.456042489000026%5D%2C%20%5B127.008544057%2C%2037.45805628699998%5D%2C%20%5B127.00822678700001%2C%2037.459104508%5D%2C%20%5B127.00705377999998%2C%2037.460220048999986%5D%2C%20%5B127.00578293399997%2C%2037.46255271299998%5D%2C%20%5B127.00448335800002%2C%2037.464077615%5D%2C%20%5B127.00492006399998%2C%2037.46584683899999%5D%2C%20%5B127.00369035799997%2C%2037.46772444499999%5D%2C%20%5B127.00274926199995%2C%2037.467128762000016%5D%2C%20%5B126.99879405800004%2C%2037.46723823899998%5D%2C%20%5B126.99676674199998%2C%2037.467076564000024%5D%2C%20%5B126.996245907%2C%2037.46661254999998%5D%2C%20%5B126.99663361800003%2C%2037.46478279199999%5D%2C%20%5B126.997384828%2C%2037.46369182699999%5D%2C%20%5B126.996782414%2C%2037.461877791%5D%2C%20%5B126.993368691%2C%2037.46150029400002%5D%2C%20%5B126.99266931099999%2C%2037.46034838899999%5D%2C%20%5B126.99171279500001%2C%2037.46052342799999%5D%2C%20%5B126.99000889599995%2C%2037.45946741%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2016%2C%20%22SHAPE_AREA%22%3A%200.004776%2C%20%22SHAPE_LEN%22%3A%200.437399%2C%20%22SIG_CD%22%3A%20%2211650%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seocho-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.97782373300004%2C%2037.63391334300002%5D%2C%20%5B126.981433809%2C%2037.634832856%5D%2C%20%5B126.983724113%2C%2037.636460397%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2017%2C%20%22SHAPE_AREA%22%3A%200.002511%2C%20%22SHAPE_LEN%22%3A%200.318673%2C%20%22SIG_CD%22%3A%20%2211290%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05182101699995%2C%2037.687069296%5D%2C%20%5B127.05519755800003%2C%2037.689209908%5D%2C%20%5B127.05840713099997%2C%2037.689689869%5D%2C%20%5B127.05969812599994%2C%2037.690342471%5D%2C%20%5B127.06130232099997%2C%2037.69219614100001%5D%2C%20%5B127.06249161400001%2C%2037.69305951199999%5D%2C%20%5B127.06274278700005%2C%2037.69467682200002%5D%2C%20%5B127.06356049299995%2C%2037.69497166299999%5D%2C%20%5B127.06650615199999%2C%2037.694498565%5D%2C%20%5B127.068049664%2C%2037.694837018999976%5D%2C%20%5B127.069366791%2C%2037.69375161099998%5D%2C%20%5B127.07291609799995%2C%2037.69387412100002%5D%2C%20%5B127.07522005500005%2C%2037.695310147999976%5D%2C%20%5B127.07856303100004%2C%2037.696092556999986%5D%2C%20%5B127.08150813999998%2C%2037.69625423399998%5D%2C%20%5B127.084240346%2C%2037.694384093%5D%2C%20%5B127.08425235100003%2C%2037.69188345499998%5D%2C%20%5B127.08551096600002%2C%2037.69048708600002%5D%2C%20%5B127.08687032%2C%2037.69001125599999%5D%2C%20%5B127.09093103999999%2C%2037.68968092699998%5D%2C%20%5B127.09345258600001%2C%2037.690087849%5D%2C%20%5B127.096066758%2C%2037.687916323000024%5D%2C%20%5B127.09641572400005%2C%2037.685647404%5D%2C%20%5B127.09580503999996%2C%2037.684650288%5D%2C%20%5B127.09428565300004%2C%2037.683427962999986%5D%2C%20%5B127.09292189500002%2C%2037.68156068000002%5D%2C%20%5B127.09282796000002%2C%2037.68005578999998%5D%2C%20%5B127.09197763199995%2C%2037.67920024699998%5D%2C%20%5B127.09221964100004%2C%2037.678039738%5D%2C%20%5B127.09388436100005%2C%2037.676669027%5D%2C%20%5B127.09464265999998%2C%2037.67354516199998%5D%2C%20%5B127.09577928399995%2C%2037.672681931%5D%2C%20%5B127.09577485800003%2C%2037.67062784799998%5D%2C%20%5B127.09626208899999%2C%2037.668837161%5D%2C%20%5B127.09500097800003%2C%2037.667606956999975%5D%2C%20%5B127.09469822699998%2C%2037.664927189000025%5D%2C%20%5B127.09542861800003%2C%2037.663895431000014%5D%2C%20%5B127.094157774%2C%2037.66330400700002%5D%2C%20%5B127.091221335%2C%2037.65934019600002%5D%2C%20%5B127.09114721900005%2C%2037.658248809999975%5D%2C%20%5B127.09190627600003%2C%2037.65744077099998%5D%2C%20%5B127.09236725000005%2C%2037.65549788499999%5D%2C%20%5B127.09401619699997%2C%2037.65230150399998%5D%2C%20%5B127.09248088799995%2C%2037.64971631999998%5D%2C%20%5B127.09364724600005%2C%2037.646632361%5D%2C%20%5B127.09448968499999%2C%2037.645856236999975%5D%2C%20%5B127.09458566599994%2C%2037.644576943%5D%2C%20%5B127.09757475499998%2C%2037.643971025999974%5D%2C%20%5B127.10160801200004%2C%2037.64477886399999%5D%2C%20%5B127.103936524%2C%2037.645499323000024%5D%2C%20%5B127.10658921100003%2C%2037.645384454%5D%2C%20%5B127.10770375100003%2C%2037.64497346799999%5D%2C%20%5B127.10930262900001%2C%2037.64286426299998%5D%2C%20%5B127.11076511800002%2C%2037.64273476300002%5D%2C%20%5B127.11223881%2C%2037.64037708500001%5D%2C%20%5B127.110580357%2C%2037.639364204%5D%2C%20%5B127.11249787%2C%2037.63648892100002%5D%2C%20%5B127.11242501499999%2C%2037.63425247999999%5D%2C%20%5B127.11162094999997%2C%2037.633851106%5D%2C%20%5B127.11222175299997%2C%2037.632645615%5D%2C%20%5B127.111649757%2C%2037.63150772300003%5D%2C%20%5B127.108900973%2C%2037.62930749999998%5D%2C%20%5B127.10597855699996%2C%2037.627337269%5D%2C%20%5B127.10434078499998%2C%2037.62327850299999%5D%2C%20%5B127.10407893299998%2C%2037.62166095499998%5D%2C%20%5B127.10492130800003%2C%2037.62157580500002%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2018%2C%20%22SHAPE_AREA%22%3A%200.00364%2C%20%22SHAPE_LEN%22%3A%200.309723%2C%20%22SIG_CD%22%3A%20%2211350%22%2C%20%22SIG_ENG_NM%22%3A%20%22Nowon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.144621648%2C%2037.51561223599998%5D%2C%20%5B127.14215399099999%2C%2037.51566003099998%5D%2C%20%5B127.14138931900004%2C%2037.514975155%5D%2C%20%5B127.14341256199998%2C%2037.51387651099998%5D%2C%20%5B127.14354620699999%2C%2037.512663953000015%5D%2C%20%5B127.1414575%2C%2037.51240795199999%5D%2C%20%5B127.14096457300002%2C%2037.51040582000002%5D%2C%20%5B127.13998328499997%2C%2037.50864916199998%5D%2C%20%5B127.141534557%2C%2037.506717287000015%5D%2C%20%5B127.14102941299996%2C%2037.505493221999984%5D%2C%20%5B127.144121606%2C%2037.50440674700002%5D%2C%20%5B127.14597136099997%2C%2037.503236346999984%5D%2C%20%5B127.14771286899997%2C%2037.503223030000015%5D%2C%20%5B127.15022088800004%2C%2037.50474927699997%5D%2C%20%5B127.15212420600005%2C%2037.50297788%5D%2C%20%5B127.15285967099999%2C%2037.503062426999975%5D%2C%20%5B127.15656939099995%2C%2037.50190080300001%5D%2C%20%5B127.157743278%2C%2037.50318546800003%5D%2C%20%5B127.15887991900001%2C%2037.50239690400002%5D%2C%20%5B127.159410547%2C%2037.501369312%5D%2C%20%5B127.16140357300003%2C%2037.50020737699998%5D%2C%20%5B127.15977913799998%2C%2037.49679815600001%5D%2C%20%5B127.16001896600005%2C%2037.49433240399998%5D%2C%20%5B127.15848990500001%2C%2037.492042157000014%5D%2C%20%5B127.158182699%2C%2037.49051667399999%5D%2C%20%5B127.156300945%2C%2037.48949331900002%5D%2C%20%5B127.154795626%2C%2037.48731379999998%5D%2C%20%5B127.15219320300002%2C%2037.48624278199998%5D%2C%20%5B127.150769406%2C%2037.48474927500001%5D%2C%20%5B127.15104542899996%2C%2037.483961959%5D%2C%20%5B127.15068947400005%2C%2037.48202662699998%5D%2C%20%5B127.14915395599996%2C%2037.48074899300002%5D%2C%20%5B127.14783019699996%2C%2037.478664200000026%5D%2C%20%5B127.14439008099998%2C%2037.476456739000014%5D%2C%20%5B127.14354172599997%2C%2037.475545689%5D%2C%20%5B127.139692276%2C%2037.47419605099998%5D%2C%20%5B127.13937783300003%2C%2037.473447066%5D%2C%20%5B127.13558350400001%2C%2037.474578011%5D%2C%20%5B127.13368163799998%2C%2037.47568719499998%5D%2C%20%5B127.13267746400004%2C%2037.475808433%5D%2C%20%5B127.13024850199997%2C%2037.47510375899998%5D%2C%20%5B127.13042151800005%2C%2037.47189767899999%5D%2C%20%5B127.13538943799995%2C%2037.46932071200001%5D%2C%20%5B127.13404767700001%2C%2037.46859963000003%5D%2C%20%5B127.13014299300005%2C%2037.467765722000024%5D%2C%20%5B127.12704911399999%2C%2037.46842315399999%5D%2C%20%5B127.12521919400001%2C%2037.46962967899998%5D%2C%20%5B127.125063372%2C%2037.46727479200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2019%2C%20%22SHAPE_AREA%22%3A%200.003449%2C%20%22SHAPE_LEN%22%3A%200.316773%2C%20%22SIG_CD%22%3A%20%2211710%22%2C%20%22SIG_ENG_NM%22%3A%20%22Songpa-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2020%2C%20%22SHAPE_AREA%22%3A%200.001714%2C%20%22SHAPE_LEN%22%3A%200.193793%2C%20%22SIG_CD%22%3A%20%2211200%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2021%2C%20%22SHAPE_AREA%22%3A%200.001811%2C%20%22SHAPE_LEN%22%3A%200.231131%2C%20%22SIG_CD%22%3A%20%2211410%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seodaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.858345407%2C%2037.50992339800001%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.85562926600005%2C%2037.50916078300003%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.84873119999997%2C%2037.508888764%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84076167%2C%2037.50604941%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.837508791%2C%2037.50328696299999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.826851927%2C%2037.510431685000015%5D%2C%20%5B126.82420556800002%2C%2037.51053149099999%5D%2C%20%5B126.82387418500002%2C%2037.51088144200003%5D%2C%20%5B126.824425776%2C%2037.513182637%5D%2C%20%5B126.824273167%2C%2037.514473764%5D%2C%20%5B126.82341871799997%2C%2037.51495970899998%5D%2C%20%5B126.82312208200005%2C%2037.51622650500002%5D%2C%20%5B126.82451390100005%2C%2037.51772160199999%5D%2C%20%5B126.82563535199995%2C%2037.520085435%5D%2C%20%5B126.82515520799996%2C%2037.523010279%5D%2C%20%5B126.82676624299995%2C%2037.525130822999984%5D%2C%20%5B126.82816515299999%2C%2037.525836518%5D%2C%20%5B126.82844406000004%2C%2037.52670751199997%5D%2C%20%5B126.827086602%2C%2037.529715297%5D%2C%20%5B126.825575308%2C%2037.529853316000015%5D%2C%20%5B126.82344723300002%2C%2037.532623817%5D%2C%20%5B126.82356902000004%2C%2037.533770715%5D%2C%20%5B126.82193790199995%2C%2037.534835165%5D%2C%20%5B126.82244956399995%2C%2037.536550956999974%5D%2C%20%5B126.82233701300004%2C%2037.53793206300003%5D%2C%20%5B126.82160576399997%2C%2037.539720295999985%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2022%2C%20%22SHAPE_AREA%22%3A%200.001775%2C%20%22SHAPE_LEN%22%3A%200.294094%2C%20%22SIG_CD%22%3A%20%2211470%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yangcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88272419199996%2C%2037.51576865800001%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2023%2C%20%22SHAPE_AREA%22%3A%200.002512%2C%20%22SHAPE_LEN%22%3A%200.262953%2C%20%22SIG_CD%22%3A%20%2211560%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yeongdeungpo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2024%2C%20%22SHAPE_AREA%22%3A%200.002234%2C%20%22SHAPE_LEN%22%3A%200.214496%2C%20%22SIG_CD%22%3A%20%2211170%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yongsan-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20var%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc%20%3D%20%7B%7D%3B%0A%0A%20%20%20%20%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.color%20%3D%20d3.scale.threshold%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B1.0%2C%209.56312625250501%2C%2018.12625250501002%2C%2026.68937875751503%2C%2035.25250501002004%2C%2043.81563126252505%2C%2052.37875751503006%2C%2060.94188376753507%2C%2069.50501002004007%2C%2078.06813627254509%2C%2086.6312625250501%2C%2095.1943887775551%2C%20103.75751503006012%2C%20112.32064128256513%2C%20120.88376753507013%2C%20129.44689378757516%2C%20138.01002004008015%2C%20146.57314629258516%2C%20155.13627254509018%2C%20163.6993987975952%2C%20172.2625250501002%2C%20180.82565130260522%2C%20189.3887775551102%2C%20197.95190380761522%2C%20206.51503006012024%2C%20215.07815631262525%2C%20223.64128256513027%2C%20232.20440881763528%2C%20240.76753507014027%2C%20249.33066132264528%2C%20257.8937875751503%2C%20266.4569138276553%2C%20275.0200400801603%2C%20283.5831663326653%2C%20292.14629258517033%2C%20300.70941883767534%2C%20309.27254509018036%2C%20317.8356713426854%2C%20326.3987975951904%2C%20334.9619238476954%2C%20343.5250501002004%2C%20352.08817635270543%2C%20360.65130260521045%2C%20369.2144288577154%2C%20377.7775551102204%2C%20386.34068136272543%2C%20394.90380761523045%2C%20403.46693386773546%2C%20412.0300601202405%2C%20420.5931863727455%2C%20429.1563126252505%2C%20437.7194388777555%2C%20446.28256513026054%2C%20454.84569138276555%2C%20463.40881763527057%2C%20471.9719438877755%2C%20480.53507014028054%2C%20489.09819639278555%2C%20497.66132264529057%2C%20506.2244488977956%2C%20514.7875751503007%2C%20523.3507014028056%2C%20531.9138276553106%2C%20540.4769539078156%2C%20549.0400801603206%2C%20557.6032064128257%2C%20566.1663326653306%2C%20574.7294589178357%2C%20583.2925851703407%2C%20591.8557114228457%2C%20600.4188376753507%2C%20608.9819639278558%2C%20617.5450901803607%2C%20626.1082164328658%2C%20634.6713426853707%2C%20643.2344689378757%2C%20651.7975951903808%2C%20660.3607214428857%2C%20668.9238476953908%2C%20677.4869739478958%2C%20686.0501002004008%2C%20694.6132264529058%2C%20703.1763527054109%2C%20711.7394789579158%2C%20720.3026052104209%2C%20728.8657314629259%2C%20737.4288577154308%2C%20745.9919839679359%2C%20754.5551102204408%2C%20763.1182364729459%2C%20771.6813627254509%2C%20780.2444889779559%2C%20788.8076152304609%2C%20797.370741482966%2C%20805.9338677354709%2C%20814.496993987976%2C%20823.060120240481%2C%20831.623246492986%2C%20840.186372745491%2C%20848.7494989979959%2C%20857.312625250501%2C%20865.875751503006%2C%20874.438877755511%2C%20883.002004008016%2C%20891.5651302605211%2C%20900.128256513026%2C%20908.6913827655311%2C%20917.2545090180361%2C%20925.8176352705411%2C%20934.3807615230461%2C%20942.943887775551%2C%20951.5070140280561%2C%20960.0701402805611%2C%20968.6332665330661%2C%20977.1963927855711%2C%20985.7595190380762%2C%20994.3226452905811%2C%201002.8857715430862%2C%201011.4488977955912%2C%201020.0120240480962%2C%201028.5751503006013%2C%201037.1382765531062%2C%201045.7014028056112%2C%201054.2645290581163%2C%201062.8276553106211%2C%201071.3907815631262%2C%201079.9539078156313%2C%201088.5170340681364%2C%201097.0801603206412%2C%201105.6432865731463%2C%201114.2064128256513%2C%201122.7695390781564%2C%201131.3326653306613%2C%201139.8957915831663%2C%201148.4589178356714%2C%201157.0220440881762%2C%201165.5851703406813%2C%201174.1482965931864%2C%201182.7114228456915%2C%201191.2745490981963%2C%201199.8376753507014%2C%201208.4008016032064%2C%201216.9639278557115%2C%201225.5270541082164%2C%201234.0901803607214%2C%201242.6533066132265%2C%201251.2164328657316%2C%201259.7795591182364%2C%201268.3426853707415%2C%201276.9058116232466%2C%201285.4689378757514%2C%201294.0320641282565%2C%201302.5951903807616%2C%201311.1583166332666%2C%201319.7214428857715%2C%201328.2845691382765%2C%201336.8476953907816%2C%201345.4108216432867%2C%201353.9739478957915%2C%201362.5370741482966%2C%201371.1002004008017%2C%201379.6633266533065%2C%201388.2264529058116%2C%201396.7895791583167%2C%201405.3527054108217%2C%201413.9158316633266%2C%201422.4789579158316%2C%201431.0420841683367%2C%201439.6052104208418%2C%201448.1683366733466%2C%201456.7314629258517%2C%201465.2945891783568%2C%201473.8577154308616%2C%201482.4208416833667%2C%201490.9839679358718%2C%201499.5470941883768%2C%201508.1102204408817%2C%201516.6733466933867%2C%201525.2364729458918%2C%201533.799599198397%2C%201542.3627254509017%2C%201550.9258517034068%2C%201559.4889779559119%2C%201568.0521042084167%2C%201576.6152304609218%2C%201585.1783567134269%2C%201593.741482965932%2C%201602.3046092184368%2C%201610.8677354709419%2C%201619.430861723447%2C%201627.993987975952%2C%201636.5571142284568%2C%201645.120240480962%2C%201653.683366733467%2C%201662.246492985972%2C%201670.809619238477%2C%201679.372745490982%2C%201687.935871743487%2C%201696.4989979959919%2C%201705.062124248497%2C%201713.625250501002%2C%201722.188376753507%2C%201730.751503006012%2C%201739.314629258517%2C%201747.877755511022%2C%201756.4408817635272%2C%201765.004008016032%2C%201773.567134268537%2C%201782.1302605210421%2C%201790.693386773547%2C%201799.256513026052%2C%201807.8196392785571%2C%201816.3827655310622%2C%201824.945891783567%2C%201833.5090180360721%2C%201842.0721442885772%2C%201850.6352705410823%2C%201859.198396793587%2C%201867.7615230460922%2C%201876.3246492985973%2C%201884.887775551102%2C%201893.4509018036072%2C%201902.0140280561122%2C%201910.5771543086173%2C%201919.1402805611222%2C%201927.7034068136272%2C%201936.2665330661323%2C%201944.8296593186374%2C%201953.3927855711422%2C%201961.9559118236473%2C%201970.5190380761524%2C%201979.0821643286574%2C%201987.6452905811623%2C%201996.2084168336673%2C%202004.7715430861724%2C%202013.3346693386773%2C%202021.8977955911823%2C%202030.4609218436874%2C%202039.0240480961925%2C%202047.5871743486973%2C%202056.1503006012026%2C%202064.7134268537075%2C%202073.2765531062123%2C%202081.8396793587176%2C%202090.4028056112224%2C%202098.9659318637273%2C%202107.5290581162326%2C%202116.0921843687374%2C%202124.6553106212423%2C%202133.2184368737476%2C%202141.7815631262524%2C%202150.3446893787577%2C%202158.9078156312626%2C%202167.4709418837674%2C%202176.0340681362727%2C%202184.5971943887776%2C%202193.1603206412824%2C%202201.7234468937877%2C%202210.2865731462925%2C%202218.8496993987974%2C%202227.4128256513027%2C%202235.9759519038075%2C%202244.539078156313%2C%202253.1022044088177%2C%202261.6653306613225%2C%202270.228456913828%2C%202278.7915831663327%2C%202287.3547094188375%2C%202295.917835671343%2C%202304.4809619238476%2C%202313.0440881763525%2C%202321.607214428858%2C%202330.1703406813626%2C%202338.733466933868%2C%202347.296593186373%2C%202355.8597194388776%2C%202364.422845691383%2C%202372.9859719438878%2C%202381.5490981963926%2C%202390.112224448898%2C%202398.6753507014027%2C%202407.2384769539076%2C%202415.801603206413%2C%202424.3647294589177%2C%202432.927855711423%2C%202441.490981963928%2C%202450.0541082164327%2C%202458.617234468938%2C%202467.180360721443%2C%202475.7434869739477%2C%202484.306613226453%2C%202492.869739478958%2C%202501.432865731463%2C%202509.995991983968%2C%202518.559118236473%2C%202527.122244488978%2C%202535.685370741483%2C%202544.248496993988%2C%202552.811623246493%2C%202561.374749498998%2C%202569.937875751503%2C%202578.501002004008%2C%202587.064128256513%2C%202595.6272545090183%2C%202604.190380761523%2C%202612.753507014028%2C%202621.3166332665332%2C%202629.879759519038%2C%202638.442885771543%2C%202647.0060120240482%2C%202655.569138276553%2C%202664.132264529058%2C%202672.695390781563%2C%202681.258517034068%2C%202689.8216432865734%2C%202698.384769539078%2C%202706.947895791583%2C%202715.5110220440883%2C%202724.074148296593%2C%202732.637274549098%2C%202741.2004008016033%2C%202749.763527054108%2C%202758.326653306613%2C%202766.8897795591183%2C%202775.452905811623%2C%202784.0160320641285%2C%202792.5791583166333%2C%202801.142284569138%2C%202809.7054108216435%2C%202818.2685370741483%2C%202826.831663326653%2C%202835.3947895791584%2C%202843.9579158316633%2C%202852.521042084168%2C%202861.0841683366734%2C%202869.6472945891783%2C%202878.2104208416836%2C%202886.7735470941884%2C%202895.3366733466933%2C%202903.8997995991986%2C%202912.4629258517034%2C%202921.0260521042082%2C%202929.5891783567135%2C%202938.1523046092184%2C%202946.7154308617232%2C%202955.2785571142285%2C%202963.8416833667334%2C%202972.4048096192387%2C%202980.9679358717435%2C%202989.5310621242484%2C%202998.0941883767537%2C%203006.6573146292585%2C%203015.2204408817634%2C%203023.7835671342687%2C%203032.3466933867735%2C%203040.9098196392783%2C%203049.4729458917836%2C%203058.0360721442885%2C%203066.599198396794%2C%203075.1623246492986%2C%203083.7254509018035%2C%203092.2885771543088%2C%203100.8517034068136%2C%203109.4148296593185%2C%203117.9779559118238%2C%203126.5410821643286%2C%203135.1042084168334%2C%203143.6673346693387%2C%203152.2304609218436%2C%203160.793587174349%2C%203169.3567134268537%2C%203177.9198396793586%2C%203186.482965931864%2C%203195.0460921843687%2C%203203.6092184368736%2C%203212.172344689379%2C%203220.7354709418837%2C%203229.298597194389%2C%203237.861723446894%2C%203246.4248496993987%2C%203254.987975951904%2C%203263.551102204409%2C%203272.1142284569137%2C%203280.677354709419%2C%203289.240480961924%2C%203297.8036072144287%2C%203306.366733466934%2C%203314.929859719439%2C%203323.492985971944%2C%203332.056112224449%2C%203340.619238476954%2C%203349.182364729459%2C%203357.745490981964%2C%203366.308617234469%2C%203374.871743486974%2C%203383.434869739479%2C%203391.9979959919838%2C%203400.561122244489%2C%203409.124248496994%2C%203417.687374749499%2C%203426.250501002004%2C%203434.813627254509%2C%203443.376753507014%2C%203451.939879759519%2C%203460.503006012024%2C%203469.066132264529%2C%203477.629258517034%2C%203486.192384769539%2C%203494.755511022044%2C%203503.318637274549%2C%203511.8817635270543%2C%203520.444889779559%2C%203529.008016032064%2C%203537.5711422845693%2C%203546.134268537074%2C%203554.697394789579%2C%203563.2605210420843%2C%203571.823647294589%2C%203580.386773547094%2C%203588.9498997995993%2C%203597.513026052104%2C%203606.0761523046094%2C%203614.6392785571143%2C%203623.202404809619%2C%203631.7655310621244%2C%203640.3286573146293%2C%203648.891783567134%2C%203657.4549098196394%2C%203666.0180360721442%2C%203674.581162324649%2C%203683.1442885771544%2C%203691.7074148296592%2C%203700.2705410821645%2C%203708.8336673346694%2C%203717.396793587174%2C%203725.9599198396795%2C%203734.5230460921844%2C%203743.086172344689%2C%203751.6492985971945%2C%203760.2124248496993%2C%203768.775551102204%2C%203777.3386773547095%2C%203785.9018036072143%2C%203794.4649298597196%2C%203803.0280561122245%2C%203811.5911823647293%2C%203820.1543086172346%2C%203828.7174348697395%2C%203837.2805611222443%2C%203845.8436873747496%2C%203854.4068136272545%2C%203862.9699398797593%2C%203871.5330661322646%2C%203880.0961923847694%2C%203888.6593186372747%2C%203897.2224448897796%2C%203905.7855711422844%2C%203914.3486973947897%2C%203922.9118236472946%2C%203931.4749498997994%2C%203940.0380761523047%2C%203948.6012024048096%2C%203957.164328657315%2C%203965.7274549098197%2C%203974.2905811623245%2C%203982.85370741483%2C%203991.4168336673347%2C%203999.9799599198395%2C%204008.543086172345%2C%204017.1062124248497%2C%204025.6693386773545%2C%204034.23246492986%2C%204042.7955911823647%2C%204051.35871743487%2C%204059.921843687375%2C%204068.4849699398796%2C%204077.048096192385%2C%204085.61122244489%2C%204094.1743486973946%2C%204102.7374749498995%2C%204111.300601202405%2C%204119.86372745491%2C%204128.426853707415%2C%204136.98997995992%2C%204145.553106212425%2C%204154.116232464929%2C%204162.679358717435%2C%204171.24248496994%2C%204179.805611222445%2C%204188.36873747495%2C%204196.931863727455%2C%204205.49498997996%2C%204214.058116232465%2C%204222.62124248497%2C%204231.184368737475%2C%204239.74749498998%2C%204248.310621242485%2C%204256.87374749499%2C%204265.436873747495%2C%204274.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23edf8e9ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23c7e9c0ff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%23a1d99bff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2374c476ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%2331a354ff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%2C%20%27%23006d2cff%27%5D%29%3B%0A%20%20%20%20%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x%20%3D%20d3.scale.linear%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B1.0%2C%204274.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B0%2C%20400%5D%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.legend%20%3D%20L.control%28%7Bposition%3A%20%27topright%27%7D%29%3B%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.legend.onAdd%20%3D%20function%20%28map%29%20%7Bvar%20div%20%3D%20L.DomUtil.create%28%27div%27%2C%20%27legend%27%29%3B%20return%20div%7D%3B%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.legend.addTo%28map_1fa756f29afb4313accf910c33f13947%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.xAxis%20%3D%20d3.svg.axis%28%29%0A%20%20%20%20%20%20%20%20.scale%28color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x%29%0A%20%20%20%20%20%20%20%20.orient%28%22top%22%29%0A%20%20%20%20%20%20%20%20.tickSize%281%29%0A%20%20%20%20%20%20%20%20.tickValues%28%5B1.0%2C%20713.1666666666666%2C%201425.3333333333333%2C%202137.5%2C%202849.6666666666665%2C%203561.833333333333%2C%204274.0%5D%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.svg%20%3D%20d3.select%28%22.legend.leaflet-control%22%29.append%28%22svg%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22id%22%2C%20%27legend%27%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20450%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2040%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.g%20%3D%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.svg.append%28%22g%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22key%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22transform%22%2C%20%22translate%2825%2C16%29%22%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.g.selectAll%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.data%28color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.color.range%28%29.map%28function%28d%2C%20i%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20return%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20x0%3A%20i%20%3F%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x%28color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.color.domain%28%29%5Bi%20-%201%5D%29%20%3A%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x.range%28%29%5B0%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x1%3A%20i%20%3C%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.color.domain%28%29.length%20%3F%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x%28color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.color.domain%28%29%5Bi%5D%29%20%3A%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.x.range%28%29%5B1%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20z%3A%20d%0A%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%7D%29%29%0A%20%20%20%20%20%20.enter%28%29.append%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2010%29%0A%20%20%20%20%20%20%20%20.attr%28%22x%22%2C%20function%28d%29%20%7B%20return%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20function%28d%29%20%7B%20return%20d.x1%20-%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.style%28%22fill%22%2C%20function%28d%29%20%7B%20return%20d.z%3B%20%7D%29%3B%0A%0A%20%20%20%20color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.g.call%28color_map_ffeb1a8e5a15490ca2f59f0ef25c29fc.xAxis%29.append%28%22text%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22caption%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22y%22%2C%2021%29%0A%20%20%20%20%20%20%20%20.text%28%27%27%29%3B%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20map_d6fb1ba880dc49459feac09a2f6f713d.sync%28map_1fa756f29afb4313accf910c33f13947%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20map_1fa756f29afb4313accf910c33f13947.sync%28map_d6fb1ba880dc49459feac09a2f6f713d%29%3B%0A%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



## Summary

- Mapo has distinctive patterns in commute time.
- During 7am ~ 10am: inflow is approximately 30~35%
- During 5pm ~ 8pm: outflow is approximately 25~30%
- During commute time, there's highest traffic at stations in DMC and Hongik University.
- Traffics with other counties are mostly with nearby counties.


```python

```
