## Intro

In our last episode, we looked at Seoul Bike Stations across the city of Seoul.

In this post, we will explore the trips taken in April of 2021.

On top of the GPS/location feature from station dataset, the trips dataset has a time feature (time of rent, time of return) which poses exciting questions like

How does trips differ by counties, hours of the day, or days of the week?

Do counties or stations have distinct characteristics with which we can cluster them into a few groups?

# Data

In this post, we will explore the trips taken in April of 2021.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import folium
from folium import plugins
import missingno as mnso
import warnings
warnings.filterwarnings('ignore')
```


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




    
![png](output_13_1.png)
    



```python
s_eng = pd.read_csv('C:\\Users\\82104\\Desktop\\New_Seoul_Bike\\Data\\Station\\Seoul_Distict_ID_English.csv', encoding = 'euc-kr')
s_eng = s_eng.drop(columns=['county_id','lat','long'])
s_kr = pd.read_csv('C:\\Users\\82104\\Desktop\\New_Seoul_Bike\\Data\\Station\\station_df_facilities_info.csv', encoding = 'cp949')
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
# after_merged = len(df)
# loss = before_merged - after_merged

# print(df.shape)
# print("%d loss. (%d%%)" %(loss, float(loss)/before_merged*100))
```

    (578407, 20)
    470151 loss. (44%)
    

# Which Day of Week and Hour had the most frequent trips? 

Let's start with the simplest question.
We will look at trips taken by **Day of Week, Hour, County** respectively

## Trips by Day of Week


```python
use_by_dayofweek = df.groupby('start_dayofweek').size()
use_by_dayofweek.index = "Mon Tue Wed Thu Fri Sat Sun".split()
use_by_dayofweek.plot(kind='bar', figsize=(12, 5), rot=0, title="Number of trips by day of week")
plt.box(False)
plt.tight_layout()
plt.show()
```


    
![png](output_30_0.png)
    


There are  27.1% more trips taken during weekends than weekdays.
During weekdays, people rode more often on Tuesday and Wednesday.
During weekends, people came out to ride bikes more on Saturday than Sunday.



```python
mean_weekday = use_by_dayofweek.loc["Mon Tue Wed Thu Fri".split()].mean()
mean_weekend = use_by_dayofweek.loc["Sat Sun".split()].mean()

pd.Series(data=[
    mean_weekday,
    mean_weekend
], index=["Weekdays", "Weekend"]).plot(kind='bar', figsize=(5, 5), rot=0, title="Average # of Trips: Weekdays vs. Weekend")
plt.box(False)
plt.tight_layout()
plt.show()
```


    
![png](output_32_0.png)
    



```python
diff = (mean_weekend - mean_weekday) / mean_weekday * 100
print("There are %.1f%% more trips during weekends than weekdays." %diff)
```

    There are 27.1% more trips during weekends than weekdays.
    

There are 27.1% more trips during weekends than weekdays.

## Trips by Hour

We can presume that the trip pattern might look different between weekdays and weekends. Let's compare them and see if the difference exists.

### On Weekdays


```python
pd.DataFrame(data={
    "Rent": df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby('start_hour').size()//5,
    "Return": df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby('end_hour').size()//5
}).plot(kind='bar', figsize=(24, 5), rot=0, title="Average # of Trips by Hour (Weekdays)")
plt.xlabel("Hour")
plt.box(False)
plt.legend(frameon=False)
plt.tight_layout()
plt.show()
```


    
![png](output_38_0.png)
    


### On Weekends


```python
pd.DataFrame(data={
    "Rent": df[df['start_dayofweek'].isin(set(range(5, 7)))].groupby('start_hour').size()//5,
    "Return": df[df['start_dayofweek'].isin(set(range(5, 7)))].groupby('end_hour').size()//5
}).plot(kind='bar', figsize=(24, 5), rot=0, title="시간에 따른 평균 이용량(주말)")
plt.xlabel("Hour")
plt.box(False)
plt.legend(frameon=False)
plt.tight_layout()
plt.show()
```


    
![png](output_40_0.png)
    


We can observe the following:
- **On Weekdays, relatively more trips are taken at 8 am and 18 pm.** This may be due to commute time. Yes, people work 9 to 6 in Korea (one more hour than 9-5 in the US).
- **On Weekends, more trips are taken as it gets late in the afternoon**  and hits the peak around 18pm. This may vary depending on the season and the sunset time.
- Otherwise, except for commute time. **Rent > Return in the afternoon and the other way around at night.**

## 1.3. Does Hourly Trip Pattern Look Different by County?

### Rent


```python
pvt_table = (df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby(['county_eng_x', 'start_hour']).size()//5).unstack()

plt.figure(figsize=(24, 13))
sns.heatmap(pvt_table, annot=True, fmt='d', cmap="Greens", cbar=False, linewidth=1)
plt.title("Average # of Rent (x-axis: Hour, y-axis: County")
plt.ylabel("")
plt.tight_layout()
plt.show()
```


    
![png](output_44_0.png)
    


### Return


```python
pvt_table = (df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby(['county_eng_y', 'end_hour']).size()//5).unstack()

plt.figure(figsize=(24, 13))
sns.heatmap(pvt_table, annot=True, fmt='d', cmap="Oranges", cbar=False, linewidth=1)
plt.title("Average # of Return (x-axis: Hour, y-axis: County")
plt.ylabel("")
plt.tight_layout()
plt.show()
```


    
![png](output_46_0.png)
    


**All counties have a very simliar hourly trip patterns.**

The dark colors in commute time sticks out which makes me wonder:
- In which county is the # of Rent > # of Return?
- In which county is the # of Rent < # of Return?

which sums up to the following question:

- **Are the counties that have higher Rent or Return during commute time?**

### 1.4. Are the counties that have higher Rent or Return during commute time?

Let's extract the trips taken during communte time (8am and 18pm). In addition, let's look at the Rent to Return Raio.


```python
pvt_table = (df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby(['county_eng_x', 'start_hour']).size()//5)
rent = pd.DataFrame([pvt_table.xs(8,level=1), pvt_table.xs(18,level=1)], index=[8, 18])

pvt_table = (df[df['start_dayofweek'].isin(set(range(0, 5)))].groupby(['county_eng_y', 'end_hour']).size()//5)
ret = pd.DataFrame([pvt_table.xs(8,level=1), pvt_table.xs(18,level=1)], index=[8, 18])
```


```python
total = rent+ret
rent = rent / total
ret = ret / total
```


```python
morning_diff = pd.DataFrame([rent.T[8], ret.T[8]], index=['Rent', 'REturn']).T.sort_values('Rent')
eve_diff = pd.DataFrame([rent.T[18], ret.T[18]], index=['Rent', 'Return']).T.sort_values('Rent')
```


```python
fig, axes = plt.subplots(1, 2, figsize=(24, 10))

for i, (diff, time_name) in enumerate(zip([morning_diff, eve_diff], ["Going to Work(08:00)", "Getting off Work(18:00)"])):
    ax = diff.plot(kind='barh', color=['green', 'orange'], stacked=True, title="Usage Ratio when %s"%time_name, ax=axes[i])

    for p in ax.patches: 
        left, bottom, width, height = p.get_bbox().bounds 
        ax.annotate("%.1f %%"%(width*100), xy=(left+width/2, bottom+height/2), ha='center', va='center')

    plt.sca(ax)
    plt.xticks([])
    plt.box(False)
axes[0].get_legend().remove()
plt.legend(loc='center left', bbox_to_anchor=(0.97, 0.5), frameon=False)
plt.tight_layout()
plt.show()
```


    
![png](output_53_0.png)
    


- In the morning commute time, **Seongbuk had 26% more Rents than Returns.**
- In the evening commute time, **Geumcheon had 10% more Returns than Rents.**
- In the evening commute time, **Seongbuk had 15% more Returns than Rents.**

However, the two plots above have different order of y-axis, so it's quiet difficult to compare the difference in morning and evening commute time.

Let's visualize just the Rents this time because Returns = Trips - Rents. 


```python
morning_eve_diff = pd.DataFrame([morning_diff['Rent'], eve_diff['Rent']], index=['Morning', 'Evening']).T
morning_eve_diff.sort_values('Morning', inplace=True, ascending=False)

morning_eve_diff.plot(kind='bar', figsize=(24, 5), title="Morning Commute vs. Evening Communte), Rent Ratio by County", rot=0, color=['royalblue', 'midnightblue'])
plt.box(False)
plt.xticks(rotation = 45)
plt.legend(frameon=False)
plt.show()
```


    
![png](output_55_0.png)
    


The morning commute bars graudlly decrease, while the evening commute bars gradually increase.

In other words, **counties with higher Rent rate in the morning have higher Return rate in the evening (The correlation is 0.8).**

We can imagine that **people ride Seoul bikes from home -> work -> home on weekdays**

Well, this may be pretty obvious, but we can verify our groundless inference with this visualization now!

## 2. Which County has High Trip Distance and Trip Hour?

We can assume that a county with high average trip distance and trip hour has more riders who travel a far distance or ride longer.

Let's first look at the trip distance

## Trip Distance

Let's draw the distribution in boxplot and distplot.


```python
distance = df['distance']

# Exclude 0
distance = distance[distance != 0]
```


```python
def draw_box_distplot(series, title, xlabel, axvline=False, color='C0', bins=100):
    # Cut the window in 2 parts
    f, (ax_box, ax_hist) = plt.subplots(2, sharex=True, gridspec_kw={"height_ratios": (.15, .85)}, figsize=(24, 7))

    # Add a graph in each part
    sns.boxplot(series, ax=ax_box, boxprops={'alpha':0.6}, color=color)
    sns.distplot(series, ax=ax_hist, color=color, bins=bins)

    # Remove x axis name for the boxplot
    ax_box.set(xlabel='')

    plt.sca(ax_box)
    plt.box(False)
    plt.title(title)
    plt.sca(ax_hist)
    plt.box(False)
    plt.xlabel(xlabel)
    
    if axvline:
        plt.axvline(series.mean(), color='green')
        plt.axvline(series.value_counts().idxmax(), color='red')
        plt.axvline(series.median(), color='blue')
    
    plt.tight_layout()
    plt.show()
    
draw_box_distplot(distance, "Distance(m) Distribution", "Distance(m)", color='green')
```


    
![png](output_61_0.png)
    


The distribution is skewed due to outliers.
Let's remove the outliers for the sake of better analysis.


```python
def remove_outlier(series):
    q1 = series.quantile(0.25)
    q3 = series.quantile(0.75)
    iqr = q3-q1 #Interquartile range
    fence_low  = q1-1.5*iqr
    fence_high = q3+1.5*iqr
    series_out = series.loc[(series > fence_low) & (series < fence_high)]
    return series_out
```


```python
distance = remove_outlier(distance)

draw_box_distplot(distance, "Distance(m) Distribution", "Distance(m)", axvline=True, color='green')
```


    
![png](output_64_0.png)
    



```python
#print("Mode: %dm" %distance.value_counts().idxmax())
print("Median: %dm" %distance.median())
print("Truncated Mean(5~95%%): %dm" %distance.mean())
```

    Median: 1798m
    Truncated Mean(5~95%): 2370m
    

The distribution is still skewed a little bit, but the median is 1798 meters.

Let's dig a little deeper by looking at distributions by county.


```python
distance = df['distance']

# Exclude 0 
distance = distance[distance != 0]
```


```python
distance_df_total = None
dist_by_region = df.groupby('county_eng_x')

for name, g_df in dist_by_region:
    distance = remove_outlier(g_df['distance']).reset_index(drop=True)
    distance_df = pd.DataFrame(distance)
    distance_df['county_eng'] = name
    
    if distance_df_total is None:
        distance_df_total = distance_df
    else:
        distance_df_total = distance_df_total.append(distance_df, ignore_index=True)

distance_df_total.head()
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
      <th>distance</th>
      <th>county_eng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1984.84</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2950.00</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>2</th>
      <td>976.96</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>3</th>
      <td>317.95</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>4</th>
      <td>976.99</td>
      <td>Dobong</td>
    </tr>
  </tbody>
</table>
</div>




```python
medians = distance_df_total.groupby('county_eng')['distance'].median()
medians = medians.sort_values(ascending=False)
medians
```




    county_eng
    Yongsan         2633.600
    Gangnam         2617.935
    Seongdong       2370.000
    Seocho          2120.470
    Mapo            2091.800
    Seodaemun       1997.865
    Dongjak         1985.550
    Guro            1956.540
    Dobong          1931.530
    Songpa          1902.760
    Seongbuk        1889.220
    Gwanak          1880.710
    Geumcheon       1870.000
    Nowon           1806.325
    Gangbuk         1782.775
    Yeongdeungpo    1759.990
    Dongdaemun      1642.770
    Gwangjin        1629.950
    Jung            1624.960
    Eunpyeong       1604.430
    Gangdong        1570.580
    Jungnang        1568.530
    Yangcheon       1517.460
    Jongno          1421.980
    Gangseo         1345.650
    Name: distance, dtype: float64




```python
distance_medians = medians
```


```python
plt.figure(figsize=(24, 10))
sns.violinplot(x='county_eng', y='distance', data=distance_df_total, order=medians.index, palette="Purples_r")
plt.tight_layout()
plt.xticks(rotation = 45)
plt.box(False)
plt.show()
```


    
![png](output_71_0.png)
    



```python
plt.figure(figsize=(24, 5))
sns.barplot(x='county_eng', y='distance', data=medians.reset_index(), palette="Purples_r")
plt.xlabel("")
plt.ylabel("")
plt.title("Median Distance by County (m)")
plt.box(False)
plt.tight_layout()
plt.show()
```


    
![png](output_72_0.png)
    


- The distributions are skewed in every county.
- Counties with longer distance tend to have higher median as well as more outliers.

## 2.2 Trip Hour

### Distribution of All Trips


```python
time = df['duration']
time = time[time != 0]

draw_box_distplot(time, "Duration (in minutes) Distribution", "Duration (in minutes)", color='brown')
```


    
![png](output_76_0.png)
    


### Distribution without Outliers


```python
time = remove_outlier(time)

draw_box_distplot(time, "Duration (in minutes) Distribution", "Duration (in minutes)", axvline=True, color='brown', bins=74)
```


    
![png](output_78_0.png)
    



```python
print("Mode: %d Minutes" %time.value_counts().idxmax())
print("Median: %d Minutes" %time.median())
print("Truncated Mean(5~95%%): %d Minutes" %time.mean())
```

    Mode: 5 Minutes
    Median: 16 Minutes
    Truncated Mean(5~95%): 23 Minutes
    

Distributions for distance and hour look very simliar.
Mode is 5 minutes, and median is 16 minutes.


```python
usetime_df_total = None
usetime_by_region = df.groupby('county_eng_x')

for name, g_df in usetime_by_region:
    usetime = remove_outlier(g_df['duration']).reset_index(drop=True)
    usetime_df = pd.DataFrame(usetime)
    usetime_df['county_eng'] = name
    
    if usetime_df_total is None:
        usetime_df_total = usetime_df
    else:
        usetime_df_total = usetime_df_total.append(usetime_df, ignore_index=True)

usetime_df_total.head()
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
      <th>duration</th>
      <th>county_eng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>1</th>
      <td>31.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12.0</td>
      <td>Dobong</td>
    </tr>
  </tbody>
</table>
</div>




```python
medians = usetime_df_total.groupby('county_eng')['duration'].median()
medians = medians.sort_values(ascending=False)
medians
```




    county_eng
    Yongsan         25.0
    Gangnam         25.0
    Seongdong       23.0
    Mapo            21.0
    Seocho          20.0
    Dongjak         18.0
    Guro            17.0
    Yeongdeungpo    17.0
    Seodaemun       17.0
    Dobong          17.0
    Geumcheon       17.0
    Gangbuk         17.0
    Nowon           16.0
    Seongbuk        16.0
    Songpa          16.0
    Gwanak          15.0
    Gwangjin        15.0
    Jung            15.0
    Dongdaemun      14.0
    Jungnang        14.0
    Gangdong        14.0
    Eunpyeong       14.0
    Yangcheon       13.0
    Jongno          13.0
    Gangseo         10.0
    Name: duration, dtype: float64




```python
plt.figure(figsize=(24, 10))
sns.violinplot(x='county_eng', y='duration', data=usetime_df_total, order=medians.index, palette='YlOrBr_r')
plt.tight_layout()
plt.box(False)
plt.xlabel("")
plt.ylabel("Duration (in minutes)")
plt.show()
```


    
![png](output_83_0.png)
    



```python
plt.figure(figsize=(24, 5))
sns.barplot(x='county_eng', y='duration', data=medians.reset_index(), palette='YlOrBr_r')
plt.xlabel("")
plt.ylabel("")
plt.title("Median Duration (in minutes) by County")
plt.box(False)
plt.tight_layout()
plt.show()
```


    
![png](output_84_0.png)
    


Trip hour is very simliar to Trip distance.

It's interesting that some counties (Yongsan, Gangnam) have more outlier than others (Gangseo). 

Putting both distance and trip hour into maps:


```python
usetime_medians = medians
```


```python
usetime_df_total
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
      <th>duration</th>
      <th>county_eng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>1</th>
      <td>31.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12.0</td>
      <td>Dobong</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>543068</th>
      <td>98.0</td>
      <td>Yongsan</td>
    </tr>
    <tr>
      <th>543069</th>
      <td>38.0</td>
      <td>Yongsan</td>
    </tr>
    <tr>
      <th>543070</th>
      <td>38.0</td>
      <td>Yongsan</td>
    </tr>
    <tr>
      <th>543071</th>
      <td>33.0</td>
      <td>Yongsan</td>
    </tr>
    <tr>
      <th>543072</th>
      <td>33.0</td>
      <td>Yongsan</td>
    </tr>
  </tbody>
</table>
<p>543073 rows × 2 columns</p>
</div>




```python
usetime_df_total = None
usetime_by_region = df.groupby('county_x')

for name, g_df in usetime_by_region:
    usetime = remove_outlier(g_df['duration']).reset_index(drop=True)
    usetime_df = pd.DataFrame(usetime)
    usetime_df['county'] = name
    
    if usetime_df_total is None:
        usetime_df_total = usetime_df
    else:
        usetime_df_total = usetime_df_total.append(usetime_df, ignore_index=True)

usetime_df_total.head()
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
      <th>duration</th>
      <th>county</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>88.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>1</th>
      <td>68.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>109.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>57.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>76.0</td>
      <td>강남구</td>
    </tr>
  </tbody>
</table>
</div>




```python
distance_df_total = None
dist_by_region = df.groupby('county_x')

for name, g_df in dist_by_region:
    distance = remove_outlier(g_df['distance']).reset_index(drop=True)
    distance_df = pd.DataFrame(distance)
    distance_df['county'] = name
    
    if distance_df_total is None:
        distance_df_total = distance_df
    else:
        distance_df_total = distance_df_total.append(distance_df, ignore_index=True)

distance_df_total.head()
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
      <th>distance</th>
      <th>county</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12586.15</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11725.22</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11650.81</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9965.30</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5535.63</td>
      <td>강남구</td>
    </tr>
  </tbody>
</table>
</div>




```python
usetime_medians = usetime_df_total.groupby('county')['duration'].median().sort_values(ascending=False)
distance_medians = distance_df_total.groupby('county')['distance'].median().sort_values(ascending=False)
```


```python
bike_map = folium.plugins.DualMap(location=[37.541, 126.986], zoom_start=10, tiles='cartodbpositron', zoom_control=False)
folium.Choropleth(geo_data=geo_str,
                  data=distance_medians,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='Purples',
                  line_color='grey', line_opacity=0.5).add_to(bike_map.m1)
folium.Choropleth(geo_data=geo_str,
                  data=usetime_medians,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='YlOrBr',
                  line_color='grey', line_opacity=0.5).add_to(bike_map.m2)
bike_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_21d2760c69a444c08bed3d41c690fe16%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20absolute%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js%22%3E%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_a6aaaf43e579464b91cdf5a9af4616bd%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20absolute%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%2050.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/gh/jieter/Leaflet.Sync/L.Map.Sync.min.js%22%3E%3C/script%3E%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_21d2760c69a444c08bed3d41c690fe16%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_a6aaaf43e579464b91cdf5a9af4616bd%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_21d2760c69a444c08bed3d41c690fe16%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_21d2760c69a444c08bed3d41c690fe16%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.541%2C%20126.986%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_ca85ebc9961347babe1e42ab93625d09%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//cartodb-basemaps-%7Bs%7D.global.ssl.fastly.net/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%20contributors%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eCartoDB%5Cu003c/a%5Cu003e%2C%20CartoDB%20%5Cu003ca%20href%20%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eattributions%5Cu003c/a%5Cu003e%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_21d2760c69a444c08bed3d41c690fe16%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20choropleth_baf44a9ce98b4757828f24b7af0edc36%20%3D%20L.featureGroup%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_21d2760c69a444c08bed3d41c690fe16%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.properties.SIG_CD%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211380%22%3A%20case%20%2211230%22%3A%20case%20%2211260%22%3A%20case%20%2211140%22%3A%20case%20%2211740%22%3A%20case%20%2211215%22%3A%20case%20%2211560%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23dadaeb%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211110%22%3A%20case%20%2211500%22%3A%20case%20%2211470%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23f2f0f7%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211680%22%3A%20case%20%2211170%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%2354278f%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211440%22%3A%20case%20%2211650%22%3A%20case%20%2211410%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%239e9ac8%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211200%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23756bb1%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23bcbddc%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_styler%2C%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28choropleth_baf44a9ce98b4757828f24b7af0edc36%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_60cf4e7a6ec14960b8229675ce0d57c8_add%28%7B%22features%22%3A%20%5B%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00831558599998%2C%2037.69001680000002%5D%2C%20%5B127.00764424700003%2C%2037.69150638999997%5D%2C%20%5B127.00973504700005%2C%2037.69324682000001%5D%2C%20%5B127.00969102500005%2C%2037.696558160999984%5D%2C%20%5B127.01214887599997%2C%2037.69724089499999%5D%2C%20%5B127.01390304300003%2C%2037.698660491%5D%2C%20%5B127.01544040299996%2C%2037.70130154100002%5D%2C%20%5B127.019851357%2C%2037.700884901999984%5D%2C%20%5B127.02217147700003%2C%2037.69960736799999%5D%2C%20%5B127.02341184299996%2C%2037.69995983299998%5D%2C%20%5B127.02533791899998%2C%2037.69948105499998%5D%2C%20%5B127.02771004600004%2C%2037.700837447000026%5D%2C%20%5B127.02930740500005%2C%2037.699210467%5D%2C%20%5B127.02975002699998%2C%2037.69621560600001%5D%2C%20%5B127.03104551199999%2C%2037.69304682699999%5D%2C%20%5B127.032188658%2C%2037.69279409400002%5D%2C%20%5B127.03243270899998%2C%2037.69182616099999%5D%2C%20%5B127.03678096299996%2C%2037.692638805%5D%2C%20%5B127.038515888%2C%2037.69328696000002%5D%2C%20%5B127.03896796200002%2C%2037.69397974100002%5D%2C%20%5B127.04112667799996%2C%2037.695296503%5D%2C%20%5B127.04309027199997%2C%2037.69523619%5D%2C%20%5B127.04376597600003%2C%2037.693511828%5D%2C%20%5B127.04566611500002%2C%2037.692384920999984%5D%2C%20%5B127.04783268899996%2C%2037.69329455000002%5D%2C%20%5B127.04864899799998%2C%2037.69406779399998%5D%2C%20%5B127.04938379999999%2C%2037.691693921000024%5D%2C%20%5B127.04998367500002%2C%2037.690875163999976%5D%2C%20%5B127.04983654099999%2C%2037.687983056%5D%2C%20%5B127.05093367799998%2C%2037.686160674%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%200%2C%20%22SHAPE_AREA%22%3A%200.00211%2C%20%22SHAPE_LEN%22%3A%200.239901%2C%20%22SIG_CD%22%3A%20%2211320%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dobong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88359471499996%2C%2037.591855451000015%5D%2C%20%5B126.88502350199997%2C%2037.593642095%5D%2C%20%5B126.88720157499995%2C%2037.59368140999999%5D%2C%20%5B126.88561580199996%2C%2037.591502085%5D%2C%20%5B126.88577464100001%2C%2037.589569513000015%5D%2C%20%5B126.88717970799996%2C%2037.58853613399998%5D%2C%20%5B126.88927454400005%2C%2037.58883090799998%5D%2C%20%5B126.89225666100003%2C%2037.58852532399999%5D%2C%20%5B126.89334847199996%2C%2037.589070849%5D%2C%20%5B126.89687879200005%2C%2037.588570065%5D%2C%20%5B126.89810143800003%2C%2037.58954265199998%5D%2C%20%5B126.89966320099995%2C%2037.58982035399998%5D%2C%20%5B126.899585279%2C%2037.591694511000014%5D%2C%20%5B126.89898042499999%2C%2037.592683197999975%5D%2C%20%5B126.90037465800003%2C%2037.59421526400001%5D%2C%20%5B126.90184962900003%2C%2037.59514499199997%5D%2C%20%5B126.90103258399995%2C%2037.597339986%5D%2C%20%5B126.901343312%2C%2037.59936941699999%5D%2C%20%5B126.90000258099997%2C%2037.602042943000015%5D%2C%20%5B126.90017282600002%2C%2037.60311939799999%5D%2C%20%5B126.90212880000001%2C%2037.60374458000001%5D%2C%20%5B126.90062064300002%2C%2037.60932366100002%5D%2C%20%5B126.90032356100005%2C%2037.611195898%5D%2C%20%5B126.90175330700004%2C%2037.61407416499998%5D%2C%20%5B126.901843654%2C%2037.615756572%5D%2C%20%5B126.90314656400005%2C%2037.61730752099999%5D%2C%20%5B126.90338372099995%2C%2037.61884112299998%5D%2C%20%5B126.90525097600005%2C%2037.61907907300002%5D%2C%20%5B126.90563169100005%2C%2037.620708664%5D%2C%20%5B126.90713704200004%2C%2037.62194588400001%5D%2C%20%5B126.90716203600005%2C%2037.62331746299998%5D%2C%20%5B126.90664762899996%2C%2037.62449994799999%5D%2C%20%5B126.908630388%2C%2037.62627061799998%5D%2C%20%5B126.90879064700005%2C%2037.629332764000026%5D%2C%20%5B126.90731509700004%2C%2037.63087173100001%5D%2C%20%5B126.90677806899998%2C%2037.63274096499998%5D%2C%20%5B126.90711683699999%2C%2037.63340278800001%5D%2C%20%5B126.910875079%2C%2037.635322556%5D%2C%20%5B126.91123094800002%2C%2037.635911695%5D%2C%20%5B126.91011226199998%2C%2037.638515595%5D%2C%20%5B126.91081456899997%2C%2037.64003255799997%5D%2C%20%5B126.91230017800001%2C%2037.64432203199999%5D%2C%20%5B126.91016072699995%2C%2037.645083612%5D%2C%20%5B126.90772442000002%2C%2037.64629642699998%5D%2C%20%5B126.90843309399997%2C%2037.647262366%5D%2C%20%5B126.90974709700004%2C%2037.646882914%5D%2C%20%5B126.91234747099998%2C%2037.645182541%5D%2C%20%5B126.913740469%2C%2037.64476066499998%5D%2C%20%5B126.92414365900004%2C%2037.64611043100001%5D%2C%20%5B126.92715483799998%2C%2037.64810585499998%5D%2C%20%5B126.92972022900005%2C%2037.65004959800001%5D%2C%20%5B126.93224238100004%2C%2037.65038895999999%5D%2C%20%5B126.935727004%2C%2037.65120514099999%5D%2C%20%5B126.93718405799996%2C%2037.652317728000014%5D%2C%20%5B126.93847852299996%2C%2037.654773405000014%5D%2C%20%5B126.94014671100001%2C%2037.65663874799998%5D%2C%20%5B126.94211837499995%2C%2037.65772510800002%5D%2C%20%5B126.94441488899997%2C%2037.65816625799999%5D%2C%20%5B126.94758363699998%2C%2037.659217403000014%5D%2C%20%5B126.94790109500002%2C%2037.65712764900002%5D%2C%20%5B126.94922168000005%2C%2037.656772183999976%5D%2C%20%5B126.94980975299995%2C%2037.655913363000025%5D%2C%20%5B126.95139494700004%2C%2037.654951117999985%5D%2C%20%5B126.95437039%2C%2037.654596949999984%5D%2C%20%5B126.95712830900004%2C%2037.65283925900002%5D%2C%20%5B126.95781039600001%2C%2037.64981070699997%5D%2C%20%5B126.95786031099999%2C%2037.64804963099999%5D%2C%20%5B126.95904808800003%2C%2037.64678533599999%5D%2C%20%5B126.95892784399996%2C%2037.64476220400002%5D%2C%20%5B126.95928919999994%2C%2037.642134535000025%5D%2C%20%5B126.959992629%2C%2037.64146349599997%5D%2C%20%5B126.96162582500006%2C%2037.63711909400001%5D%2C%20%5B126.96256224599995%2C%2037.63582057799999%5D%2C%20%5B126.96335032800005%2C%2037.633246824000025%5D%2C%20%5B126.96164091799994%2C%2037.631889602%5D%2C%20%5B126.95984451699996%2C%2037.629762777999986%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%201%2C%20%22SHAPE_AREA%22%3A%200.003041%2C%20%22SHAPE_LEN%22%3A%200.327143%2C%20%22SIG_CD%22%3A%20%2211380%22%2C%20%22SIG_ENG_NM%22%3A%20%22Eunpyeong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%202%2C%20%22SHAPE_AREA%22%3A%200.001453%2C%20%22SHAPE_LEN%22%3A%200.182837%2C%20%22SIG_CD%22%3A%20%2211230%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongdaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%203%2C%20%22SHAPE_AREA%22%3A%200.00167%2C%20%22SHAPE_LEN%22%3A%200.237796%2C%20%22SIG_CD%22%3A%20%2211590%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongjak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92829901899995%2C%2037.449360259%5D%2C%20%5B126.926538273%2C%2037.44839745899998%5D%2C%20%5B126.92500225699996%2C%2037.44678811799997%5D%2C%20%5B126.92327265100005%2C%2037.44577942699999%5D%2C%20%5B126.92278057800002%2C%2037.444047814999976%5D%2C%20%5B126.92118598900004%2C%2037.442552917%5D%2C%20%5B126.92028939299996%2C%2037.44047332299999%5D%2C%20%5B126.91927808399998%2C%2037.43985946599997%5D%2C%20%5B126.91613959300003%2C%2037.440060542000026%5D%2C%20%5B126.91232267199996%2C%2037.438587175%5D%2C%20%5B126.911215782%2C%2037.43720903600001%5D%2C%20%5B126.91134523699998%2C%2037.436179159%5D%2C%20%5B126.90940698500003%2C%2037.433868523%5D%2C%20%5B126.90726602899997%2C%2037.43352405000002%5D%2C%20%5B126.90611662000003%2C%2037.43400280399999%5D%2C%20%5B126.90299885399997%2C%2037.43407347499999%5D%2C%20%5B126.902763%2C%2037.43586855199999%5D%2C%20%5B126.90086236399998%2C%2037.43699089%5D%2C%20%5B126.89877013700004%2C%2037.439283811999985%5D%2C%20%5B126.89991895399999%2C%2037.439562146000014%5D%2C%20%5B126.897287549%2C%2037.445443447%5D%2C%20%5B126.89580121999995%2C%2037.445640562999984%5D%2C%20%5B126.89466890899996%2C%2037.446706108%5D%2C%20%5B126.89476993300002%2C%2037.447710443%5D%2C%20%5B126.89592769599994%2C%2037.44842231299998%5D%2C%20%5B126.89400040400005%2C%2037.452723881%5D%2C%20%5B126.89277619200004%2C%2037.452046575%5D%2C%20%5B126.88951816999997%2C%2037.45270753599999%5D%2C%20%5B126.889280761%2C%2037.45448889199997%5D%2C%20%5B126.88641922299996%2C%2037.456180116999974%5D%2C%20%5B126.88589653600002%2C%2037.457514893999985%5D%2C%20%5B126.88630108799998%2C%2037.459120703%5D%2C%20%5B126.885364977%2C%2037.459868728%5D%2C%20%5B126.886087029%2C%2037.460865333000015%5D%2C%20%5B126.88889275600002%2C%2037.460953249%5D%2C%20%5B126.88713625100002%2C%2037.462909712%5D%2C%20%5B126.88515609299998%2C%2037.462473671%5D%2C%20%5B126.88433205199999%2C%2037.46272970799998%5D%2C%20%5B126.88276046399994%2C%2037.464396598%5D%2C%20%5B126.88463173000002%2C%2037.46601745999999%5D%2C%20%5B126.88302833499995%2C%2037.467069503%5D%2C%20%5B126.881647767%2C%2037.468466234%5D%2C%20%5B126.88161172399998%2C%2037.469425546000025%5D%2C%20%5B126.878031942%2C%2037.47383573500002%5D%2C%20%5B126.87600453499999%2C%2037.477111283%5D%2C%20%5B126.87178398799995%2C%2037.48527755399999%5D%2C%20%5B126.87315863100002%2C%2037.486160427000016%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%204%2C%20%22SHAPE_AREA%22%3A%200.001325%2C%20%22SHAPE_LEN%22%3A%200.211649%2C%20%22SIG_CD%22%3A%20%2211545%22%2C%20%22SIG_ENG_NM%22%3A%20%22Geumcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.88272419199996%2C%2037.515768666999975%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.87599162499998%2C%2037.48691906800002%5D%2C%20%5B126.87680700600004%2C%2037.488578991%5D%2C%20%5B126.87583373899997%2C%2037.489035374000025%5D%2C%20%5B126.87309748400003%2C%2037.488261317000024%5D%2C%20%5B126.87245908399996%2C%2037.489543242000025%5D%2C%20%5B126.87024621900002%2C%2037.48960327200001%5D%2C%20%5B126.86936249799999%2C%2037.49164208000002%5D%2C%20%5B126.86972553299995%2C%2037.49410186699998%5D%2C%20%5B126.86836486200002%2C%2037.49511503000002%5D%2C%20%5B126.86738599800003%2C%2037.49308047%5D%2C%20%5B126.8663199%2C%2037.492637301%5D%2C%20%5B126.86541991000001%2C%2037.49151324600001%5D%2C%20%5B126.86260271100002%2C%2037.49076571799998%5D%2C%20%5B126.86139963599999%2C%2037.489972641%5D%2C%20%5B126.85798566000005%2C%2037.48609330599999%5D%2C%20%5B126.855521178%2C%2037.485415111%5D%2C%20%5B126.853961895%2C%2037.48297475200002%5D%2C%20%5B126.85271384700002%2C%2037.48182609899999%5D%2C%20%5B126.85043164800004%2C%2037.48157525800002%5D%2C%20%5B126.84848405499997%2C%2037.482095259%5D%2C%20%5B126.84642145600003%2C%2037.48150571299999%5D%2C%20%5B126.84594900499997%2C%2037.480323644%5D%2C%20%5B126.84537853999996%2C%2037.473821137000016%5D%2C%20%5B126.842799474%2C%2037.47498250699999%5D%2C%20%5B126.84104901299997%2C%2037.474656347%5D%2C%20%5B126.83831728799998%2C%2037.47540135999998%5D%2C%20%5B126.83484912899996%2C%2037.47443089%5D%2C%20%5B126.83488769799999%2C%2037.47575841499997%5D%2C%20%5B126.83349606700006%2C%2037.47716231800001%5D%2C%20%5B126.83176399199999%2C%2037.47765735500002%5D%2C%20%5B126.82788973599997%2C%2037.47599654700002%5D%2C%20%5B126.82435334800005%2C%2037.476259733%5D%2C%20%5B126.82212405999996%2C%2037.47575425399998%5D%2C%20%5B126.82116790299995%2C%2037.476307167000016%5D%2C%20%5B126.81909562600003%2C%2037.476138418%5D%2C%20%5B126.818336391%2C%2037.47530395899997%5D%2C%20%5B126.818647371%2C%2037.474085166%5D%2C%20%5B126.81708964200004%2C%2037.47329169699998%5D%2C%20%5B126.81657626699996%2C%2037.473998096%5D%2C%20%5B126.81464318099995%2C%2037.47465330900002%5D%2C%20%5B126.81531048900001%2C%2037.47636645199998%5D%2C%20%5B126.81722552199994%2C%2037.47814126600002%5D%2C%20%5B126.81827151499999%2C%2037.478170568%5D%2C%20%5B126.81901268700005%2C%2037.47916943500002%5D%2C%20%5B126.81999807199998%2C%2037.48164884%5D%2C%20%5B126.81959813599997%2C%2037.48244711299998%5D%2C%20%5B126.81934165300004%2C%2037.48551259499999%5D%2C%20%5B126.82130494499995%2C%2037.48618739800003%5D%2C%20%5B126.82351963899998%2C%2037.487744236000026%5D%2C%20%5B126.82277597899997%2C%2037.48998772700003%5D%2C%20%5B126.81790518800005%2C%2037.49154342000003%5D%2C%20%5B126.81459113400001%2C%2037.49320034200002%5D%2C%20%5B126.81436892199997%2C%2037.49443227400002%5D%2C%20%5B126.81302860799997%2C%2037.496407777%5D%2C%20%5B126.81415555599995%2C%2037.49813429599999%5D%2C%20%5B126.81611235499997%2C%2037.49765224499998%5D%2C%20%5B126.818224268%2C%2037.49886467499999%5D%2C%20%5B126.81965596999999%2C%2037.499219118999974%5D%2C%20%5B126.81943123600001%2C%2037.50079738%5D%2C%20%5B126.82158116200003%2C%2037.50216396000002%5D%2C%20%5B126.82184366299998%2C%2037.504545139000015%5D%2C%20%5B126.82256759899997%2C%2037.505692083999975%5D%2C%20%5B126.82223955400002%2C%2037.50769455599999%5D%2C%20%5B126.82615287800002%2C%2037.50872472999998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.837508791%2C%2037.50328697200001%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.84076167%2C%2037.50604941900002%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84873119999997%2C%2037.508888773000024%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85562926600005%2C%2037.50916079199999%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.858345407%2C%2037.509923406999974%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%205%2C%20%22SHAPE_AREA%22%3A%200.002047%2C%20%22SHAPE_LEN%22%3A%200.347568%2C%20%22SIG_CD%22%3A%20%2211530%22%2C%20%22SIG_ENG_NM%22%3A%20%22Guro-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.96146158%2C%2037.62996095%5D%2C%20%5B126.96378891899997%2C%2037.62979475499998%5D%2C%20%5B126.96524997699998%2C%2037.63076970399999%5D%2C%20%5B126.968256763%2C%2037.63073174200002%5D%2C%20%5B126.97134811%2C%2037.63162062700002%5D%2C%20%5B126.973049887%2C%2037.632376954999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%206%2C%20%22SHAPE_AREA%22%3A%200.002448%2C%20%22SHAPE_LEN%22%3A%200.2901%2C%20%22SIG_CD%22%3A%20%2211110%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jongno-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98421633700002%2C%2037.63634100199999%5D%2C%20%5B126.98510166300002%2C%2037.637318127000015%5D%2C%20%5B126.98569312400002%2C%2037.640048433%5D%2C%20%5B126.98528469200005%2C%2037.641354295999975%5D%2C%20%5B126.98427374300002%2C%2037.64164886999998%5D%2C%20%5B126.98326407499997%2C%2037.643687516%5D%2C%20%5B126.98512211000002%2C%2037.64573277%5D%2C%20%5B126.98462050299997%2C%2037.647616578%5D%2C%20%5B126.98399871900006%2C%2037.64818593799998%5D%2C%20%5B126.98395619999997%2C%2037.649613708%5D%2C%20%5B126.98251481800003%2C%2037.650544008%5D%2C%20%5B126.98169600799997%2C%2037.652320175%5D%2C%20%5B126.98106737800003%2C%2037.652716521%5D%2C%20%5B126.97996370600004%2C%2037.65481924699998%5D%2C%20%5B126.97967724299997%2C%2037.65604285799998%5D%2C%20%5B126.98302857199997%2C%2037.65700805%5D%2C%20%5B126.98603858499996%2C%2037.65894778400002%5D%2C%20%5B126.987294798%2C%2037.660604260000014%5D%2C%20%5B126.98782001899997%2C%2037.66359695900002%5D%2C%20%5B126.98828632200002%2C%2037.66439418800002%5D%2C%20%5B126.99001520599995%2C%2037.66450267%5D%2C%20%5B126.99224251800001%2C%2037.66523496299999%5D%2C%20%5B126.99414712600003%2C%2037.667033076%5D%2C%20%5B126.99357858400003%2C%2037.66780672099998%5D%2C%20%5B126.99437535000004%2C%2037.66956914000002%5D%2C%20%5B126.99382043900005%2C%2037.670199244%5D%2C%20%5B126.99409198599994%2C%2037.672763257999975%5D%2C%20%5B126.99356165699999%2C%2037.67410523500001%5D%2C%20%5B126.99399960799997%2C%2037.674909437999986%5D%2C%20%5B126.99321780000002%2C%2037.675722317%5D%2C%20%5B126.99326569200002%2C%2037.67766322599999%5D%2C%20%5B126.99221614400005%2C%2037.67962970100001%5D%2C%20%5B126.99409428599995%2C%2037.68032697899997%5D%2C%20%5B126.99475084200003%2C%2037.681246918%5D%2C%20%5B126.99675604100003%2C%2037.68239214%5D%2C%20%5B126.997333905%2C%2037.68337292299998%5D%2C%20%5B127.00170893100005%2C%2037.68426214499999%5D%2C%20%5B127.003605311%2C%2037.684165339%5D%2C%20%5B127.00454835300002%2C%2037.68501282099999%5D%2C%20%5B127.00607232000004%2C%2037.684984327%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%207%2C%20%22SHAPE_AREA%22%3A%200.002412%2C%20%22SHAPE_LEN%22%3A%200.267441%2C%20%22SIG_CD%22%3A%20%2211305%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10914571%2C%2037.62052080400002%5D%2C%20%5B127.11053916100002%2C%2037.62105968399999%5D%2C%20%5B127.11216023500003%2C%2037.620311775%5D%2C%20%5B127.11503737099997%2C%2037.61954149500002%5D%2C%20%5B127.11716289399999%2C%2037.61789223%5D%2C%20%5B127.11668740899995%2C%2037.61601964499999%5D%2C%20%5B127.11749802600002%2C%2037.61177613000001%5D%2C%20%5B127.11677896900005%2C%2037.610134802%5D%2C%20%5B127.116712333%2C%2037.60885401899998%5D%2C%20%5B127.11849163500005%2C%2037.60761747%5D%2C%20%5B127.118075341%2C%2037.606595825%5D%2C%20%5B127.118061087%2C%2037.604606131000025%5D%2C%20%5B127.115739211%2C%2037.60211296%5D%2C%20%5B127.11408091199996%2C%2037.599527276%5D%2C%20%5B127.11689451500001%2C%2037.595503880000024%5D%2C%20%5B127.11668170400003%2C%2037.594023413%5D%2C%20%5B127.11335498100004%2C%2037.59326654199998%5D%2C%20%5B127.110693577%2C%2037.58916251699998%5D%2C%20%5B127.11000566899997%2C%2037.586023006%5D%2C%20%5B127.10898535199999%2C%2037.58338535199999%5D%2C%20%5B127.10345938600005%2C%2037.58060086099999%5D%2C%20%5B127.10304415999997%2C%2037.578912213000024%5D%2C%20%5B127.10116216200004%2C%2037.57607578599999%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%208%2C%20%22SHAPE_AREA%22%3A%200.001893%2C%20%22SHAPE_LEN%22%3A%200.184716%2C%20%22SIG_CD%22%3A%20%2211260%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jungnang-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.12187312100002%2C%2037.46514826399999%5D%2C%20%5B127.12145320599996%2C%2037.46438168499998%5D%2C%20%5B127.11748896200004%2C%2037.462210077%5D%2C%20%5B127.11700353900005%2C%2037.46149384900002%5D%2C%20%5B127.116910318%2C%2037.458650388000024%5D%2C%20%5B127.11590693999995%2C%2037.45860154000002%5D%2C%20%5B127.11328498199998%2C%2037.460064267%5D%2C%20%5B127.11298953899995%2C%2037.46114412899999%5D%2C%20%5B127.11183504999997%2C%2037.461654001%5D%2C%20%5B127.10648246400001%2C%2037.46243379200001%5D%2C%20%5B127.10435658899996%2C%2037.46218425799998%5D%2C%20%5B127.10479563499996%2C%2037.46116971399999%5D%2C%20%5B127.10395681800003%2C%2037.46006054700001%5D%2C%20%5B127.10139603200003%2C%2037.45898237099999%5D%2C%20%5B127.10129219299995%2C%2037.45825120400002%5D%2C%20%5B127.09975283000006%2C%2037.457364144%5D%2C%20%5B127.099109139%2C%2037.45622823999997%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%209%2C%20%22SHAPE_AREA%22%3A%200.004027%2C%20%22SHAPE_LEN%22%3A%200.348412%2C%20%22SIG_CD%22%3A%20%2211680%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangnam-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.81658266%2C%2037.540568317%5D%2C%20%5B126.81234887999994%2C%2037.54074469400001%5D%2C%20%5B126.81060934799996%2C%2037.54150444200002%5D%2C%20%5B126.80725610699994%2C%2037.54362225199998%5D%2C%20%5B126.80186598399996%2C%2037.542723611999975%5D%2C%20%5B126.80048554400003%2C%2037.54177814600001%5D%2C%20%5B126.80040658300004%2C%2037.540804252999976%5D%2C%20%5B126.79887833%2C%2037.5402999%5D%2C%20%5B126.79875436099996%2C%2037.539406192%5D%2C%20%5B126.79949804199998%2C%2037.537775685999975%5D%2C%20%5B126.79782849100002%2C%2037.537056888999984%5D%2C%20%5B126.79605918200002%2C%2037.53675834400002%5D%2C%20%5B126.79481063799994%2C%2037.535972963%5D%2C%20%5B126.79397960699998%2C%2037.537369168%5D%2C%20%5B126.79390197299995%2C%2037.539558371%5D%2C%20%5B126.79496077700003%2C%2037.541381135999984%5D%2C%20%5B126.79182670499995%2C%2037.54191169%5D%2C%20%5B126.79159995700002%2C%2037.54307308199998%5D%2C%20%5B126.79077654100001%2C%2037.543847862%5D%2C%20%5B126.78872746499997%2C%2037.54482803399998%5D%2C%20%5B126.78727682700003%2C%2037.54608495799999%5D%2C%20%5B126.78332478000004%2C%2037.54603042999997%5D%2C%20%5B126.77875616300003%2C%2037.546729674%5D%2C%20%5B126.77757101300006%2C%2037.54672980700002%5D%2C%20%5B126.77675212199995%2C%2037.548221075000015%5D%2C%20%5B126.775301227%2C%2037.548988871%5D%2C%20%5B126.77258571799996%2C%2037.548773638%5D%2C%20%5B126.77169496199997%2C%2037.548329146000015%5D%2C%20%5B126.76992633199995%2C%2037.550196289999974%5D%2C%20%5B126.769793875%2C%2037.551086956%5D%2C%20%5B126.767468484%2C%2037.551972233000015%5D%2C%20%5B126.76742527299996%2C%2037.554237899999976%5D%2C%20%5B126.76458060899995%2C%2037.555466594999984%5D%2C%20%5B126.76720647299999%2C%2037.55666916000001%5D%2C%20%5B126.76988573200003%2C%2037.55724517599998%5D%2C%20%5B126.772397404%2C%2037.55699389%5D%2C%20%5B126.773200245%2C%2037.557536051%5D%2C%20%5B126.77183969700002%2C%2037.55861259400001%5D%2C%20%5B126.77320943200004%2C%2037.55929739800001%5D%2C%20%5B126.77612545800002%2C%2037.56187228599998%5D%2C%20%5B126.77795148799999%2C%2037.56016005399999%5D%2C%20%5B126.7771186%2C%2037.563728853999976%5D%2C%20%5B126.77515191199996%2C%2037.565895781999984%5D%2C%20%5B126.77478348299996%2C%2037.56772791100002%5D%2C%20%5B126.77635320499996%2C%2037.567107521000025%5D%2C%20%5B126.777826946%2C%2037.567006467%5D%2C%20%5B126.78025671600005%2C%2037.567453419%5D%2C%20%5B126.780491568%2C%2037.56829614100002%5D%2C%20%5B126.781642583%2C%2037.568909259%5D%2C%20%5B126.78133354800002%2C%2037.570185116%5D%2C%20%5B126.782647215%2C%2037.57055544799999%5D%2C%20%5B126.78191796099998%2C%2037.57181951000001%5D%2C%20%5B126.78242742400005%2C%2037.57361885900002%5D%2C%20%5B126.78526689%2C%2037.57487061799998%5D%2C%20%5B126.787630605%2C%2037.57558866%5D%2C%20%5B126.78935921899995%2C%2037.57772807700002%5D%2C%20%5B126.79058199500003%2C%2037.577776819%5D%2C%20%5B126.79074497399995%2C%2037.58061836799999%5D%2C%20%5B126.79228461599996%2C%2037.580056910999986%5D%2C%20%5B126.79275533099997%2C%2037.57684457%5D%2C%20%5B126.79323752200003%2C%2037.57689948500001%5D%2C%20%5B126.79288111400001%2C%2037.58022635499998%5D%2C%20%5B126.79390501399996%2C%2037.582257949%5D%2C%20%5B126.79411019999998%2C%2037.58425737099998%5D%2C%20%5B126.79555674699998%2C%2037.58515790000001%5D%2C%20%5B126.79569588599998%2C%2037.583367872%5D%2C%20%5B126.79711367100003%2C%2037.58427787900001%5D%2C%20%5B126.79670218499996%2C%2037.58491212500002%5D%2C%20%5B126.79878484200003%2C%2037.58807839000002%5D%2C%20%5B126.80075747499995%2C%2037.587841243000014%5D%2C%20%5B126.80053959500003%2C%2037.590315551%5D%2C%20%5B126.798817848%2C%2037.59132633199999%5D%2C%20%5B126.79867010299995%2C%2037.59366814100002%5D%2C%20%5B126.79745919100003%2C%2037.595248595999976%5D%2C%20%5B126.79708180299997%2C%2037.59773291099998%5D%2C%20%5B126.79756259099997%2C%2037.59793496100002%5D%2C%20%5B126.79787158299996%2C%2037.599898203%5D%2C%20%5B126.799935054%2C%2037.601445969%5D%2C%20%5B126.79994633199999%2C%2037.602546226000015%5D%2C%20%5B126.80258612299997%2C%2037.605033999%5D%2C%20%5B126.80587186000002%2C%2037.60258296500001%5D%2C%20%5B126.808249756%2C%2037.601211834000026%5D%2C%20%5B126.81138204800004%2C%2037.59896582800002%5D%2C%20%5B126.81396850600004%2C%2037.59762536400001%5D%2C%20%5B126.816573747%2C%2037.595688145999986%5D%2C%20%5B126.81754378000005%2C%2037.59533660400001%5D%2C%20%5B126.81932148199996%2C%2037.592855004%5D%2C%20%5B126.82783387699999%2C%2037.58748727300002%5D%2C%20%5B126.83178168200004%2C%2037.585846134%5D%2C%20%5B126.837170002%2C%2037.582568493%5D%2C%20%5B126.83811851799999%2C%2037.581798564999986%5D%2C%20%5B126.84326033599996%2C%2037.578886398%5D%2C%20%5B126.84840263399997%2C%2037.57516434500002%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2010%2C%20%22SHAPE_AREA%22%3A%200.004227%2C%20%22SHAPE_LEN%22%3A%200.435694%2C%20%22SIG_CD%22%3A%20%2211500%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangseo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2011%2C%20%22SHAPE_AREA%22%3A%200.001017%2C%20%22SHAPE_LEN%22%3A%200.191242%2C%20%22SIG_CD%22%3A%20%2211140%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jung-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.11428417299999%2C%2037.554386276%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11566284000003%2C%2037.557702898%5D%2C%20%5B127.11735874600004%2C%2037.559467202%5D%2C%20%5B127.12011629799997%2C%2037.56120324699998%5D%2C%20%5B127.12297571600004%2C%2037.56349386%5D%2C%20%5B127.12862746099995%2C%2037.56616545000003%5D%2C%20%5B127.134283364%2C%2037.567961086000025%5D%2C%20%5B127.13768321700002%2C%2037.56843244200002%5D%2C%20%5B127.14896618800003%2C%2037.568444026%5D%2C%20%5B127.154998245%2C%2037.57204888699999%5D%2C%20%5B127.15691686699995%2C%2037.572958835%5D%2C%20%5B127.16257241200003%2C%2037.576732698%5D%2C%20%5B127.16676596299999%2C%2037.57897662%5D%2C%20%5B127.17026691299998%2C%2037.57901745999999%5D%2C%20%5B127.17185117300005%2C%2037.57926277399997%5D%2C%20%5B127.17521472800001%2C%2037.58046697200001%5D%2C%20%5B127.17734763199996%2C%2037.581575015%5D%2C%20%5B127.17701410100005%2C%2037.579511809%5D%2C%20%5B127.17550670900005%2C%2037.578472404000024%5D%2C%20%5B127.17534214%2C%2037.57723767800002%5D%2C%20%5B127.17569917399999%2C%2037.57490180399998%5D%2C%20%5B127.17789114499999%2C%2037.57187252799997%5D%2C%20%5B127.17922984100005%2C%2037.56892072800002%5D%2C%20%5B127.17959913300001%2C%2037.56549991899999%5D%2C%20%5B127.181200053%2C%2037.56301996299999%5D%2C%20%5B127.18200949499999%2C%2037.56099529800002%5D%2C%20%5B127.181566868%2C%2037.557559535%5D%2C%20%5B127.18169641500003%2C%2037.55600353400001%5D%2C%20%5B127.18135382599996%2C%2037.552973123000015%5D%2C%20%5B127.18294495500004%2C%2037.55177914199999%5D%2C%20%5B127.183131151%2C%2037.551111615000025%5D%2C%20%5B127.18268913400004%2C%2037.54791615900001%5D%2C%20%5B127.182837425%2C%2037.54639986199999%5D%2C%20%5B127.17934642900002%2C%2037.54657307399998%5D%2C%20%5B127.17573322400006%2C%2037.545213538999974%5D%2C%20%5B127.17392236499995%2C%2037.545588527%5D%2C%20%5B127.17212391600003%2C%2037.545541722999985%5D%2C%20%5B127.16978076600003%2C%2037.544813007000016%5D%2C%20%5B127.16699588400002%2C%2037.54521138699999%5D%2C%20%5B127.16665500399995%2C%2037.54426145999997%5D%2C%20%5B127.16317850999997%2C%2037.544997455999976%5D%2C%20%5B127.16032596900004%2C%2037.541629215%5D%2C%20%5B127.15766934199996%2C%2037.53927536100002%5D%2C%20%5B127.15696225299996%2C%2037.537974744999985%5D%2C%20%5B127.15527845999998%2C%2037.53622368600003%5D%2C%20%5B127.15358375799997%2C%2037.533672017000015%5D%2C%20%5B127.15395052600002%2C%2037.531926252%5D%2C%20%5B127.15270367899996%2C%2037.528535456999975%5D%2C%20%5B127.15041863099998%2C%2037.52637816499998%5D%2C%20%5B127.14781481299997%2C%2037.52215484599998%5D%2C%20%5B127.145682889%2C%2037.52193569399998%5D%2C%20%5B127.144801873%2C%2037.519630431999985%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2012%2C%20%22SHAPE_AREA%22%3A%200.002504%2C%20%22SHAPE_LEN%22%3A%200.242596%2C%20%22SIG_CD%22%3A%20%2211740%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11411142500003%2C%2037.55409376599999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.10168325400002%2C%2037.57240515400002%5D%2C%20%5B127.103148411%2C%2037.57228406899998%5D%2C%20%5B127.10424898199994%2C%2037.571392263%5D%2C%20%5B127.10347415499996%2C%2037.57005447199998%5D%2C%20%5B127.10240523599998%2C%2037.56564081%5D%2C%20%5B127.10225269499995%2C%2037.564314869999976%5D%2C%20%5B127.10122363599999%2C%2037.56158417300003%5D%2C%20%5B127.10187808800003%2C%2037.559498245999976%5D%2C%20%5B127.10429532800003%2C%2037.55756740800001%5D%2C%20%5B127.10494775899997%2C%2037.55642552299997%5D%2C%20%5B127.10641446099999%2C%2037.55646542699998%5D%2C%20%5B127.10753783500002%2C%2037.55761021400002%5D%2C%20%5B127.10954748999995%2C%2037.558557784000016%5D%2C%20%5B127.11036889499997%2C%2037.55825685100001%5D%2C%20%5B127.11231650399998%2C%2037.55900405400001%5D%2C%20%5B127.11388262800006%2C%2037.55843289500001%5D%2C%20%5B127.11333075200002%2C%2037.55683318299998%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2013%2C%20%22SHAPE_AREA%22%3A%200.001737%2C%20%22SHAPE_LEN%22%3A%200.186732%2C%20%22SIG_CD%22%3A%20%2211215%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwangjin-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85364710199997%2C%2037.573804395000025%5D%2C%20%5B126.85930276099998%2C%2037.57492123999998%5D%2C%20%5B126.85931123%2C%2037.575246692%5D%2C%20%5B126.86495888599995%2C%2037.57747814700002%5D%2C%20%5B126.868396404%2C%2037.577526395%5D%2C%20%5B126.870619272%2C%2037.578013113%5D%2C%20%5B126.87393583999994%2C%2037.57828489799999%5D%2C%20%5B126.87628222700005%2C%2037.57818935199998%5D%2C%20%5B126.87765407400002%2C%2037.57975304299998%5D%2C%20%5B126.87668444099995%2C%2037.581263981%5D%2C%20%5B126.87707186%2C%2037.58221029200001%5D%2C%20%5B126.87710903799996%2C%2037.58483216500002%5D%2C%20%5B126.877564388%2C%2037.586252527%5D%2C%20%5B126.87915565399999%2C%2037.58677640399998%5D%2C%20%5B126.88036942999997%2C%2037.58950964899998%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2014%2C%20%22SHAPE_AREA%22%3A%200.002415%2C%20%22SHAPE_LEN%22%3A%200.289336%2C%20%22SIG_CD%22%3A%20%2211440%22%2C%20%22SIG_ENG_NM%22%3A%20%22Mapo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98662702599995%2C%2037.457216786%5D%2C%20%5B126.98290313899997%2C%2037.456831314%5D%2C%20%5B126.98241282200001%2C%2037.455895618%5D%2C%20%5B126.97849284100005%2C%2037.455739153000025%5D%2C%20%5B126.97777249%2C%2037.455203476%5D%2C%20%5B126.97459526800003%2C%2037.45441720500003%5D%2C%20%5B126.97285906000002%2C%2037.45238675000002%5D%2C%20%5B126.97176838200005%2C%2037.45174170500002%5D%2C%20%5B126.97058548899997%2C%2037.449457128%5D%2C%20%5B126.96767973600004%2C%2037.44838496%5D%2C%20%5B126.964307298%2C%2037.446271943%5D%2C%20%5B126.96395069300002%2C%2037.44521969200002%5D%2C%20%5B126.96443595200003%2C%2037.44431798300002%5D%2C%20%5B126.96464967600002%2C%2037.44204566000002%5D%2C%20%5B126.96295770799998%2C%2037.44028699400002%5D%2C%20%5B126.96007066799996%2C%2037.44040704499997%5D%2C%20%5B126.95898017000002%2C%2037.43907666199999%5D%2C%20%5B126.95512268899995%2C%2037.43869914800001%5D%2C%20%5B126.95242507900002%2C%2037.43919711699999%5D%2C%20%5B126.95148094199999%2C%2037.43858365%5D%2C%20%5B126.94929766799999%2C%2037.43825732%5D%2C%20%5B126.94837678299996%2C%2037.43871966299997%5D%2C%20%5B126.94680271000004%2C%2037.43817427499999%5D%2C%20%5B126.94510345499998%2C%2037.43709568600002%5D%2C%20%5B126.94143729300004%2C%2037.437411664000024%5D%2C%20%5B126.940232358%2C%2037.435720186000026%5D%2C%20%5B126.93858713099996%2C%2037.43642808300001%5D%2C%20%5B126.93775941000001%2C%2037.437393122%5D%2C%20%5B126.93726096700004%2C%2037.439219657000024%5D%2C%20%5B126.93787838100002%2C%2037.44020405700002%5D%2C%20%5B126.93666192499995%2C%2037.44175181600002%5D%2C%20%5B126.93176733099995%2C%2037.444982075999974%5D%2C%20%5B126.93057246299998%2C%2037.445474494%5D%2C%20%5B126.93018248700002%2C%2037.44638402800001%5D%2C%20%5B126.93018641599997%2C%2037.448390587%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2015%2C%20%22SHAPE_AREA%22%3A%200.003012%2C%20%22SHAPE_LEN%22%3A%200.280092%2C%20%22SIG_CD%22%3A%20%2211620%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwanak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.093554412%2C%2037.45589802400002%5D%2C%20%5B127.09273821199997%2C%2037.453643511999985%5D%2C%20%5B127.09097172%2C%2037.45279201300002%5D%2C%20%5B127.09031546300002%2C%2037.45127626300001%5D%2C%20%5B127.088339345%2C%2037.44863720799998%5D%2C%20%5B127.08817389900003%2C%2037.44539478899998%5D%2C%20%5B127.08643383499998%2C%2037.444272048000016%5D%2C%20%5B127.08387430300002%2C%2037.44393121899998%5D%2C%20%5B127.08244417699996%2C%2037.441589282%5D%2C%20%5B127.08018384599995%2C%2037.441060924%5D%2C%20%5B127.07602324899995%2C%2037.4421491%5D%2C%20%5B127.07300687899999%2C%2037.442027784%5D%2C%20%5B127.07163946599997%2C%2037.441534323999974%5D%2C%20%5B127.07202035099999%2C%2037.43886650600001%5D%2C%20%5B127.07377263199999%2C%2037.4377991%5D%2C%20%5B127.07303654400005%2C%2037.43641643799998%5D%2C%20%5B127.07145178799999%2C%2037.43587310999999%5D%2C%20%5B127.07153192800001%2C%2037.434126424%5D%2C%20%5B127.07064098%2C%2037.432054365%5D%2C%20%5B127.07121871100003%2C%2037.430827679%5D%2C%20%5B127.06828150800004%2C%2037.43068911%5D%2C%20%5B127.06673287599995%2C%2037.43013986099999%5D%2C%20%5B127.06569412299996%2C%2037.42899537400001%5D%2C%20%5B127.06332888700001%2C%2037.429761353%5D%2C%20%5B127.06115775299997%2C%2037.429994996%5D%2C%20%5B127.05997391899996%2C%2037.429578751%5D%2C%20%5B127.05808265300004%2C%2037.430020472000024%5D%2C%20%5B127.05369652000002%2C%2037.428984660000026%5D%2C%20%5B127.05218338899999%2C%2037.42834757000003%5D%2C%20%5B127.04959235199999%2C%2037.430279794%5D%2C%20%5B127.04738197699999%2C%2037.43071133900003%5D%2C%20%5B127.04723810300004%2C%2037.43204956%5D%2C%20%5B127.04632535799999%2C%2037.43345562899998%5D%2C%20%5B127.04496506500004%2C%2037.43385551799997%5D%2C%20%5B127.044199833%2C%2037.435063092%5D%2C%20%5B127.04006758800006%2C%2037.438243274%5D%2C%20%5B127.03708267800005%2C%2037.43828114399997%5D%2C%20%5B127.03558860400005%2C%2037.43901011600002%5D%2C%20%5B127.03518600799998%2C%2037.440947884000025%5D%2C%20%5B127.03644468300001%2C%2037.44183865799999%5D%2C%20%5B127.03749597800004%2C%2037.443272594%5D%2C%20%5B127.038243717%2C%2037.44518611900003%5D%2C%20%5B127.03726765%2C%2037.44645425300001%5D%2C%20%5B127.03717336600005%2C%2037.44801560799999%5D%2C%20%5B127.03782636200003%2C%2037.44892774700003%5D%2C%20%5B127.03692812600002%2C%2037.45081401300001%5D%2C%20%5B127.03579206200004%2C%2037.452201682%5D%2C%20%5B127.03477066100004%2C%2037.45262298599999%5D%2C%20%5B127.03714478699999%2C%2037.45521278799998%5D%2C%20%5B127.03675713500002%2C%2037.45644861599999%5D%2C%20%5B127.03503165799998%2C%2037.45789780400003%5D%2C%20%5B127.03494060599996%2C%2037.46018196900002%5D%2C%20%5B127.033746501%2C%2037.461242513%5D%2C%20%5B127.03471186900003%2C%2037.46416056200002%5D%2C%20%5B127.03315839200002%2C%2037.465024089999986%5D%2C%20%5B127.03121260499995%2C%2037.46563217200003%5D%2C%20%5B127.02954249100003%2C%2037.46537973%5D%2C%20%5B127.02970909099997%2C%2037.463910751000014%5D%2C%20%5B127.02933072500002%2C%2037.46270655%5D%2C%20%5B127.02805724300003%2C%2037.46125024600002%5D%2C%20%5B127.02837573700003%2C%2037.459898619%5D%2C%20%5B127.02658642100005%2C%2037.459646512%5D%2C%20%5B127.02645936199997%2C%2037.45834107600001%5D%2C%20%5B127.02529273699997%2C%2037.457498327%5D%2C%20%5B127.02280887500001%2C%2037.457254868%5D%2C%20%5B127.02152010400005%2C%2037.456229913000016%5D%2C%20%5B127.01970823600004%2C%2037.45579098899998%5D%2C%20%5B127.01756149699997%2C%2037.45613975200001%5D%2C%20%5B127.01630291799995%2C%2037.455234355000016%5D%2C%20%5B127.01452250900002%2C%2037.45486602300002%5D%2C%20%5B127.01066817699996%2C%2037.456042489000026%5D%2C%20%5B127.008544057%2C%2037.45805628699998%5D%2C%20%5B127.00822678700001%2C%2037.459104508%5D%2C%20%5B127.00705377999998%2C%2037.460220048999986%5D%2C%20%5B127.00578293399997%2C%2037.46255271299998%5D%2C%20%5B127.00448335800002%2C%2037.464077615%5D%2C%20%5B127.00492006399998%2C%2037.46584683899999%5D%2C%20%5B127.00369035799997%2C%2037.46772444499999%5D%2C%20%5B127.00274926199995%2C%2037.467128762000016%5D%2C%20%5B126.99879405800004%2C%2037.46723823899998%5D%2C%20%5B126.99676674199998%2C%2037.467076564000024%5D%2C%20%5B126.996245907%2C%2037.46661254999998%5D%2C%20%5B126.99663361800003%2C%2037.46478279199999%5D%2C%20%5B126.997384828%2C%2037.46369182699999%5D%2C%20%5B126.996782414%2C%2037.461877791%5D%2C%20%5B126.993368691%2C%2037.46150029400002%5D%2C%20%5B126.99266931099999%2C%2037.46034838899999%5D%2C%20%5B126.99171279500001%2C%2037.46052342799999%5D%2C%20%5B126.99000889599995%2C%2037.45946741%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2016%2C%20%22SHAPE_AREA%22%3A%200.004776%2C%20%22SHAPE_LEN%22%3A%200.437399%2C%20%22SIG_CD%22%3A%20%2211650%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seocho-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.97782373300004%2C%2037.63391334300002%5D%2C%20%5B126.981433809%2C%2037.634832856%5D%2C%20%5B126.983724113%2C%2037.636460397%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2017%2C%20%22SHAPE_AREA%22%3A%200.002511%2C%20%22SHAPE_LEN%22%3A%200.318673%2C%20%22SIG_CD%22%3A%20%2211290%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05182101699995%2C%2037.687069296%5D%2C%20%5B127.05519755800003%2C%2037.689209908%5D%2C%20%5B127.05840713099997%2C%2037.689689869%5D%2C%20%5B127.05969812599994%2C%2037.690342471%5D%2C%20%5B127.06130232099997%2C%2037.69219614100001%5D%2C%20%5B127.06249161400001%2C%2037.69305951199999%5D%2C%20%5B127.06274278700005%2C%2037.69467682200002%5D%2C%20%5B127.06356049299995%2C%2037.69497166299999%5D%2C%20%5B127.06650615199999%2C%2037.694498565%5D%2C%20%5B127.068049664%2C%2037.694837018999976%5D%2C%20%5B127.069366791%2C%2037.69375161099998%5D%2C%20%5B127.07291609799995%2C%2037.69387412100002%5D%2C%20%5B127.07522005500005%2C%2037.695310147999976%5D%2C%20%5B127.07856303100004%2C%2037.696092556999986%5D%2C%20%5B127.08150813999998%2C%2037.69625423399998%5D%2C%20%5B127.084240346%2C%2037.694384093%5D%2C%20%5B127.08425235100003%2C%2037.69188345499998%5D%2C%20%5B127.08551096600002%2C%2037.69048708600002%5D%2C%20%5B127.08687032%2C%2037.69001125599999%5D%2C%20%5B127.09093103999999%2C%2037.68968092699998%5D%2C%20%5B127.09345258600001%2C%2037.690087849%5D%2C%20%5B127.096066758%2C%2037.687916323000024%5D%2C%20%5B127.09641572400005%2C%2037.685647404%5D%2C%20%5B127.09580503999996%2C%2037.684650288%5D%2C%20%5B127.09428565300004%2C%2037.683427962999986%5D%2C%20%5B127.09292189500002%2C%2037.68156068000002%5D%2C%20%5B127.09282796000002%2C%2037.68005578999998%5D%2C%20%5B127.09197763199995%2C%2037.67920024699998%5D%2C%20%5B127.09221964100004%2C%2037.678039738%5D%2C%20%5B127.09388436100005%2C%2037.676669027%5D%2C%20%5B127.09464265999998%2C%2037.67354516199998%5D%2C%20%5B127.09577928399995%2C%2037.672681931%5D%2C%20%5B127.09577485800003%2C%2037.67062784799998%5D%2C%20%5B127.09626208899999%2C%2037.668837161%5D%2C%20%5B127.09500097800003%2C%2037.667606956999975%5D%2C%20%5B127.09469822699998%2C%2037.664927189000025%5D%2C%20%5B127.09542861800003%2C%2037.663895431000014%5D%2C%20%5B127.094157774%2C%2037.66330400700002%5D%2C%20%5B127.091221335%2C%2037.65934019600002%5D%2C%20%5B127.09114721900005%2C%2037.658248809999975%5D%2C%20%5B127.09190627600003%2C%2037.65744077099998%5D%2C%20%5B127.09236725000005%2C%2037.65549788499999%5D%2C%20%5B127.09401619699997%2C%2037.65230150399998%5D%2C%20%5B127.09248088799995%2C%2037.64971631999998%5D%2C%20%5B127.09364724600005%2C%2037.646632361%5D%2C%20%5B127.09448968499999%2C%2037.645856236999975%5D%2C%20%5B127.09458566599994%2C%2037.644576943%5D%2C%20%5B127.09757475499998%2C%2037.643971025999974%5D%2C%20%5B127.10160801200004%2C%2037.64477886399999%5D%2C%20%5B127.103936524%2C%2037.645499323000024%5D%2C%20%5B127.10658921100003%2C%2037.645384454%5D%2C%20%5B127.10770375100003%2C%2037.64497346799999%5D%2C%20%5B127.10930262900001%2C%2037.64286426299998%5D%2C%20%5B127.11076511800002%2C%2037.64273476300002%5D%2C%20%5B127.11223881%2C%2037.64037708500001%5D%2C%20%5B127.110580357%2C%2037.639364204%5D%2C%20%5B127.11249787%2C%2037.63648892100002%5D%2C%20%5B127.11242501499999%2C%2037.63425247999999%5D%2C%20%5B127.11162094999997%2C%2037.633851106%5D%2C%20%5B127.11222175299997%2C%2037.632645615%5D%2C%20%5B127.111649757%2C%2037.63150772300003%5D%2C%20%5B127.108900973%2C%2037.62930749999998%5D%2C%20%5B127.10597855699996%2C%2037.627337269%5D%2C%20%5B127.10434078499998%2C%2037.62327850299999%5D%2C%20%5B127.10407893299998%2C%2037.62166095499998%5D%2C%20%5B127.10492130800003%2C%2037.62157580500002%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2018%2C%20%22SHAPE_AREA%22%3A%200.00364%2C%20%22SHAPE_LEN%22%3A%200.309723%2C%20%22SIG_CD%22%3A%20%2211350%22%2C%20%22SIG_ENG_NM%22%3A%20%22Nowon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.144621648%2C%2037.51561223599998%5D%2C%20%5B127.14215399099999%2C%2037.51566003099998%5D%2C%20%5B127.14138931900004%2C%2037.514975155%5D%2C%20%5B127.14341256199998%2C%2037.51387651099998%5D%2C%20%5B127.14354620699999%2C%2037.512663953000015%5D%2C%20%5B127.1414575%2C%2037.51240795199999%5D%2C%20%5B127.14096457300002%2C%2037.51040582000002%5D%2C%20%5B127.13998328499997%2C%2037.50864916199998%5D%2C%20%5B127.141534557%2C%2037.506717287000015%5D%2C%20%5B127.14102941299996%2C%2037.505493221999984%5D%2C%20%5B127.144121606%2C%2037.50440674700002%5D%2C%20%5B127.14597136099997%2C%2037.503236346999984%5D%2C%20%5B127.14771286899997%2C%2037.503223030000015%5D%2C%20%5B127.15022088800004%2C%2037.50474927699997%5D%2C%20%5B127.15212420600005%2C%2037.50297788%5D%2C%20%5B127.15285967099999%2C%2037.503062426999975%5D%2C%20%5B127.15656939099995%2C%2037.50190080300001%5D%2C%20%5B127.157743278%2C%2037.50318546800003%5D%2C%20%5B127.15887991900001%2C%2037.50239690400002%5D%2C%20%5B127.159410547%2C%2037.501369312%5D%2C%20%5B127.16140357300003%2C%2037.50020737699998%5D%2C%20%5B127.15977913799998%2C%2037.49679815600001%5D%2C%20%5B127.16001896600005%2C%2037.49433240399998%5D%2C%20%5B127.15848990500001%2C%2037.492042157000014%5D%2C%20%5B127.158182699%2C%2037.49051667399999%5D%2C%20%5B127.156300945%2C%2037.48949331900002%5D%2C%20%5B127.154795626%2C%2037.48731379999998%5D%2C%20%5B127.15219320300002%2C%2037.48624278199998%5D%2C%20%5B127.150769406%2C%2037.48474927500001%5D%2C%20%5B127.15104542899996%2C%2037.483961959%5D%2C%20%5B127.15068947400005%2C%2037.48202662699998%5D%2C%20%5B127.14915395599996%2C%2037.48074899300002%5D%2C%20%5B127.14783019699996%2C%2037.478664200000026%5D%2C%20%5B127.14439008099998%2C%2037.476456739000014%5D%2C%20%5B127.14354172599997%2C%2037.475545689%5D%2C%20%5B127.139692276%2C%2037.47419605099998%5D%2C%20%5B127.13937783300003%2C%2037.473447066%5D%2C%20%5B127.13558350400001%2C%2037.474578011%5D%2C%20%5B127.13368163799998%2C%2037.47568719499998%5D%2C%20%5B127.13267746400004%2C%2037.475808433%5D%2C%20%5B127.13024850199997%2C%2037.47510375899998%5D%2C%20%5B127.13042151800005%2C%2037.47189767899999%5D%2C%20%5B127.13538943799995%2C%2037.46932071200001%5D%2C%20%5B127.13404767700001%2C%2037.46859963000003%5D%2C%20%5B127.13014299300005%2C%2037.467765722000024%5D%2C%20%5B127.12704911399999%2C%2037.46842315399999%5D%2C%20%5B127.12521919400001%2C%2037.46962967899998%5D%2C%20%5B127.125063372%2C%2037.46727479200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2019%2C%20%22SHAPE_AREA%22%3A%200.003449%2C%20%22SHAPE_LEN%22%3A%200.316773%2C%20%22SIG_CD%22%3A%20%2211710%22%2C%20%22SIG_ENG_NM%22%3A%20%22Songpa-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2020%2C%20%22SHAPE_AREA%22%3A%200.001714%2C%20%22SHAPE_LEN%22%3A%200.193793%2C%20%22SIG_CD%22%3A%20%2211200%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2021%2C%20%22SHAPE_AREA%22%3A%200.001811%2C%20%22SHAPE_LEN%22%3A%200.231131%2C%20%22SIG_CD%22%3A%20%2211410%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seodaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.858345407%2C%2037.50992339800001%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.85562926600005%2C%2037.50916078300003%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.84873119999997%2C%2037.508888764%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84076167%2C%2037.50604941%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.837508791%2C%2037.50328696299999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.826851927%2C%2037.510431685000015%5D%2C%20%5B126.82420556800002%2C%2037.51053149099999%5D%2C%20%5B126.82387418500002%2C%2037.51088144200003%5D%2C%20%5B126.824425776%2C%2037.513182637%5D%2C%20%5B126.824273167%2C%2037.514473764%5D%2C%20%5B126.82341871799997%2C%2037.51495970899998%5D%2C%20%5B126.82312208200005%2C%2037.51622650500002%5D%2C%20%5B126.82451390100005%2C%2037.51772160199999%5D%2C%20%5B126.82563535199995%2C%2037.520085435%5D%2C%20%5B126.82515520799996%2C%2037.523010279%5D%2C%20%5B126.82676624299995%2C%2037.525130822999984%5D%2C%20%5B126.82816515299999%2C%2037.525836518%5D%2C%20%5B126.82844406000004%2C%2037.52670751199997%5D%2C%20%5B126.827086602%2C%2037.529715297%5D%2C%20%5B126.825575308%2C%2037.529853316000015%5D%2C%20%5B126.82344723300002%2C%2037.532623817%5D%2C%20%5B126.82356902000004%2C%2037.533770715%5D%2C%20%5B126.82193790199995%2C%2037.534835165%5D%2C%20%5B126.82244956399995%2C%2037.536550956999974%5D%2C%20%5B126.82233701300004%2C%2037.53793206300003%5D%2C%20%5B126.82160576399997%2C%2037.539720295999985%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2022%2C%20%22SHAPE_AREA%22%3A%200.001775%2C%20%22SHAPE_LEN%22%3A%200.294094%2C%20%22SIG_CD%22%3A%20%2211470%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yangcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88272419199996%2C%2037.51576865800001%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2023%2C%20%22SHAPE_AREA%22%3A%200.002512%2C%20%22SHAPE_LEN%22%3A%200.262953%2C%20%22SIG_CD%22%3A%20%2211560%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yeongdeungpo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2024%2C%20%22SHAPE_AREA%22%3A%200.002234%2C%20%22SHAPE_LEN%22%3A%200.214496%2C%20%22SIG_CD%22%3A%20%2211170%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yongsan-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20var%20color_map_88ae6e52e31f47ec99716bf890aa0b88%20%3D%20%7B%7D%3B%0A%0A%20%20%20%20%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.color%20%3D%20d3.scale.threshold%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B1345.65%2C%201348.2310621242486%2C%201350.8121242484972%2C%201353.3931863727455%2C%201355.974248496994%2C%201358.5553106212426%2C%201361.1363727454911%2C%201363.7174348697395%2C%201366.298496993988%2C%201368.8795591182366%2C%201371.460621242485%2C%201374.0416833667337%2C%201376.622745490982%2C%201379.2038076152305%2C%201381.784869739479%2C%201384.3659318637276%2C%201386.946993987976%2C%201389.5280561122245%2C%201392.109118236473%2C%201394.6901803607216%2C%201397.2712424849701%2C%201399.8523046092184%2C%201402.433366733467%2C%201405.0144288577155%2C%201407.595490981964%2C%201410.1765531062124%2C%201412.757615230461%2C%201415.3386773547095%2C%201417.919739478958%2C%201420.5008016032066%2C%201423.081863727455%2C%201425.6629258517034%2C%201428.243987975952%2C%201430.8250501002005%2C%201433.4061122244489%2C%201435.9871743486974%2C%201438.568236472946%2C%201441.1492985971945%2C%201443.730360721443%2C%201446.3114228456914%2C%201448.89248496994%2C%201451.4735470941885%2C%201454.054609218437%2C%201456.6356713426853%2C%201459.2167334669339%2C%201461.7977955911824%2C%201464.378857715431%2C%201466.9599198396795%2C%201469.5409819639278%2C%201472.1220440881764%2C%201474.703106212425%2C%201477.2841683366735%2C%201479.865230460922%2C%201482.4462925851703%2C%201485.027354709419%2C%201487.6084168336674%2C%201490.189478957916%2C%201492.7705410821645%2C%201495.3516032064128%2C%201497.9326653306614%2C%201500.51372745491%2C%201503.0947895791583%2C%201505.6758517034068%2C%201508.2569138276554%2C%201510.837975951904%2C%201513.4190380761524%2C%201516.0001002004008%2C%201518.5811623246493%2C%201521.1622244488979%2C%201523.7432865731464%2C%201526.3243486973947%2C%201528.9054108216433%2C%201531.4864729458918%2C%201534.0675350701404%2C%201536.648597194389%2C%201539.2296593186375%2C%201541.8107214428858%2C%201544.3917835671343%2C%201546.9728456913829%2C%201549.5539078156314%2C%201552.1349699398797%2C%201554.7160320641283%2C%201557.2970941883768%2C%201559.8781563126254%2C%201562.459218436874%2C%201565.0402805611222%2C%201567.6213426853708%2C%201570.2024048096193%2C%201572.7834669338677%2C%201575.3645290581162%2C%201577.9455911823648%2C%201580.5266533066133%2C%201583.1077154308618%2C%201585.6887775551104%2C%201588.2698396793587%2C%201590.8509018036073%2C%201593.4319639278558%2C%201596.0130260521044%2C%201598.5940881763527%2C%201601.1751503006012%2C%201603.7562124248498%2C%201606.3372745490983%2C%201608.9183366733469%2C%201611.4993987975952%2C%201614.0804609218437%2C%201616.6615230460923%2C%201619.2425851703408%2C%201621.8236472945891%2C%201624.4047094188377%2C%201626.9857715430862%2C%201629.5668336673348%2C%201632.1478957915833%2C%201634.7289579158316%2C%201637.3100200400802%2C%201639.8910821643287%2C%201642.472144288577%2C%201645.0532064128256%2C%201647.6342685370741%2C%201650.2153306613227%2C%201652.7963927855712%2C%201655.3774549098198%2C%201657.958517034068%2C%201660.5395791583167%2C%201663.1206412825652%2C%201665.7017034068137%2C%201668.282765531062%2C%201670.8638276553106%2C%201673.4448897795592%2C%201676.0259519038077%2C%201678.6070140280563%2C%201681.1880761523046%2C%201683.7691382765531%2C%201686.3502004008017%2C%201688.9312625250502%2C%201691.5123246492985%2C%201694.093386773547%2C%201696.6744488977956%2C%201699.2555110220442%2C%201701.8365731462927%2C%201704.4176352705413%2C%201706.9986973947896%2C%201709.5797595190381%2C%201712.1608216432865%2C%201714.741883767535%2C%201717.3229458917835%2C%201719.904008016032%2C%201722.4850701402806%2C%201725.0661322645292%2C%201727.6471943887777%2C%201730.228256513026%2C%201732.8093186372746%2C%201735.3903807615231%2C%201737.9714428857715%2C%201740.55250501002%2C%201743.1335671342686%2C%201745.714629258517%2C%201748.2956913827657%2C%201750.876753507014%2C%201753.4578156312625%2C%201756.038877755511%2C%201758.6199398797596%2C%201761.201002004008%2C%201763.7820641282565%2C%201766.363126252505%2C%201768.9441883767536%2C%201771.5252505010021%2C%201774.1063126252507%2C%201776.687374749499%2C%201779.2684368737475%2C%201781.849498997996%2C%201784.4305611222444%2C%201787.011623246493%2C%201789.5926853707415%2C%201792.17374749499%2C%201794.7548096192386%2C%201797.3358717434871%2C%201799.9169338677355%2C%201802.497995991984%2C%201805.0790581162325%2C%201807.6601202404809%2C%201810.2411823647294%2C%201812.822244488978%2C%201815.4033066132265%2C%201817.984368737475%2C%201820.5654308617234%2C%201823.146492985972%2C%201825.7275551102205%2C%201828.308617234469%2C%201830.8896793587176%2C%201833.4707414829659%2C%201836.0518036072144%2C%201838.632865731463%2C%201841.2139278557115%2C%201843.79498997996%2C%201846.3760521042084%2C%201848.957114228457%2C%201851.5381763527055%2C%201854.1192384769538%2C%201856.7003006012023%2C%201859.281362725451%2C%201861.8624248496994%2C%201864.443486973948%2C%201867.0245490981965%2C%201869.605611222445%2C%201872.1866733466936%2C%201874.7677354709417%2C%201877.3487975951903%2C%201879.9298597194388%2C%201882.5109218436874%2C%201885.091983967936%2C%201887.6730460921844%2C%201890.254108216433%2C%201892.8351703406815%2C%201895.4162324649299%2C%201897.9972945891784%2C%201900.5783567134267%2C%201903.1594188376753%2C%201905.7404809619238%2C%201908.3215430861724%2C%201910.902605210421%2C%201913.4836673346695%2C%201916.0647294589178%2C%201918.6457915831663%2C%201921.2268537074149%2C%201923.8079158316632%2C%201926.3889779559117%2C%201928.9700400801603%2C%201931.5511022044088%2C%201934.1321643286574%2C%201936.713226452906%2C%201939.2942885771542%2C%201941.8753507014028%2C%201944.4564128256513%2C%201947.0374749499%2C%201949.6185370741482%2C%201952.1995991983968%2C%201954.7806613226453%2C%201957.3617234468938%2C%201959.9427855711424%2C%201962.5238476953907%2C%201965.1049098196393%2C%201967.6859719438878%2C%201970.2670340681361%2C%201972.8480961923847%2C%201975.4291583166332%2C%201978.0102204408818%2C%201980.5912825651303%2C%201983.1723446893789%2C%201985.7534068136274%2C%201988.3344689378757%2C%201990.9155310621243%2C%201993.4965931863726%2C%201996.0776553106211%2C%201998.6587174348697%2C%202001.2397795591182%2C%202003.8208416833668%2C%202006.4019038076153%2C%202008.9829659318639%2C%202011.5640280561124%2C%202014.1450901803605%2C%202016.726152304609%2C%202019.3072144288576%2C%202021.8882765531062%2C%202024.4693386773547%2C%202027.0504008016032%2C%202029.6314629258518%2C%202032.2125250501003%2C%202034.7935871743489%2C%202037.3746492985972%2C%202039.9557114228455%2C%202042.536773547094%2C%202045.1178356713426%2C%202047.6988977955912%2C%202050.2799599198397%2C%202052.8610220440883%2C%202055.442084168337%2C%202058.023146292585%2C%202060.604208416834%2C%202063.185270541082%2C%202065.7663326653305%2C%202068.347394789579%2C%202070.9284569138276%2C%202073.509519038076%2C%202076.0905811623247%2C%202078.6716432865733%2C%202081.2527054108214%2C%202083.8337675350704%2C%202086.4148296593185%2C%202088.9958917835675%2C%202091.5769539078156%2C%202094.158016032064%2C%202096.7390781563126%2C%202099.320140280561%2C%202101.9012024048097%2C%202104.482264529058%2C%202107.063326653307%2C%202109.644388777555%2C%202112.2254509018035%2C%202114.806513026052%2C%202117.3875751503006%2C%202119.968637274549%2C%202122.5496993987977%2C%202125.130761523046%2C%202127.7118236472943%2C%202130.2928857715433%2C%202132.8739478957914%2C%202135.45501002004%2C%202138.0360721442885%2C%202140.617134268537%2C%202143.1981963927856%2C%202145.779258517034%2C%202148.3603206412827%2C%202150.941382765531%2C%202153.5224448897798%2C%202156.103507014028%2C%202158.6845691382764%2C%202161.265631262525%2C%202163.8466933867735%2C%202166.427755511022%2C%202169.0088176352706%2C%202171.589879759519%2C%202174.1709418837677%2C%202176.7520040080162%2C%202179.3330661322643%2C%202181.914128256513%2C%202184.4951903807614%2C%202187.07625250501%2C%202189.6573146292585%2C%202192.238376753507%2C%202194.8194388777556%2C%202197.400501002004%2C%202199.9815631262527%2C%202202.562625250501%2C%202205.1436873747493%2C%202207.724749498998%2C%202210.3058116232464%2C%202212.886873747495%2C%202215.4679358717435%2C%202218.048997995992%2C%202220.63006012024%2C%202223.211122244489%2C%202225.7921843687373%2C%202228.3732464929863%2C%202230.9543086172343%2C%202233.535370741483%2C%202236.1164328657314%2C%202238.69749498998%2C%202241.2785571142285%2C%202243.8596192384766%2C%202246.4406813627256%2C%202249.0217434869737%2C%202251.6028056112227%2C%202254.183867735471%2C%202256.7649298597194%2C%202259.345991983968%2C%202261.9270541082165%2C%202264.508116232465%2C%202267.089178356713%2C%202269.670240480962%2C%202272.25130260521%2C%202274.8323647294587%2C%202277.4134268537073%2C%202279.994488977956%2C%202282.5755511022044%2C%202285.156613226453%2C%202287.7376753507015%2C%202290.31873747495%2C%202292.8997995991986%2C%202295.4808617234467%2C%202298.061923847695%2C%202300.6429859719437%2C%202303.2240480961923%2C%202305.805110220441%2C%202308.3861723446894%2C%202310.967234468938%2C%202313.5482965931865%2C%202316.129358717435%2C%202318.710420841683%2C%202321.2914829659317%2C%202323.87254509018%2C%202326.4536072144288%2C%202329.0346693386773%2C%202331.615731462926%2C%202334.1967935871744%2C%202336.777855711423%2C%202339.3589178356715%2C%202341.9399799599196%2C%202344.521042084168%2C%202347.1021042084167%2C%202349.683166332665%2C%202352.2642284569138%2C%202354.8452905811623%2C%202357.426352705411%2C%202360.0074148296594%2C%202362.588476953908%2C%202365.169539078156%2C%202367.750601202405%2C%202370.331663326653%2C%202372.9127254509017%2C%202375.4937875751502%2C%202378.074849699399%2C%202380.6559118236473%2C%202383.2369739478954%2C%202385.8180360721444%2C%202388.3990981963925%2C%202390.9801603206415%2C%202393.5612224448896%2C%202396.142284569138%2C%202398.7233466933867%2C%202401.3044088176352%2C%202403.885470941884%2C%202406.4665330661323%2C%202409.047595190381%2C%202411.628657314629%2C%202414.209719438878%2C%202416.790781563126%2C%202419.371843687375%2C%202421.952905811623%2C%202424.5339679358717%2C%202427.1150300601203%2C%202429.6960921843684%2C%202432.2771543086174%2C%202434.8582164328654%2C%202437.4392785571144%2C%202440.0203406813625%2C%202442.601402805611%2C%202445.1824649298596%2C%202447.763527054108%2C%202450.3445891783567%2C%202452.9256513026053%2C%202455.506713426854%2C%202458.087775551102%2C%202460.668837675351%2C%202463.249899799599%2C%202465.8309619238476%2C%202468.412024048096%2C%202470.9930861723446%2C%202473.574148296593%2C%202476.1552104208417%2C%202478.7362725450903%2C%202481.3173346693384%2C%202483.8983967935874%2C%202486.4794589178355%2C%202489.0605210420845%2C%202491.6415831663326%2C%202494.222645290581%2C%202496.8037074148297%2C%202499.3847695390778%2C%202501.9658316633268%2C%202504.546893787575%2C%202507.1279559118234%2C%202509.709018036072%2C%202512.2900801603205%2C%202514.871142284569%2C%202517.4522044088176%2C%202520.033266533066%2C%202522.6143286573147%2C%202525.195390781563%2C%202527.7764529058113%2C%202530.3575150300603%2C%202532.9385771543084%2C%202535.5196392785574%2C%202538.1007014028055%2C%202540.6817635270536%2C%202543.2628256513026%2C%202545.8438877755507%2C%202548.4249498997997%2C%202551.006012024048%2C%202553.5870741482963%2C%202556.168136272545%2C%202558.7491983967934%2C%202561.330260521042%2C%202563.9113226452905%2C%202566.492384769539%2C%202569.0734468937876%2C%202571.654509018036%2C%202574.2355711422842%2C%202576.8166332665332%2C%202579.3976953907813%2C%202581.9787575150303%2C%202584.5598196392784%2C%202587.140881763527%2C%202589.7219438877755%2C%202592.3030060120236%2C%202594.8840681362726%2C%202597.4651302605207%2C%202600.0461923847697%2C%202602.627254509018%2C%202605.2083166332663%2C%202607.789378757515%2C%202610.3704408817634%2C%202612.951503006012%2C%202615.5325651302605%2C%202618.113627254509%2C%202620.694689378757%2C%202623.275751503006%2C%202625.8568136272543%2C%202628.437875751503%2C%202631.0189378757514%2C%202633.6%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23f2f0f7ff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23dadaebff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%23bcbddcff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%239e9ac8ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%23756bb1ff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%2C%20%27%2354278fff%27%5D%29%3B%0A%20%20%20%20%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.x%20%3D%20d3.scale.linear%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B1345.65%2C%202633.6%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B0%2C%20400%5D%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.legend%20%3D%20L.control%28%7Bposition%3A%20%27topright%27%7D%29%3B%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.legend.onAdd%20%3D%20function%20%28map%29%20%7Bvar%20div%20%3D%20L.DomUtil.create%28%27div%27%2C%20%27legend%27%29%3B%20return%20div%7D%3B%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.legend.addTo%28map_21d2760c69a444c08bed3d41c690fe16%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.xAxis%20%3D%20d3.svg.axis%28%29%0A%20%20%20%20%20%20%20%20.scale%28color_map_88ae6e52e31f47ec99716bf890aa0b88.x%29%0A%20%20%20%20%20%20%20%20.orient%28%22top%22%29%0A%20%20%20%20%20%20%20%20.tickSize%281%29%0A%20%20%20%20%20%20%20%20.tickValues%28%5B1345.65%2C%201560.3083333333334%2C%201774.9666666666667%2C%201989.625%2C%202204.2833333333333%2C%202418.9416666666666%2C%202633.6%5D%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.svg%20%3D%20d3.select%28%22.legend.leaflet-control%22%29.append%28%22svg%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22id%22%2C%20%27legend%27%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20450%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2040%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.g%20%3D%20color_map_88ae6e52e31f47ec99716bf890aa0b88.svg.append%28%22g%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22key%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22transform%22%2C%20%22translate%2825%2C16%29%22%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.g.selectAll%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.data%28color_map_88ae6e52e31f47ec99716bf890aa0b88.color.range%28%29.map%28function%28d%2C%20i%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20return%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20x0%3A%20i%20%3F%20color_map_88ae6e52e31f47ec99716bf890aa0b88.x%28color_map_88ae6e52e31f47ec99716bf890aa0b88.color.domain%28%29%5Bi%20-%201%5D%29%20%3A%20color_map_88ae6e52e31f47ec99716bf890aa0b88.x.range%28%29%5B0%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x1%3A%20i%20%3C%20color_map_88ae6e52e31f47ec99716bf890aa0b88.color.domain%28%29.length%20%3F%20color_map_88ae6e52e31f47ec99716bf890aa0b88.x%28color_map_88ae6e52e31f47ec99716bf890aa0b88.color.domain%28%29%5Bi%5D%29%20%3A%20color_map_88ae6e52e31f47ec99716bf890aa0b88.x.range%28%29%5B1%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20z%3A%20d%0A%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%7D%29%29%0A%20%20%20%20%20%20.enter%28%29.append%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2010%29%0A%20%20%20%20%20%20%20%20.attr%28%22x%22%2C%20function%28d%29%20%7B%20return%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20function%28d%29%20%7B%20return%20d.x1%20-%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.style%28%22fill%22%2C%20function%28d%29%20%7B%20return%20d.z%3B%20%7D%29%3B%0A%0A%20%20%20%20color_map_88ae6e52e31f47ec99716bf890aa0b88.g.call%28color_map_88ae6e52e31f47ec99716bf890aa0b88.xAxis%29.append%28%22text%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22caption%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22y%22%2C%2021%29%0A%20%20%20%20%20%20%20%20.text%28%27%27%29%3B%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_a6aaaf43e579464b91cdf5a9af4616bd%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_a6aaaf43e579464b91cdf5a9af4616bd%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B37.541%2C%20126.986%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_778e5960525542a1b2041ac2e29bc08f%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//cartodb-basemaps-%7Bs%7D.global.ssl.fastly.net/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%20contributors%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eCartoDB%5Cu003c/a%5Cu003e%2C%20CartoDB%20%5Cu003ca%20href%20%3D%5C%22http%3A//cartodb.com/attributions%5C%22%5Cu003eattributions%5Cu003c/a%5Cu003e%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_a6aaaf43e579464b91cdf5a9af4616bd%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20choropleth_8406b27cb2f94c53a2458ccb75b73693%20%3D%20L.featureGroup%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_a6aaaf43e579464b91cdf5a9af4616bd%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_4fd3c80256744b62b54f470445643a49_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.properties.SIG_CD%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211380%22%3A%20case%20%2211230%22%3A%20case%20%2211110%22%3A%20case%20%2211260%22%3A%20case%20%2211740%22%3A%20case%20%2211470%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23fee391%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211590%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23fe9929%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211680%22%3A%20case%20%2211200%22%3A%20case%20%2211170%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23993404%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211500%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23ffffd4%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20case%20%2211440%22%3A%20case%20%2211650%22%3A%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23d95f0e%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22color%22%3A%20%22grey%22%2C%20%22fillColor%22%3A%20%22%23fec44f%22%2C%20%22fillOpacity%22%3A%200.6%2C%20%22opacity%22%3A%200.5%2C%20%22weight%22%3A%201%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_4fd3c80256744b62b54f470445643a49_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_4fd3c80256744b62b54f470445643a49%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_4fd3c80256744b62b54f470445643a49_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_4fd3c80256744b62b54f470445643a49_styler%2C%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_4fd3c80256744b62b54f470445643a49_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_4fd3c80256744b62b54f470445643a49%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28choropleth_8406b27cb2f94c53a2458ccb75b73693%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_4fd3c80256744b62b54f470445643a49_add%28%7B%22features%22%3A%20%5B%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00831558599998%2C%2037.69001680000002%5D%2C%20%5B127.00764424700003%2C%2037.69150638999997%5D%2C%20%5B127.00973504700005%2C%2037.69324682000001%5D%2C%20%5B127.00969102500005%2C%2037.696558160999984%5D%2C%20%5B127.01214887599997%2C%2037.69724089499999%5D%2C%20%5B127.01390304300003%2C%2037.698660491%5D%2C%20%5B127.01544040299996%2C%2037.70130154100002%5D%2C%20%5B127.019851357%2C%2037.700884901999984%5D%2C%20%5B127.02217147700003%2C%2037.69960736799999%5D%2C%20%5B127.02341184299996%2C%2037.69995983299998%5D%2C%20%5B127.02533791899998%2C%2037.69948105499998%5D%2C%20%5B127.02771004600004%2C%2037.700837447000026%5D%2C%20%5B127.02930740500005%2C%2037.699210467%5D%2C%20%5B127.02975002699998%2C%2037.69621560600001%5D%2C%20%5B127.03104551199999%2C%2037.69304682699999%5D%2C%20%5B127.032188658%2C%2037.69279409400002%5D%2C%20%5B127.03243270899998%2C%2037.69182616099999%5D%2C%20%5B127.03678096299996%2C%2037.692638805%5D%2C%20%5B127.038515888%2C%2037.69328696000002%5D%2C%20%5B127.03896796200002%2C%2037.69397974100002%5D%2C%20%5B127.04112667799996%2C%2037.695296503%5D%2C%20%5B127.04309027199997%2C%2037.69523619%5D%2C%20%5B127.04376597600003%2C%2037.693511828%5D%2C%20%5B127.04566611500002%2C%2037.692384920999984%5D%2C%20%5B127.04783268899996%2C%2037.69329455000002%5D%2C%20%5B127.04864899799998%2C%2037.69406779399998%5D%2C%20%5B127.04938379999999%2C%2037.691693921000024%5D%2C%20%5B127.04998367500002%2C%2037.690875163999976%5D%2C%20%5B127.04983654099999%2C%2037.687983056%5D%2C%20%5B127.05093367799998%2C%2037.686160674%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%200%2C%20%22SHAPE_AREA%22%3A%200.00211%2C%20%22SHAPE_LEN%22%3A%200.239901%2C%20%22SIG_CD%22%3A%20%2211320%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dobong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3c4%5Cubd09%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88359471499996%2C%2037.591855451000015%5D%2C%20%5B126.88502350199997%2C%2037.593642095%5D%2C%20%5B126.88720157499995%2C%2037.59368140999999%5D%2C%20%5B126.88561580199996%2C%2037.591502085%5D%2C%20%5B126.88577464100001%2C%2037.589569513000015%5D%2C%20%5B126.88717970799996%2C%2037.58853613399998%5D%2C%20%5B126.88927454400005%2C%2037.58883090799998%5D%2C%20%5B126.89225666100003%2C%2037.58852532399999%5D%2C%20%5B126.89334847199996%2C%2037.589070849%5D%2C%20%5B126.89687879200005%2C%2037.588570065%5D%2C%20%5B126.89810143800003%2C%2037.58954265199998%5D%2C%20%5B126.89966320099995%2C%2037.58982035399998%5D%2C%20%5B126.899585279%2C%2037.591694511000014%5D%2C%20%5B126.89898042499999%2C%2037.592683197999975%5D%2C%20%5B126.90037465800003%2C%2037.59421526400001%5D%2C%20%5B126.90184962900003%2C%2037.59514499199997%5D%2C%20%5B126.90103258399995%2C%2037.597339986%5D%2C%20%5B126.901343312%2C%2037.59936941699999%5D%2C%20%5B126.90000258099997%2C%2037.602042943000015%5D%2C%20%5B126.90017282600002%2C%2037.60311939799999%5D%2C%20%5B126.90212880000001%2C%2037.60374458000001%5D%2C%20%5B126.90062064300002%2C%2037.60932366100002%5D%2C%20%5B126.90032356100005%2C%2037.611195898%5D%2C%20%5B126.90175330700004%2C%2037.61407416499998%5D%2C%20%5B126.901843654%2C%2037.615756572%5D%2C%20%5B126.90314656400005%2C%2037.61730752099999%5D%2C%20%5B126.90338372099995%2C%2037.61884112299998%5D%2C%20%5B126.90525097600005%2C%2037.61907907300002%5D%2C%20%5B126.90563169100005%2C%2037.620708664%5D%2C%20%5B126.90713704200004%2C%2037.62194588400001%5D%2C%20%5B126.90716203600005%2C%2037.62331746299998%5D%2C%20%5B126.90664762899996%2C%2037.62449994799999%5D%2C%20%5B126.908630388%2C%2037.62627061799998%5D%2C%20%5B126.90879064700005%2C%2037.629332764000026%5D%2C%20%5B126.90731509700004%2C%2037.63087173100001%5D%2C%20%5B126.90677806899998%2C%2037.63274096499998%5D%2C%20%5B126.90711683699999%2C%2037.63340278800001%5D%2C%20%5B126.910875079%2C%2037.635322556%5D%2C%20%5B126.91123094800002%2C%2037.635911695%5D%2C%20%5B126.91011226199998%2C%2037.638515595%5D%2C%20%5B126.91081456899997%2C%2037.64003255799997%5D%2C%20%5B126.91230017800001%2C%2037.64432203199999%5D%2C%20%5B126.91016072699995%2C%2037.645083612%5D%2C%20%5B126.90772442000002%2C%2037.64629642699998%5D%2C%20%5B126.90843309399997%2C%2037.647262366%5D%2C%20%5B126.90974709700004%2C%2037.646882914%5D%2C%20%5B126.91234747099998%2C%2037.645182541%5D%2C%20%5B126.913740469%2C%2037.64476066499998%5D%2C%20%5B126.92414365900004%2C%2037.64611043100001%5D%2C%20%5B126.92715483799998%2C%2037.64810585499998%5D%2C%20%5B126.92972022900005%2C%2037.65004959800001%5D%2C%20%5B126.93224238100004%2C%2037.65038895999999%5D%2C%20%5B126.935727004%2C%2037.65120514099999%5D%2C%20%5B126.93718405799996%2C%2037.652317728000014%5D%2C%20%5B126.93847852299996%2C%2037.654773405000014%5D%2C%20%5B126.94014671100001%2C%2037.65663874799998%5D%2C%20%5B126.94211837499995%2C%2037.65772510800002%5D%2C%20%5B126.94441488899997%2C%2037.65816625799999%5D%2C%20%5B126.94758363699998%2C%2037.659217403000014%5D%2C%20%5B126.94790109500002%2C%2037.65712764900002%5D%2C%20%5B126.94922168000005%2C%2037.656772183999976%5D%2C%20%5B126.94980975299995%2C%2037.655913363000025%5D%2C%20%5B126.95139494700004%2C%2037.654951117999985%5D%2C%20%5B126.95437039%2C%2037.654596949999984%5D%2C%20%5B126.95712830900004%2C%2037.65283925900002%5D%2C%20%5B126.95781039600001%2C%2037.64981070699997%5D%2C%20%5B126.95786031099999%2C%2037.64804963099999%5D%2C%20%5B126.95904808800003%2C%2037.64678533599999%5D%2C%20%5B126.95892784399996%2C%2037.64476220400002%5D%2C%20%5B126.95928919999994%2C%2037.642134535000025%5D%2C%20%5B126.959992629%2C%2037.64146349599997%5D%2C%20%5B126.96162582500006%2C%2037.63711909400001%5D%2C%20%5B126.96256224599995%2C%2037.63582057799999%5D%2C%20%5B126.96335032800005%2C%2037.633246824000025%5D%2C%20%5B126.96164091799994%2C%2037.631889602%5D%2C%20%5B126.95984451699996%2C%2037.629762777999986%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%201%2C%20%22SHAPE_AREA%22%3A%200.003041%2C%20%22SHAPE_LEN%22%3A%200.327143%2C%20%22SIG_CD%22%3A%20%2211380%22%2C%20%22SIG_ENG_NM%22%3A%20%22Eunpyeong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc740%5Cud3c9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%202%2C%20%22SHAPE_AREA%22%3A%200.001453%2C%20%22SHAPE_LEN%22%3A%200.182837%2C%20%22SIG_CD%22%3A%20%2211230%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongdaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%203%2C%20%22SHAPE_AREA%22%3A%200.00167%2C%20%22SHAPE_LEN%22%3A%200.237796%2C%20%22SIG_CD%22%3A%20%2211590%22%2C%20%22SIG_ENG_NM%22%3A%20%22Dongjak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub3d9%5Cuc791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92829901899995%2C%2037.449360259%5D%2C%20%5B126.926538273%2C%2037.44839745899998%5D%2C%20%5B126.92500225699996%2C%2037.44678811799997%5D%2C%20%5B126.92327265100005%2C%2037.44577942699999%5D%2C%20%5B126.92278057800002%2C%2037.444047814999976%5D%2C%20%5B126.92118598900004%2C%2037.442552917%5D%2C%20%5B126.92028939299996%2C%2037.44047332299999%5D%2C%20%5B126.91927808399998%2C%2037.43985946599997%5D%2C%20%5B126.91613959300003%2C%2037.440060542000026%5D%2C%20%5B126.91232267199996%2C%2037.438587175%5D%2C%20%5B126.911215782%2C%2037.43720903600001%5D%2C%20%5B126.91134523699998%2C%2037.436179159%5D%2C%20%5B126.90940698500003%2C%2037.433868523%5D%2C%20%5B126.90726602899997%2C%2037.43352405000002%5D%2C%20%5B126.90611662000003%2C%2037.43400280399999%5D%2C%20%5B126.90299885399997%2C%2037.43407347499999%5D%2C%20%5B126.902763%2C%2037.43586855199999%5D%2C%20%5B126.90086236399998%2C%2037.43699089%5D%2C%20%5B126.89877013700004%2C%2037.439283811999985%5D%2C%20%5B126.89991895399999%2C%2037.439562146000014%5D%2C%20%5B126.897287549%2C%2037.445443447%5D%2C%20%5B126.89580121999995%2C%2037.445640562999984%5D%2C%20%5B126.89466890899996%2C%2037.446706108%5D%2C%20%5B126.89476993300002%2C%2037.447710443%5D%2C%20%5B126.89592769599994%2C%2037.44842231299998%5D%2C%20%5B126.89400040400005%2C%2037.452723881%5D%2C%20%5B126.89277619200004%2C%2037.452046575%5D%2C%20%5B126.88951816999997%2C%2037.45270753599999%5D%2C%20%5B126.889280761%2C%2037.45448889199997%5D%2C%20%5B126.88641922299996%2C%2037.456180116999974%5D%2C%20%5B126.88589653600002%2C%2037.457514893999985%5D%2C%20%5B126.88630108799998%2C%2037.459120703%5D%2C%20%5B126.885364977%2C%2037.459868728%5D%2C%20%5B126.886087029%2C%2037.460865333000015%5D%2C%20%5B126.88889275600002%2C%2037.460953249%5D%2C%20%5B126.88713625100002%2C%2037.462909712%5D%2C%20%5B126.88515609299998%2C%2037.462473671%5D%2C%20%5B126.88433205199999%2C%2037.46272970799998%5D%2C%20%5B126.88276046399994%2C%2037.464396598%5D%2C%20%5B126.88463173000002%2C%2037.46601745999999%5D%2C%20%5B126.88302833499995%2C%2037.467069503%5D%2C%20%5B126.881647767%2C%2037.468466234%5D%2C%20%5B126.88161172399998%2C%2037.469425546000025%5D%2C%20%5B126.878031942%2C%2037.47383573500002%5D%2C%20%5B126.87600453499999%2C%2037.477111283%5D%2C%20%5B126.87178398799995%2C%2037.48527755399999%5D%2C%20%5B126.87315863100002%2C%2037.486160427000016%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%204%2C%20%22SHAPE_AREA%22%3A%200.001325%2C%20%22SHAPE_LEN%22%3A%200.211649%2C%20%22SIG_CD%22%3A%20%2211545%22%2C%20%22SIG_ENG_NM%22%3A%20%22Geumcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuae08%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.88272419199996%2C%2037.515768666999975%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89771162399995%2C%2037.47891983300002%5D%2C%20%5B126.89650016%2C%2037.4787078%5D%2C%20%5B126.89623107199998%2C%2037.47868023199999%5D%2C%20%5B126.89547755700005%2C%2037.478512998999975%5D%2C%20%5B126.89143281500003%2C%2037.479128694999986%5D%2C%20%5B126.88964944999998%2C%2037.47948708400003%5D%2C%20%5B126.88938470300002%2C%2037.47958364700003%5D%2C%20%5B126.88832528800003%2C%2037.480080283%5D%2C%20%5B126.88817227899995%2C%2037.48017473300001%5D%2C%20%5B126.88801927400004%2C%2037.480272571%5D%2C%20%5B126.88771737800005%2C%2037.48048289600001%5D%2C%20%5B126.88769721799997%2C%2037.480507663000026%5D%2C%20%5B126.88723547500001%2C%2037.48107383399997%5D%2C%20%5B126.88708539100003%2C%2037.48125842600001%5D%2C%20%5B126.887007337%2C%2037.48135411200002%5D%2C%20%5B126.88672553100002%2C%2037.481699675000016%5D%2C%20%5B126.88669534400003%2C%2037.48173676699997%5D%2C%20%5B126.886536836%2C%2037.48193146199998%5D%2C%20%5B126.88640327099995%2C%2037.482089121%5D%2C%20%5B126.88621126600003%2C%2037.482313331%5D%2C%20%5B126.88573284100005%2C%2037.482699266%5D%2C%20%5B126.885692489%2C%2037.48272511800002%5D%2C%20%5B126.88535591300001%2C%2037.48294430499999%5D%2C%20%5B126.88529309%2C%2037.482984778%5D%2C%20%5B126.885089247%2C%2037.483120825000015%5D%2C%20%5B126.88490716199999%2C%2037.483243372%5D%2C%20%5B126.88477988199998%2C%2037.48332767900001%5D%2C%20%5B126.88460522900004%2C%2037.48344460200002%5D%2C%20%5B126.88448555000002%2C%2037.483524423%5D%2C%20%5B126.88436214800004%2C%2037.483608747%5D%2C%20%5B126.88322788100004%2C%2037.48440887100003%5D%2C%20%5B126.88311944300006%2C%2037.484486799000024%5D%2C%20%5B126.88266028400005%2C%2037.484802494%5D%2C%20%5B126.88248637000004%2C%2037.48490623399999%5D%2C%20%5B126.88241684699994%2C%2037.484949094%5D%2C%20%5B126.88204037399998%2C%2037.485172380999984%5D%2C%20%5B126.88137250299997%2C%2037.485565259%5D%2C%20%5B126.88105180699995%2C%2037.485677517%5D%2C%20%5B126.879470288%2C%2037.48622544599999%5D%2C%20%5B126.878479578%2C%2037.486669673999984%5D%2C%20%5B126.877641165%2C%2037.486087836000024%5D%2C%20%5B126.87457691500003%2C%2037.485375523000016%5D%2C%20%5B126.87599162499998%2C%2037.48691906800002%5D%2C%20%5B126.87680700600004%2C%2037.488578991%5D%2C%20%5B126.87583373899997%2C%2037.489035374000025%5D%2C%20%5B126.87309748400003%2C%2037.488261317000024%5D%2C%20%5B126.87245908399996%2C%2037.489543242000025%5D%2C%20%5B126.87024621900002%2C%2037.48960327200001%5D%2C%20%5B126.86936249799999%2C%2037.49164208000002%5D%2C%20%5B126.86972553299995%2C%2037.49410186699998%5D%2C%20%5B126.86836486200002%2C%2037.49511503000002%5D%2C%20%5B126.86738599800003%2C%2037.49308047%5D%2C%20%5B126.8663199%2C%2037.492637301%5D%2C%20%5B126.86541991000001%2C%2037.49151324600001%5D%2C%20%5B126.86260271100002%2C%2037.49076571799998%5D%2C%20%5B126.86139963599999%2C%2037.489972641%5D%2C%20%5B126.85798566000005%2C%2037.48609330599999%5D%2C%20%5B126.855521178%2C%2037.485415111%5D%2C%20%5B126.853961895%2C%2037.48297475200002%5D%2C%20%5B126.85271384700002%2C%2037.48182609899999%5D%2C%20%5B126.85043164800004%2C%2037.48157525800002%5D%2C%20%5B126.84848405499997%2C%2037.482095259%5D%2C%20%5B126.84642145600003%2C%2037.48150571299999%5D%2C%20%5B126.84594900499997%2C%2037.480323644%5D%2C%20%5B126.84537853999996%2C%2037.473821137000016%5D%2C%20%5B126.842799474%2C%2037.47498250699999%5D%2C%20%5B126.84104901299997%2C%2037.474656347%5D%2C%20%5B126.83831728799998%2C%2037.47540135999998%5D%2C%20%5B126.83484912899996%2C%2037.47443089%5D%2C%20%5B126.83488769799999%2C%2037.47575841499997%5D%2C%20%5B126.83349606700006%2C%2037.47716231800001%5D%2C%20%5B126.83176399199999%2C%2037.47765735500002%5D%2C%20%5B126.82788973599997%2C%2037.47599654700002%5D%2C%20%5B126.82435334800005%2C%2037.476259733%5D%2C%20%5B126.82212405999996%2C%2037.47575425399998%5D%2C%20%5B126.82116790299995%2C%2037.476307167000016%5D%2C%20%5B126.81909562600003%2C%2037.476138418%5D%2C%20%5B126.818336391%2C%2037.47530395899997%5D%2C%20%5B126.818647371%2C%2037.474085166%5D%2C%20%5B126.81708964200004%2C%2037.47329169699998%5D%2C%20%5B126.81657626699996%2C%2037.473998096%5D%2C%20%5B126.81464318099995%2C%2037.47465330900002%5D%2C%20%5B126.81531048900001%2C%2037.47636645199998%5D%2C%20%5B126.81722552199994%2C%2037.47814126600002%5D%2C%20%5B126.81827151499999%2C%2037.478170568%5D%2C%20%5B126.81901268700005%2C%2037.47916943500002%5D%2C%20%5B126.81999807199998%2C%2037.48164884%5D%2C%20%5B126.81959813599997%2C%2037.48244711299998%5D%2C%20%5B126.81934165300004%2C%2037.48551259499999%5D%2C%20%5B126.82130494499995%2C%2037.48618739800003%5D%2C%20%5B126.82351963899998%2C%2037.487744236000026%5D%2C%20%5B126.82277597899997%2C%2037.48998772700003%5D%2C%20%5B126.81790518800005%2C%2037.49154342000003%5D%2C%20%5B126.81459113400001%2C%2037.49320034200002%5D%2C%20%5B126.81436892199997%2C%2037.49443227400002%5D%2C%20%5B126.81302860799997%2C%2037.496407777%5D%2C%20%5B126.81415555599995%2C%2037.49813429599999%5D%2C%20%5B126.81611235499997%2C%2037.49765224499998%5D%2C%20%5B126.818224268%2C%2037.49886467499999%5D%2C%20%5B126.81965596999999%2C%2037.499219118999974%5D%2C%20%5B126.81943123600001%2C%2037.50079738%5D%2C%20%5B126.82158116200003%2C%2037.50216396000002%5D%2C%20%5B126.82184366299998%2C%2037.504545139000015%5D%2C%20%5B126.82256759899997%2C%2037.505692083999975%5D%2C%20%5B126.82223955400002%2C%2037.50769455599999%5D%2C%20%5B126.82615287800002%2C%2037.50872472999998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.837508791%2C%2037.50328697200001%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.84076167%2C%2037.50604941900002%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84873119999997%2C%2037.508888773000024%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85562926600005%2C%2037.50916079199999%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.858345407%2C%2037.509923406999974%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%205%2C%20%22SHAPE_AREA%22%3A%200.002047%2C%20%22SHAPE_LEN%22%3A%200.347568%2C%20%22SIG_CD%22%3A%20%2211530%22%2C%20%22SIG_ENG_NM%22%3A%20%22Guro-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad6c%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02316607%2C%2037.57770457999999%5D%2C%20%5B127.02317476600001%2C%2037.577454805%5D%2C%20%5B127.02321042100004%2C%2037.57647763699998%5D%2C%20%5B127.02323057299998%2C%2037.575884438%5D%2C%20%5B127.02325246299995%2C%2037.57542703399997%5D%2C%20%5B127.02325476800002%2C%2037.575367640000024%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.94982865199995%2C%2037.611660546999985%5D%2C%20%5B126.95078813299995%2C%2037.61386663799999%5D%2C%20%5B126.94985370899997%2C%2037.618770648%5D%2C%20%5B126.94992061100004%2C%2037.620926634%5D%2C%20%5B126.94891749700002%2C%2037.62325859700002%5D%2C%20%5B126.94902146799996%2C%2037.624339281%5D%2C%20%5B126.95043234800005%2C%2037.626422925999975%5D%2C%20%5B126.95159893000005%2C%2037.62655885100003%5D%2C%20%5B126.95522594199997%2C%2037.628153574%5D%2C%20%5B126.956400197%2C%2037.628221783000015%5D%2C%20%5B126.95844027600003%2C%2037.62947148400002%5D%2C%20%5B126.96146158%2C%2037.62996095%5D%2C%20%5B126.96378891899997%2C%2037.62979475499998%5D%2C%20%5B126.96524997699998%2C%2037.63076970399999%5D%2C%20%5B126.968256763%2C%2037.63073174200002%5D%2C%20%5B126.97134811%2C%2037.63162062700002%5D%2C%20%5B126.973049887%2C%2037.632376954999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%206%2C%20%22SHAPE_AREA%22%3A%200.002448%2C%20%22SHAPE_LEN%22%3A%200.2901%2C%20%22SIG_CD%22%3A%20%2211110%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jongno-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc885%5Cub85c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.008684346%2C%2037.68439513300001%5D%2C%20%5B127.00843994399997%2C%2037.683188873%5D%2C%20%5B127.009435564%2C%2037.679556897%5D%2C%20%5B127.012151378%2C%2037.679097421999984%5D%2C%20%5B127.01329872400004%2C%2037.677882114%5D%2C%20%5B127.01349362899998%2C%2037.67718832999998%5D%2C%20%5B127.01392402%2C%2037.675242336999986%5D%2C%20%5B127.01513538200004%2C%2037.67415612399998%5D%2C%20%5B127.01623870699996%2C%2037.673934359999976%5D%2C%20%5B127.01850911899999%2C%2037.67015732200002%5D%2C%20%5B127.01842022400001%2C%2037.668889713%5D%2C%20%5B127.01623571799996%2C%2037.66745846600003%5D%2C%20%5B127.01636096899995%2C%2037.667360386999974%5D%2C%20%5B127.01580328199998%2C%2037.66703756200002%5D%2C%20%5B127.01534875899995%2C%2037.664080711%5D%2C%20%5B127.01667029600003%2C%2037.662212047000025%5D%2C%20%5B127.01589974499996%2C%2037.66132864000002%5D%2C%20%5B127.01467797500004%2C%2037.66148801600002%5D%2C%20%5B127.01413119999995%2C%2037.65950582300002%5D%2C%20%5B127.014105223%2C%2037.659425837000015%5D%2C%20%5B127.01246283700004%2C%2037.652188685%5D%2C%20%5B127.01292392699997%2C%2037.650751845%5D%2C%20%5B127.01422799%2C%2037.650086412%5D%2C%20%5B127.01426408299994%2C%2037.65002643600002%5D%2C%20%5B127.01445003200001%2C%2037.64987242199999%5D%2C%20%5B127.01461456599998%2C%2037.649725441999976%5D%2C%20%5B127.015111177%2C%2037.64928874499998%5D%2C%20%5B127.01771548%2C%2037.64858196099999%5D%2C%20%5B127.02114661899998%2C%2037.64900586300001%5D%2C%20%5B127.02268095700003%2C%2037.647853215%5D%2C%20%5B127.024696501%2C%2037.647349873999985%5D%2C%20%5B127.02590005800005%2C%2037.64572796800002%5D%2C%20%5B127.02742464799996%2C%2037.64503923299998%5D%2C%20%5B127.031962991%2C%2037.64217890100002%5D%2C%20%5B127.03284573899998%2C%2037.64141351799998%5D%2C%20%5B127.03458564100004%2C%2037.637785278000024%5D%2C%20%5B127.037764358%2C%2037.636110051%5D%2C%20%5B127.03778471400005%2C%2037.63609174599998%5D%2C%20%5B127.03815895299999%2C%2037.63426091299999%5D%2C%20%5B127.03978313599998%2C%2037.632200351999984%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98421633700002%2C%2037.63634100199999%5D%2C%20%5B126.98510166300002%2C%2037.637318127000015%5D%2C%20%5B126.98569312400002%2C%2037.640048433%5D%2C%20%5B126.98528469200005%2C%2037.641354295999975%5D%2C%20%5B126.98427374300002%2C%2037.64164886999998%5D%2C%20%5B126.98326407499997%2C%2037.643687516%5D%2C%20%5B126.98512211000002%2C%2037.64573277%5D%2C%20%5B126.98462050299997%2C%2037.647616578%5D%2C%20%5B126.98399871900006%2C%2037.64818593799998%5D%2C%20%5B126.98395619999997%2C%2037.649613708%5D%2C%20%5B126.98251481800003%2C%2037.650544008%5D%2C%20%5B126.98169600799997%2C%2037.652320175%5D%2C%20%5B126.98106737800003%2C%2037.652716521%5D%2C%20%5B126.97996370600004%2C%2037.65481924699998%5D%2C%20%5B126.97967724299997%2C%2037.65604285799998%5D%2C%20%5B126.98302857199997%2C%2037.65700805%5D%2C%20%5B126.98603858499996%2C%2037.65894778400002%5D%2C%20%5B126.987294798%2C%2037.660604260000014%5D%2C%20%5B126.98782001899997%2C%2037.66359695900002%5D%2C%20%5B126.98828632200002%2C%2037.66439418800002%5D%2C%20%5B126.99001520599995%2C%2037.66450267%5D%2C%20%5B126.99224251800001%2C%2037.66523496299999%5D%2C%20%5B126.99414712600003%2C%2037.667033076%5D%2C%20%5B126.99357858400003%2C%2037.66780672099998%5D%2C%20%5B126.99437535000004%2C%2037.66956914000002%5D%2C%20%5B126.99382043900005%2C%2037.670199244%5D%2C%20%5B126.99409198599994%2C%2037.672763257999975%5D%2C%20%5B126.99356165699999%2C%2037.67410523500001%5D%2C%20%5B126.99399960799997%2C%2037.674909437999986%5D%2C%20%5B126.99321780000002%2C%2037.675722317%5D%2C%20%5B126.99326569200002%2C%2037.67766322599999%5D%2C%20%5B126.99221614400005%2C%2037.67962970100001%5D%2C%20%5B126.99409428599995%2C%2037.68032697899997%5D%2C%20%5B126.99475084200003%2C%2037.681246918%5D%2C%20%5B126.99675604100003%2C%2037.68239214%5D%2C%20%5B126.997333905%2C%2037.68337292299998%5D%2C%20%5B127.00170893100005%2C%2037.68426214499999%5D%2C%20%5B127.003605311%2C%2037.684165339%5D%2C%20%5B127.00454835300002%2C%2037.68501282099999%5D%2C%20%5B127.00607232000004%2C%2037.684984327%5D%2C%20%5B127.008684346%2C%2037.68439513300001%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%207%2C%20%22SHAPE_AREA%22%3A%200.002412%2C%20%22SHAPE_LEN%22%3A%200.267441%2C%20%22SIG_CD%22%3A%20%2211305%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.07817366799998%2C%2037.571926374999975%5D%2C%20%5B127.077250134%2C%2037.57292520499999%5D%2C%20%5B127.077244704%2C%2037.573514312999976%5D%2C%20%5B127.07712932799996%2C%2037.57439263100002%5D%2C%20%5B127.07712827399996%2C%2037.574400516000026%5D%2C%20%5B127.07671563099996%2C%2037.57753555900001%5D%2C%20%5B127.076706124%2C%2037.57760169599999%5D%2C%20%5B127.07670347500004%2C%2037.577620835%5D%2C%20%5B127.076439628%2C%2037.579552481%5D%2C%20%5B127.07643873699999%2C%2037.57955726199998%5D%2C%20%5B127.07643115899998%2C%2037.579590752%5D%2C%20%5B127.07642991700004%2C%2037.57959610099999%5D%2C%20%5B127.076360905%2C%2037.57986203199999%5D%2C%20%5B127.07635931200002%2C%2037.579867380999985%5D%2C%20%5B127.07621554699995%2C%2037.58026220800002%5D%2C%20%5B127.07619947%2C%2037.580298798%5D%2C%20%5B127.074273189%2C%2037.58248067400001%5D%2C%20%5B127.073947814%2C%2037.582867693000026%5D%2C%20%5B127.07392149500004%2C%2037.58289813599998%5D%2C%20%5B127.07392026699995%2C%2037.58290320499998%5D%2C%20%5B127.07392222199996%2C%2037.58290968799997%5D%2C%20%5B127.073920983%2C%2037.58291532499999%5D%2C%20%5B127.073060941%2C%2037.583958273%5D%2C%20%5B127.07305458799999%2C%2037.58396616499999%5D%2C%20%5B127.07287404299996%2C%2037.584170527000026%5D%2C%20%5B127.07278765499996%2C%2037.584269189%5D%2C%20%5B127.07254740400003%2C%2037.584543736%5D%2C%20%5B127.07218048699997%2C%2037.584961206%5D%2C%20%5B127.07218865599998%2C%2037.58498769%5D%2C%20%5B127.072328363%2C%2037.585430953000014%5D%2C%20%5B127.07236474399997%2C%2037.58554927900002%5D%2C%20%5B127.07266186799995%2C%2037.586515487999975%5D%2C%20%5B127.07262647200002%2C%2037.586539146%5D%2C%20%5B127.07237974199995%2C%2037.586762448%5D%2C%20%5B127.07238008800005%2C%2037.58677484200001%5D%2C%20%5B127.07238431600001%2C%2037.586831415%5D%2C%20%5B127.07224516400004%2C%2037.58705523499998%5D%2C%20%5B127.07223826400002%2C%2037.58706593900001%5D%2C%20%5B127.072213476%2C%2037.58709719400002%5D%2C%20%5B127.07215576500005%2C%2037.58716927400002%5D%2C%20%5B127.07214603199998%2C%2037.587183915000026%5D%2C%20%5B127.07136886%2C%2037.58853606299999%5D%2C%20%5B127.07109588799995%2C%2037.58890909399997%5D%2C%20%5B127.07093747600004%2C%2037.58901355699999%5D%2C%20%5B127.07093181899995%2C%2037.58901580999998%5D%2C%20%5B127.07091148500001%2C%2037.58902426499998%5D%2C%20%5B127.07090529599998%2C%2037.58902708599999%5D%2C%20%5B127.07089928799996%2C%2037.589029617999984%5D%2C%20%5B127.07089026899996%2C%2037.58903327799999%5D%2C%20%5B127.07081530599999%2C%2037.58906960600001%5D%2C%20%5B127.07077499000002%2C%2037.58908312800003%5D%2C%20%5B127.07073910999998%2C%2037.58909467699999%5D%2C%20%5B127.07066255100005%2C%2037.58913467100001%5D%2C%20%5B127.07062312899996%2C%2037.58915522799998%5D%2C%20%5B127.07059908400004%2C%2037.58917915699999%5D%2C%20%5B127.07031955399998%2C%2037.589538398%5D%2C%20%5B127.07028755700003%2C%2037.589819585999976%5D%2C%20%5B127.07024655099997%2C%2037.58996202499998%5D%2C%20%5B127.07032858000002%2C%2037.590194741%5D%2C%20%5B127.07045644599998%2C%2037.59055658800003%5D%2C%20%5B127.07046889799994%2C%2037.59059203700002%5D%2C%20%5B127.07047725300004%2C%2037.59061596%5D%2C%20%5B127.07049094700005%2C%2037.59065506500002%5D%2C%20%5B127.07050387499999%2C%2037.59069196899998%5D%2C%20%5B127.070510431%2C%2037.59071054999998%5D%2C%20%5B127.07051875499997%2C%2037.59073024700001%5D%2C%20%5B127.07056961399996%2C%2037.590845365%5D%2C%20%5B127.07057280100003%2C%2037.59085239900003%5D%2C%20%5B127.07058538800004%2C%2037.590880543000026%5D%2C%20%5B127.07062064299998%2C%2037.59095962700002%5D%2C%20%5B127.070708373%2C%2037.59135990599998%5D%2C%20%5B127.07074059800004%2C%2037.59189552399999%5D%2C%20%5B127.07058647199995%2C%2037.59245507999998%5D%2C%20%5B127.06945351000002%2C%2037.59512585300001%5D%2C%20%5B127.06944824899995%2C%2037.59521424600001%5D%2C%20%5B127.06944207599997%2C%2037.595331913999985%5D%2C%20%5B127.06946696199998%2C%2037.595502726%5D%2C%20%5B127.06985706%2C%2037.59773406099998%5D%2C%20%5B127.07080962299995%2C%2037.59863019900001%5D%2C%20%5B127.07250759199997%2C%2037.60024822899999%5D%2C%20%5B127.07281704000002%2C%2037.60084306800002%5D%2C%20%5B127.07242288400005%2C%2037.602880202999984%5D%2C%20%5B127.07154505000005%2C%2037.60464406300002%5D%2C%20%5B127.07124734399997%2C%2037.60631163199997%5D%2C%20%5B127.07124158199997%2C%2037.60635220199998%5D%2C%20%5B127.07123234899996%2C%2037.606424048%5D%2C%20%5B127.071188119%2C%2037.60669169900001%5D%2C%20%5B127.07109526%2C%2037.60732730400002%5D%2C%20%5B127.07109216200001%2C%2037.607369568000024%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10914571%2C%2037.62052080400002%5D%2C%20%5B127.11053916100002%2C%2037.62105968399999%5D%2C%20%5B127.11216023500003%2C%2037.620311775%5D%2C%20%5B127.11503737099997%2C%2037.61954149500002%5D%2C%20%5B127.11716289399999%2C%2037.61789223%5D%2C%20%5B127.11668740899995%2C%2037.61601964499999%5D%2C%20%5B127.11749802600002%2C%2037.61177613000001%5D%2C%20%5B127.11677896900005%2C%2037.610134802%5D%2C%20%5B127.116712333%2C%2037.60885401899998%5D%2C%20%5B127.11849163500005%2C%2037.60761747%5D%2C%20%5B127.118075341%2C%2037.606595825%5D%2C%20%5B127.118061087%2C%2037.604606131000025%5D%2C%20%5B127.115739211%2C%2037.60211296%5D%2C%20%5B127.11408091199996%2C%2037.599527276%5D%2C%20%5B127.11689451500001%2C%2037.595503880000024%5D%2C%20%5B127.11668170400003%2C%2037.594023413%5D%2C%20%5B127.11335498100004%2C%2037.59326654199998%5D%2C%20%5B127.110693577%2C%2037.58916251699998%5D%2C%20%5B127.11000566899997%2C%2037.586023006%5D%2C%20%5B127.10898535199999%2C%2037.58338535199999%5D%2C%20%5B127.10345938600005%2C%2037.58060086099999%5D%2C%20%5B127.10304415999997%2C%2037.578912213000024%5D%2C%20%5B127.10116216200004%2C%2037.57607578599999%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%208%2C%20%22SHAPE_AREA%22%3A%200.001893%2C%20%22SHAPE_LEN%22%3A%200.184716%2C%20%22SIG_CD%22%3A%20%2211260%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jungnang-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cub791%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.12187312100002%2C%2037.46514826399999%5D%2C%20%5B127.12145320599996%2C%2037.46438168499998%5D%2C%20%5B127.11748896200004%2C%2037.462210077%5D%2C%20%5B127.11700353900005%2C%2037.46149384900002%5D%2C%20%5B127.116910318%2C%2037.458650388000024%5D%2C%20%5B127.11590693999995%2C%2037.45860154000002%5D%2C%20%5B127.11328498199998%2C%2037.460064267%5D%2C%20%5B127.11298953899995%2C%2037.46114412899999%5D%2C%20%5B127.11183504999997%2C%2037.461654001%5D%2C%20%5B127.10648246400001%2C%2037.46243379200001%5D%2C%20%5B127.10435658899996%2C%2037.46218425799998%5D%2C%20%5B127.10479563499996%2C%2037.46116971399999%5D%2C%20%5B127.10395681800003%2C%2037.46006054700001%5D%2C%20%5B127.10139603200003%2C%2037.45898237099999%5D%2C%20%5B127.10129219299995%2C%2037.45825120400002%5D%2C%20%5B127.09975283000006%2C%2037.457364144%5D%2C%20%5B127.099109139%2C%2037.45622823999997%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%209%2C%20%22SHAPE_AREA%22%3A%200.004027%2C%20%22SHAPE_LEN%22%3A%200.348412%2C%20%22SIG_CD%22%3A%20%2211680%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangnam-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub0a8%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.81658266%2C%2037.540568317%5D%2C%20%5B126.81234887999994%2C%2037.54074469400001%5D%2C%20%5B126.81060934799996%2C%2037.54150444200002%5D%2C%20%5B126.80725610699994%2C%2037.54362225199998%5D%2C%20%5B126.80186598399996%2C%2037.542723611999975%5D%2C%20%5B126.80048554400003%2C%2037.54177814600001%5D%2C%20%5B126.80040658300004%2C%2037.540804252999976%5D%2C%20%5B126.79887833%2C%2037.5402999%5D%2C%20%5B126.79875436099996%2C%2037.539406192%5D%2C%20%5B126.79949804199998%2C%2037.537775685999975%5D%2C%20%5B126.79782849100002%2C%2037.537056888999984%5D%2C%20%5B126.79605918200002%2C%2037.53675834400002%5D%2C%20%5B126.79481063799994%2C%2037.535972963%5D%2C%20%5B126.79397960699998%2C%2037.537369168%5D%2C%20%5B126.79390197299995%2C%2037.539558371%5D%2C%20%5B126.79496077700003%2C%2037.541381135999984%5D%2C%20%5B126.79182670499995%2C%2037.54191169%5D%2C%20%5B126.79159995700002%2C%2037.54307308199998%5D%2C%20%5B126.79077654100001%2C%2037.543847862%5D%2C%20%5B126.78872746499997%2C%2037.54482803399998%5D%2C%20%5B126.78727682700003%2C%2037.54608495799999%5D%2C%20%5B126.78332478000004%2C%2037.54603042999997%5D%2C%20%5B126.77875616300003%2C%2037.546729674%5D%2C%20%5B126.77757101300006%2C%2037.54672980700002%5D%2C%20%5B126.77675212199995%2C%2037.548221075000015%5D%2C%20%5B126.775301227%2C%2037.548988871%5D%2C%20%5B126.77258571799996%2C%2037.548773638%5D%2C%20%5B126.77169496199997%2C%2037.548329146000015%5D%2C%20%5B126.76992633199995%2C%2037.550196289999974%5D%2C%20%5B126.769793875%2C%2037.551086956%5D%2C%20%5B126.767468484%2C%2037.551972233000015%5D%2C%20%5B126.76742527299996%2C%2037.554237899999976%5D%2C%20%5B126.76458060899995%2C%2037.555466594999984%5D%2C%20%5B126.76720647299999%2C%2037.55666916000001%5D%2C%20%5B126.76988573200003%2C%2037.55724517599998%5D%2C%20%5B126.772397404%2C%2037.55699389%5D%2C%20%5B126.773200245%2C%2037.557536051%5D%2C%20%5B126.77183969700002%2C%2037.55861259400001%5D%2C%20%5B126.77320943200004%2C%2037.55929739800001%5D%2C%20%5B126.77612545800002%2C%2037.56187228599998%5D%2C%20%5B126.77795148799999%2C%2037.56016005399999%5D%2C%20%5B126.7771186%2C%2037.563728853999976%5D%2C%20%5B126.77515191199996%2C%2037.565895781999984%5D%2C%20%5B126.77478348299996%2C%2037.56772791100002%5D%2C%20%5B126.77635320499996%2C%2037.567107521000025%5D%2C%20%5B126.777826946%2C%2037.567006467%5D%2C%20%5B126.78025671600005%2C%2037.567453419%5D%2C%20%5B126.780491568%2C%2037.56829614100002%5D%2C%20%5B126.781642583%2C%2037.568909259%5D%2C%20%5B126.78133354800002%2C%2037.570185116%5D%2C%20%5B126.782647215%2C%2037.57055544799999%5D%2C%20%5B126.78191796099998%2C%2037.57181951000001%5D%2C%20%5B126.78242742400005%2C%2037.57361885900002%5D%2C%20%5B126.78526689%2C%2037.57487061799998%5D%2C%20%5B126.787630605%2C%2037.57558866%5D%2C%20%5B126.78935921899995%2C%2037.57772807700002%5D%2C%20%5B126.79058199500003%2C%2037.577776819%5D%2C%20%5B126.79074497399995%2C%2037.58061836799999%5D%2C%20%5B126.79228461599996%2C%2037.580056910999986%5D%2C%20%5B126.79275533099997%2C%2037.57684457%5D%2C%20%5B126.79323752200003%2C%2037.57689948500001%5D%2C%20%5B126.79288111400001%2C%2037.58022635499998%5D%2C%20%5B126.79390501399996%2C%2037.582257949%5D%2C%20%5B126.79411019999998%2C%2037.58425737099998%5D%2C%20%5B126.79555674699998%2C%2037.58515790000001%5D%2C%20%5B126.79569588599998%2C%2037.583367872%5D%2C%20%5B126.79711367100003%2C%2037.58427787900001%5D%2C%20%5B126.79670218499996%2C%2037.58491212500002%5D%2C%20%5B126.79878484200003%2C%2037.58807839000002%5D%2C%20%5B126.80075747499995%2C%2037.587841243000014%5D%2C%20%5B126.80053959500003%2C%2037.590315551%5D%2C%20%5B126.798817848%2C%2037.59132633199999%5D%2C%20%5B126.79867010299995%2C%2037.59366814100002%5D%2C%20%5B126.79745919100003%2C%2037.595248595999976%5D%2C%20%5B126.79708180299997%2C%2037.59773291099998%5D%2C%20%5B126.79756259099997%2C%2037.59793496100002%5D%2C%20%5B126.79787158299996%2C%2037.599898203%5D%2C%20%5B126.799935054%2C%2037.601445969%5D%2C%20%5B126.79994633199999%2C%2037.602546226000015%5D%2C%20%5B126.80258612299997%2C%2037.605033999%5D%2C%20%5B126.80587186000002%2C%2037.60258296500001%5D%2C%20%5B126.808249756%2C%2037.601211834000026%5D%2C%20%5B126.81138204800004%2C%2037.59896582800002%5D%2C%20%5B126.81396850600004%2C%2037.59762536400001%5D%2C%20%5B126.816573747%2C%2037.595688145999986%5D%2C%20%5B126.81754378000005%2C%2037.59533660400001%5D%2C%20%5B126.81932148199996%2C%2037.592855004%5D%2C%20%5B126.82783387699999%2C%2037.58748727300002%5D%2C%20%5B126.83178168200004%2C%2037.585846134%5D%2C%20%5B126.837170002%2C%2037.582568493%5D%2C%20%5B126.83811851799999%2C%2037.581798564999986%5D%2C%20%5B126.84326033599996%2C%2037.578886398%5D%2C%20%5B126.84840263399997%2C%2037.57516434500002%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2010%2C%20%22SHAPE_AREA%22%3A%200.004227%2C%20%22SHAPE_LEN%22%3A%200.435694%2C%20%22SIG_CD%22%3A%20%2211500%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangseo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cuc11c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.02338131199997%2C%2037.57191744099998%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96688100899996%2C%2037.56598896600002%5D%2C%20%5B126.96690327500005%2C%2037.566010662%5D%2C%20%5B126.96911916800002%2C%2037.56825034899998%5D%2C%20%5B126.97106417099997%2C%2037.568786338999985%5D%2C%20%5B126.97256951199995%2C%2037.568476987%5D%2C%20%5B126.973221824%2C%2037.56932127499999%5D%2C%20%5B126.97638956699996%2C%2037.56948908200002%5D%2C%20%5B126.98057786100003%2C%2037.568996454%5D%2C%20%5B126.98270044499998%2C%2037.56901410400002%5D%2C%20%5B126.98665814000003%2C%2037.56823705099998%5D%2C%20%5B126.99017151700002%2C%2037.568138827999974%5D%2C%20%5B126.99663981900005%2C%2037.568687872%5D%2C%20%5B126.99685960399995%2C%2037.56871100699999%5D%2C%20%5B126.99704744200005%2C%2037.56873248300002%5D%2C%20%5B127.00080872399997%2C%2037.56938090699998%5D%2C%20%5B127.00132850499995%2C%2037.569477446%5D%2C%20%5B127.01005354999995%2C%2037.569772985999975%5D%2C%20%5B127.01463347699996%2C%2037.569720932999985%5D%2C%20%5B127.01472223099995%2C%2037.569723151%5D%2C%20%5B127.01762628400002%2C%2037.570211508%5D%2C%20%5B127.01779205900004%2C%2037.57029209299998%5D%2C%20%5B127.02079279999998%2C%2037.57172399299998%5D%2C%20%5B127.02338131199997%2C%2037.57191744099998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2011%2C%20%22SHAPE_AREA%22%3A%200.001017%2C%20%22SHAPE_LEN%22%3A%200.191242%2C%20%22SIG_CD%22%3A%20%2211140%22%2C%20%22SIG_ENG_NM%22%3A%20%22Jung-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc911%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.11428417299999%2C%2037.554386276%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11566284000003%2C%2037.557702898%5D%2C%20%5B127.11735874600004%2C%2037.559467202%5D%2C%20%5B127.12011629799997%2C%2037.56120324699998%5D%2C%20%5B127.12297571600004%2C%2037.56349386%5D%2C%20%5B127.12862746099995%2C%2037.56616545000003%5D%2C%20%5B127.134283364%2C%2037.567961086000025%5D%2C%20%5B127.13768321700002%2C%2037.56843244200002%5D%2C%20%5B127.14896618800003%2C%2037.568444026%5D%2C%20%5B127.154998245%2C%2037.57204888699999%5D%2C%20%5B127.15691686699995%2C%2037.572958835%5D%2C%20%5B127.16257241200003%2C%2037.576732698%5D%2C%20%5B127.16676596299999%2C%2037.57897662%5D%2C%20%5B127.17026691299998%2C%2037.57901745999999%5D%2C%20%5B127.17185117300005%2C%2037.57926277399997%5D%2C%20%5B127.17521472800001%2C%2037.58046697200001%5D%2C%20%5B127.17734763199996%2C%2037.581575015%5D%2C%20%5B127.17701410100005%2C%2037.579511809%5D%2C%20%5B127.17550670900005%2C%2037.578472404000024%5D%2C%20%5B127.17534214%2C%2037.57723767800002%5D%2C%20%5B127.17569917399999%2C%2037.57490180399998%5D%2C%20%5B127.17789114499999%2C%2037.57187252799997%5D%2C%20%5B127.17922984100005%2C%2037.56892072800002%5D%2C%20%5B127.17959913300001%2C%2037.56549991899999%5D%2C%20%5B127.181200053%2C%2037.56301996299999%5D%2C%20%5B127.18200949499999%2C%2037.56099529800002%5D%2C%20%5B127.181566868%2C%2037.557559535%5D%2C%20%5B127.18169641500003%2C%2037.55600353400001%5D%2C%20%5B127.18135382599996%2C%2037.552973123000015%5D%2C%20%5B127.18294495500004%2C%2037.55177914199999%5D%2C%20%5B127.183131151%2C%2037.551111615000025%5D%2C%20%5B127.18268913400004%2C%2037.54791615900001%5D%2C%20%5B127.182837425%2C%2037.54639986199999%5D%2C%20%5B127.17934642900002%2C%2037.54657307399998%5D%2C%20%5B127.17573322400006%2C%2037.545213538999974%5D%2C%20%5B127.17392236499995%2C%2037.545588527%5D%2C%20%5B127.17212391600003%2C%2037.545541722999985%5D%2C%20%5B127.16978076600003%2C%2037.544813007000016%5D%2C%20%5B127.16699588400002%2C%2037.54521138699999%5D%2C%20%5B127.16665500399995%2C%2037.54426145999997%5D%2C%20%5B127.16317850999997%2C%2037.544997455999976%5D%2C%20%5B127.16032596900004%2C%2037.541629215%5D%2C%20%5B127.15766934199996%2C%2037.53927536100002%5D%2C%20%5B127.15696225299996%2C%2037.537974744999985%5D%2C%20%5B127.15527845999998%2C%2037.53622368600003%5D%2C%20%5B127.15358375799997%2C%2037.533672017000015%5D%2C%20%5B127.15395052600002%2C%2037.531926252%5D%2C%20%5B127.15270367899996%2C%2037.528535456999975%5D%2C%20%5B127.15041863099998%2C%2037.52637816499998%5D%2C%20%5B127.14781481299997%2C%2037.52215484599998%5D%2C%20%5B127.145682889%2C%2037.52193569399998%5D%2C%20%5B127.144801873%2C%2037.519630431999985%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2012%2C%20%22SHAPE_AREA%22%3A%200.002504%2C%20%22SHAPE_LEN%22%3A%200.242596%2C%20%22SIG_CD%22%3A%20%2211740%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gangdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuac15%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.11526904799996%2C%2037.55676291499998%5D%2C%20%5B127.11411142500003%2C%2037.55409376599999%5D%2C%20%5B127.11154955500001%2C%2037.55038274399999%5D%2C%20%5B127.111480035%2C%2037.54694839699999%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06548714999997%2C%2037.525071298%5D%2C%20%5B127.065474956%2C%2037.52507298699999%5D%2C%20%5B127.06017559300005%2C%2037.52709427399998%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.072410408%2C%2037.55998452%5D%2C%20%5B127.072687901%2C%2037.560686033000025%5D%2C%20%5B127.07269020499996%2C%2037.560691951000024%5D%2C%20%5B127.07438993000005%2C%2037.56492125199998%5D%2C%20%5B127.07604864500001%2C%2037.56691571699997%5D%2C%20%5B127.07689686399999%2C%2037.567936557999985%5D%2C%20%5B127.07800928699999%2C%2037.571244407999984%5D%2C%20%5B127.07822080799997%2C%2037.57187485700001%5D%2C%20%5B127.07824470900005%2C%2037.57189172699998%5D%2C%20%5B127.08334681999997%2C%2037.57137561899998%5D%2C%20%5B127.08575452299999%2C%2037.570925004%5D%2C%20%5B127.09043403600003%2C%2037.56970029799999%5D%2C%20%5B127.09481359899996%2C%2037.570760388%5D%2C%20%5B127.09949334999999%2C%2037.57275202300002%5D%2C%20%5B127.10090644800005%2C%2037.57376806899998%5D%2C%20%5B127.10168325400002%2C%2037.57240515400002%5D%2C%20%5B127.103148411%2C%2037.57228406899998%5D%2C%20%5B127.10424898199994%2C%2037.571392263%5D%2C%20%5B127.10347415499996%2C%2037.57005447199998%5D%2C%20%5B127.10240523599998%2C%2037.56564081%5D%2C%20%5B127.10225269499995%2C%2037.564314869999976%5D%2C%20%5B127.10122363599999%2C%2037.56158417300003%5D%2C%20%5B127.10187808800003%2C%2037.559498245999976%5D%2C%20%5B127.10429532800003%2C%2037.55756740800001%5D%2C%20%5B127.10494775899997%2C%2037.55642552299997%5D%2C%20%5B127.10641446099999%2C%2037.55646542699998%5D%2C%20%5B127.10753783500002%2C%2037.55761021400002%5D%2C%20%5B127.10954748999995%2C%2037.558557784000016%5D%2C%20%5B127.11036889499997%2C%2037.55825685100001%5D%2C%20%5B127.11231650399998%2C%2037.55900405400001%5D%2C%20%5B127.11388262800006%2C%2037.55843289500001%5D%2C%20%5B127.11333075200002%2C%2037.55683318299998%5D%2C%20%5B127.11526904799996%2C%2037.55676291499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2013%2C%20%22SHAPE_AREA%22%3A%200.001737%2C%20%22SHAPE_LEN%22%3A%200.186732%2C%20%22SIG_CD%22%3A%20%2211215%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwangjin-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad11%5Cuc9c4%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.96306480600003%2C%2037.55759743700003%5D%2C%20%5B126.963065465%2C%2037.55744648699999%5D%2C%20%5B126.96337208399996%2C%2037.55668744600001%5D%2C%20%5B126.96239721899997%2C%2037.555754579999984%5D%2C%20%5B126.96233057500001%2C%2037.55568336599998%5D%2C%20%5B126.96220840900003%2C%2037.55557836899999%5D%2C%20%5B126.96215607800002%2C%2037.55553360800002%5D%2C%20%5B126.96179134700003%2C%2037.554985018000025%5D%2C%20%5B126.96234193400005%2C%2037.551993829000025%5D%2C%20%5B126.96230190799997%2C%2037.55188377000002%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.87420295100003%2C%2037.559487495999974%5D%2C%20%5B126.86896973199998%2C%2037.562022953%5D%2C%20%5B126.86630929900002%2C%2037.56305722399998%5D%2C%20%5B126.863843846%2C%2037.56449447900002%5D%2C%20%5B126.85608917399998%2C%2037.569447969%5D%2C%20%5B126.85365036099995%2C%2037.571795441%5D%2C%20%5B126.85364710199997%2C%2037.573804395000025%5D%2C%20%5B126.85930276099998%2C%2037.57492123999998%5D%2C%20%5B126.85931123%2C%2037.575246692%5D%2C%20%5B126.86495888599995%2C%2037.57747814700002%5D%2C%20%5B126.868396404%2C%2037.577526395%5D%2C%20%5B126.870619272%2C%2037.578013113%5D%2C%20%5B126.87393583999994%2C%2037.57828489799999%5D%2C%20%5B126.87628222700005%2C%2037.57818935199998%5D%2C%20%5B126.87765407400002%2C%2037.57975304299998%5D%2C%20%5B126.87668444099995%2C%2037.581263981%5D%2C%20%5B126.87707186%2C%2037.58221029200001%5D%2C%20%5B126.87710903799996%2C%2037.58483216500002%5D%2C%20%5B126.877564388%2C%2037.586252527%5D%2C%20%5B126.87915565399999%2C%2037.58677640399998%5D%2C%20%5B126.88036942999997%2C%2037.58950964899998%5D%2C%20%5B126.88207576399998%2C%2037.59079562900001%5D%2C%20%5B126.88592100400001%2C%2037.58703245200002%5D%2C%20%5B126.88873743800002%2C%2037.584754343999975%5D%2C%20%5B126.89755448699998%2C%2037.57909892700002%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2014%2C%20%22SHAPE_AREA%22%3A%200.002415%2C%20%22SHAPE_LEN%22%3A%200.289336%2C%20%22SIG_CD%22%3A%20%2211440%22%2C%20%22SIG_ENG_NM%22%3A%20%22Mapo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub9c8%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.92849317399998%2C%2037.49483149399998%5D%2C%20%5B126.92850643099996%2C%2037.494823343%5D%2C%20%5B126.92987695900001%2C%2037.494001422%5D%2C%20%5B126.93008725799996%2C%2037.493996605%5D%2C%20%5B126.93135406500005%2C%2037.493270822%5D%2C%20%5B126.93136032799998%2C%2037.49323533299997%5D%2C%20%5B126.93241296199994%2C%2037.49298252300002%5D%2C%20%5B126.932455828%2C%2037.492932689999975%5D%2C%20%5B126.933840153%2C%2037.49315721800002%5D%2C%20%5B126.93538540400004%2C%2037.49277250099999%5D%2C%20%5B126.93656389%2C%2037.491999152%5D%2C%20%5B126.93915324%2C%2037.492247038000016%5D%2C%20%5B126.93925041199998%2C%2037.492214429%5D%2C%20%5B126.93950568000002%2C%2037.492478366%5D%2C%20%5B126.93958069200005%2C%2037.49256420199998%5D%2C%20%5B126.93974068800003%2C%2037.49264855299998%5D%2C%20%5B126.93975036899997%2C%2037.49279093000001%5D%2C%20%5B126.93975406499999%2C%2037.492797403%5D%2C%20%5B126.939963466%2C%2037.492900021000025%5D%2C%20%5B126.94263397199995%2C%2037.49215547%5D%2C%20%5B126.94367456600003%2C%2037.49238951000001%5D%2C%20%5B126.94384122099996%2C%2037.492455155000016%5D%2C%20%5B126.94447754999999%2C%2037.49308442%5D%2C%20%5B126.94454450900002%2C%2037.49310238800001%5D%2C%20%5B126.94698896199998%2C%2037.494088480000016%5D%2C%20%5B126.94718649499998%2C%2037.49401064900002%5D%2C%20%5B126.94834759399998%2C%2037.49369153700002%5D%2C%20%5B126.94882142699998%2C%2037.493310698000016%5D%2C%20%5B126.94958498200003%2C%2037.49274311900001%5D%2C%20%5B126.95171069800006%2C%2037.492183727%5D%2C%20%5B126.95373000300003%2C%2037.49063185699998%5D%2C%20%5B126.95391885499998%2C%2037.49075917599998%5D%2C%20%5B126.95749468500003%2C%2037.491983721%5D%2C%20%5B126.958962595%2C%2037.49361080699998%5D%2C%20%5B126.96030551299998%2C%2037.49374965099997%5D%2C%20%5B126.96139404300004%2C%2037.492879129000016%5D%2C%20%5B126.96141462100002%2C%2037.491560521%5D%2C%20%5B126.96141530700004%2C%2037.491499541999985%5D%2C%20%5B126.96079767900005%2C%2037.49083862700002%5D%2C%20%5B126.960796276%2C%2037.490830448%5D%2C%20%5B126.96196229300006%2C%2037.48860120799998%5D%2C%20%5B126.961909881%2C%2037.48541996599999%5D%2C%20%5B126.96200037799997%2C%2037.48518336799998%5D%2C%20%5B126.96108986000002%2C%2037.483568373000026%5D%2C%20%5B126.96206297900005%2C%2037.48245202700002%5D%2C%20%5B126.96395912900005%2C%2037.48102475600001%5D%2C%20%5B126.96442976799995%2C%2037.479822540999976%5D%2C%20%5B126.96635728900003%2C%2037.478930159000015%5D%2C%20%5B126.96779700900004%2C%2037.47784124200001%5D%2C%20%5B126.96806025700005%2C%2037.477500217%5D%2C%20%5B126.968845%2C%2037.476608681000016%5D%2C%20%5B126.97029023300001%2C%2037.476489499000024%5D%2C%20%5B126.97053839099999%2C%2037.47538229899999%5D%2C%20%5B126.97430276399996%2C%2037.47620602799998%5D%2C%20%5B126.97844049499997%2C%2037.47665586599999%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98662702599995%2C%2037.457216786%5D%2C%20%5B126.98290313899997%2C%2037.456831314%5D%2C%20%5B126.98241282200001%2C%2037.455895618%5D%2C%20%5B126.97849284100005%2C%2037.455739153000025%5D%2C%20%5B126.97777249%2C%2037.455203476%5D%2C%20%5B126.97459526800003%2C%2037.45441720500003%5D%2C%20%5B126.97285906000002%2C%2037.45238675000002%5D%2C%20%5B126.97176838200005%2C%2037.45174170500002%5D%2C%20%5B126.97058548899997%2C%2037.449457128%5D%2C%20%5B126.96767973600004%2C%2037.44838496%5D%2C%20%5B126.964307298%2C%2037.446271943%5D%2C%20%5B126.96395069300002%2C%2037.44521969200002%5D%2C%20%5B126.96443595200003%2C%2037.44431798300002%5D%2C%20%5B126.96464967600002%2C%2037.44204566000002%5D%2C%20%5B126.96295770799998%2C%2037.44028699400002%5D%2C%20%5B126.96007066799996%2C%2037.44040704499997%5D%2C%20%5B126.95898017000002%2C%2037.43907666199999%5D%2C%20%5B126.95512268899995%2C%2037.43869914800001%5D%2C%20%5B126.95242507900002%2C%2037.43919711699999%5D%2C%20%5B126.95148094199999%2C%2037.43858365%5D%2C%20%5B126.94929766799999%2C%2037.43825732%5D%2C%20%5B126.94837678299996%2C%2037.43871966299997%5D%2C%20%5B126.94680271000004%2C%2037.43817427499999%5D%2C%20%5B126.94510345499998%2C%2037.43709568600002%5D%2C%20%5B126.94143729300004%2C%2037.437411664000024%5D%2C%20%5B126.940232358%2C%2037.435720186000026%5D%2C%20%5B126.93858713099996%2C%2037.43642808300001%5D%2C%20%5B126.93775941000001%2C%2037.437393122%5D%2C%20%5B126.93726096700004%2C%2037.439219657000024%5D%2C%20%5B126.93787838100002%2C%2037.44020405700002%5D%2C%20%5B126.93666192499995%2C%2037.44175181600002%5D%2C%20%5B126.93176733099995%2C%2037.444982075999974%5D%2C%20%5B126.93057246299998%2C%2037.445474494%5D%2C%20%5B126.93018248700002%2C%2037.44638402800001%5D%2C%20%5B126.93018641599997%2C%2037.448390587%5D%2C%20%5B126.92841761800003%2C%2037.450219984%5D%2C%20%5B126.92848764999997%2C%2037.45074018600002%5D%2C%20%5B126.928471113%2C%2037.45081842500002%5D%2C%20%5B126.927343597%2C%2037.45100593799998%5D%2C%20%5B126.92629329099998%2C%2037.452544569%5D%2C%20%5B126.92480360499997%2C%2037.452836038999976%5D%2C%20%5B126.92336253200006%2C%2037.45420163599999%5D%2C%20%5B126.92226575100005%2C%2037.45658259099997%5D%2C%20%5B126.92220212799998%2C%2037.45663504800001%5D%2C%20%5B126.921334011%2C%2037.45677051400003%5D%2C%20%5B126.92089818700003%2C%2037.45683564799998%5D%2C%20%5B126.91969206600004%2C%2037.45676440400001%5D%2C%20%5B126.91643131199999%2C%2037.457390843999974%5D%2C%20%5B126.91627684800005%2C%2037.457347287%5D%2C%20%5B126.91404088499996%2C%2037.457896307%5D%2C%20%5B126.91440474800004%2C%2037.46158584%5D%2C%20%5B126.91331723999997%2C%2037.463054605000025%5D%2C%20%5B126.91314457399994%2C%2037.46330904000001%5D%2C%20%5B126.913549322%2C%2037.46512347999999%5D%2C%20%5B126.91370338299998%2C%2037.46538695700002%5D%2C%20%5B126.91239499200003%2C%2037.46588423600002%5D%2C%20%5B126.91105616599998%2C%2037.468088841999986%5D%2C%20%5B126.90998565899997%2C%2037.468795648000025%5D%2C%20%5B126.910061799%2C%2037.4689239%5D%2C%20%5B126.909609091%2C%2037.47070677800002%5D%2C%20%5B126.90822009800002%2C%2037.47269093699998%5D%2C%20%5B126.90821443000004%2C%2037.47273741700002%5D%2C%20%5B126.90858257399998%2C%2037.47296422900001%5D%2C%20%5B126.90933983599996%2C%2037.47319861%5D%2C%20%5B126.910771221%2C%2037.473607087%5D%2C%20%5B126.91132204500002%2C%2037.47476077499999%5D%2C%20%5B126.91132182599995%2C%2037.47479031400002%5D%2C%20%5B126.91184131600005%2C%2037.47781328299999%5D%2C%20%5B126.90949628800001%2C%2037.47802530199999%5D%2C%20%5B126.90948000100002%2C%2037.478035697999985%5D%2C%20%5B126.90871552199997%2C%2037.479385396%5D%2C%20%5B126.908714454%2C%2037.479390465999984%5D%2C%20%5B126.90980158499997%2C%2037.48077253399998%5D%2C%20%5B126.90525384399996%2C%2037.48012032299999%5D%2C%20%5B126.903848273%2C%2037.47986373499998%5D%2C%20%5B126.899010599%2C%2037.47915640799999%5D%2C%20%5B126.89903022099998%2C%2037.47923194499998%5D%2C%20%5B126.89910722900004%2C%2037.479485625%5D%2C%20%5B126.89911828200002%2C%2037.47951944%5D%2C%20%5B126.89929689999997%2C%2037.47996366799998%5D%2C%20%5B126.89930376500001%2C%2037.47997831999999%5D%2C%20%5B126.89933846400004%2C%2037.480050456000015%5D%2C%20%5B126.89937228500003%2C%2037.48011921099999%5D%2C%20%5B126.89942496499998%2C%2037.48022064700001%5D%2C%20%5B126.89948170699995%2C%2037.48032097%5D%2C%20%5B126.89949563699997%2C%2037.48034464199998%5D%2C%20%5B126.899597331%2C%2037.480512615%5D%2C%20%5B126.899628904%2C%2037.48056220699999%5D%2C%20%5B126.89972993599997%2C%2037.48071195900002%5D%2C%20%5B126.89985975699994%2C%2037.480888762%5D%2C%20%5B126.89990903499995%2C%2037.48095182399999%5D%2C%20%5B126.90015273200004%2C%2037.48126779900002%5D%2C%20%5B126.90074018500002%2C%2037.48198279799999%5D%2C%20%5B126.90083500900005%2C%2037.482098784000016%5D%2C%20%5B126.90089647599996%2C%2037.482174309000015%5D%2C%20%5B126.90093521899996%2C%2037.482221658000014%5D%2C%20%5B126.90121553300003%2C%2037.48256557299999%5D%2C%20%5B126.90125409100006%2C%2037.48261293799999%5D%2C%20%5B126.90219688699995%2C%2037.483766118%5D%2C%20%5B126.902322%2C%2037.48391963400002%5D%2C%20%5B126.90256421100003%2C%2037.484218433000024%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90611436799998%2C%2037.48496907100002%5D%2C%20%5B126.91228813299995%2C%2037.486756008999976%5D%2C%20%5B126.91232745699995%2C%2037.48678281100001%5D%2C%20%5B126.912432309%2C%2037.48690065%5D%2C%20%5B126.91246614399995%2C%2037.48693814400002%5D%2C%20%5B126.91309285900002%2C%2037.48740636799999%5D%2C%20%5B126.91320422900003%2C%2037.48753885899998%5D%2C%20%5B126.91451151700005%2C%2037.48850797099999%5D%2C%20%5B126.91485424400003%2C%2037.488683408999975%5D%2C%20%5B126.91666942100005%2C%2037.489563437000015%5D%2C%20%5B126.91872214099999%2C%2037.490153088%5D%2C%20%5B126.91902122299996%2C%2037.490212586999974%5D%2C%20%5B126.92213104300004%2C%2037.49017681999999%5D%2C%20%5B126.92217789200004%2C%2037.490162742999985%5D%2C%20%5B126.92417074800005%2C%2037.48997139699998%5D%2C%20%5B126.924779651%2C%2037.490848184000015%5D%2C%20%5B126.92479305400002%2C%2037.490864250000016%5D%2C%20%5B126.92512815500004%2C%2037.49166297199997%5D%2C%20%5B126.92513396899994%2C%2037.491671419%5D%2C%20%5B126.92611525699999%2C%2037.49318942799999%5D%2C%20%5B126.92615198700003%2C%2037.493207196000014%5D%2C%20%5B126.92620227500004%2C%2037.493263265%5D%2C%20%5B126.92623723999998%2C%2037.493281869999976%5D%2C%20%5B126.92768716199998%2C%2037.49503581%5D%2C%20%5B126.92779671000005%2C%2037.495058132999986%5D%2C%20%5B126.92789513100001%2C%2037.49507819500002%5D%2C%20%5B126.92849317399998%2C%2037.49483149399998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2015%2C%20%22SHAPE_AREA%22%3A%200.003012%2C%20%22SHAPE_LEN%22%3A%200.280092%2C%20%22SIG_CD%22%3A%20%2211620%22%2C%20%22SIG_ENG_NM%22%3A%20%22Gwanak-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuad00%5Cuc545%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.03410511300001%2C%2037.484373084000026%5D%2C%20%5B127.03462801199998%2C%2037.484446408999986%5D%2C%20%5B127.03743121399998%2C%2037.48483468400002%5D%2C%20%5B127.03775238399999%2C%2037.484880825%5D%2C%20%5B127.03812275300004%2C%2037.48493503399999%5D%2C%20%5B127.038293802%2C%2037.484959043%5D%2C%20%5B127.04162613699998%2C%2037.48524357000002%5D%2C%20%5B127.04390604000002%2C%2037.480564211%5D%2C%20%5B127.04402181499995%2C%2037.479945714%5D%2C%20%5B127.04505713499998%2C%2037.47732183099998%5D%2C%20%5B127.05084195200004%2C%2037.471641508%5D%2C%20%5B127.04868603%2C%2037.47006535999998%5D%2C%20%5B127.05105024299996%2C%2037.46936231900003%5D%2C%20%5B127.05077112699996%2C%2037.46730241099999%5D%2C%20%5B127.05081771300001%2C%2037.46727397699999%5D%2C%20%5B127.05540305600005%2C%2037.46882300300001%5D%2C%20%5B127.06004414200004%2C%2037.469096901%5D%2C%20%5B127.060433397%2C%2037.46911046600002%5D%2C%20%5B127.06511498999998%2C%2037.469340827%5D%2C%20%5B127.06858970500002%2C%2037.47101304900002%5D%2C%20%5B127.07043815500003%2C%2037.47105584500002%5D%2C%20%5B127.072700038%2C%2037.47196330100002%5D%2C%20%5B127.07386142099995%2C%2037.47294075799999%5D%2C%20%5B127.07538860299996%2C%2037.473494921%5D%2C%20%5B127.07643531099995%2C%2037.47442209799999%5D%2C%20%5B127.07645818200001%2C%2037.47444214799998%5D%2C%20%5B127.07692872400003%2C%2037.47516319499999%5D%2C%20%5B127.079009015%2C%2037.47477282%5D%2C%20%5B127.08452367699999%2C%2037.475620564%5D%2C%20%5B127.08419105799999%2C%2037.47282587299998%5D%2C%20%5B127.08504561100006%2C%2037.471207665%5D%2C%20%5B127.08660644500003%2C%2037.47076724300001%5D%2C%20%5B127.08754885400003%2C%2037.46976963999998%5D%2C%20%5B127.08818068200003%2C%2037.46832342099998%5D%2C%20%5B127.09045288000004%2C%2037.46703465500002%5D%2C%20%5B127.09202158699998%2C%2037.464931131000014%5D%2C%20%5B127.092039956%2C%2037.46228391%5D%2C%20%5B127.09320980099994%2C%2037.46128151%5D%2C%20%5B127.09569257800001%2C%2037.46101392600002%5D%2C%20%5B127.094911626%2C%2037.45848998600002%5D%2C%20%5B127.095462623%2C%2037.457491342000026%5D%2C%20%5B127.09523686299997%2C%2037.45640378600001%5D%2C%20%5B127.093554412%2C%2037.45589802400002%5D%2C%20%5B127.09273821199997%2C%2037.453643511999985%5D%2C%20%5B127.09097172%2C%2037.45279201300002%5D%2C%20%5B127.09031546300002%2C%2037.45127626300001%5D%2C%20%5B127.088339345%2C%2037.44863720799998%5D%2C%20%5B127.08817389900003%2C%2037.44539478899998%5D%2C%20%5B127.08643383499998%2C%2037.444272048000016%5D%2C%20%5B127.08387430300002%2C%2037.44393121899998%5D%2C%20%5B127.08244417699996%2C%2037.441589282%5D%2C%20%5B127.08018384599995%2C%2037.441060924%5D%2C%20%5B127.07602324899995%2C%2037.4421491%5D%2C%20%5B127.07300687899999%2C%2037.442027784%5D%2C%20%5B127.07163946599997%2C%2037.441534323999974%5D%2C%20%5B127.07202035099999%2C%2037.43886650600001%5D%2C%20%5B127.07377263199999%2C%2037.4377991%5D%2C%20%5B127.07303654400005%2C%2037.43641643799998%5D%2C%20%5B127.07145178799999%2C%2037.43587310999999%5D%2C%20%5B127.07153192800001%2C%2037.434126424%5D%2C%20%5B127.07064098%2C%2037.432054365%5D%2C%20%5B127.07121871100003%2C%2037.430827679%5D%2C%20%5B127.06828150800004%2C%2037.43068911%5D%2C%20%5B127.06673287599995%2C%2037.43013986099999%5D%2C%20%5B127.06569412299996%2C%2037.42899537400001%5D%2C%20%5B127.06332888700001%2C%2037.429761353%5D%2C%20%5B127.06115775299997%2C%2037.429994996%5D%2C%20%5B127.05997391899996%2C%2037.429578751%5D%2C%20%5B127.05808265300004%2C%2037.430020472000024%5D%2C%20%5B127.05369652000002%2C%2037.428984660000026%5D%2C%20%5B127.05218338899999%2C%2037.42834757000003%5D%2C%20%5B127.04959235199999%2C%2037.430279794%5D%2C%20%5B127.04738197699999%2C%2037.43071133900003%5D%2C%20%5B127.04723810300004%2C%2037.43204956%5D%2C%20%5B127.04632535799999%2C%2037.43345562899998%5D%2C%20%5B127.04496506500004%2C%2037.43385551799997%5D%2C%20%5B127.044199833%2C%2037.435063092%5D%2C%20%5B127.04006758800006%2C%2037.438243274%5D%2C%20%5B127.03708267800005%2C%2037.43828114399997%5D%2C%20%5B127.03558860400005%2C%2037.43901011600002%5D%2C%20%5B127.03518600799998%2C%2037.440947884000025%5D%2C%20%5B127.03644468300001%2C%2037.44183865799999%5D%2C%20%5B127.03749597800004%2C%2037.443272594%5D%2C%20%5B127.038243717%2C%2037.44518611900003%5D%2C%20%5B127.03726765%2C%2037.44645425300001%5D%2C%20%5B127.03717336600005%2C%2037.44801560799999%5D%2C%20%5B127.03782636200003%2C%2037.44892774700003%5D%2C%20%5B127.03692812600002%2C%2037.45081401300001%5D%2C%20%5B127.03579206200004%2C%2037.452201682%5D%2C%20%5B127.03477066100004%2C%2037.45262298599999%5D%2C%20%5B127.03714478699999%2C%2037.45521278799998%5D%2C%20%5B127.03675713500002%2C%2037.45644861599999%5D%2C%20%5B127.03503165799998%2C%2037.45789780400003%5D%2C%20%5B127.03494060599996%2C%2037.46018196900002%5D%2C%20%5B127.033746501%2C%2037.461242513%5D%2C%20%5B127.03471186900003%2C%2037.46416056200002%5D%2C%20%5B127.03315839200002%2C%2037.465024089999986%5D%2C%20%5B127.03121260499995%2C%2037.46563217200003%5D%2C%20%5B127.02954249100003%2C%2037.46537973%5D%2C%20%5B127.02970909099997%2C%2037.463910751000014%5D%2C%20%5B127.02933072500002%2C%2037.46270655%5D%2C%20%5B127.02805724300003%2C%2037.46125024600002%5D%2C%20%5B127.02837573700003%2C%2037.459898619%5D%2C%20%5B127.02658642100005%2C%2037.459646512%5D%2C%20%5B127.02645936199997%2C%2037.45834107600001%5D%2C%20%5B127.02529273699997%2C%2037.457498327%5D%2C%20%5B127.02280887500001%2C%2037.457254868%5D%2C%20%5B127.02152010400005%2C%2037.456229913000016%5D%2C%20%5B127.01970823600004%2C%2037.45579098899998%5D%2C%20%5B127.01756149699997%2C%2037.45613975200001%5D%2C%20%5B127.01630291799995%2C%2037.455234355000016%5D%2C%20%5B127.01452250900002%2C%2037.45486602300002%5D%2C%20%5B127.01066817699996%2C%2037.456042489000026%5D%2C%20%5B127.008544057%2C%2037.45805628699998%5D%2C%20%5B127.00822678700001%2C%2037.459104508%5D%2C%20%5B127.00705377999998%2C%2037.460220048999986%5D%2C%20%5B127.00578293399997%2C%2037.46255271299998%5D%2C%20%5B127.00448335800002%2C%2037.464077615%5D%2C%20%5B127.00492006399998%2C%2037.46584683899999%5D%2C%20%5B127.00369035799997%2C%2037.46772444499999%5D%2C%20%5B127.00274926199995%2C%2037.467128762000016%5D%2C%20%5B126.99879405800004%2C%2037.46723823899998%5D%2C%20%5B126.99676674199998%2C%2037.467076564000024%5D%2C%20%5B126.996245907%2C%2037.46661254999998%5D%2C%20%5B126.99663361800003%2C%2037.46478279199999%5D%2C%20%5B126.997384828%2C%2037.46369182699999%5D%2C%20%5B126.996782414%2C%2037.461877791%5D%2C%20%5B126.993368691%2C%2037.46150029400002%5D%2C%20%5B126.99266931099999%2C%2037.46034838899999%5D%2C%20%5B126.99171279500001%2C%2037.46052342799999%5D%2C%20%5B126.99000889599995%2C%2037.45946741%5D%2C%20%5B126.98865006699998%2C%2037.45817040899999%5D%2C%20%5B126.98767243999998%2C%2037.459731788%5D%2C%20%5B126.98765602200001%2C%2037.45977936700001%5D%2C%20%5B126.98753916199996%2C%2037.460673848%5D%2C%20%5B126.98835488199995%2C%2037.46400365400001%5D%2C%20%5B126.98774291200004%2C%2037.46641709099998%5D%2C%20%5B126.98773406199996%2C%2037.466432028999975%5D%2C%20%5B126.98763787300004%2C%2037.466598523000016%5D%2C%20%5B126.98761753600002%2C%2037.466633178%5D%2C%20%5B126.98694022699999%2C%2037.467976436000015%5D%2C%20%5B126.98403984100003%2C%2037.47054971799997%5D%2C%20%5B126.98300881299997%2C%2037.472016715%5D%2C%20%5B126.98300404999998%2C%2037.472022621%5D%2C%20%5B126.98217250599998%2C%2037.473727688%5D%2C%20%5B126.981705294%2C%2037.47653534300002%5D%2C%20%5B126.98220285000002%2C%2037.486734081%5D%2C%20%5B126.98266488800004%2C%2037.49074970700002%5D%2C%20%5B126.98266648699996%2C%2037.490795875%5D%2C%20%5B126.9826971%2C%2037.49167122900002%5D%2C%20%5B126.98267928999996%2C%2037.49188692199999%5D%2C%20%5B126.98284317800005%2C%2037.493331137999974%5D%2C%20%5B126.98285657099996%2C%2037.49383837800002%5D%2C%20%5B126.98294082100006%2C%2037.496998254%5D%2C%20%5B126.98378613499995%2C%2037.498036680999974%5D%2C%20%5B126.98451928700001%2C%2037.49885515400001%5D%2C%20%5B126.98540105999996%2C%2037.499865699%5D%2C%20%5B126.98204229199996%2C%2037.50204822500001%5D%2C%20%5B126.98055896999995%2C%2037.50269589800001%5D%2C%20%5B126.97985151099999%2C%2037.504182980999985%5D%2C%20%5B126.98041422699998%2C%2037.504594743999974%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.01319429499995%2C%2037.52259168299997%5D%2C%20%5B127.01524810199999%2C%2037.524849631%5D%2C%20%5B127.01783754500002%2C%2037.52181485800003%5D%2C%20%5B127.020530345%2C%2037.51280506699999%5D%2C%20%5B127.03410511300001%2C%2037.484373084000026%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2016%2C%20%22SHAPE_AREA%22%3A%200.004776%2C%20%22SHAPE_LEN%22%3A%200.437399%2C%20%22SIG_CD%22%3A%20%2211650%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seocho-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cucd08%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.98593509%2C%2037.635792451999976%5D%2C%20%5B126.98941711299994%2C%2037.634040097000025%5D%2C%20%5B126.98994433099995%2C%2037.632808297999986%5D%2C%20%5B126.98995070900003%2C%2037.632802389%5D%2C%20%5B126.99138411900003%2C%2037.632614284%5D%2C%20%5B126.99139668199996%2C%2037.63260893400002%5D%2C%20%5B126.993710029%2C%2037.63147100600003%5D%2C%20%5B126.99606041200002%2C%2037.62934997000002%5D%2C%20%5B126.99651955000002%2C%2037.62909314900003%5D%2C%20%5B126.99653639500002%2C%2037.62907062900001%5D%2C%20%5B126.999288385%2C%2037.62619623799998%5D%2C%20%5B127.00178266700004%2C%2037.62564837799999%5D%2C%20%5B127.00353743699998%2C%2037.624932172%5D%2C%20%5B127.00390656800005%2C%2037.624230626999974%5D%2C%20%5B127.005583291%2C%2037.623851102%5D%2C%20%5B127.00560114899997%2C%2037.62385308199998%5D%2C%20%5B127.00787101200001%2C%2037.62402666700001%5D%2C%20%5B127.00759821999998%2C%2037.62150355099999%5D%2C%20%5B127.00759276199994%2C%2037.62147681599998%5D%2C%20%5B127.00722642899996%2C%2037.62083427599998%5D%2C%20%5B127.00726484300003%2C%2037.62079262399999%5D%2C%20%5B127.00758730500002%2C%2037.620050427000024%5D%2C%20%5B127.00760977899995%2C%2037.620025661%5D%2C%20%5B127.00788007699998%2C%2037.61986667000002%5D%2C%20%5B127.00788272700004%2C%2037.619854284999974%5D%2C%20%5B127.00796057000002%2C%2037.619416897%5D%2C%20%5B127.00796570199998%2C%2037.61941407900002%5D%2C%20%5B127.00804872000003%2C%2037.61939945300003%5D%2C%20%5B127.00818324500005%2C%2037.61932601%5D%2C%20%5B127.01007020099996%2C%2037.617665294%5D%2C%20%5B127.01015525499997%2C%2037.616951777%5D%2C%20%5B127.01013187700005%2C%2037.616900831%5D%2C%20%5B127.010954792%2C%2037.61629594599998%5D%2C%20%5B127.01096629699998%2C%2037.61629482799998%5D%2C%20%5B127.01341735899996%2C%2037.615541442999984%5D%2C%20%5B127.01446604199998%2C%2037.614686769%5D%2C%20%5B127.01654109699996%2C%2037.614942274999976%5D%2C%20%5B127.01655437600004%2C%2037.614943660999984%5D%2C%20%5B127.01779510300003%2C%2037.61443326400001%5D%2C%20%5B127.01783574700005%2C%2037.61439327300002%5D%2C%20%5B127.01783646499996%2C%2037.614380595%5D%2C%20%5B127.01855824699999%2C%2037.61385730500001%5D%2C%20%5B127.01856391700005%2C%2037.61385730699999%5D%2C%20%5B127.02041404199997%2C%2037.61260025299998%5D%2C%20%5B127.020449934%2C%2037.61258928500001%5D%2C%20%5B127.02067693799995%2C%2037.61251758499998%5D%2C%20%5B127.02068348900002%2C%2037.612515615%5D%2C%20%5B127.02068949800002%2C%2037.61251392299999%5D%2C%20%5B127.02198785200005%2C%2037.61227773899998%5D%2C%20%5B127.02215458%2C%2037.61138467%5D%2C%20%5B127.02221682100003%2C%2037.61137453399999%5D%2C%20%5B127.02251655999999%2C%2037.611371716%5D%2C%20%5B127.02252327999997%2C%2037.611371717%5D%2C%20%5B127.02259567399994%2C%2037.61140409199999%5D%2C%20%5B127.02331110900002%2C%2037.61168840900001%5D%2C%20%5B127.02346174900003%2C%2037.61173710899999%5D%2C%20%5B127.02449958199998%2C%2037.61208337%5D%2C%20%5B127.02458433799995%2C%2037.61214500599999%5D%2C%20%5B127.026847095%2C%2037.61268268700002%5D%2C%20%5B127.02685433900001%2C%2037.61268156%5D%2C%20%5B127.02686016999996%2C%2037.612680434000026%5D%2C%20%5B127.03019669399998%2C%2037.612365453999985%5D%2C%20%5B127.03036127799999%2C%2037.611311376%5D%2C%20%5B127.03036198400002%2C%2037.61130124099998%5D%2C%20%5B127.03033118400003%2C%2037.610609613%5D%2C%20%5B127.030330467%2C%2037.610591597%5D%2C%20%5B127.03026655899998%2C%2037.60898282900001%5D%2C%20%5B127.030275577%2C%2037.60897803400002%5D%2C%20%5B127.03178102499999%2C%2037.609805274999985%5D%2C%20%5B127.031977966%2C%2037.60989752099999%5D%2C%20%5B127.03353454199998%2C%2037.61077921100002%5D%2C%20%5B127.03535125500002%2C%2037.61195842699999%5D%2C%20%5B127.03601975599997%2C%2037.612413091%5D%2C%20%5B127.03606542199998%2C%2037.61243531399998%5D%2C%20%5B127.03612823399999%2C%2037.612430785000015%5D%2C%20%5B127.03611259299998%2C%2037.61234323899998%5D%2C%20%5B127.03619275999995%2C%2037.61236094499998%5D%2C%20%5B127.03678252300006%2C%2037.612423431000025%5D%2C%20%5B127.03679686199996%2C%2037.61242877199999%5D%2C%20%5B127.03710027199998%2C%2037.61264394599999%5D%2C%20%5B127.03728985600003%2C%2037.61277641200002%5D%2C%20%5B127.03729570200005%2C%2037.61278035399999%5D%2C%20%5B127.03731216300002%2C%2037.612791883%5D%2C%20%5B127.03732331100002%2C%2037.612799758999984%5D%2C%20%5B127.03732757099999%2C%2037.61280296299998%5D%2C%20%5B127.03733853899996%2C%2037.61281128899998%5D%2C%20%5B127.03735800499999%2C%2037.612827924999976%5D%2C%20%5B127.03744388300004%2C%2037.612902435000024%5D%2C%20%5B127.03747361900002%2C%2037.612919594%5D%2C%20%5B127.03753415300002%2C%2037.61296318400002%5D%2C%20%5B127.03772882600003%2C%2037.613237%5D%2C%20%5B127.03779662700003%2C%2037.61347235400001%5D%2C%20%5B127.03786860399998%2C%2037.61363029%5D%2C%20%5B127.03792604900002%2C%2037.61376991399999%5D%2C%20%5B127.038167058%2C%2037.61394701900002%5D%2C%20%5B127.03822599299997%2C%2037.614018536%5D%2C%20%5B127.03831447100004%2C%2037.61409062400003%5D%2C%20%5B127.038606179%2C%2037.61421053%5D%2C%20%5B127.03866921199995%2C%2037.61433866099998%5D%2C%20%5B127.03910116899999%2C%2037.614654383000016%5D%2C%20%5B127.039605768%2C%2037.615113173%5D%2C%20%5B127.03963124999996%2C%2037.61513035799999%5D%2C%20%5B127.03965673200003%2C%2037.615147525%5D%2C%20%5B127.039713205%2C%2037.615220478000026%5D%2C%20%5B127.03991002700002%2C%2037.615490928999975%5D%2C%20%5B127.039916329%2C%2037.61557761199998%5D%2C%20%5B127.04000740900005%2C%2037.615690684000015%5D%2C%20%5B127.04001805200005%2C%2037.61571826300002%5D%2C%20%5B127.04003139099996%2C%2037.61577425500002%5D%2C%20%5B127.04050711900004%2C%2037.61614092399998%5D%2C%20%5B127.04064321099997%2C%2037.61624918000001%5D%2C%20%5B127.04086447600002%2C%2037.616453288%5D%2C%20%5B127.04117637000002%2C%2037.616684943%5D%2C%20%5B127.04118716599999%2C%2037.61669112200002%5D%2C%20%5B127.04128895600002%2C%2037.61676294199998%5D%2C%20%5B127.04136046999997%2C%2037.61681905099999%5D%2C%20%5B127.04213794500004%2C%2037.617349734000015%5D%2C%20%5B127.04235451700004%2C%2037.61750541599997%5D%2C%20%5B127.04241255199997%2C%2037.61754623500002%5D%2C%20%5B127.04259303200001%2C%2037.61767405099999%5D%2C%20%5B127.04322434699998%2C%2037.61812223599998%5D%2C%20%5B127.04336341800001%2C%2037.618221052000024%5D%2C%20%5B127.04354584999999%2C%2037.618350263000025%5D%2C%20%5B127.04360530099996%2C%2037.61839249600001%5D%2C%20%5B127.04368969100005%2C%2037.618451881%5D%2C%20%5B127.04379101400002%2C%2037.618523692999986%5D%2C%20%5B127.04403884299995%2C%2037.618728153%5D%2C%20%5B127.04411290899998%2C%2037.61880926499998%5D%2C%20%5B127.04446284899996%2C%2037.61919059799999%5D%2C%20%5B127.04458966300001%2C%2037.619328316%5D%2C%20%5B127.04469692199996%2C%2037.619446289%5D%2C%20%5B127.04472315400005%2C%2037.61949583799998%5D%2C%20%5B127.04484228299998%2C%2037.619719374%5D%2C%20%5B127.04487012599998%2C%2037.61977202100002%5D%2C%20%5B127.04491674500002%2C%2037.61985901000003%5D%2C%20%5B127.04504615400003%2C%2037.62010083299998%5D%2C%20%5B127.04506742800004%2C%2037.62014109400002%5D%2C%20%5B127.045232113%2C%2037.620450204%5D%2C%20%5B127.04561107300003%2C%2037.62115659699998%5D%2C%20%5B127.04564041900005%2C%2037.62121022500003%5D%2C%20%5B127.04572401200005%2C%2037.62136557000002%5D%2C%20%5B127.045898975%2C%2037.62160363800001%5D%2C%20%5B127.04593864200001%2C%2037.62165794600003%5D%2C%20%5B127.04609890799998%2C%2037.62187771599997%5D%2C%20%5B127.04611484099996%2C%2037.621893473%5D%2C%20%5B127.04632095399995%2C%2037.622099144%5D%2C%20%5B127.04647694599998%2C%2037.62225755600002%5D%2C%20%5B127.04651138600002%2C%2037.62229258899998%5D%2C%20%5B127.04657823499997%2C%2037.622357163%5D%2C%20%5B127.04691712700003%2C%2037.622624711000014%5D%2C%20%5B127.04745571900003%2C%2037.62294930500002%5D%2C%20%5B127.04773847299998%2C%2037.623120035%5D%2C%20%5B127.04792455799998%2C%2037.62322692599997%5D%2C%20%5B127.048244221%2C%2037.623409261%5D%2C%20%5B127.048542902%2C%2037.62357842199998%5D%2C%20%5B127.048679268%2C%2037.62365724799997%5D%2C%20%5B127.049259227%2C%2037.623987839999984%5D%2C%20%5B127.04930364200004%2C%2037.62401347000002%5D%2C%20%5B127.04937497000003%2C%2037.624054301%5D%2C%20%5B127.04972276700005%2C%2037.624250871000015%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.07174587700001%2C%2037.61350877899997%5D%2C%20%5B127.07107677700003%2C%2037.60763421199999%5D%2C%20%5B127.07058840599996%2C%2037.607474744%5D%2C%20%5B127.07054181599995%2C%2037.60736919200002%5D%2C%20%5B127.07038655099996%2C%2037.60704174099999%5D%2C%20%5B127.07037116799995%2C%2037.607043714999975%5D%2C%20%5B127.07031648899999%2C%2037.60704093099997%5D%2C%20%5B127.07027209199998%2C%2037.607038699999976%5D%2C%20%5B127.07007576599995%2C%2037.607271646000015%5D%2C%20%5B127.07000072100004%2C%2037.60726378499999%5D%2C%20%5B127.06991702000005%2C%2037.607276763000016%5D%2C%20%5B127.06986888300003%2C%2037.60728325399998%5D%2C%20%5B127.06914080299998%2C%2037.60696620800002%5D%2C%20%5B127.06908467999995%2C%2037.60691836400002%5D%2C%20%5B127.068754894%2C%2037.606784183%5D%2C%20%5B127.06875011199998%2C%2037.60677995999998%5D%2C%20%5B127.06745069800002%2C%2037.606344561000014%5D%2C%20%5B127.06740715%2C%2037.60632739499999%5D%2C%20%5B127.06553217600003%2C%2037.60548780099998%5D%2C%20%5B127.06499814899996%2C%2037.60583985300002%5D%2C%20%5B127.063113488%2C%2037.605255578000026%5D%2C%20%5B127.063105006%2C%2037.60525528300002%5D%2C%20%5B127.06225792099997%2C%2037.60514261899999%5D%2C%20%5B127.06221588899996%2C%2037.60512405600002%5D%2C%20%5B127.06220437599995%2C%2037.60512885600002%5D%2C%20%5B127.06218159900004%2C%2037.60503369899999%5D%2C%20%5B127.062194815%2C%2037.60494078200003%5D%2C%20%5B127.062176539%2C%2037.60493121500002%5D%2C%20%5B127.06211410599997%2C%2037.60489943800002%5D%2C%20%5B127.06203822199996%2C%2037.604900688999976%5D%2C%20%5B127.06197777600005%2C%2037.60489697600002%5D%2C%20%5B127.06194548600001%2C%2037.60487250599999%5D%2C%20%5B127.06145471399998%2C%2037.604243739000026%5D%2C%20%5B127.06078071699994%2C%2037.60348731%5D%2C%20%5B127.06072758799996%2C%2037.60353464100001%5D%2C%20%5B127.06044313300004%2C%2037.60341788699998%5D%2C%20%5B127.06011685099998%2C%2037.60305839599999%5D%2C%20%5B127.06015501299999%2C%2037.60284661100002%5D%2C%20%5B127.06007566799997%2C%2037.602564743000016%5D%2C%20%5B127.059971087%2C%2037.602449326%5D%2C%20%5B127.05917634800005%2C%2037.60106011800002%5D%2C%20%5B127.057226162%2C%2037.601507706%5D%2C%20%5B127.05210293100004%2C%2037.60015112299999%5D%2C%20%5B127.05181798700005%2C%2037.60049124300002%5D%2C%20%5B127.05122101500001%2C%2037.601158829999974%5D%2C%20%5B127.04976516800002%2C%2037.599760435%5D%2C%20%5B127.049419231%2C%2037.59914950899997%5D%2C%20%5B127.04934047100005%2C%2037.59903162400002%5D%2C%20%5B127.04931551699997%2C%2037.598995322%5D%2C%20%5B127.04907651400003%2C%2037.59813784099998%5D%2C%20%5B127.047323603%2C%2037.59615608299998%5D%2C%20%5B127.046786602%2C%2037.59604565500001%5D%2C%20%5B127.04582490200005%2C%2037.596436617999984%5D%2C%20%5B127.04554542200003%2C%2037.596613837%5D%2C%20%5B127.04502417000003%2C%2037.59644353800002%5D%2C%20%5B127.04480958700003%2C%2037.596369%5D%2C%20%5B127.04480326500004%2C%2037.596366948000025%5D%2C%20%5B127.04479507600001%2C%2037.59636563700002%5D%2C%20%5B127.04466137300005%2C%2037.596371523000016%5D%2C%20%5B127.04217442499998%2C%2037.59655887500003%5D%2C%20%5B127.04210851300002%2C%2037.59653585799998%5D%2C%20%5B127.04187491300002%2C%2037.59645644400001%5D%2C%20%5B127.04185933999997%2C%2037.59646177399998%5D%2C%20%5B127.041176146%2C%2037.59551177100002%5D%2C%20%5B127.04116319299999%2C%2037.595522003999974%5D%2C%20%5B127.04109137499995%2C%2037.59555297100002%5D%2C%20%5B127.04077860799998%2C%2037.59529079700002%5D%2C%20%5B127.04018348399995%2C%2037.594576378%5D%2C%20%5B127.03870036199999%2C%2037.591242098%5D%2C%20%5B127.03741559399998%2C%2037.591235691%5D%2C%20%5B127.03624678899996%2C%2037.591189344999975%5D%2C%20%5B127.03624037400004%2C%2037.59112516800002%5D%2C%20%5B127.03631229500002%2C%2037.590483587999984%5D%2C%20%5B127.03631616500002%2C%2037.59044811199999%5D%2C%20%5B127.03631968000002%2C%2037.59042221200002%5D%2C%20%5B127.03632108700003%2C%2037.590410104%5D%2C%20%5B127.036349041%2C%2037.590144916999975%5D%2C%20%5B127.036310823%2C%2037.590121297999985%5D%2C%20%5B127.036073364%2C%2037.58970006499999%5D%2C%20%5B127.03464513999995%2C%2037.58816959500001%5D%2C%20%5B127.03461099200001%2C%2037.58814172799998%5D%2C%20%5B127.03454005499998%2C%2037.58808008099999%5D%2C%20%5B127.03448608999997%2C%2037.58803223199999%5D%2C%20%5B127.034356409%2C%2037.587917385000026%5D%2C%20%5B127.03431377899994%2C%2037.58787995%5D%2C%20%5B127.03402734300005%2C%2037.58763054999997%5D%2C%20%5B127.033898195%2C%2037.58751824400002%5D%2C%20%5B127.03368372199998%2C%2037.587283161000016%5D%2C%20%5B127.03215982899997%2C%2037.58580831799998%5D%2C%20%5B127.03219093999996%2C%2037.58580378800002%5D%2C%20%5B127.03224306899995%2C%2037.585780645%5D%2C%20%5B127.03225331199997%2C%2037.58577162199998%5D%2C%20%5B127.03227752099997%2C%2037.585752737%5D%2C%20%5B127.03231900900005%2C%2037.585681145000024%5D%2C%20%5B127.03231422700003%2C%2037.58567270100002%5D%2C%20%5B127.03229460099999%2C%2037.58566398%5D%2C%20%5B127.03216406599995%2C%2037.58559307899998%5D%2C%20%5B127.03213293800002%2C%2037.585561548999976%5D%2C%20%5B127.032094906%2C%2037.58554213899998%5D%2C%20%5B127.032069621%2C%2037.58553144699999%5D%2C%20%5B127.03196863000005%2C%2037.58547686999998%5D%2C%20%5B127.031937847%2C%2037.58545238%5D%2C%20%5B127.03176161199997%2C%2037.585370849000014%5D%2C%20%5B127.03172519999998%2C%2037.58535847799999%5D%2C%20%5B127.03171778%2C%2037.58535764599998%5D%2C%20%5B127.03169214599995%2C%2037.585368931%5D%2C%20%5B127.031656096%2C%2037.58535994800002%5D%2C%20%5B127.03163506600004%2C%2037.585288709999986%5D%2C%20%5B127.031613516%2C%2037.585211012%5D%2C%20%5B127.03160591799997%2C%2037.58518369500001%5D%2C%20%5B127.03156828199997%2C%2037.58511585799999%5D%2C%20%5B127.03145077800002%2C%2037.58493628799999%5D%2C%20%5B127.03142779699999%2C%2037.58490081799999%5D%2C%20%5B127.03142002699997%2C%2037.58489125400001%5D%2C%20%5B127.03135640799997%2C%2037.58482117699998%5D%2C%20%5B127.03124631599997%2C%2037.58470270700002%5D%2C%20%5B127.03120743700003%2C%2037.584633466000014%5D%2C%20%5B127.03118393700004%2C%2037.58459180599999%5D%2C%20%5B127.03102506100004%2C%2037.58431903000002%5D%2C%20%5B127.03097093500003%2C%2037.58425484600002%5D%2C%20%5B127.030668977%2C%2037.58391760500001%5D%2C%20%5B127.02955829200005%2C%2037.582683187999976%5D%2C%20%5B127.02941127899999%2C%2037.582585255000026%5D%2C%20%5B127.02903897299996%2C%2037.582339558%5D%2C%20%5B127.02901207299999%2C%2037.58232830100002%5D%2C%20%5B127.02897081799995%2C%2037.58228663099999%5D%2C%20%5B127.02656569600003%2C%2037.580639682000026%5D%2C%20%5B127.02649454200002%2C%2037.58058815700002%5D%2C%20%5B127.02562530099999%2C%2037.57998934400001%5D%2C%20%5B127.02518450499997%2C%2037.57969053800002%5D%2C%20%5B127.02499392799996%2C%2037.57956392199998%5D%2C%20%5B127.02452820200006%2C%2037.579252171%5D%2C%20%5B127.02442432299995%2C%2037.579190847%5D%2C%20%5B127.02411572999995%2C%2037.57897784800002%5D%2C%20%5B127.02390019300003%2C%2037.578828155%5D%2C%20%5B127.02388603700001%2C%2037.57881830399998%5D%2C%20%5B127.02388072600002%2C%2037.578814360000024%5D%2C%20%5B127.02386923100005%2C%2037.578806482%5D%2C%20%5B127.023723593%2C%2037.57870518499999%5D%2C%20%5B127.02337217399997%2C%2037.57846460799999%5D%2C%20%5B127.02335253700005%2C%2037.57845139%5D%2C%20%5B127.02334315999997%2C%2037.57844519499997%5D%2C%20%5B127.02332139400005%2C%2037.57843056399997%5D%2C%20%5B127.02328617299997%2C%2037.57838496199997%5D%2C%20%5B127.023284827%2C%2037.578203561%5D%2C%20%5B127.02323896200005%2C%2037.57811120299999%5D%2C%20%5B127.02315595799996%2C%2037.57802393899999%5D%2C%20%5B127.02314959399996%2C%2037.578020563999985%5D%2C%20%5B127.02313206600002%2C%2037.57799268799999%5D%2C%20%5B127.02304763999996%2C%2037.577891912999974%5D%2C%20%5B127.02299347200005%2C%2037.57783026300001%5D%2C%20%5B127.02290061199994%2C%2037.57783033099997%5D%2C%20%5B127.022663276%2C%2037.57821303399999%5D%2C%20%5B127.02271016999998%2C%2037.578264261000015%5D%2C%20%5B127.02272327599997%2C%2037.57827834%5D%2C%20%5B127.02277460100004%2C%2037.578338019%5D%2C%20%5B127.02266513200004%2C%2037.57837724699999%5D%2C%20%5B127.02253038799995%2C%2037.57844467000001%5D%2C%20%5B127.02244622900002%2C%2037.578522196999984%5D%2C%20%5B127.02238842500003%2C%2037.57858110799998%5D%2C%20%5B127.022302674%2C%2037.578645395000024%5D%2C%20%5B127.02228145000004%2C%2037.578649917%5D%2C%20%5B127.02217180800005%2C%2037.57868549400001%5D%2C%20%5B127.02199791400005%2C%2037.57874641%5D%2C%20%5B127.02197863399999%2C%2037.57875543099999%5D%2C%20%5B127.02194521000001%2C%2037.578792899%5D%2C%20%5B127.02193442199996%2C%2037.57879825700002%5D%2C%20%5B127.02186313799996%2C%2037.57884390800001%5D%2C%20%5B127.02184243500005%2C%2037.578865888%5D%2C%20%5B127.02181006700005%2C%2037.57887519500002%5D%2C%20%5B127.02172073%2C%2037.57886482499998%5D%2C%20%5B127.02166092599998%2C%2037.578866541000025%5D%2C%20%5B127.021423153%2C%2037.57885905799998%5D%2C%20%5B127.02127966499995%2C%2037.578823376%5D%2C%20%5B127.02108045099999%2C%2037.57875814800002%5D%2C%20%5B127.02099410599999%2C%2037.578724686999976%5D%2C%20%5B127.020914994%2C%2037.57869740899997%5D%2C%20%5B127.02072599200005%2C%2037.57861272500003%5D%2C%20%5B127.02055096799995%2C%2037.57853902800002%5D%2C%20%5B127.02051698900004%2C%2037.578524399%5D%2C%20%5B127.02050849499994%2C%2037.578521024%5D%2C%20%5B127.02041805600004%2C%2037.578485297999976%5D%2C%20%5B127.02038160899997%2C%2037.57847096%5D%2C%20%5B127.02031400299995%2C%2037.578444515%5D%2C%20%5B127.020291702%2C%2037.57843579199999%5D%2C%20%5B127.02016516699996%2C%2037.578386566%5D%2C%20%5B127.020139514%2C%2037.57837644799997%5D%2C%20%5B127.02013348800006%2C%2037.578374198%5D%2C%20%5B127.02012800700004%2C%2037.57837193900002%5D%2C%20%5B127.02011685000002%2C%2037.578367727%5D%2C%20%5B127.02007527299997%2C%2037.57835169600003%5D%2C%20%5B127.02002730699996%2C%2037.578333127%5D%2C%20%5B127.02001598100003%2C%2037.57832862599997%5D%2C%20%5B127.01954934000003%2C%2037.57814926100002%5D%2C%20%5B127.01943184200002%2C%2037.578103082999974%5D%2C%20%5B127.01938812499998%2C%2037.57808467299998%5D%2C%20%5B127.01951868399999%2C%2037.57791229600002%5D%2C%20%5B127.01920376199996%2C%2037.57781827399998%5D%2C%20%5B127.019130023%2C%2037.577736458%5D%2C%20%5B127.01903796399995%2C%2037.577709954%5D%2C%20%5B127.01900254700001%2C%2037.57769330999997%5D%2C%20%5B127.01895261100003%2C%2037.577669336999975%5D%2C%20%5B127.01843394900004%2C%2037.577674480999974%5D%2C%20%5B127.01812307600005%2C%2037.577581986999974%5D%2C%20%5B127.01803500999995%2C%2037.577740839%5D%2C%20%5B127.017940995%2C%2037.57783028799997%5D%2C%20%5B127.01783020899995%2C%2037.57794316799999%5D%2C%20%5B127.01779503199998%2C%2037.577965455000026%5D%2C%20%5B127.01782213499996%2C%2037.57798182099998%5D%2C%20%5B127.01770944999998%2C%2037.57811776300002%5D%2C%20%5B127.01771104099998%2C%2037.578143391000026%5D%2C%20%5B127.01829582000005%2C%2037.57903378100002%5D%2C%20%5B127.01828290399999%2C%2037.57906786799998%5D%2C%20%5B127.01785779399995%2C%2037.57958825899999%5D%2C%20%5B127.017714651%2C%2037.579672238%5D%2C%20%5B127.01761307499999%2C%2037.579711410000016%5D%2C%20%5B127.017571696%2C%2037.57979309799998%5D%2C%20%5B127.01754959000004%2C%2037.57985224700002%5D%2C%20%5B127.017503423%2C%2037.579929986000025%5D%2C%20%5B127.01736491500003%2C%2037.580199806%5D%2C%20%5B127.01734918199998%2C%2037.580242903%5D%2C%20%5B127.01734387099998%2C%2037.580245158000025%5D%2C%20%5B127.017319455%2C%2037.580253331999984%5D%2C%20%5B127.01731768399998%2C%2037.580287403%5D%2C%20%5B127.01732177600002%2C%2037.580356111000015%5D%2C%20%5B127.01729314%2C%2037.58055796399998%5D%2C%20%5B127.01726995399997%2C%2037.580565837999984%5D%2C%20%5B127.01717561700002%2C%2037.58070056499997%5D%2C%20%5B127.01716180899996%2C%2037.580719412%5D%2C%20%5B127.01715401599995%2C%2037.580727287%5D%2C%20%5B127.01704995499995%2C%2037.58078296999997%5D%2C%20%5B127.01695544300003%2C%2037.58083556600002%5D%2C%20%5B127.01693916600004%2C%2037.58084428699999%5D%2C%20%5B127.01693225600002%2C%2037.58084878699998%5D%2C%20%5B127.01682728699996%2C%2037.58118145399999%5D%2C%20%5B127.01682086899996%2C%2037.58122125599999%5D%2C%20%5B127.01684323200004%2C%2037.581435011999986%5D%2C%20%5B127.01671805499996%2C%2037.58167123499999%5D%2C%20%5B127.01669759599997%2C%2037.58172104200003%5D%2C%20%5B127.01649181799996%2C%2037.581826879%5D%2C%20%5B127.01639821100002%2C%2037.58193191399999%5D%2C%20%5B127.01637224199999%2C%2037.58194230599997%5D%2C%20%5B127.01629053900001%2C%2037.58192435199999%5D%2C%20%5B127.016141995%2C%2037.58189125199999%5D%2C%20%5B127.01599220399999%2C%2037.58185617700002%5D%2C%20%5B127.01577594399998%2C%2037.581817753%5D%2C%20%5B127.01562687%2C%2037.58178605400002%5D%2C%20%5B127.01535740700001%2C%2037.581942218999984%5D%2C%20%5B127.01534489599999%2C%2037.581980973999976%5D%2C%20%5B127.01533182799994%2C%2037.58199360899999%5D%2C%20%5B127.01529065499994%2C%2037.58201944899997%5D%2C%20%5B127.01526220599999%2C%2037.58203910399999%5D%2C%20%5B127.01524984800005%2C%2037.582049214999984%5D%2C%20%5B127.01519684100003%2C%2037.582087977000015%5D%2C%20%5B127.015083763%2C%2037.582162959000016%5D%2C%20%5B127.015017984%2C%2037.582159322999985%5D%2C%20%5B127.01484401100004%2C%2037.582330915%5D%2C%20%5B127.01287077999996%2C%2037.581363708000026%5D%2C%20%5B127.01210020200006%2C%2037.58157834000002%5D%2C%20%5B127.01180557299995%2C%2037.581573408%5D%2C%20%5B127.01060709499995%2C%2037.58025685000001%5D%2C%20%5B127.00866045299995%2C%2037.58047182299998%5D%2C%20%5B127.00802758400005%2C%2037.581232381%5D%2C%20%5B127.00798028899999%2C%2037.58128610199998%5D%2C%20%5B127.00687707700001%2C%2037.58226271799998%5D%2C%20%5B127.00677574300005%2C%2037.582349131%5D%2C%20%5B127.00708698400001%2C%2037.584095407%5D%2C%20%5B127.00702772900001%2C%2037.584297263999986%5D%2C%20%5B127.00667273299996%2C%2037.58542715499999%5D%2C%20%5B127.00656853800001%2C%2037.58555034599999%5D%2C%20%5B127.00588462999997%2C%2037.586976170000014%5D%2C%20%5B127.00427042700005%2C%2037.587180580999984%5D%2C%20%5B127.00402086099996%2C%2037.587806682%5D%2C%20%5B127.00391033599999%2C%2037.587750976%5D%2C%20%5B127.00400656700003%2C%2037.587947825000015%5D%2C%20%5B127.003735237%2C%2037.58804613%5D%2C%20%5B127.003584101%2C%2037.58809865500001%5D%2C%20%5B127.00353556599998%2C%2037.588148122%5D%2C%20%5B127.00348107599996%2C%2037.588332267%5D%2C%20%5B127.00349018400004%2C%2037.588417744000026%5D%2C%20%5B127.003433449%2C%2037.588533482%5D%2C%20%5B127.00341152099998%2C%2037.58857149800002%5D%2C%20%5B127.00337137400004%2C%2037.588606137%5D%2C%20%5B127.00295885699995%2C%2037.58873192499999%5D%2C%20%5B127.00284086800002%2C%2037.588774417000025%5D%2C%20%5B127.00264718200003%2C%2037.58888929599999%5D%2C%20%5B127.002362759%2C%2037.589045828%5D%2C%20%5B127.00216566100005%2C%2037.58947028099999%5D%2C%20%5B127.00202422200005%2C%2037.58968687%5D%2C%20%5B127.00188646900006%2C%2037.58986401200002%5D%2C%20%5B127.00168484699998%2C%2037.590025077%5D%2C%20%5B127.001488193%2C%2037.590151290999984%5D%2C%20%5B127.00149218599995%2C%2037.590268622%5D%2C%20%5B127.00149026199995%2C%2037.59035090899999%5D%2C%20%5B127.00148307400002%2C%2037.59055380799998%5D%2C%20%5B127.00150304399995%2C%2037.59054930799999%5D%2C%20%5B127.00151065499995%2C%2037.59057355099998%5D%2C%20%5B127.00156054399997%2C%2037.59070067099998%5D%2C%20%5B127.00157275900006%2C%2037.59072180499999%5D%2C%20%5B127.00162987800002%2C%2037.590806094000015%5D%2C%20%5B127.00166839300005%2C%2037.59087848399997%5D%2C%20%5B127.00175147499999%2C%2037.59135529600002%5D%2C%20%5B127.00146400100004%2C%2037.59183083099998%5D%2C%20%5B127.00139247499999%2C%2037.591914438%5D%2C%20%5B127.00114493%2C%2037.59209822499997%5D%2C%20%5B127.00097196000002%2C%2037.59230373499997%5D%2C%20%5B127.000771987%2C%2037.59232392400003%5D%2C%20%5B127.00054479799996%2C%2037.59231594%5D%2C%20%5B127.000458708%2C%2037.59230970200002%5D%2C%20%5B126.99992558899999%2C%2037.59221569099998%5D%2C%20%5B126.99978748299998%2C%2037.592239565%5D%2C%20%5B126.999220735%2C%2037.592331969999975%5D%2C%20%5B126.99887956600003%2C%2037.592434036999975%5D%2C%20%5B126.99882209099997%2C%2037.592448096%5D%2C%20%5B126.99880140100004%2C%2037.59245315200002%5D%2C%20%5B126.99874833700005%2C%2037.59246946600001%5D%2C%20%5B126.998461927%2C%2037.59246933600002%5D%2C%20%5B126.99827015999995%2C%2037.592433862%5D%2C%20%5B126.99808352900004%2C%2037.59239670699998%5D%2C%20%5B126.99789920199999%2C%2037.592356172999985%5D%2C%20%5B126.99780313999997%2C%2037.592333653000026%5D%2C%20%5B126.99769419300003%2C%2037.59229310799998%5D%2C%20%5B126.99767403199996%2C%2037.59228663%5D%2C%20%5B126.99699232299997%2C%2037.592219684999975%5D%2C%20%5B126.99688508500003%2C%2037.59222054600002%5D%2C%20%5B126.99679094800001%2C%2037.59222309%5D%2C%20%5B126.99652759900005%2C%2037.59212369800002%5D%2C%20%5B126.99643401499998%2C%2037.59210343%5D%2C%20%5B126.99620448200005%2C%2037.59212542099999%5D%2C%20%5B126.99612293999996%2C%2037.59213246600001%5D%2C%20%5B126.99585864200003%2C%2037.59219925899998%5D%2C%20%5B126.995462287%2C%2037.592270008000014%5D%2C%20%5B126.99538217700001%2C%2037.59228212300002%5D%2C%20%5B126.995277558%2C%2037.592220462%5D%2C%20%5B126.99516545300003%2C%2037.59206471800002%5D%2C%20%5B126.99511839000002%2C%2037.592022192%5D%2C%20%5B126.99497434299997%2C%2037.591904485999976%5D%2C%20%5B126.99361781100004%2C%2037.591698483000016%5D%2C%20%5B126.99360945599994%2C%2037.591714535999984%5D%2C%20%5B126.98921696599996%2C%2037.59127129199999%5D%2C%20%5B126.988536684%2C%2037.59194487100001%5D%2C%20%5B126.98823492999998%2C%2037.59204277600003%5D%2C%20%5B126.98708071099998%2C%2037.59273273000002%5D%2C%20%5B126.98671764899996%2C%2037.59275293500002%5D%2C%20%5B126.98628759099995%2C%2037.593279661%5D%2C%20%5B126.98619199200004%2C%2037.59351277799999%5D%2C%20%5B126.986139761%2C%2037.59370113199998%5D%2C%20%5B126.986138148%2C%2037.59378024900002%5D%2C%20%5B126.98591781599998%2C%2037.59401474700002%5D%2C%20%5B126.98593320400005%2C%2037.59406909799998%5D%2C%20%5B126.98579833099996%2C%2037.59425828299999%5D%2C%20%5B126.98544884499995%2C%2037.594452768%5D%2C%20%5B126.98501447000001%2C%2037.594459727000014%5D%2C%20%5B126.984898036%2C%2037.594469839%5D%2C%20%5B126.98466149599994%2C%2037.59439883700003%5D%2C%20%5B126.98463495399994%2C%2037.59439235299999%5D%2C%20%5B126.98452348599994%2C%2037.594377968%5D%2C%20%5B126.98386333400003%2C%2037.59443864799999%5D%2C%20%5B126.98348621000002%2C%2037.594687749%5D%2C%20%5B126.98157477400002%2C%2037.595319717999985%5D%2C%20%5B126.98150735700005%2C%2037.59533884899997%5D%2C%20%5B126.98104525300005%2C%2037.59576052800003%5D%2C%20%5B126.98092950900002%2C%2037.595841870000015%5D%2C%20%5B126.980741771%2C%2037.59586518899999%5D%2C%20%5B126.98064745%2C%2037.59591641499998%5D%2C%20%5B126.979376548%2C%2037.59635449699999%5D%2C%20%5B126.97912987200004%2C%2037.596397232000015%5D%2C%20%5B126.97867132800002%2C%2037.59670261000002%5D%2C%20%5B126.97859079700004%2C%2037.596788753%5D%2C%20%5B126.97837223600004%2C%2037.597443618%5D%2C%20%5B126.97818395000002%2C%2037.597503252000024%5D%2C%20%5B126.977972443%2C%2037.597686785%5D%2C%20%5B126.97731953200002%2C%2037.597662961000026%5D%2C%20%5B126.97692404400004%2C%2037.59771916699998%5D%2C%20%5B126.976779427%2C%2037.59786694600001%5D%2C%20%5B126.97673747800002%2C%2037.59790888600003%5D%2C%20%5B126.97704825000005%2C%2037.598293591000015%5D%2C%20%5B126.97716090799997%2C%2037.59845805399999%5D%2C%20%5B126.97722807900004%2C%2037.59864586899999%5D%2C%20%5B126.97743354800002%2C%2037.59910375099997%5D%2C%20%5B126.97743167399994%2C%2037.599394603%5D%2C%20%5B126.97743502900005%2C%2037.599419382%5D%2C%20%5B126.97770160699997%2C%2037.60023288799999%5D%2C%20%5B126.97795426000005%2C%2037.60030982799998%5D%2C%20%5B126.97826925000004%2C%2037.60080094800003%5D%2C%20%5B126.97852099399995%2C%2037.60100318000002%5D%2C%20%5B126.98049376100005%2C%2037.601473871999985%5D%2C%20%5B126.98068837899996%2C%2037.60162532200002%5D%2C%20%5B126.98130635099994%2C%2037.601830588999974%5D%2C%20%5B126.98258556899998%2C%2037.602260254999976%5D%2C%20%5B126.98400154199999%2C%2037.603210456%5D%2C%20%5B126.985776633%2C%2037.60345489000002%5D%2C%20%5B126.98595470199996%2C%2037.603646089999984%5D%2C%20%5B126.98668572600002%2C%2037.60402012399999%5D%2C%20%5B126.98679855199998%2C%2037.60447453099999%5D%2C%20%5B126.98677423200002%2C%2037.60609022199998%5D%2C%20%5B126.98613063899995%2C%2037.607393840999976%5D%2C%20%5B126.98685213700003%2C%2037.60887230100002%5D%2C%20%5B126.98589052099999%2C%2037.60946627300001%5D%2C%20%5B126.98566685599997%2C%2037.611381728000026%5D%2C%20%5B126.986203477%2C%2037.61302193900002%5D%2C%20%5B126.98615749500004%2C%2037.61322589500003%5D%2C%20%5B126.98660388400003%2C%2037.61398160800002%5D%2C%20%5B126.98543426200001%2C%2037.615143476000014%5D%2C%20%5B126.98518296600002%2C%2037.61550731400001%5D%2C%20%5B126.98456299700001%2C%2037.61600205000002%5D%2C%20%5B126.98318453700006%2C%2037.61925462200003%5D%2C%20%5B126.98366887300006%2C%2037.621149323%5D%2C%20%5B126.98184958499996%2C%2037.624325628%5D%2C%20%5B126.98170777999997%2C%2037.624499043000014%5D%2C%20%5B126.97962309599995%2C%2037.62637744900002%5D%2C%20%5B126.98020520900002%2C%2037.62793079199997%5D%2C%20%5B126.98000165400003%2C%2037.628434196%5D%2C%20%5B126.979949228%2C%2037.62856370700001%5D%2C%20%5B126.97955639300005%2C%2037.628890227%5D%2C%20%5B126.979544713%2C%2037.628896414%5D%2C%20%5B126.979525943%2C%2037.62890288699998%5D%2C%20%5B126.97942842299994%2C%2037.628935523%5D%2C%20%5B126.979315315%2C%2037.62897378500003%5D%2C%20%5B126.97845846999996%2C%2037.62928975300002%5D%2C%20%5B126.978438116%2C%2037.629291711%5D%2C%20%5B126.97664316400005%2C%2037.629126176%5D%2C%20%5B126.97638232199995%2C%2037.62910300200002%5D%2C%20%5B126.97497686500003%2C%2037.629001760999984%5D%2C%20%5B126.975114716%2C%2037.631235275999984%5D%2C%20%5B126.97538881000003%2C%2037.63170052800001%5D%2C%20%5B126.97782373300004%2C%2037.63391334300002%5D%2C%20%5B126.981433809%2C%2037.634832856%5D%2C%20%5B126.983724113%2C%2037.636460397%5D%2C%20%5B126.98593509%2C%2037.635792451999976%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2017%2C%20%22SHAPE_AREA%22%3A%200.002511%2C%20%22SHAPE_LEN%22%3A%200.318673%2C%20%22SIG_CD%22%3A%20%2211290%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongbuk-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cubd81%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.105568661%2C%2037.62037839200002%5D%2C%20%5B127.10170814399999%2C%2037.62003044099998%5D%2C%20%5B127.10171305200004%2C%2037.620010164%5D%2C%20%5B127.10081617100002%2C%2037.61949304199999%5D%2C%20%5B127.09913280399996%2C%2037.62022241300002%5D%2C%20%5B127.09904414100004%2C%2037.620277621000014%5D%2C%20%5B127.09903953000003%2C%2037.620280724%5D%2C%20%5B127.09903334499995%2C%2037.62028467499999%5D%2C%20%5B127.09902731700004%2C%2037.62028833699998%5D%2C%20%5B127.09899740900005%2C%2037.620306938%5D%2C%20%5B127.09899191399995%2C%2037.62031033%5D%2C%20%5B127.098982364%2C%2037.62031624799999%5D%2C%20%5B127.09897705100002%2C%2037.62031962999998%5D%2C%20%5B127.09897208799998%2C%2037.620322724%5D%2C%20%5B127.09896536999997%2C%2037.620326955%5D%2C%20%5B127.09895845999995%2C%2037.62033118599999%5D%2C%20%5B127.09895190999998%2C%2037.62033457000001%5D%2C%20%5B127.098946597%2C%2037.620337673999984%5D%2C%20%5B127.09893986600002%2C%2037.62034133700001%5D%2C%20%5B127.09842738099996%2C%2037.62018371300002%5D%2C%20%5B127.09842188899995%2C%2037.620181703000014%5D%2C%20%5B127.098218975%2C%2037.620106846%5D%2C%20%5B127.09795415500002%2C%2037.620011589%5D%2C%20%5B127.09794724899996%2C%2037.620009290999974%5D%2C%20%5B127.09793009600003%2C%2037.62000297600002%5D%2C%20%5B127.09784305300002%2C%2037.61997198400002%5D%2C%20%5B127.09766066899999%2C%2037.619906852999975%5D%2C%20%5B127.09765429499998%2C%2037.619904556999984%5D%2C%20%5B127.097647219%2C%2037.61990196900001%5D%2C%20%5B127.09763890800002%2C%2037.61989910199998%5D%2C%20%5B127.09759591900001%2C%2037.61988360399999%5D%2C%20%5B127.097550984%2C%2037.619868386%5D%2C%20%5B127.09753807699997%2C%2037.61986407500001%5D%2C%20%5B127.09736214600002%2C%2037.61980634299999%5D%2C%20%5B127.09735330199999%2C%2037.619803252999986%5D%2C%20%5B127.09734692699999%2C%2037.61980100699998%5D%2C%20%5B127.097341616%2C%2037.619799039999975%5D%2C%20%5B127.097335252%2C%2037.619796802999986%5D%2C%20%5B127.09732976%2C%2037.61979483599998%5D%2C%20%5B127.09732109799995%2C%2037.619791745999976%5D%2C%20%5B127.09731490399997%2C%2037.619789500000024%5D%2C%20%5B127.09730889100001%2C%2037.619787255%5D%2C%20%5B127.097298099%2C%2037.61978332000001%5D%2C%20%5B127.097292788%2C%2037.619781362000026%5D%2C%20%5B127.09728606199997%2C%2037.619779117%5D%2C%20%5B127.09727951699995%2C%2037.619776582999975%5D%2C%20%5B127.09723157300004%2C%2037.61975946199999%5D%2C%20%5B127.097196005%2C%2037.61974655400002%5D%2C%20%5B127.09718999200004%2C%2037.619744587000014%5D%2C%20%5B127.09681756199996%2C%2037.619612114%5D%2C%20%5B127.09677633599995%2C%2037.61959868600002%5D%2C%20%5B127.09668664900005%2C%2037.619569313%5D%2C%20%5B127.09667426099998%2C%2037.61956511300002%5D%2C%20%5B127.09666594999999%2C%2037.61956231099998%5D%2C%20%5B127.09663064699998%2C%2037.61943297099998%5D%2C%20%5B127.09662108600003%2C%2037.61942369299999%5D%2C%20%5B127.09661773200003%2C%2037.61942003799999%5D%2C%20%5B127.09661100300002%2C%2037.61941356800003%5D%2C%20%5B127.09660232600004%2C%2037.61940485600002%5D%2C%20%5B127.09659135100003%2C%2037.61939391300001%5D%2C%20%5B127.09658693300003%2C%2037.61938942%5D%2C%20%5B127.096582323%2C%2037.619385206%5D%2C%20%5B127.09657667%2C%2037.61937959%5D%2C%20%5B127.09656994199997%2C%2037.61937285099998%5D%2C%20%5B127.09656444799998%2C%2037.61936723500003%5D%2C%20%5B127.09655719800003%2C%2037.619360217%5D%2C%20%5B127.09654427299995%2C%2037.619347017%5D%2C%20%5B127.096540377%2C%2037.61934336899998%5D%2C%20%5B127.09653577799997%2C%2037.61933887599997%5D%2C%20%5B127.09652604799999%2C%2037.61932904600002%5D%2C%20%5B127.0965218%2C%2037.61932483099997%5D%2C%20%5B127.09651808399997%2C%2037.61932118300001%5D%2C%20%5B127.09651383599999%2C%2037.61931696900001%5D%2C%20%5B127.09650975900001%2C%2037.61931304199999%5D%2C%20%5B127.096506224%2C%2037.61930939400003%5D%2C%20%5B127.09650161399998%2C%2037.619304613%5D%2C%20%5B127.09649506699998%2C%2037.61929844000002%5D%2C%20%5B127.09649153299995%2C%2037.619294792%5D%2C%20%5B127.09648781700002%2C%2037.619291135000026%5D%2C%20%5B127.09648373899995%2C%2037.619287208%5D%2C%20%5B127.09647967299998%2C%2037.619282994%5D%2C%20%5B127.09647524399998%2C%2037.61927878%5D%2C%20%5B127.09647170999995%2C%2037.61927513099999%5D%2C%20%5B127.09646196799997%2C%2037.619265300999984%5D%2C%20%5B127.09645808300002%2C%2037.619261653000024%5D%2C%20%5B127.09645383500003%2C%2037.61925716000002%5D%2C%20%5B127.09644993799998%2C%2037.619253512%5D%2C%20%5B127.09644409299995%2C%2037.61924789599999%5D%2C%20%5B127.09644020799999%2C%2037.61924396%5D%2C%20%5B127.09643631100005%2C%2037.619239755000024%5D%2C%20%5B127.09643099899995%2C%2037.61923469599998%5D%2C%20%5B127.09642693199999%2C%2037.61923076900001%5D%2C%20%5B127.096421438%2C%2037.619225153%5D%2C%20%5B127.09641312400004%2C%2037.61921700300002%5D%2C%20%5B127.096226753%2C%2037.61903167299999%5D%2C%20%5B127.09621985599995%2C%2037.619026335%5D%2C%20%5B127.09621136199996%2C%2037.619021852%5D%2C%20%5B127.09446646399999%2C%2037.61900122999998%5D%2C%20%5B127.09367062700005%2C%2037.618075807000025%5D%2C%20%5B127.09365859900004%2C%2037.61807475099999%5D%2C%20%5B127.09323919099995%2C%2037.618044079000015%5D%2C%20%5B127.09324431499999%2C%2037.618037035999976%5D%2C%20%5B127.093071195%2C%2037.61808083300002%5D%2C%20%5B127.09303491900005%2C%2037.61807861%5D%2C%20%5B127.09294828600002%2C%2037.618105437%5D%2C%20%5B127.09293749100004%2C%2037.61810881600002%5D%2C%20%5B127.09142105%2C%2037.61895247699999%5D%2C%20%5B127.09140032200003%2C%2037.61896286299998%5D%2C%20%5B127.091292834%2C%2037.618991188%5D%2C%20%5B127.09118711199994%2C%2037.61901528599998%5D%2C%20%5B127.09109751200003%2C%2037.619052334%5D%2C%20%5B127.09088696000003%2C%2037.61912584100003%5D%2C%20%5B127.09058359100004%2C%2037.61929405%5D%2C%20%5B127.09038681000004%2C%2037.619356568%5D%2C%20%5B127.090280452%2C%2037.619390235000026%5D%2C%20%5B127.09025939799994%2C%2037.61939674500002%5D%2C%20%5B127.09025373500003%2C%2037.619398718000014%5D%2C%20%5B127.09024754%2C%2037.61940070000003%5D%2C%20%5B127.09023462899995%2C%2037.61940466599998%5D%2C%20%5B127.09021587400002%2C%2037.619410604%5D%2C%20%5B127.08894840599999%2C%2037.620068437999976%5D%2C%20%5B127.08889187399996%2C%2037.620131517%5D%2C%20%5B127.08887967700002%2C%2037.620139693%5D%2C%20%5B127.08887383900003%2C%2037.620143632%5D%2C%20%5B127.08834961699995%2C%2037.61991678800001%5D%2C%20%5B127.08831171400004%2C%2037.61989991299998%5D%2C%20%5B127.08807051199994%2C%2037.61991299699997%5D%2C%20%5B127.08796064900002%2C%2037.61993754600002%5D%2C%20%5B127.08697588999996%2C%2037.620239808%5D%2C%20%5B127.08669526400001%2C%2037.62027823699998%5D%2C%20%5B127.08403802199996%2C%2037.61984682799999%5D%2C%20%5B127.08380421000004%2C%2037.61978259900002%5D%2C%20%5B127.08380403399997%2C%2037.619777251000016%5D%2C%20%5B127.081442702%2C%2037.61920071700001%5D%2C%20%5B127.08141613600003%2C%2037.61919543%5D%2C%20%5B127.07774268399999%2C%2037.617923659999974%5D%2C%20%5B127.07771171800005%2C%2037.61791125100001%5D%2C%20%5B127.07774592400006%2C%2037.617875653%5D%2C%20%5B127.07727085800002%2C%2037.61775898600001%5D%2C%20%5B127.07653320700001%2C%2037.617504359%5D%2C%20%5B127.07654155099999%2C%2037.617474148999975%5D%2C%20%5B127.07633587500004%2C%2037.617434995%5D%2C%20%5B127.07621148199996%2C%2037.617371804000015%5D%2C%20%5B127.07555026800003%2C%2037.617169089000015%5D%2C%20%5B127.07538879900005%2C%2037.61717396900002%5D%2C%20%5B127.075382569%2C%2037.61715114899999%5D%2C%20%5B127.07537583299995%2C%2037.61714861899998%5D%2C%20%5B127.07457949599996%2C%2037.61701021099998%5D%2C%20%5B127.07448959099997%2C%2037.61704407899998%5D%2C%20%5B127.07431845300005%2C%2037.61698726899999%5D%2C%20%5B127.07431259999998%2C%2037.61698361200001%5D%2C%20%5B127.07425606200002%2C%2037.61694306700002%5D%2C%20%5B127.07422770100004%2C%2037.616921666999986%5D%2C%20%5B127.07417234000002%2C%2037.61683828600002%5D%2C%20%5B127.07397644800005%2C%2037.616804316000014%5D%2C%20%5B127.073964047%2C%2037.61680206%5D%2C%20%5B127.07394952599998%2C%2037.616799535999974%5D%2C%20%5B127.07394244099999%2C%2037.61679813400002%5D%2C%20%5B127.07220733300005%2C%2037.616534719000015%5D%2C%20%5B127.07219547299997%2C%2037.61653330799999%5D%2C%20%5B127.07197741699997%2C%2037.61651096600002%5D%2C%20%5B127.07196700899999%2C%2037.616540525%5D%2C%20%5B127.07194187100004%2C%2037.616536008000026%5D%2C%20%5B127.07184824%2C%2037.616519073%5D%2C%20%5B127.07139541000004%2C%2037.61640258599999%5D%2C%20%5B127.07017895399997%2C%2037.61539946800002%5D%2C%20%5B127.06817132000003%2C%2037.61545522900002%5D%2C%20%5B127.06607829100005%2C%2037.615024663999975%5D%2C%20%5B127.06342664700003%2C%2037.61432122100001%5D%2C%20%5B127.06309735800005%2C%2037.614302821000024%5D%2C%20%5B127.06295282400004%2C%2037.614320071%5D%2C%20%5B127.06239452399996%2C%2037.61437856700002%5D%2C%20%5B127.062048075%2C%2037.61443152800001%5D%2C%20%5B127.06177353800001%2C%2037.614498406999985%5D%2C%20%5B127.06148469499999%2C%2037.614635953%5D%2C%20%5B127.06112969699996%2C%2037.614918381999985%5D%2C%20%5B127.06110664300002%2C%2037.614941593000026%5D%2C%20%5B127.06099273699999%2C%2037.615107232000014%5D%2C%20%5B127.06095022800002%2C%2037.61516924699998%5D%2C%20%5B127.06089608100001%2C%2037.615257136000025%5D%2C%20%5B127.06086982600004%2C%2037.615305815%5D%2C%20%5B127.06084102700004%2C%2037.61535903599997%5D%2C%20%5B127.06073892799998%2C%2037.615555873%5D%2C%20%5B127.06068329100003%2C%2037.615667960999986%5D%2C%20%5B127.05950317700001%2C%2037.61630130399999%5D%2C%20%5B127.05936722700005%2C%2037.616373727%5D%2C%20%5B127.059003083%2C%2037.61657251999998%5D%2C%20%5B127.05889487000002%2C%2037.61664244899998%5D%2C%20%5B127.05887912799994%2C%2037.616652031%5D%2C%20%5B127.05834272000004%2C%2037.61698833000003%5D%2C%20%5B127.05809590000001%2C%2037.617128353%5D%2C%20%5B127.057455248%2C%2037.617474129000016%5D%2C%20%5B127.05718000800005%2C%2037.617738234%5D%2C%20%5B127.05707011100003%2C%2037.617850546%5D%2C%20%5B127.05622734899998%2C%2037.618674414%5D%2C%20%5B127.056148914%2C%2037.61874342499999%5D%2C%20%5B127.05557420100001%2C%2037.61917132999997%5D%2C%20%5B127.05555242100002%2C%2037.619182878%5D%2C%20%5B127.05516465999995%2C%2037.61938773999998%5D%2C%20%5B127.05482396399998%2C%2037.619556037999985%5D%2C%20%5B127.05442376899998%2C%2037.61980001500001%5D%2C%20%5B127.054137093%2C%2037.62005487%5D%2C%20%5B127.05409706499995%2C%2037.620090371%5D%2C%20%5B127.05408396500002%2C%2037.62010333699999%5D%2C%20%5B127.05369230999997%2C%2037.620526132%5D%2C%20%5B127.05244650500003%2C%2037.621842843000024%5D%2C%20%5B127.05208653700004%2C%2037.622160446%5D%2C%20%5B127.05212812599996%2C%2037.62224602399999%5D%2C%20%5B127.05121275399995%2C%2037.62305264899999%5D%2C%20%5B127.049754269%2C%2037.624269174%5D%2C%20%5B127.04707421900002%2C%2037.627527047%5D%2C%20%5B127.044443129%2C%2037.628828758%5D%2C%20%5B127.04261642300003%2C%2037.63045736999999%5D%2C%20%5B127.04167535700003%2C%2037.63147208800001%5D%2C%20%5B127.04393530799996%2C%2037.63348294799999%5D%2C%20%5B127.04500996299998%2C%2037.635869506%5D%2C%20%5B127.04504562299996%2C%2037.63669604299997%5D%2C%20%5B127.04601795999997%2C%2037.63790943499998%5D%2C%20%5B127.04678299900002%2C%2037.639714513%5D%2C%20%5B127.04673406899997%2C%2037.64122815500002%5D%2C%20%5B127.04905372400003%2C%2037.64215242300003%5D%2C%20%5B127.049737725%2C%2037.643517594%5D%2C%20%5B127.05137096700003%2C%2037.64482203900002%5D%2C%20%5B127.05298397900003%2C%2037.641679633000024%5D%2C%20%5B127.05474370299999%2C%2037.64014199899998%5D%2C%20%5B127.05513914699998%2C%2037.64133787499998%5D%2C%20%5B127.05516606799995%2C%2037.643454948%5D%2C%20%5B127.05566746900001%2C%2037.64460474399999%5D%2C%20%5B127.05584227600002%2C%2037.64705617300001%5D%2C%20%5B127.05545541100003%2C%2037.64824690299997%5D%2C%20%5B127.05396756200003%2C%2037.650582568%5D%2C%20%5B127.05376492699997%2C%2037.652060053000014%5D%2C%20%5B127.054073077%2C%2037.654496732999974%5D%2C%20%5B127.05343323299996%2C%2037.65751023399997%5D%2C%20%5B127.051469816%2C%2037.66042898699999%5D%2C%20%5B127.05142672199997%2C%2037.663973570999985%5D%2C%20%5B127.04893704599999%2C%2037.667590002%5D%2C%20%5B127.04820611399998%2C%2037.67048413200001%5D%2C%20%5B127.04887969799995%2C%2037.67518859900002%5D%2C%20%5B127.04973706099997%2C%2037.676724165999985%5D%2C%20%5B127.050936224%2C%2037.68010014399999%5D%2C%20%5B127.05124372099999%2C%2037.68200315500002%5D%2C%20%5B127.05195432999994%2C%2037.68295554600002%5D%2C%20%5B127.05181987399999%2C%2037.685815952999974%5D%2C%20%5B127.05182101699995%2C%2037.687069296%5D%2C%20%5B127.05519755800003%2C%2037.689209908%5D%2C%20%5B127.05840713099997%2C%2037.689689869%5D%2C%20%5B127.05969812599994%2C%2037.690342471%5D%2C%20%5B127.06130232099997%2C%2037.69219614100001%5D%2C%20%5B127.06249161400001%2C%2037.69305951199999%5D%2C%20%5B127.06274278700005%2C%2037.69467682200002%5D%2C%20%5B127.06356049299995%2C%2037.69497166299999%5D%2C%20%5B127.06650615199999%2C%2037.694498565%5D%2C%20%5B127.068049664%2C%2037.694837018999976%5D%2C%20%5B127.069366791%2C%2037.69375161099998%5D%2C%20%5B127.07291609799995%2C%2037.69387412100002%5D%2C%20%5B127.07522005500005%2C%2037.695310147999976%5D%2C%20%5B127.07856303100004%2C%2037.696092556999986%5D%2C%20%5B127.08150813999998%2C%2037.69625423399998%5D%2C%20%5B127.084240346%2C%2037.694384093%5D%2C%20%5B127.08425235100003%2C%2037.69188345499998%5D%2C%20%5B127.08551096600002%2C%2037.69048708600002%5D%2C%20%5B127.08687032%2C%2037.69001125599999%5D%2C%20%5B127.09093103999999%2C%2037.68968092699998%5D%2C%20%5B127.09345258600001%2C%2037.690087849%5D%2C%20%5B127.096066758%2C%2037.687916323000024%5D%2C%20%5B127.09641572400005%2C%2037.685647404%5D%2C%20%5B127.09580503999996%2C%2037.684650288%5D%2C%20%5B127.09428565300004%2C%2037.683427962999986%5D%2C%20%5B127.09292189500002%2C%2037.68156068000002%5D%2C%20%5B127.09282796000002%2C%2037.68005578999998%5D%2C%20%5B127.09197763199995%2C%2037.67920024699998%5D%2C%20%5B127.09221964100004%2C%2037.678039738%5D%2C%20%5B127.09388436100005%2C%2037.676669027%5D%2C%20%5B127.09464265999998%2C%2037.67354516199998%5D%2C%20%5B127.09577928399995%2C%2037.672681931%5D%2C%20%5B127.09577485800003%2C%2037.67062784799998%5D%2C%20%5B127.09626208899999%2C%2037.668837161%5D%2C%20%5B127.09500097800003%2C%2037.667606956999975%5D%2C%20%5B127.09469822699998%2C%2037.664927189000025%5D%2C%20%5B127.09542861800003%2C%2037.663895431000014%5D%2C%20%5B127.094157774%2C%2037.66330400700002%5D%2C%20%5B127.091221335%2C%2037.65934019600002%5D%2C%20%5B127.09114721900005%2C%2037.658248809999975%5D%2C%20%5B127.09190627600003%2C%2037.65744077099998%5D%2C%20%5B127.09236725000005%2C%2037.65549788499999%5D%2C%20%5B127.09401619699997%2C%2037.65230150399998%5D%2C%20%5B127.09248088799995%2C%2037.64971631999998%5D%2C%20%5B127.09364724600005%2C%2037.646632361%5D%2C%20%5B127.09448968499999%2C%2037.645856236999975%5D%2C%20%5B127.09458566599994%2C%2037.644576943%5D%2C%20%5B127.09757475499998%2C%2037.643971025999974%5D%2C%20%5B127.10160801200004%2C%2037.64477886399999%5D%2C%20%5B127.103936524%2C%2037.645499323000024%5D%2C%20%5B127.10658921100003%2C%2037.645384454%5D%2C%20%5B127.10770375100003%2C%2037.64497346799999%5D%2C%20%5B127.10930262900001%2C%2037.64286426299998%5D%2C%20%5B127.11076511800002%2C%2037.64273476300002%5D%2C%20%5B127.11223881%2C%2037.64037708500001%5D%2C%20%5B127.110580357%2C%2037.639364204%5D%2C%20%5B127.11249787%2C%2037.63648892100002%5D%2C%20%5B127.11242501499999%2C%2037.63425247999999%5D%2C%20%5B127.11162094999997%2C%2037.633851106%5D%2C%20%5B127.11222175299997%2C%2037.632645615%5D%2C%20%5B127.111649757%2C%2037.63150772300003%5D%2C%20%5B127.108900973%2C%2037.62930749999998%5D%2C%20%5B127.10597855699996%2C%2037.627337269%5D%2C%20%5B127.10434078499998%2C%2037.62327850299999%5D%2C%20%5B127.10407893299998%2C%2037.62166095499998%5D%2C%20%5B127.10492130800003%2C%2037.62157580500002%5D%2C%20%5B127.105568661%2C%2037.62037839200002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2018%2C%20%22SHAPE_AREA%22%3A%200.00364%2C%20%22SHAPE_LEN%22%3A%200.309723%2C%20%22SIG_CD%22%3A%20%2211350%22%2C%20%22SIG_ENG_NM%22%3A%20%22Nowon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cub178%5Cuc6d0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.14511013100002%2C%2037.51684043%5D%2C%20%5B127.144621648%2C%2037.51561223599998%5D%2C%20%5B127.14215399099999%2C%2037.51566003099998%5D%2C%20%5B127.14138931900004%2C%2037.514975155%5D%2C%20%5B127.14341256199998%2C%2037.51387651099998%5D%2C%20%5B127.14354620699999%2C%2037.512663953000015%5D%2C%20%5B127.1414575%2C%2037.51240795199999%5D%2C%20%5B127.14096457300002%2C%2037.51040582000002%5D%2C%20%5B127.13998328499997%2C%2037.50864916199998%5D%2C%20%5B127.141534557%2C%2037.506717287000015%5D%2C%20%5B127.14102941299996%2C%2037.505493221999984%5D%2C%20%5B127.144121606%2C%2037.50440674700002%5D%2C%20%5B127.14597136099997%2C%2037.503236346999984%5D%2C%20%5B127.14771286899997%2C%2037.503223030000015%5D%2C%20%5B127.15022088800004%2C%2037.50474927699997%5D%2C%20%5B127.15212420600005%2C%2037.50297788%5D%2C%20%5B127.15285967099999%2C%2037.503062426999975%5D%2C%20%5B127.15656939099995%2C%2037.50190080300001%5D%2C%20%5B127.157743278%2C%2037.50318546800003%5D%2C%20%5B127.15887991900001%2C%2037.50239690400002%5D%2C%20%5B127.159410547%2C%2037.501369312%5D%2C%20%5B127.16140357300003%2C%2037.50020737699998%5D%2C%20%5B127.15977913799998%2C%2037.49679815600001%5D%2C%20%5B127.16001896600005%2C%2037.49433240399998%5D%2C%20%5B127.15848990500001%2C%2037.492042157000014%5D%2C%20%5B127.158182699%2C%2037.49051667399999%5D%2C%20%5B127.156300945%2C%2037.48949331900002%5D%2C%20%5B127.154795626%2C%2037.48731379999998%5D%2C%20%5B127.15219320300002%2C%2037.48624278199998%5D%2C%20%5B127.150769406%2C%2037.48474927500001%5D%2C%20%5B127.15104542899996%2C%2037.483961959%5D%2C%20%5B127.15068947400005%2C%2037.48202662699998%5D%2C%20%5B127.14915395599996%2C%2037.48074899300002%5D%2C%20%5B127.14783019699996%2C%2037.478664200000026%5D%2C%20%5B127.14439008099998%2C%2037.476456739000014%5D%2C%20%5B127.14354172599997%2C%2037.475545689%5D%2C%20%5B127.139692276%2C%2037.47419605099998%5D%2C%20%5B127.13937783300003%2C%2037.473447066%5D%2C%20%5B127.13558350400001%2C%2037.474578011%5D%2C%20%5B127.13368163799998%2C%2037.47568719499998%5D%2C%20%5B127.13267746400004%2C%2037.475808433%5D%2C%20%5B127.13024850199997%2C%2037.47510375899998%5D%2C%20%5B127.13042151800005%2C%2037.47189767899999%5D%2C%20%5B127.13538943799995%2C%2037.46932071200001%5D%2C%20%5B127.13404767700001%2C%2037.46859963000003%5D%2C%20%5B127.13014299300005%2C%2037.467765722000024%5D%2C%20%5B127.12704911399999%2C%2037.46842315399999%5D%2C%20%5B127.12521919400001%2C%2037.46962967899998%5D%2C%20%5B127.125063372%2C%2037.46727479200001%5D%2C%20%5B127.12422200499998%2C%2037.46653068400002%5D%2C%20%5B127.11972185399998%2C%2037.47277094200001%5D%2C%20%5B127.11670068800004%2C%2037.476760110999976%5D%2C%20%5B127.11661307300005%2C%2037.476874286%5D%2C%20%5B127.11522846900004%2C%2037.47870581900003%5D%2C%20%5B127.11507001899997%2C%2037.47891645300001%5D%2C%20%5B127.111525573%2C%2037.48403468399999%5D%2C%20%5B127.10702363799999%2C%2037.49029834800001%5D%2C%20%5B127.10353662099999%2C%2037.49270810199999%5D%2C%20%5B127.09881574300005%2C%2037.494899765000014%5D%2C%20%5B127.09804850499995%2C%2037.495061401999976%5D%2C%20%5B127.09757487399997%2C%2037.495615964000024%5D%2C%20%5B127.097411577%2C%2037.49570356200002%5D%2C%20%5B127.094461453%2C%2037.49678522%5D%2C%20%5B127.094361983%2C%2037.49681289199998%5D%2C%20%5B127.09133828899996%2C%2037.49766548399998%5D%2C%20%5B127.09127723699999%2C%2037.49768245299998%5D%2C%20%5B127.09073518499997%2C%2037.49783561200002%5D%2C%20%5B127.09065772999998%2C%2037.49785652700001%5D%2C%20%5B127.08423766299995%2C%2037.49965926099998%5D%2C%20%5B127.07700902900001%2C%2037.50207652099999%5D%2C%20%5B127.07618946800005%2C%2037.50215425099998%5D%2C%20%5B127.07595697900001%2C%2037.502176561%5D%2C%20%5B127.07229181499997%2C%2037.50251013600001%5D%2C%20%5B127.07195844099999%2C%2037.50254188500003%5D%2C%20%5B127.06981054799996%2C%2037.502915734%5D%2C%20%5B127.06964225599995%2C%2037.50415385299999%5D%2C%20%5B127.06762596099998%2C%2037.520873596%5D%2C%20%5B127.06752008299998%2C%2037.52458673299998%5D%2C%20%5B127.06868275399995%2C%2037.523874679000016%5D%2C%20%5B127.07240582899999%2C%2037.52333952999999%5D%2C%20%5B127.07258890699995%2C%2037.523296141%5D%2C%20%5B127.07686379699999%2C%2037.52253424600002%5D%2C%20%5B127.07696470500002%2C%2037.52252098399998%5D%2C%20%5B127.07999463199997%2C%2037.52295635199999%5D%2C%20%5B127.085651558%2C%2037.52476491200002%5D%2C%20%5B127.09016038899995%2C%2037.527010054000016%5D%2C%20%5B127.09911344099999%2C%2037.53411786700002%5D%2C%20%5B127.09923242699995%2C%2037.53421491199998%5D%2C%20%5B127.10829759600006%2C%2037.541412825%5D%2C%20%5B127.10915912500002%2C%2037.543170621%5D%2C%20%5B127.11306236099995%2C%2037.54298559799997%5D%2C%20%5B127.11889123200001%2C%2037.541080018%5D%2C%20%5B127.121876327%2C%2037.539287715%5D%2C%20%5B127.12345794400005%2C%2037.538648282%5D%2C%20%5B127.12015481200001%2C%2037.529669963%5D%2C%20%5B127.11911248499996%2C%2037.52796713499998%5D%2C%20%5B127.14384594399996%2C%2037.517038566999986%5D%2C%20%5B127.14511013100002%2C%2037.51684043%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2019%2C%20%22SHAPE_AREA%22%3A%200.003449%2C%20%22SHAPE_LEN%22%3A%200.316773%2C%20%22SIG_CD%22%3A%20%2211710%22%2C%20%22SIG_ENG_NM%22%3A%20%22Songpa-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc1a1%5Cud30c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B127.07240207200005%2C%2037.55996818900002%5D%2C%20%5B127.07375319200003%2C%2037.55942005899999%5D%2C%20%5B127.06748119300005%2C%2037.548333146%5D%2C%20%5B127.06399131399996%2C%2037.54214780199999%5D%2C%20%5B127.05822943500004%2C%2037.53193715100002%5D%2C%20%5B127.05623661599998%2C%2037.528327074%5D%2C%20%5B127.05514476500002%2C%2037.528670655999974%5D%2C%20%5B127.04951337499995%2C%2037.53221947499998%5D%2C%20%5B127.04613452000001%2C%2037.53420143%5D%2C%20%5B127.04603770599999%2C%2037.534260027000016%5D%2C%20%5B127.03981273800002%2C%2037.535828337%5D%2C%20%5B127.03706651699997%2C%2037.53584514800002%5D%2C%20%5B127.03291623799998%2C%2037.535843096%5D%2C%20%5B127.03170159399997%2C%2037.535842057000025%5D%2C%20%5B127.02266610699996%2C%2037.53581941700003%5D%2C%20%5B127.02145905199995%2C%2037.53582472900001%5D%2C%20%5B127.02118792199997%2C%2037.535824079%5D%2C%20%5B127.02038664899999%2C%2037.53545835599999%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00817125799995%2C%2037.545066427999984%5D%2C%20%5B127.00922899700004%2C%2037.546389986%5D%2C%20%5B127.00969648299997%2C%2037.54789438199998%5D%2C%20%5B127.01161154199997%2C%2037.548492732%5D%2C%20%5B127.01218373999995%2C%2037.549419613%5D%2C%20%5B127.01587377299995%2C%2037.552472548000026%5D%2C%20%5B127.01595370099994%2C%2037.55255136400001%5D%2C%20%5B127.015998609%2C%2037.55259049699998%5D%2C%20%5B127.01624583199998%2C%2037.552803031999986%5D%2C%20%5B127.01790785599997%2C%2037.55670534199999%5D%2C%20%5B127.01811182699998%2C%2037.556877523000026%5D%2C%20%5B127.019697973%2C%2037.55726674800002%5D%2C%20%5B127.01959446800004%2C%2037.557499285%5D%2C%20%5B127.021749484%2C%2037.55753185499998%5D%2C%20%5B127.02181049800004%2C%2037.55751436999998%5D%2C%20%5B127.02293212999996%2C%2037.55786605399999%5D%2C%20%5B127.022803586%2C%2037.55937128%5D%2C%20%5B127.02336744000002%2C%2037.56066693899999%5D%2C%20%5B127.02653042899999%2C%2037.56337140699998%5D%2C%20%5B127.02680257099996%2C%2037.56504333700002%5D%2C%20%5B127.02360224999995%2C%2037.565181224000014%5D%2C%20%5B127.02354931499997%2C%2037.566834884%5D%2C%20%5B127.02354861000003%2C%2037.56684754600002%5D%2C%20%5B127.02355265599999%2C%2037.56691480799998%5D%2C%20%5B127.02354744900003%2C%2037.56715984800002%5D%2C%20%5B127.02340767400005%2C%2037.571910409%5D%2C%20%5B127.02460599699998%2C%2037.57105023399998%5D%2C%20%5B127.02617816700001%2C%2037.57125315899998%5D%2C%20%5B127.02769262599998%2C%2037.57041509700002%5D%2C%20%5B127.03128573699996%2C%2037.56980279700002%5D%2C%20%5B127.03307690999998%2C%2037.57051669100002%5D%2C%20%5B127.03420836600003%2C%2037.57139252500002%5D%2C%20%5B127.03566033699997%2C%2037.57200219600003%5D%2C%20%5B127.03814805900004%2C%2037.57226823799999%5D%2C%20%5B127.03819617500005%2C%2037.573023611999986%5D%2C%20%5B127.04192662699995%2C%2037.57297117600001%5D%2C%20%5B127.04797413899996%2C%2037.57038284999999%5D%2C%20%5B127.05878053100002%2C%2037.562319322%5D%2C%20%5B127.07087406300002%2C%2037.560589956%5D%2C%20%5B127.07240207200005%2C%2037.55996818900002%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2020%2C%20%22SHAPE_AREA%22%3A%200.001714%2C%20%22SHAPE_LEN%22%3A%200.193793%2C%20%22SIG_CD%22%3A%20%2211200%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seongdong-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc131%5Cub3d9%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.953065821%2C%2037.599076496%5D%2C%20%5B126.95303030499997%2C%2037.59893801499999%5D%2C%20%5B126.95490273799999%2C%2037.59931146299999%5D%2C%20%5B126.95806204799999%2C%2037.59815816600002%5D%2C%20%5B126.95907047000003%2C%2037.59503662499998%5D%2C%20%5B126.95753628%2C%2037.593531073%5D%2C%20%5B126.95783824600005%2C%2037.59146206899999%5D%2C%20%5B126.95734372699997%2C%2037.588701662%5D%2C%20%5B126.95748048999997%2C%2037.58602578900002%5D%2C%20%5B126.95800221699994%2C%2037.584431987000016%5D%2C%20%5B126.95756617100005%2C%2037.582303430000024%5D%2C%20%5B126.95817750200001%2C%2037.581170713%5D%2C%20%5B126.95721857599995%2C%2037.57981962299999%5D%2C%20%5B126.95569425899998%2C%2037.57968449499998%5D%2C%20%5B126.95357504399999%2C%2037.578749668%5D%2C%20%5B126.955465893%2C%2037.57654704999999%5D%2C%20%5B126.958353321%2C%2037.57409909699999%5D%2C%20%5B126.96401578500002%2C%2037.56863901000003%5D%2C%20%5B126.96671421400004%2C%2037.56582271000002%5D%2C%20%5B126.96684742499997%2C%2037.56567956399999%5D%2C%20%5B126.967638211%2C%2037.56480180800003%5D%2C%20%5B126.96766816299998%2C%2037.56476074300002%5D%2C%20%5B126.96795605600005%2C%2037.564365807%5D%2C%20%5B126.96801557900005%2C%2037.56428443200002%5D%2C%20%5B126.96948235499997%2C%2037.56197813599999%5D%2C%20%5B126.96664950599995%2C%2037.56113058400001%5D%2C%20%5B126.96657450600003%2C%2037.56107646800001%5D%2C%20%5B126.96637532900002%2C%2037.56093243%5D%2C%20%5B126.96634772699997%2C%2037.56091242399998%5D%2C%20%5B126.96596405599996%2C%2037.560633783000014%5D%2C%20%5B126.96593561700001%2C%2037.560613461%5D%2C%20%5B126.964234357%2C%2037.55959337799999%5D%2C%20%5B126.96419925099997%2C%2037.55957309899998%5D%2C%20%5B126.96393016399998%2C%2037.55943875000003%5D%2C%20%5B126.96390582499998%2C%2037.559429619000014%5D%2C%20%5B126.96319348999998%2C%2037.559194177999984%5D%2C%20%5B126.96309676500005%2C%2037.559187388%5D%2C%20%5B126.96232487999998%2C%2037.559101829999975%5D%2C%20%5B126.96197012799996%2C%2037.559060332%5D%2C%20%5B126.96170649400005%2C%2037.558984517%5D%2C%20%5B126.95914877799999%2C%2037.55724429999998%5D%2C%20%5B126.95620622299998%2C%2037.55740926599998%5D%2C%20%5B126.94138350900005%2C%2037.55654660099998%5D%2C%20%5B126.93625552499998%2C%2037.554894509%5D%2C%20%5B126.93396630200004%2C%2037.553452935%5D%2C%20%5B126.93289803100004%2C%2037.55432018699997%5D%2C%20%5B126.93261962500003%2C%2037.55545525299999%5D%2C%20%5B126.93262354599995%2C%2037.555558925000014%5D%2C%20%5B126.93281136500002%2C%2037.556581901000015%5D%2C%20%5B126.92661636399998%2C%2037.55885124500003%5D%2C%20%5B126.92662051100001%2C%2037.55944762299998%5D%2C%20%5B126.92662346400004%2C%2037.559501977000025%5D%2C%20%5B126.92686756299997%2C%2037.56128305300001%5D%2C%20%5B126.92830228800005%2C%2037.56333432500003%5D%2C%20%5B126.926034996%2C%2037.56510655599999%5D%2C%20%5B126.92475336099994%2C%2037.56567633899999%5D%2C%20%5B126.91845893699997%2C%2037.56711322699999%5D%2C%20%5B126.91105651%2C%2037.57042899700002%5D%2C%20%5B126.91096095600005%2C%2037.57046693%5D%2C%20%5B126.90880560799997%2C%2037.57144989599999%5D%2C%20%5B126.902102937%2C%2037.576007618%5D%2C%20%5B126.90185072600002%2C%2037.57654631499997%5D%2C%20%5B126.904138687%2C%2037.57849389699999%5D%2C%20%5B126.91067547399996%2C%2037.58464564299999%5D%2C%20%5B126.91096941499995%2C%2037.584917345%5D%2C%20%5B126.91286579799998%2C%2037.58722638099999%5D%2C%20%5B126.91571674299996%2C%2037.585983397%5D%2C%20%5B126.91602474199999%2C%2037.58310641700001%5D%2C%20%5B126.91890764499999%2C%2037.58338949099999%5D%2C%20%5B126.92087799%2C%2037.582942883999976%5D%2C%20%5B126.92176922399995%2C%2037.58345890700002%5D%2C%20%5B126.92186544499998%2C%2037.58468884500002%5D%2C%20%5B126.92367083800002%2C%2037.58615331599998%5D%2C%20%5B126.92387602999997%2C%2037.587100857999985%5D%2C%20%5B126.92633484400005%2C%2037.58729347600001%5D%2C%20%5B126.92801735900002%2C%2037.588390409%5D%2C%20%5B126.927623445%2C%2037.58939674599998%5D%2C%20%5B126.92778464399998%2C%2037.591579792%5D%2C%20%5B126.93026736700006%2C%2037.594553845%5D%2C%20%5B126.93281481899999%2C%2037.59631526499999%5D%2C%20%5B126.93453831600004%2C%2037.596636406000016%5D%2C%20%5B126.93756142899997%2C%2037.59787207%5D%2C%20%5B126.938649199%2C%2037.597503867%5D%2C%20%5B126.94089729799998%2C%2037.598720316000026%5D%2C%20%5B126.94140240700006%2C%2037.60068057900003%5D%2C%20%5B126.93985536599996%2C%2037.60187368499999%5D%2C%20%5B126.93997629900002%2C%2037.60307851099998%5D%2C%20%5B126.94126023499996%2C%2037.60313710299999%5D%2C%20%5B126.94214877499996%2C%2037.60494473900002%5D%2C%20%5B126.94368385200005%2C%2037.60504592299998%5D%2C%20%5B126.94376183400004%2C%2037.60621278000002%5D%2C%20%5B126.94623054099998%2C%2037.60739522099999%5D%2C%20%5B126.94874208700003%2C%2037.609028416%5D%2C%20%5B126.95037992899995%2C%2037.610620949%5D%2C%20%5B126.95061630099997%2C%2037.608363673999975%5D%2C%20%5B126.951468212%2C%2037.608018157%5D%2C%20%5B126.95273025300003%2C%2037.60591681599999%5D%2C%20%5B126.95408125100005%2C%2037.605075141999976%5D%2C%20%5B126.95239360200003%2C%2037.60247920400002%5D%2C%20%5B126.95338309600004%2C%2037.60017021700003%5D%2C%20%5B126.953065821%2C%2037.599076496%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2021%2C%20%22SHAPE_AREA%22%3A%200.001811%2C%20%22SHAPE_LEN%22%3A%200.231131%2C%20%22SIG_CD%22%3A%20%2211410%22%2C%20%22SIG_ENG_NM%22%3A%20%22Seodaemun-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc11c%5Cub300%5Cubb38%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.87497518099997%2C%2037.54697092499998%5D%2C%20%5B126.875011094%2C%2037.546972363%5D%2C%20%5B126.87514333499996%2C%2037.54697837499998%5D%2C%20%5B126.87518571099997%2C%2037.546980072999986%5D%2C%20%5B126.87797081300005%2C%2037.547103134%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87961291299996%2C%2037.51764897700002%5D%2C%20%5B126.87958544499998%2C%2037.517588095%5D%2C%20%5B126.87930741800005%2C%2037.517494376%5D%2C%20%5B126.878693763%2C%2037.51758981900002%5D%2C%20%5B126.87855131699996%2C%2037.51739811200002%5D%2C%20%5B126.87861125799998%2C%2037.517301223%5D%2C%20%5B126.87898967399997%2C%2037.51703769300002%5D%2C%20%5B126.879022534%2C%2037.516906973%5D%2C%20%5B126.87892647900003%2C%2037.51680885000002%5D%2C%20%5B126.87841015900005%2C%2037.51630315800003%5D%2C%20%5B126.87826555200002%2C%2037.51624565399999%5D%2C%20%5B126.87779565000005%2C%2037.51522560500001%5D%2C%20%5B126.87766317600006%2C%2037.514435708%5D%2C%20%5B126.87767502600002%2C%2037.51438914%5D%2C%20%5B126.87791691500001%2C%2037.51345160699998%5D%2C%20%5B126.87767547600004%2C%2037.513363599%5D%2C%20%5B126.87666684400006%2C%2037.513690188%5D%2C%20%5B126.876594834%2C%2037.51363155199999%5D%2C%20%5B126.87639996500002%2C%2037.513311594000015%5D%2C%20%5B126.87646375300005%2C%2037.51294127900002%5D%2C%20%5B126.87641212100004%2C%2037.512819639999975%5D%2C%20%5B126.87621775900004%2C%2037.51252669199999%5D%2C%20%5B126.87517960000002%2C%2037.512430897%5D%2C%20%5B126.87514378599997%2C%2037.51238695699999%5D%2C%20%5B126.87412014799997%2C%2037.51140628600001%5D%2C%20%5B126.87404973699995%2C%2037.51133640199998%5D%2C%20%5B126.87395469499995%2C%2037.51117755000001%5D%2C%20%5B126.87401808699997%2C%2037.511025632999974%5D%2C%20%5B126.87409670399995%2C%2037.51083209699999%5D%2C%20%5B126.87457056699998%2C%2037.51070094300002%5D%2C%20%5B126.87481361599998%2C%2037.51086108800001%5D%2C%20%5B126.87491023200005%2C%2037.51104133199999%5D%2C%20%5B126.87488010300001%2C%2037.51131149899999%5D%2C%20%5B126.87484003700001%2C%2037.51163792599999%5D%2C%20%5B126.87519821900003%2C%2037.51183762800002%5D%2C%20%5B126.87547908399995%2C%2037.51164094500001%5D%2C%20%5B126.87607668500004%2C%2037.51122960599997%5D%2C%20%5B126.87643582299995%2C%2037.511186113%5D%2C%20%5B126.87668761299994%2C%2037.51108622100003%5D%2C%20%5B126.876759397%2C%2037.511016504999986%5D%2C%20%5B126.87640855500001%2C%2037.510722249000025%5D%2C%20%5B126.87636871799998%2C%2037.510813401%5D%2C%20%5B126.87538706700002%2C%2037.51093382%5D%2C%20%5B126.87520165599994%2C%2037.510528319%5D%2C%20%5B126.87520184599998%2C%2037.510519312999975%5D%2C%20%5B126.87527600700002%2C%2037.51018953%5D%2C%20%5B126.87561522099998%2C%2037.50966277499998%5D%2C%20%5B126.87554601099998%2C%2037.50957480599999%5D%2C%20%5B126.87548982199996%2C%2037.50951052300002%5D%2C%20%5B126.87544788800005%2C%2037.50946539400002%5D%2C%20%5B126.87542320199998%2C%2037.50944959100002%5D%2C%20%5B126.87480680600004%2C%2037.50933397900002%5D%2C%20%5B126.874423966%2C%2037.50949468099998%5D%2C%20%5B126.87411762500005%2C%2037.50943349%5D%2C%20%5B126.87401180699999%2C%2037.509364644000016%5D%2C%20%5B126.87399719400003%2C%2037.50934885200002%5D%2C%20%5B126.87397200199996%2C%2037.50931839899999%5D%2C%20%5B126.87394735199996%2C%2037.50928794700002%5D%2C%20%5B126.87391847200001%2C%2037.50924960200001%5D%2C%20%5B126.87391919499998%2C%2037.508676045000016%5D%2C%20%5B126.87444050600004%2C%2037.508449007000024%5D%2C%20%5B126.87477277599999%2C%2037.50830964099998%5D%2C%20%5B126.87495693300002%2C%2037.50798306399997%5D%2C%20%5B126.875049397%2C%2037.507736399%5D%2C%20%5B126.87500859399995%2C%2037.50759549000003%5D%2C%20%5B126.87474459700002%2C%2037.507489282999984%5D%2C%20%5B126.87463181199996%2C%2037.507449709000014%5D%2C%20%5B126.87449596099998%2C%2037.507384202000026%5D%2C%20%5B126.87472443000001%2C%2037.506635346999985%5D%2C%20%5B126.87470556300002%2C%2037.506539659%5D%2C%20%5B126.87457002600001%2C%2037.50631615999998%5D%2C%20%5B126.87441977200001%2C%2037.506049689%5D%2C%20%5B126.87407221399997%2C%2037.50517583499999%5D%2C%20%5B126.87372020999999%2C%2037.504759122999985%5D%2C%20%5B126.87371440100003%2C%2037.50444758399999%5D%2C%20%5B126.87377190899997%2C%2037.504235489999985%5D%2C%20%5B126.87377088400001%2C%2037.50421073299998%5D%2C%20%5B126.87359425299996%2C%2037.503562991000024%5D%2C%20%5B126.872756998%2C%2037.50384008999998%5D%2C%20%5B126.87259622900001%2C%2037.503937888999985%5D%2C%20%5B126.87184140299996%2C%2037.504205309999975%5D%2C%20%5B126.87183448500002%2C%2037.50422335799999%5D%2C%20%5B126.87148326099998%2C%2037.50469375199998%5D%2C%20%5B126.87065566900003%2C%2037.505759743%5D%2C%20%5B126.86870085700002%2C%2037.50486748399999%5D%2C%20%5B126.86715745699996%2C%2037.50529986%5D%2C%20%5B126.86652236999998%2C%2037.50535542799997%5D%2C%20%5B126.86644081400004%2C%2037.50539252200002%5D%2C%20%5B126.86606616999995%2C%2037.505530738%5D%2C%20%5B126.86595564900006%2C%2037.505547525%5D%2C%20%5B126.86545582899998%2C%2037.50557854800002%5D%2C%20%5B126.86517882700002%2C%2037.505569236999975%5D%2C%20%5B126.86483013700001%2C%2037.50535024499999%5D%2C%20%5B126.86479709100001%2C%2037.50533669700002%5D%2C%20%5B126.86474403199998%2C%2037.505222815000025%5D%2C%20%5B126.86462709900002%2C%2037.50501760499998%5D%2C%20%5B126.86403527499999%2C%2037.50505866399999%5D%2C%20%5B126.86363638399996%2C%2037.505054168000015%5D%2C%20%5B126.86283780099996%2C%2037.50613905900002%5D%2C%20%5B126.862916531%2C%2037.506613129000016%5D%2C%20%5B126.86280736699996%2C%2037.50669406600002%5D%2C%20%5B126.86280696899996%2C%2037.50696316300002%5D%2C%20%5B126.863088038%2C%2037.5072968%5D%2C%20%5B126.863764801%2C%2037.50787936299997%5D%2C%20%5B126.86380943799998%2C%2037.508023466%5D%2C%20%5B126.86410005799996%2C%2037.508344798%5D%2C%20%5B126.86373739299995%2C%2037.508107844999984%5D%2C%20%5B126.86318581499995%2C%2037.507855366%5D%2C%20%5B126.86168582100004%2C%2037.50711042199998%5D%2C%20%5B126.86118514500004%2C%2037.506963430999974%5D%2C%20%5B126.860456043%2C%2037.50674747199997%5D%2C%20%5B126.859374592%2C%2037.508212151%5D%2C%20%5B126.858345407%2C%2037.50992339800001%5D%2C%20%5B126.85723987899996%2C%2037.50963325700002%5D%2C%20%5B126.85562926600005%2C%2037.50916078300003%5D%2C%20%5B126.85405801700006%2C%2037.51022482000002%5D%2C%20%5B126.85217559299997%2C%2037.510481905%5D%2C%20%5B126.85140792799996%2C%2037.51013296100001%5D%2C%20%5B126.849535715%2C%2037.50964766499999%5D%2C%20%5B126.84873119999997%2C%2037.508888764%5D%2C%20%5B126.84764115400003%2C%2037.50934484700002%5D%2C%20%5B126.84761497299996%2C%2037.509358335%5D%2C%20%5B126.847054835%2C%2037.509376804%5D%2C%20%5B126.84630913299998%2C%2037.50870375%5D%2C%20%5B126.84610971899997%2C%2037.508154037%5D%2C%20%5B126.84606851399997%2C%2037.508092057%5D%2C%20%5B126.84597292700005%2C%2037.508022122%5D%2C%20%5B126.845355165%2C%2037.507775865999974%5D%2C%20%5B126.84507536900003%2C%2037.507535673%5D%2C%20%5B126.84477361300003%2C%2037.50709728099997%5D%2C%20%5B126.84360051099998%2C%2037.50582680100001%5D%2C%20%5B126.84234321600002%2C%2037.50554362299999%5D%2C%20%5B126.84211428499998%2C%2037.50566272999998%5D%2C%20%5B126.84076167%2C%2037.50604941%5D%2C%20%5B126.83933433899995%2C%2037.504454082%5D%2C%20%5B126.83843497099997%2C%2037.50436615799998%5D%2C%20%5B126.837508791%2C%2037.50328696299999%5D%2C%20%5B126.83526085899996%2C%2037.50287677599999%5D%2C%20%5B126.83406682400005%2C%2037.50335796799999%5D%2C%20%5B126.83400839199999%2C%2037.50338505299999%5D%2C%20%5B126.83391122800003%2C%2037.50349796299997%5D%2C%20%5B126.833779618%2C%2037.50368009800002%5D%2C%20%5B126.83362079100004%2C%2037.50388647099999%5D%2C%20%5B126.83357629%2C%2037.503916624%5D%2C%20%5B126.83308794899995%2C%2037.504059032999976%5D%2C%20%5B126.83304605599994%2C%2037.50408686399999%5D%2C%20%5B126.83300996100002%2C%2037.50414198300001%5D%2C%20%5B126.83294767899997%2C%2037.50424917399999%5D%2C%20%5B126.83202386200003%2C%2037.504685274%5D%2C%20%5B126.83188911399998%2C%2037.504828273999976%5D%2C%20%5B126.83161766499995%2C%2037.50511817099999%5D%2C%20%5B126.83147429500002%2C%2037.50551996199999%5D%2C%20%5B126.831205823%2C%2037.50585819499997%5D%2C%20%5B126.83091721100004%2C%2037.50609452399999%5D%2C%20%5B126.83089585699997%2C%2037.506110309%5D%2C%20%5B126.83067831200003%2C%2037.50628107599999%5D%2C%20%5B126.83104406999996%2C%2037.507833059%5D%2C%20%5B126.83105542299995%2C%2037.50801630699999%5D%2C%20%5B126.82901468399996%2C%2037.50853064299997%5D%2C%20%5B126.82767283700002%2C%2037.50868181599998%5D%2C%20%5B126.82729017999998%2C%2037.50855008399998%5D%2C%20%5B126.827077831%2C%2037.50850626699997%5D%2C%20%5B126.826851927%2C%2037.510431685000015%5D%2C%20%5B126.82420556800002%2C%2037.51053149099999%5D%2C%20%5B126.82387418500002%2C%2037.51088144200003%5D%2C%20%5B126.824425776%2C%2037.513182637%5D%2C%20%5B126.824273167%2C%2037.514473764%5D%2C%20%5B126.82341871799997%2C%2037.51495970899998%5D%2C%20%5B126.82312208200005%2C%2037.51622650500002%5D%2C%20%5B126.82451390100005%2C%2037.51772160199999%5D%2C%20%5B126.82563535199995%2C%2037.520085435%5D%2C%20%5B126.82515520799996%2C%2037.523010279%5D%2C%20%5B126.82676624299995%2C%2037.525130822999984%5D%2C%20%5B126.82816515299999%2C%2037.525836518%5D%2C%20%5B126.82844406000004%2C%2037.52670751199997%5D%2C%20%5B126.827086602%2C%2037.529715297%5D%2C%20%5B126.825575308%2C%2037.529853316000015%5D%2C%20%5B126.82344723300002%2C%2037.532623817%5D%2C%20%5B126.82356902000004%2C%2037.533770715%5D%2C%20%5B126.82193790199995%2C%2037.534835165%5D%2C%20%5B126.82244956399995%2C%2037.536550956999974%5D%2C%20%5B126.82233701300004%2C%2037.53793206300003%5D%2C%20%5B126.82160576399997%2C%2037.539720295999985%5D%2C%20%5B126.822137097%2C%2037.54068649300001%5D%2C%20%5B126.82395048199999%2C%2037.54130234500002%5D%2C%20%5B126.82406155299998%2C%2037.541341639%5D%2C%20%5B126.82540241100003%2C%2037.54168811599999%5D%2C%20%5B126.82609412900001%2C%2037.54295950300002%5D%2C%20%5B126.82602990199996%2C%2037.544892118%5D%2C%20%5B126.82799376499997%2C%2037.54756725200002%5D%2C%20%5B126.82977393500005%2C%2037.54773905299999%5D%2C%20%5B126.83047124400002%2C%2037.54578202599998%5D%2C%20%5B126.82973283599995%2C%2037.54373183299998%5D%2C%20%5B126.83024905800005%2C%2037.542640431%5D%2C%20%5B126.83000058799996%2C%2037.54166378899998%5D%2C%20%5B126.83311934799997%2C%2037.541857161999985%5D%2C%20%5B126.83374954400006%2C%2037.53985017000002%5D%2C%20%5B126.83517650099998%2C%2037.537539795999976%5D%2C%20%5B126.83442875399999%2C%2037.53646269400002%5D%2C%20%5B126.84049665600003%2C%2037.526492566%5D%2C%20%5B126.84889070199995%2C%2037.52790219899998%5D%2C%20%5B126.86399120500005%2C%2037.52979477299999%5D%2C%20%5B126.86346027599996%2C%2037.536019463%5D%2C%20%5B126.86380706299997%2C%2037.540478968%5D%2C%20%5B126.862137233%2C%2037.544494887999974%5D%2C%20%5B126.86422519899997%2C%2037.55115840799999%5D%2C%20%5B126.86577025199995%2C%2037.55065963800001%5D%2C%20%5B126.87027588700005%2C%2037.547949199000016%5D%2C%20%5B126.87331303200006%2C%2037.546983667%5D%2C%20%5B126.87497518099997%2C%2037.54697092499998%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2022%2C%20%22SHAPE_AREA%22%3A%200.001775%2C%20%22SHAPE_LEN%22%3A%200.294094%2C%20%22SIG_CD%22%3A%20%2211470%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yangcheon-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc591%5Cucc9c%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.880482788%2C%2037.556271095%5D%2C%20%5B126.88140990299996%2C%2037.55581268899999%5D%2C%20%5B126.886843064%2C%2037.553092693%5D%2C%20%5B126.88827627700005%2C%2037.55221745099999%5D%2C%20%5B126.89470263500004%2C%2037.54939842599998%5D%2C%20%5B126.89896042400005%2C%2037.54801076299998%5D%2C%20%5B126.90090612999995%2C%2037.54598863000001%5D%2C%20%5B126.90091160600002%2C%2037.54598303400002%5D%2C%20%5B126.90091566000001%2C%2037.545978843%5D%2C%20%5B126.90220230199998%2C%2037.544136991000016%5D%2C%20%5B126.90482979900003%2C%2037.54143665%5D%2C%20%5B126.90484853500004%2C%2037.54142033199997%5D%2C%20%5B126.90486495899995%2C%2037.54142006500001%5D%2C%20%5B126.91676526900005%2C%2037.54142882299999%5D%2C%20%5B126.91678366199994%2C%2037.54142883899999%5D%2C%20%5B126.917158427%2C%2037.541428586999984%5D%2C%20%5B126.93012850599996%2C%2037.541440755%5D%2C%20%5B126.93417119499998%2C%2037.53992407099997%5D%2C%20%5B126.93535165799995%2C%2037.53935763700002%5D%2C%20%5B126.93857483800002%2C%2037.53783994299999%5D%2C%20%5B126.94423608600005%2C%2037.53423989100003%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.94957704%2C%2037.517625033%5D%2C%20%5B126.93929870600005%2C%2037.51608439199998%5D%2C%20%5B126.93415342799995%2C%2037.51592078599998%5D%2C%20%5B126.93240130200002%2C%2037.515581852000025%5D%2C%20%5B126.92901054499998%2C%2037.515424834999976%5D%2C%20%5B126.92687757299996%2C%2037.515549326999974%5D%2C%20%5B126.926776636%2C%2037.51446263399998%5D%2C%20%5B126.92668629000002%2C%2037.514402394%5D%2C%20%5B126.92676249399995%2C%2037.51422774299999%5D%2C%20%5B126.92699479199996%2C%2037.51401377500002%5D%2C%20%5B126.92683638999995%2C%2037.513168414%5D%2C%20%5B126.92677221500003%2C%2037.513143345%5D%2C%20%5B126.926805457%2C%2037.51281227499999%5D%2C%20%5B126.92680191600004%2C%2037.512806090000026%5D%2C%20%5B126.92555671299999%2C%2037.512574954%5D%2C%20%5B126.925404264%2C%2037.512535403000015%5D%2C%20%5B126.92539879499998%2C%2037.512530895%5D%2C%20%5B126.92097461399999%2C%2037.50158352300002%5D%2C%20%5B126.92079040199997%2C%2037.50111921500002%5D%2C%20%5B126.92036365399997%2C%2037.50004429299997%5D%2C%20%5B126.92030996599999%2C%2037.50004285099999%5D%2C%20%5B126.92030308899996%2C%2037.50004059299999%5D%2C%20%5B126.92027037599996%2C%2037.49881823099997%5D%2C%20%5B126.920261062%2C%2037.498515938000025%5D%2C%20%5B126.91942210399998%2C%2037.49760859700001%5D%2C%20%5B126.91444808300002%2C%2037.49657621900002%5D%2C%20%5B126.91422162499998%2C%2037.4965547%5D%2C%20%5B126.91359489299998%2C%2037.49649528100002%5D%2C%20%5B126.91357667499994%2C%2037.49649358%5D%2C%20%5B126.91280869299999%2C%2037.496408221000024%5D%2C%20%5B126.91253952900001%2C%2037.496360763999974%5D%2C%20%5B126.90647628500005%2C%2037.488993391%5D%2C%20%5B126.90622732199995%2C%2037.48869004300002%5D%2C%20%5B126.90335461699999%2C%2037.48518492199997%5D%2C%20%5B126.90321344100005%2C%2037.485011993%5D%2C%20%5B126.90321239399998%2C%2037.48501086700003%5D%2C%20%5B126.90173083299999%2C%2037.485236465000014%5D%2C%20%5B126.90010998499997%2C%2037.48594288700002%5D%2C%20%5B126.899946226%2C%2037.48601476499999%5D%2C%20%5B126.89961583499996%2C%2037.486188915000014%5D%2C%20%5B126.89908189799996%2C%2037.48644728900001%5D%2C%20%5B126.89856624799995%2C%2037.48670891799998%5D%2C%20%5B126.89841798299994%2C%2037.486794273999976%5D%2C%20%5B126.89816055799997%2C%2037.486944774999984%5D%2C%20%5B126.89670357700004%2C%2037.48868356600002%5D%2C%20%5B126.89642698700004%2C%2037.48901063699998%5D%2C%20%5B126.89629283700003%2C%2037.489169242%5D%2C%20%5B126.89609502600001%2C%2037.48985993100001%5D%2C%20%5B126.89587989400002%2C%2037.490499965000026%5D%2C%20%5B126.89539873199999%2C%2037.49212123%5D%2C%20%5B126.89532464399997%2C%2037.49240607500002%5D%2C%20%5B126.895186598%2C%2037.492946511000014%5D%2C%20%5B126.89509224599999%2C%2037.49331805000003%5D%2C%20%5B126.89501298799996%2C%2037.49365359%5D%2C%20%5B126.894709455%2C%2037.494817544%5D%2C%20%5B126.89470483299999%2C%2037.494837819%5D%2C%20%5B126.89463285800002%2C%2037.49510713699999%5D%2C%20%5B126.89457127000003%2C%2037.49550832699998%5D%2C%20%5B126.89433156400003%2C%2037.49644422799997%5D%2C%20%5B126.89433768399999%2C%2037.496674025%5D%2C%20%5B126.89427435699997%2C%2037.496925159%5D%2C%20%5B126.89409645700005%2C%2037.49777436199997%5D%2C%20%5B126.89406033%2C%2037.498169484000016%5D%2C%20%5B126.89405123100005%2C%2037.49820099599998%5D%2C%20%5B126.89355630800003%2C%2037.49971156700002%5D%2C%20%5B126.893471675%2C%2037.49982295%5D%2C%20%5B126.89341295400004%2C%2037.499900579999974%5D%2C%20%5B126.89336938999998%2C%2037.49995795900003%5D%2C%20%5B126.89329732500005%2C%2037.500044608%5D%2C%20%5B126.89294758100004%2C%2037.500365219%5D%2C%20%5B126.89292479200003%2C%2037.50042714099999%5D%2C%20%5B126.892875086%2C%2037.50057237200002%5D%2C%20%5B126.89281771499998%2C%2037.50118729299999%5D%2C%20%5B126.89284005399998%2C%2037.50125263799998%5D%2C%20%5B126.89286500699995%2C%2037.50132587799999%5D%2C%20%5B126.89295209%2C%2037.501582783%5D%2C%20%5B126.89300897199996%2C%2037.50248471499998%5D%2C%20%5B126.89308295299998%2C%2037.502587257000016%5D%2C%20%5B126.89314937300003%2C%2037.50268416900002%5D%2C%20%5B126.89290723%2C%2037.50303395200001%5D%2C%20%5B126.89275738000003%2C%2037.503251067%5D%2C%20%5B126.893181851%2C%2037.50459995900002%5D%2C%20%5B126.89324314999999%2C%2037.50463272799999%5D%2C%20%5B126.89326647899998%2C%2037.504637264999985%5D%2C%20%5B126.89394740900002%2C%2037.504829933999986%5D%2C%20%5B126.89397250499997%2C%2037.505063115999974%5D%2C%20%5B126.89354936400002%2C%2037.50603884100002%5D%2C%20%5B126.89351878599996%2C%2037.506116471999974%5D%2C%20%5B126.89335855499996%2C%2037.50649551700002%5D%2C%20%5B126.89327280099997%2C%2037.50670837299998%5D%2C%20%5B126.89315021699997%2C%2037.50684678699997%5D%2C%20%5B126.89264219699999%2C%2037.508245620000025%5D%2C%20%5B126.89255187000003%2C%2037.50846521300002%5D%2C%20%5B126.89214598800004%2C%2037.50934803000001%5D%2C%20%5B126.89149283200004%2C%2037.51039608299999%5D%2C%20%5B126.890689286%2C%2037.51067584100002%5D%2C%20%5B126.88982957899998%2C%2037.51172454499999%5D%2C%20%5B126.88979881700004%2C%2037.51173352%5D%2C%20%5B126.88970605600002%2C%2037.51190367200002%5D%2C%20%5B126.88972300499995%2C%2037.51224255400001%5D%2C%20%5B126.88880867700004%2C%2037.51225257700003%5D%2C%20%5B126.88837384800001%2C%2037.512353211%5D%2C%20%5B126.88786062999998%2C%2037.51302062299999%5D%2C%20%5B126.88681781000003%2C%2037.51327246400001%5D%2C%20%5B126.885101803%2C%2037.51363473499998%5D%2C%20%5B126.88461590600002%2C%2037.51371960099999%5D%2C%20%5B126.88410520000002%2C%2037.51439608700002%5D%2C%20%5B126.88272419199996%2C%2037.51576865800001%5D%2C%20%5B126.87950673%2C%2037.51777967100003%5D%2C%20%5B126.87844781700005%2C%2037.518250850000015%5D%2C%20%5B126.87823604100004%2C%2037.51914179800002%5D%2C%20%5B126.87945873700005%2C%2037.52060415400001%5D%2C%20%5B126.87942968599998%2C%2037.52083356600002%5D%2C%20%5B126.87902211699998%2C%2037.522463485%5D%2C%20%5B126.879072489%2C%2037.52252545800002%5D%2C%20%5B126.87894314200003%2C%2037.522923967999986%5D%2C%20%5B126.87890921300004%2C%2037.52320861499999%5D%2C%20%5B126.87891105899996%2C%2037.523900688000026%5D%2C%20%5B126.87887963799994%2C%2037.52409388799998%5D%2C%20%5B126.87876972000004%2C%2037.524379673%5D%2C%20%5B126.87872851400004%2C%2037.52446391400002%5D%2C%20%5B126.87864978100004%2C%2037.524624872%5D%2C%20%5B126.87864002000003%2C%2037.52464937000002%5D%2C%20%5B126.87863358300001%2C%2037.524689926%5D%2C%20%5B126.87863270800005%2C%2037.52472801099998%5D%2C%20%5B126.87862522399996%2C%2037.525054436%5D%2C%20%5B126.87862801999995%2C%2037.52507557000001%5D%2C%20%5B126.87877313900003%2C%2037.52536172399999%5D%2C%20%5B126.87906872200006%2C%2037.525608309%5D%2C%20%5B126.87916542599999%2C%2037.525665076999985%5D%2C%20%5B126.87920696499998%2C%2037.525682888%5D%2C%20%5B126.87930474300003%2C%2037.525723028000016%5D%2C%20%5B126.87934842699997%2C%2037.525732671000014%5D%2C%20%5B126.87938060199997%2C%2037.52573750099998%5D%2C%20%5B126.87946222599999%2C%2037.52574409099998%5D%2C%20%5B126.87949307999997%2C%2037.52574413000002%5D%2C%20%5B126.87955157399995%2C%2037.52574453099999%5D%2C%20%5B126.88025848799998%2C%2037.52574511900002%5D%2C%20%5B126.88034567399995%2C%2037.52574466300001%5D%2C%20%5B126.88042206099999%2C%2037.52574601499998%5D%2C%20%5B126.88046628100005%2C%2037.525746793999986%5D%2C%20%5B126.88050667599998%2C%2037.52574831599998%5D%2C%20%5B126.88059625599999%2C%2037.525757665000015%5D%2C%20%5B126.88072924100004%2C%2037.525787713%5D%2C%20%5B126.880830543%2C%2037.525843079000026%5D%2C%20%5B126.88095394100003%2C%2037.525913127000024%5D%2C%20%5B126.880958534%2C%2037.52591623500001%5D%2C%20%5B126.880962426%2C%2037.525919622%5D%2C%20%5B126.88101396000002%2C%2037.52598215%5D%2C%20%5B126.881020724%2C%2037.525990374%5D%2C%20%5B126.88112879899995%2C%2037.526121596%5D%2C%20%5B126.88114513599999%2C%2037.52614302500001%5D%2C%20%5B126.88114840599997%2C%2037.52614991399997%5D%2C%20%5B126.88115262899998%2C%2037.526162296%5D%2C%20%5B126.88115860799996%2C%2037.52618087399998%5D%2C%20%5B126.88116230100002%2C%2037.52619241000002%5D%2C%20%5B126.88116847100002%2C%2037.526211519000015%5D%2C%20%5B126.88119754700006%2C%2037.52648983300003%5D%2C%20%5B126.88119628499999%2C%2037.526508686%5D%2C%20%5B126.88119560899997%2C%2037.526521281999976%5D%2C%20%5B126.88119022499995%2C%2037.52654306900001%5D%2C%20%5B126.88118808700006%2C%2037.52655032500002%5D%2C%20%5B126.88109137799995%2C%2037.52667717600002%5D%2C%20%5B126.88067097199996%2C%2037.52719444799999%5D%2C%20%5B126.88065025399999%2C%2037.52724797399998%5D%2C%20%5B126.88064234%2C%2037.52727373%5D%2C%20%5B126.88063130700004%2C%2037.52732632499999%5D%2C%20%5B126.88062427900002%2C%2037.527408219999984%5D%2C%20%5B126.88062496600003%2C%2037.527425102%5D%2C%20%5B126.88069565%2C%2037.52755737699999%5D%2C%20%5B126.88070817799996%2C%2037.52756919699999%5D%2C%20%5B126.88076000299998%2C%2037.527607038999975%5D%2C%20%5B126.88082542200004%2C%2037.52765479200002%5D%2C%20%5B126.88092680099999%2C%2037.52771336799998%5D%2C%20%5B126.88170850300003%2C%2037.52769407%5D%2C%20%5B126.88178394900001%2C%2037.52767390299999%5D%2C%20%5B126.88258119399995%2C%2037.52733278599999%5D%2C%20%5B126.88260056900003%2C%2037.52732377500001%5D%2C%20%5B126.88274796200005%2C%2037.52725498199999%5D%2C%20%5B126.88292853899998%2C%2037.527235555%5D%2C%20%5B126.88314068600005%2C%2037.527239345%5D%2C%20%5B126.88316834700004%2C%2037.52724387500001%5D%2C%20%5B126.883243387%2C%2037.527256610999984%5D%2C%20%5B126.883304715%2C%2037.52727440199999%5D%2C%20%5B126.88367235700002%2C%2037.527481308%5D%2C%20%5B126.88369679499999%2C%2037.52750744399998%5D%2C%20%5B126.88374950000002%2C%2037.527563826%5D%2C%20%5B126.88376854600006%2C%2037.527592829000014%5D%2C%20%5B126.88386795600002%2C%2037.527805649000015%5D%2C%20%5B126.88387623200003%2C%2037.52782591599998%5D%2C%20%5B126.88391099499995%2C%2037.527992525%5D%2C%20%5B126.88391741199996%2C%2037.52807525100002%5D%2C%20%5B126.88393859099995%2C%2037.52841122000001%5D%2C%20%5B126.88394041799995%2C%2037.52841574199999%5D%2C%20%5B126.88396141500004%2C%2037.528442208%5D%2C%20%5B126.88397379399998%2C%2037.528457995%5D%2C%20%5B126.88401771899998%2C%2037.528513729%5D%2C%20%5B126.88403994700002%2C%2037.528541898000014%5D%2C%20%5B126.88404878400002%2C%2037.528550623%5D%2C%20%5B126.884054784%2C%2037.528556544000025%5D%2C%20%5B126.88416551399996%2C%2037.52864668699999%5D%2C%20%5B126.88419206399999%2C%2037.52866820600002%5D%2C%20%5B126.88431254%2C%2037.52874229600002%5D%2C%20%5B126.88432471500005%2C%2037.52874922%5D%2C%20%5B126.88435311900002%2C%2037.528764588%5D%2C%20%5B126.884441907%2C%2037.528805239%5D%2C%20%5B126.88447697699996%2C%2037.52881814800003%5D%2C%20%5B126.88451481499999%2C%2037.52883207899998%5D%2C%20%5B126.884525446%2C%2037.52883619200003%5D%2C%20%5B126.88454252899999%2C%2037.52884280799998%5D%2C%20%5B126.884716647%2C%2037.52893355200001%5D%2C%20%5B126.88500349399999%2C%2037.52908013699999%5D%2C%20%5B126.88503601399998%2C%2037.529095961%5D%2C%20%5B126.88506209499997%2C%2037.529108651%5D%2C%20%5B126.88510402600002%2C%2037.52912798199998%5D%2C%20%5B126.885202048%2C%2037.529171157%5D%2C%20%5B126.88524513699997%2C%2037.52918854400002%5D%2C%20%5B126.88583773599998%2C%2037.52938733000002%5D%2C%20%5B126.88590605299999%2C%2037.529409272%5D%2C%20%5B126.88596476700002%2C%2037.529427914%5D%2C%20%5B126.88622199199995%2C%2037.529559806%5D%2C%20%5B126.88624437500005%2C%2037.52957399899998%5D%2C%20%5B126.88625849599998%2C%2037.529582951%5D%2C%20%5B126.88708983900005%2C%2037.530141571%5D%2C%20%5B126.88712875399995%2C%2037.530163791%5D%2C%20%5B126.88718459699999%2C%2037.53019168899999%5D%2C%20%5B126.88720890900004%2C%2037.53020161400002%5D%2C%20%5B126.88733526299995%2C%2037.530245647000015%5D%2C%20%5B126.88766516700002%2C%2037.53035981199997%5D%2C%20%5B126.88770304299999%2C%2037.53037303799999%5D%2C%20%5B126.88794181499998%2C%2037.530402723%5D%2C%20%5B126.88942952599996%2C%2037.53012460100001%5D%2C%20%5B126.88945461399999%2C%2037.530106052%5D%2C%20%5B126.88950550899995%2C%2037.53005347200002%5D%2C%20%5B126.88998939099997%2C%2037.53069573%5D%2C%20%5B126.89005864299997%2C%2037.530794592%5D%2C%20%5B126.89024439699995%2C%2037.53105937700002%5D%2C%20%5B126.89027045299997%2C%2037.531095963999974%5D%2C%20%5B126.89027927400002%2C%2037.531108394%5D%2C%20%5B126.89046274600003%2C%2037.531367799%5D%2C%20%5B126.89048699099999%2C%2037.53140269900001%5D%2C%20%5B126.890701078%2C%2037.53166245800003%5D%2C%20%5B126.890597319%2C%2037.53173105799999%5D%2C%20%5B126.89030528%2C%2037.53187488999998%5D%2C%20%5B126.88967441600005%2C%2037.531914234%5D%2C%20%5B126.889539292%2C%2037.53191990099998%5D%2C%20%5B126.88946426500002%2C%2037.531917971999974%5D%2C%20%5B126.88940643399997%2C%2037.53191594200001%5D%2C%20%5B126.88910220000002%2C%2037.531914793%5D%2C%20%5B126.88857722900002%2C%2037.53225427299998%5D%2C%20%5B126.88856422699996%2C%2037.53228870100003%5D%2C%20%5B126.888552634%2C%2037.532398467%5D%2C%20%5B126.888545324%2C%2037.532560875%5D%2C%20%5B126.888562052%2C%2037.53260258900002%5D%2C%20%5B126.88857919400004%2C%2037.532645338%5D%2C%20%5B126.88870756100005%2C%2037.532810919999974%5D%2C%20%5B126.88875218099997%2C%2037.532843598%5D%2C%20%5B126.88923708599998%2C%2037.53310700700001%5D%2C%20%5B126.88947613599998%2C%2037.533215333999976%5D%2C%20%5B126.88966320400004%2C%2037.53331432099998%5D%2C%20%5B126.889748456%2C%2037.533364027%5D%2C%20%5B126.889783136%2C%2037.533384247000015%5D%2C%20%5B126.88983757799997%2C%2037.53342374499999%5D%2C%20%5B126.88984633099994%2C%2037.53343159899998%5D%2C%20%5B126.88995087800004%2C%2037.53356766000002%5D%2C%20%5B126.89005078800005%2C%2037.533715535%5D%2C%20%5B126.89012591899996%2C%2037.53396188%5D%2C%20%5B126.89016443599996%2C%2037.53408776200001%5D%2C%20%5B126.89016628800005%2C%2037.53410340900001%5D%2C%20%5B126.890167937%2C%2037.53411732799998%5D%2C%20%5B126.89016660699997%2C%2037.53414441199999%5D%2C%20%5B126.890162153%2C%2037.53423441899997%5D%2C%20%5B126.89015755399998%2C%2037.534269173999974%5D%2C%20%5B126.89014271899998%2C%2037.534381051000025%5D%2C%20%5B126.89001040799997%2C%2037.53471335%5D%2C%20%5B126.88998342900004%2C%2037.53476834999998%5D%2C%20%5B126.88975681399995%2C%2037.53514886300002%5D%2C%20%5B126.88972001599996%2C%2037.53520282%5D%2C%20%5B126.88827555099999%2C%2037.53672107199998%5D%2C%20%5B126.88809348799998%2C%2037.536979015999975%5D%2C%20%5B126.887683993%2C%2037.537571119%5D%2C%20%5B126.88737886299998%2C%2037.538148261%5D%2C%20%5B126.88712228999998%2C%2037.53859281299998%5D%2C%20%5B126.88709413000004%2C%2037.53863500199998%5D%2C%20%5B126.88705390999996%2C%2037.538681086%5D%2C%20%5B126.88701390400001%2C%2037.538715695%5D%2C%20%5B126.88691566900002%2C%2037.53878714199999%5D%2C%20%5B126.88654807800003%2C%2037.53936766100003%5D%2C%20%5B126.88654236399998%2C%2037.53937892499999%5D%2C%20%5B126.88649361900002%2C%2037.539475149%5D%2C%20%5B126.88644990900002%2C%2037.53955651299998%5D%2C%20%5B126.88640822499997%2C%2037.53963411500001%5D%2C%20%5B126.88629931800006%2C%2037.53983162999998%5D%2C%20%5B126.88616839899998%2C%2037.54010612899998%5D%2C%20%5B126.88610640000002%2C%2037.54024347400002%5D%2C%20%5B126.88552416000005%2C%2037.541581647999976%5D%2C%20%5B126.88551157999996%2C%2037.54163955500002%5D%2C%20%5B126.88546812200002%2C%2037.541844887000025%5D%2C%20%5B126.88546669100003%2C%2037.54185502299998%5D%2C%20%5B126.885462931%2C%2037.541989340999976%5D%2C%20%5B126.88545409899996%2C%2037.542312330000016%5D%2C%20%5B126.88545374099999%2C%2037.542317114000014%5D%2C%20%5B126.88472229299998%2C%2037.543331357%5D%2C%20%5B126.88471485900004%2C%2037.543336133000025%5D%2C%20%5B126.88428452999995%2C%2037.54375910700003%5D%2C%20%5B126.88428240600001%2C%2037.543867239%5D%2C%20%5B126.88425672000005%2C%2037.544324808%5D%2C%20%5B126.88424155500002%2C%2037.544405331%5D%2C%20%5B126.88232139900003%2C%2037.546488942%5D%2C%20%5B126.88218328899995%2C%2037.54667742999999%5D%2C%20%5B126.882021861%2C%2037.54685068499998%5D%2C%20%5B126.882012121%2C%2037.546859962999974%5D%2C%20%5B126.88199706399996%2C%2037.546873748%5D%2C%20%5B126.881991395%2C%2037.546878805%5D%2C%20%5B126.88080691100004%2C%2037.548092827%5D%2C%20%5B126.87922770199998%2C%2037.550313711%5D%2C%20%5B126.87913820899996%2C%2037.550415493%5D%2C%20%5B126.878063655%2C%2037.55170404699999%5D%2C%20%5B126.87802198700001%2C%2037.55303843399997%5D%2C%20%5B126.87796921400002%2C%2037.55317292500001%5D%2C%20%5B126.87923394200004%2C%2037.55315397700002%5D%2C%20%5B126.87925520399995%2C%2037.554749062999974%5D%2C%20%5B126.880482788%2C%2037.556271095%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2023%2C%20%22SHAPE_AREA%22%3A%200.002512%2C%20%22SHAPE_LEN%22%3A%200.262953%2C%20%22SIG_CD%22%3A%20%2211560%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yeongdeungpo-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc601%5Cub4f1%5Cud3ec%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B126.97727486600002%2C%2037.55501144499999%5D%2C%20%5B126.977722319%2C%2037.55467309599999%5D%2C%20%5B126.97787311599996%2C%2037.554567831999975%5D%2C%20%5B126.97812035599998%2C%2037.554414442%5D%2C%20%5B126.97874814399995%2C%2037.55320524899997%5D%2C%20%5B126.98070985499999%2C%2037.553673457%5D%2C%20%5B126.98084840700005%2C%2037.553835521999986%5D%2C%20%5B126.98173850700005%2C%2037.55386152400001%5D%2C%20%5B126.98404581499994%2C%2037.55304170199997%5D%2C%20%5B126.98543609499995%2C%2037.55370785299999%5D%2C%20%5B126.98773489899997%2C%2037.55145462199999%5D%2C%20%5B126.99072224600002%2C%2037.551121825999985%5D%2C%20%5B126.99297688900003%2C%2037.54952155199999%5D%2C%20%5B126.99314556000002%2C%2037.54908673199998%5D%2C%20%5B126.994025502%2C%2037.548413455%5D%2C%20%5B126.99432095300006%2C%2037.54788345100002%5D%2C%20%5B126.99525161199995%2C%2037.547235589000024%5D%2C%20%5B126.996710162%2C%2037.54806677800002%5D%2C%20%5B126.99677169400002%2C%2037.548242792%5D%2C%20%5B126.99739550100003%2C%2037.54900462099999%5D%2C%20%5B126.99767729200005%2C%2037.549366255999985%5D%2C%20%5B127.00053584199998%2C%2037.55002562200002%5D%2C%20%5B127.00095637200002%2C%2037.54993994599999%5D%2C%20%5B127.00283224600003%2C%2037.549636636%5D%2C%20%5B127.00286871699996%2C%2037.54964536599999%5D%2C%20%5B127.00433800400003%2C%2037.550214794%5D%2C%20%5B127.00470587300003%2C%2037.54868811099999%5D%2C%20%5B127.00614513000005%2C%2037.54819629999997%5D%2C%20%5B127.005035582%2C%2037.546176924%5D%2C%20%5B127.00728455399997%2C%2037.54386521399999%5D%2C%20%5B127.00900602399997%2C%2037.54414177899997%5D%2C%20%5B127.00877504899995%2C%2037.542004603%5D%2C%20%5B127.00996347600005%2C%2037.539392701%5D%2C%20%5B127.01150102899999%2C%2037.539013259%5D%2C%20%5B127.01389012300001%2C%2037.539073175%5D%2C%20%5B127.01754633799999%2C%2037.537656207%5D%2C%20%5B127.01745612000002%2C%2037.53399531500003%5D%2C%20%5B127.01503715700005%2C%2037.53213505799999%5D%2C%20%5B127.01231011300001%2C%2037.52954053399998%5D%2C%20%5B127.00859622099995%2C%2037.52560038199999%5D%2C%20%5B127.00650365900003%2C%2037.52346973099998%5D%2C%20%5B126.998953271%2C%2037.518326925999986%5D%2C%20%5B126.99815390399999%2C%2037.51781911699999%5D%2C%20%5B126.99669526100001%2C%2037.516895669%5D%2C%20%5B126.995872351%2C%2037.51637447600001%5D%2C%20%5B126.99070165900002%2C%2037.513098911999975%5D%2C%20%5B126.98556279299999%2C%2037.506554987000015%5D%2C%20%5B126.98042062399998%2C%2037.50655008299998%5D%2C%20%5B126.97854593299996%2C%2037.50656086599997%5D%2C%20%5B126.97818666%2C%2037.50656420299998%5D%2C%20%5B126.977737381%2C%2037.506635034%5D%2C%20%5B126.97526363400004%2C%2037.50702325999998%5D%2C%20%5B126.97013741700005%2C%2037.50901084100002%5D%2C%20%5B126.96701655599998%2C%2037.50985079399999%5D%2C%20%5B126.964993413%2C%2037.51082665400003%5D%2C%20%5B126.96120749700003%2C%2037.513467042%5D%2C%20%5B126.95730536300005%2C%2037.51499314199998%5D%2C%20%5B126.95662402599999%2C%2037.515275953000014%5D%2C%20%5B126.954724306%2C%2037.51603819799999%5D%2C%20%5B126.94990256999995%2C%2037.51751810600001%5D%2C%20%5B126.949893697%2C%2037.527031626999985%5D%2C%20%5B126.94888144000004%2C%2037.528321799000025%5D%2C%20%5B126.94869440399998%2C%2037.52856010900001%5D%2C%20%5B126.94460208400005%2C%2037.53378549500002%5D%2C%20%5B126.94511557099997%2C%2037.53496920399999%5D%2C%20%5B126.94643874200005%2C%2037.53573910599999%5D%2C%20%5B126.94861459399999%2C%2037.53558084500003%5D%2C%20%5B126.94867770600001%2C%2037.535595494%5D%2C%20%5B126.950952035%2C%2037.53616902800002%5D%2C%20%5B126.95356855800003%2C%2037.53800882899998%5D%2C%20%5B126.95363528400003%2C%2037.53819027700001%5D%2C%20%5B126.95411208799999%2C%2037.53893482%5D%2C%20%5B126.95574987099997%2C%2037.53947266199998%5D%2C%20%5B126.95700289299998%2C%2037.541638852%5D%2C%20%5B126.95819293199997%2C%2037.54231346400002%5D%2C%20%5B126.95742465%2C%2037.543226979999986%5D%2C%20%5B126.95783517899997%2C%2037.54515508600002%5D%2C%20%5B126.96014854199996%2C%2037.546376771999974%5D%2C%20%5B126.96109354299995%2C%2037.547421300999986%5D%2C%20%5B126.96335588700003%2C%2037.548645291000014%5D%2C%20%5B126.96372616999997%2C%2037.54876584599998%5D%2C%20%5B126.96391355499998%2C%2037.549765217000015%5D%2C%20%5B126.96285693200002%2C%2037.55102707499998%5D%2C%20%5B126.96266110099998%2C%2037.551166454999986%5D%2C%20%5B126.96235773199999%2C%2037.551552155000024%5D%2C%20%5B126.96572364400004%2C%2037.554153031%5D%2C%20%5B126.96919921200004%2C%2037.55566477399998%5D%2C%20%5B126.96918664999998%2C%2037.55487825199998%5D%2C%20%5B126.97242400899995%2C%2037.554879888000016%5D%2C%20%5B126.97436538399995%2C%2037.553550108000024%5D%2C%20%5B126.97644341%2C%2037.55308918600002%5D%2C%20%5B126.97727486600002%2C%2037.55501144499999%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22properties%22%3A%20%7B%22ESRI_PK%22%3A%2024%2C%20%22SHAPE_AREA%22%3A%200.002234%2C%20%22SHAPE_LEN%22%3A%200.214496%2C%20%22SIG_CD%22%3A%20%2211170%22%2C%20%22SIG_ENG_NM%22%3A%20%22Yongsan-gu%22%2C%20%22SIG_KOR_NM%22%3A%20%22%5Cuc6a9%5Cuc0b0%5Cuad6c%22%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20var%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e%20%3D%20%7B%7D%3B%0A%0A%20%20%20%20%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.color%20%3D%20d3.scale.threshold%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B10.0%2C%2010.030060120240481%2C%2010.060120240480963%2C%2010.090180360721442%2C%2010.120240480961924%2C%2010.150300601202405%2C%2010.180360721442886%2C%2010.210420841683367%2C%2010.240480961923847%2C%2010.270541082164328%2C%2010.30060120240481%2C%2010.330661322645291%2C%2010.360721442885772%2C%2010.390781563126252%2C%2010.420841683366733%2C%2010.450901803607215%2C%2010.480961923847696%2C%2010.511022044088175%2C%2010.541082164328657%2C%2010.571142284569138%2C%2010.60120240480962%2C%2010.6312625250501%2C%2010.661322645290582%2C%2010.691382765531062%2C%2010.721442885771543%2C%2010.751503006012024%2C%2010.781563126252506%2C%2010.811623246492985%2C%2010.841683366733466%2C%2010.871743486973948%2C%2010.901803607214429%2C%2010.93186372745491%2C%2010.96192384769539%2C%2010.991983967935871%2C%2011.022044088176353%2C%2011.052104208416834%2C%2011.082164328657315%2C%2011.112224448897795%2C%2011.142284569138276%2C%2011.172344689378757%2C%2011.202404809619239%2C%2011.23246492985972%2C%2011.2625250501002%2C%2011.292585170340681%2C%2011.322645290581162%2C%2011.352705410821644%2C%2011.382765531062125%2C%2011.412825651302605%2C%2011.442885771543086%2C%2011.472945891783567%2C%2011.503006012024048%2C%2011.53306613226453%2C%2011.56312625250501%2C%2011.59318637274549%2C%2011.623246492985972%2C%2011.653306613226453%2C%2011.683366733466933%2C%2011.713426853707414%2C%2011.743486973947896%2C%2011.773547094188377%2C%2011.803607214428858%2C%2011.83366733466934%2C%2011.863727454909819%2C%2011.8937875751503%2C%2011.923847695390782%2C%2011.953907815631263%2C%2011.983967935871743%2C%2012.014028056112224%2C%2012.044088176352705%2C%2012.074148296593187%2C%2012.104208416833668%2C%2012.13426853707415%2C%2012.164328657314629%2C%2012.19438877755511%2C%2012.224448897795591%2C%2012.254509018036073%2C%2012.284569138276552%2C%2012.314629258517034%2C%2012.344689378757515%2C%2012.374749498997996%2C%2012.404809619238478%2C%2012.434869739478959%2C%2012.464929859719438%2C%2012.49498997995992%2C%2012.525050100200401%2C%2012.555110220440882%2C%2012.585170340681362%2C%2012.615230460921843%2C%2012.645290581162325%2C%2012.675350701402806%2C%2012.705410821643287%2C%2012.735470941883769%2C%2012.765531062124248%2C%2012.79559118236473%2C%2012.82565130260521%2C%2012.85571142284569%2C%2012.885771543086172%2C%2012.915831663326653%2C%2012.945891783567134%2C%2012.975951903807616%2C%2013.006012024048097%2C%2013.036072144288577%2C%2013.066132264529058%2C%2013.09619238476954%2C%2013.12625250501002%2C%2013.1563126252505%2C%2013.186372745490981%2C%2013.216432865731463%2C%2013.246492985971944%2C%2013.276553106212425%2C%2013.306613226452907%2C%2013.336673346693386%2C%2013.366733466933868%2C%2013.396793587174349%2C%2013.42685370741483%2C%2013.45691382765531%2C%2013.486973947895791%2C%2013.517034068136272%2C%2013.547094188376754%2C%2013.577154308617235%2C%2013.607214428857716%2C%2013.637274549098196%2C%2013.667334669338677%2C%2013.697394789579159%2C%2013.72745490981964%2C%2013.75751503006012%2C%2013.7875751503006%2C%2013.817635270541082%2C%2013.847695390781563%2C%2013.877755511022045%2C%2013.907815631262526%2C%2013.937875751503006%2C%2013.967935871743487%2C%2013.997995991983968%2C%2014.028056112224448%2C%2014.05811623246493%2C%2014.08817635270541%2C%2014.118236472945892%2C%2014.148296593186373%2C%2014.178356713426854%2C%2014.208416833667336%2C%2014.238476953907815%2C%2014.268537074148297%2C%2014.298597194388778%2C%2014.328657314629258%2C%2014.358717434869739%2C%2014.38877755511022%2C%2014.418837675350701%2C%2014.448897795591183%2C%2014.478957915831664%2C%2014.509018036072145%2C%2014.539078156312625%2C%2014.569138276553106%2C%2014.599198396793586%2C%2014.629258517034067%2C%2014.659318637274549%2C%2014.68937875751503%2C%2014.719438877755511%2C%2014.749498997995993%2C%2014.779559118236474%2C%2014.809619238476955%2C%2014.839679358717435%2C%2014.869739478957916%2C%2014.899799599198396%2C%2014.929859719438877%2C%2014.959919839679358%2C%2014.98997995991984%2C%2015.02004008016032%2C%2015.050100200400802%2C%2015.080160320641284%2C%2015.110220440881765%2C%2015.140280561122244%2C%2015.170340681362726%2C%2015.200400801603205%2C%2015.230460921843687%2C%2015.260521042084168%2C%2015.29058116232465%2C%2015.32064128256513%2C%2015.350701402805612%2C%2015.380761523046093%2C%2015.410821643286573%2C%2015.440881763527054%2C%2015.470941883767535%2C%2015.501002004008015%2C%2015.531062124248496%2C%2015.561122244488978%2C%2015.591182364729459%2C%2015.62124248496994%2C%2015.651302605210422%2C%2015.681362725450903%2C%2015.711422845691382%2C%2015.741482965931864%2C%2015.771543086172345%2C%2015.801603206412825%2C%2015.831663326653306%2C%2015.861723446893787%2C%2015.891783567134269%2C%2015.92184368737475%2C%2015.951903807615231%2C%2015.981963927855713%2C%2016.012024048096194%2C%2016.04208416833667%2C%2016.072144288577153%2C%2016.102204408817634%2C%2016.132264529058116%2C%2016.162324649298597%2C%2016.19238476953908%2C%2016.22244488977956%2C%2016.25250501002004%2C%2016.282565130260522%2C%2016.312625250501%2C%2016.342685370741485%2C%2016.372745490981963%2C%2016.402805611222444%2C%2016.432865731462925%2C%2016.462925851703407%2C%2016.492985971943888%2C%2016.52304609218437%2C%2016.55310621242485%2C%2016.58316633266533%2C%2016.613226452905813%2C%2016.64328657314629%2C%2016.673346693386772%2C%2016.703406813627254%2C%2016.733466933867735%2C%2016.763527054108216%2C%2016.793587174348698%2C%2016.82364729458918%2C%2016.85370741482966%2C%2016.88376753507014%2C%2016.91382765531062%2C%2016.943887775551104%2C%2016.973947895791582%2C%2017.004008016032063%2C%2017.034068136272545%2C%2017.064128256513026%2C%2017.094188376753507%2C%2017.12424849699399%2C%2017.15430861723447%2C%2017.184368737474948%2C%2017.214428857715433%2C%2017.24448897795591%2C%2017.274549098196392%2C%2017.304609218436873%2C%2017.334669338677354%2C%2017.364729458917836%2C%2017.394789579158317%2C%2017.4248496993988%2C%2017.45490981963928%2C%2017.48496993987976%2C%2017.51503006012024%2C%2017.54509018036072%2C%2017.5751503006012%2C%2017.605210420841683%2C%2017.635270541082164%2C%2017.665330661322646%2C%2017.695390781563127%2C%2017.725450901803608%2C%2017.75551102204409%2C%2017.785571142284567%2C%2017.815631262525052%2C%2017.84569138276553%2C%2017.87575150300601%2C%2017.905811623246493%2C%2017.935871743486974%2C%2017.965931863727455%2C%2017.995991983967937%2C%2018.026052104208418%2C%2018.056112224448896%2C%2018.08617234468938%2C%2018.11623246492986%2C%2018.146292585170343%2C%2018.17635270541082%2C%2018.206412825651302%2C%2018.236472945891784%2C%2018.266533066132265%2C%2018.296593186372746%2C%2018.326653306613224%2C%2018.35671342685371%2C%2018.386773547094187%2C%2018.41683366733467%2C%2018.44689378757515%2C%2018.47695390781563%2C%2018.507014028056112%2C%2018.537074148296593%2C%2018.567134268537075%2C%2018.597194388777556%2C%2018.627254509018037%2C%2018.657314629258515%2C%2018.687374749499%2C%2018.717434869739478%2C%2018.747494989979963%2C%2018.77755511022044%2C%2018.80761523046092%2C%2018.837675350701403%2C%2018.867735470941884%2C%2018.897795591182366%2C%2018.927855711422843%2C%2018.95791583166333%2C%2018.987975951903806%2C%2019.01803607214429%2C%2019.04809619238477%2C%2019.07815631262525%2C%2019.10821643286573%2C%2019.138276553106213%2C%2019.168336673346694%2C%2019.19839679358717%2C%2019.228456913827657%2C%2019.258517034068134%2C%2019.28857715430862%2C%2019.318637274549097%2C%2019.34869739478958%2C%2019.37875751503006%2C%2019.40881763527054%2C%2019.438877755511022%2C%2019.468937875751504%2C%2019.498997995991985%2C%2019.529058116232463%2C%2019.559118236472948%2C%2019.589178356713425%2C%2019.61923847695391%2C%2019.649298597194388%2C%2019.67935871743487%2C%2019.70941883767535%2C%2019.739478957915832%2C%2019.769539078156313%2C%2019.79959919839679%2C%2019.829659318637276%2C%2019.859719438877754%2C%2019.88977955911824%2C%2019.919839679358716%2C%2019.949899799599198%2C%2019.97995991983968%2C%2020.01002004008016%2C%2020.04008016032064%2C%2020.070140280561123%2C%2020.100200400801604%2C%2020.130260521042082%2C%2020.160320641282567%2C%2020.190380761523045%2C%2020.22044088176353%2C%2020.250501002004007%2C%2020.28056112224449%2C%2020.31062124248497%2C%2020.34068136272545%2C%2020.370741482965933%2C%2020.40080160320641%2C%2020.430861723446895%2C%2020.460921843687373%2C%2020.490981963927858%2C%2020.521042084168336%2C%2020.551102204408817%2C%2020.5811623246493%2C%2020.61122244488978%2C%2020.64128256513026%2C%2020.67134268537074%2C%2020.701402805611224%2C%2020.7314629258517%2C%2020.761523046092186%2C%2020.791583166332664%2C%2020.821643286573146%2C%2020.851703406813627%2C%2020.881763527054108%2C%2020.91182364729459%2C%2020.94188376753507%2C%2020.971943887775552%2C%2021.00200400801603%2C%2021.032064128256515%2C%2021.062124248496993%2C%2021.092184368737477%2C%2021.122244488977955%2C%2021.152304609218437%2C%2021.182364729458918%2C%2021.2124248496994%2C%2021.24248496993988%2C%2021.27254509018036%2C%2021.302605210420843%2C%2021.33266533066132%2C%2021.362725450901806%2C%2021.392785571142284%2C%2021.422845691382765%2C%2021.452905811623246%2C%2021.482965931863728%2C%2021.51302605210421%2C%2021.54308617234469%2C%2021.57314629258517%2C%2021.60320641282565%2C%2021.633266533066134%2C%2021.663326653306612%2C%2021.693386773547093%2C%2021.723446893787575%2C%2021.753507014028056%2C%2021.783567134268537%2C%2021.81362725450902%2C%2021.8436873747495%2C%2021.873747494989978%2C%2021.903807615230463%2C%2021.93386773547094%2C%2021.963927855711425%2C%2021.993987975951903%2C%2022.024048096192384%2C%2022.054108216432866%2C%2022.084168336673347%2C%2022.11422845691383%2C%2022.144288577154306%2C%2022.17434869739479%2C%2022.20440881763527%2C%2022.234468937875754%2C%2022.26452905811623%2C%2022.294589178356713%2C%2022.324649298597194%2C%2022.354709418837675%2C%2022.384769539078157%2C%2022.414829659318638%2C%2022.44488977955912%2C%2022.474949899799597%2C%2022.505010020040082%2C%2022.53507014028056%2C%2022.565130260521045%2C%2022.595190380761522%2C%2022.625250501002004%2C%2022.655310621242485%2C%2022.685370741482966%2C%2022.715430861723448%2C%2022.745490981963925%2C%2022.77555110220441%2C%2022.805611222444888%2C%2022.835671342685373%2C%2022.86573146292585%2C%2022.895791583166332%2C%2022.925851703406813%2C%2022.955911823647295%2C%2022.985971943887776%2C%2023.016032064128254%2C%2023.04609218436874%2C%2023.076152304609217%2C%2023.1062124248497%2C%2023.13627254509018%2C%2023.16633266533066%2C%2023.196392785571142%2C%2023.226452905811623%2C%2023.256513026052104%2C%2023.286573146292586%2C%2023.316633266533067%2C%2023.346693386773545%2C%2023.37675350701403%2C%2023.406813627254508%2C%2023.436873747494992%2C%2023.46693386773547%2C%2023.49699398797595%2C%2023.527054108216433%2C%2023.557114228456914%2C%2023.587174348697395%2C%2023.617234468937873%2C%2023.647294589178358%2C%2023.677354709418836%2C%2023.70741482965932%2C%2023.7374749498998%2C%2023.76753507014028%2C%2023.79759519038076%2C%2023.827655310621243%2C%2023.857715430861724%2C%2023.887775551102205%2C%2023.917835671342687%2C%2023.947895791583164%2C%2023.97795591182365%2C%2024.008016032064127%2C%2024.038076152304612%2C%2024.06813627254509%2C%2024.09819639278557%2C%2024.128256513026052%2C%2024.158316633266534%2C%2024.188376753507015%2C%2024.218436873747493%2C%2024.248496993987978%2C%2024.278557114228455%2C%2024.30861723446894%2C%2024.338677354709418%2C%2024.3687374749499%2C%2024.39879759519038%2C%2024.428857715430862%2C%2024.458917835671343%2C%2024.48897795591182%2C%2024.519038076152306%2C%2024.549098196392784%2C%2024.57915831663327%2C%2024.609218436873746%2C%2024.639278557114228%2C%2024.66933867735471%2C%2024.69939879759519%2C%2024.72945891783567%2C%2024.759519038076153%2C%2024.789579158316634%2C%2024.819639278557112%2C%2024.849699398797597%2C%2024.879759519038075%2C%2024.90981963927856%2C%2024.939879759519037%2C%2024.96993987975952%2C%2025.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23ffffd4ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fee391ff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fec44fff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23fe9929ff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23d95f0eff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%2C%20%27%23993404ff%27%5D%29%3B%0A%20%20%20%20%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x%20%3D%20d3.scale.linear%28%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.domain%28%5B10.0%2C%2025.0%5D%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20.range%28%5B0%2C%20400%5D%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.legend%20%3D%20L.control%28%7Bposition%3A%20%27topright%27%7D%29%3B%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.legend.onAdd%20%3D%20function%20%28map%29%20%7Bvar%20div%20%3D%20L.DomUtil.create%28%27div%27%2C%20%27legend%27%29%3B%20return%20div%7D%3B%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.legend.addTo%28map_a6aaaf43e579464b91cdf5a9af4616bd%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.xAxis%20%3D%20d3.svg.axis%28%29%0A%20%20%20%20%20%20%20%20.scale%28color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x%29%0A%20%20%20%20%20%20%20%20.orient%28%22top%22%29%0A%20%20%20%20%20%20%20%20.tickSize%281%29%0A%20%20%20%20%20%20%20%20.tickValues%28%5B10.0%2C%2012.5%2C%2015.0%2C%2017.5%2C%2020.0%2C%2022.5%2C%2025.0%5D%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.svg%20%3D%20d3.select%28%22.legend.leaflet-control%22%29.append%28%22svg%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22id%22%2C%20%27legend%27%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20450%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2040%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.g%20%3D%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.svg.append%28%22g%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22key%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22transform%22%2C%20%22translate%2825%2C16%29%22%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.g.selectAll%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.data%28color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.color.range%28%29.map%28function%28d%2C%20i%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20return%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20x0%3A%20i%20%3F%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x%28color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.color.domain%28%29%5Bi%20-%201%5D%29%20%3A%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x.range%28%29%5B0%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x1%3A%20i%20%3C%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.color.domain%28%29.length%20%3F%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x%28color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.color.domain%28%29%5Bi%5D%29%20%3A%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.x.range%28%29%5B1%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20z%3A%20d%0A%20%20%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20%7D%29%29%0A%20%20%20%20%20%20.enter%28%29.append%28%22rect%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22height%22%2C%2010%29%0A%20%20%20%20%20%20%20%20.attr%28%22x%22%2C%20function%28d%29%20%7B%20return%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.attr%28%22width%22%2C%20function%28d%29%20%7B%20return%20d.x1%20-%20d.x0%3B%20%7D%29%0A%20%20%20%20%20%20%20%20.style%28%22fill%22%2C%20function%28d%29%20%7B%20return%20d.z%3B%20%7D%29%3B%0A%0A%20%20%20%20color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.g.call%28color_map_4b647da6784e4fb2ba5ff748a0dc4d8e.xAxis%29.append%28%22text%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22class%22%2C%20%22caption%22%29%0A%20%20%20%20%20%20%20%20.attr%28%22y%22%2C%2021%29%0A%20%20%20%20%20%20%20%20.text%28%27%27%29%3B%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20map_21d2760c69a444c08bed3d41c690fe16.sync%28map_a6aaaf43e579464b91cdf5a9af4616bd%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20map_a6aaaf43e579464b91cdf5a9af4616bd.sync%28map_21d2760c69a444c08bed3d41c690fe16%29%3B%0A%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



- **Yongsan has the highest distance and trip hours.**
- Distance and trip hour has a very strong correlation (0.95). This is not a surprise since a rider has to take a longer trip to go further.

## Which County Has High Outflow/Inflow of riders?

Outflow/Inflow represent trips of riders that traveled across from A to B county.
We can get the traffic traveling across counties this way.

Since the scale of the nunmber of trips differs by counties, we will look at a relative ratio between counties instead of absolute value.

- Outflow Ratio: Return at a different county after Renting at County A / Total number of Rents at County A
- Inflow Ratio: Return at County A after Renting at a different county / Total number of Returns at County A

In short, we want to know **how much % of Rents(Returns) at County A are Returned(Rented) at another county.**

### Mapping Outflow/Inflow Ratio of Counties

We can first check which counties have high outflow ratio or inflow ratio.


```python
df.columns
```




    Index(['start_time', 'start_station_id', 'start_station_name', 'end_time',
           'end_station_id', 'end_station_name', 'duration', 'distance',
           'start_hour', 'start_day', 'start_dayofweek', 'end_hour', 'county_x',
           'county_eng_x', 'lat_x', 'long_x', 'county_y', 'county_eng_y', 'lat_y',
           'long_y'],
          dtype='object')




```python
df['county_x'].unique()
```




    ['강북구', '도봉구', '성북구', '노원구', '동대문구', ..., '양천구', '금천구', '관악구', '강서구', '은평구']
    Length: 25
    Categories (25, object): ['강남구', '강동구', '강북구', '강서구', ..., '은평구', '종로구', '중구', '중랑구']




```python
df.groupby('county_x')
```




    <pandas.core.groupby.generic.DataFrameGroupBy object at 0x000001B6085F04F0>




```python
usetime_df_total = None
usetime_by_region = df.groupby('county_x')

for name, g_df in usetime_by_region:
    usetime = remove_outlier(g_df['duration']).reset_index(drop=True)
    usetime_df = pd.DataFrame(usetime)
    usetime_df['county'] = name
    
    if usetime_df_total is None:
        usetime_df_total = usetime_df
    else:
        usetime_df_total = usetime_df_total.append(usetime_df, ignore_index=True)

usetime_df_total.head()
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
      <th>duration</th>
      <th>county</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>88.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>1</th>
      <td>68.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>109.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>57.0</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>76.0</td>
      <td>강남구</td>
    </tr>
  </tbody>
</table>
</div>




```python
group_by_region = df.groupby('county_x')

outflow_by_region = {}
for name, g_df in group_by_region:
    total = len(g_df)
    print(total)
    print(name)
    outflow = len(g_df[g_df['county_y'] != name])
    print(outflow)
    outflow_by_region[name] = float(outflow) / total * 100

outflow_by_region
```

    18314
    강남구
    6584
    19807
    강동구
    3779
    11449
    강북구
    3775
    50747
    강서구
    5372
    18495
    관악구
    4933
    28413
    광진구
    7953
    22637
    구로구
    8792
    9132
    금천구
    2944
    30216
    노원구
    6779
    12587
    도봉구
    4701
    23260
    동대문구
    9512
    13967
    동작구
    6641
    34062
    마포구
    9988
    14960
    서대문구
    7178
    22306
    서초구
    6395
    30247
    성동구
    12603
    18731
    성북구
    8351
    39790
    송파구
    6973
    30137
    양천구
    7713
    47877
    영등포구
    12787
    12044
    용산구
    4720
    17475
    은평구
    3682
    22092
    종로구
    8415
    12789
    중구
    7366
    16873
    중랑구
    4693
    




    {'강남구': 35.95063885552037,
     '강동구': 19.079113444741758,
     '강북구': 32.97231199231374,
     '강서구': 10.585847439257492,
     '관악구': 26.6720735333874,
     '광진구': 27.990708478513355,
     '구로구': 38.83906878119892,
     '금천구': 32.23828296101621,
     '노원구': 22.43513370399788,
     '도봉구': 37.34805751966314,
     '동대문구': 40.894239036973346,
     '동작구': 47.54779122216654,
     '마포구': 29.322999236686044,
     '서대문구': 47.98128342245989,
     '서초구': 28.669416300546942,
     '성동구': 41.66694217608358,
     '성북구': 44.58384496289574,
     '송파구': 17.52450364413169,
     '양천구': 25.593124730397847,
     '영등포구': 26.708022641351796,
     '용산구': 39.18963799402192,
     '은평구': 21.070100143061516,
     '종로구': 38.090711569799026,
     '중구': 57.596371882086174,
     '중랑구': 27.81366680495466}




```python
df[df['county_x'] != df['county_y']]
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
      <th>24</th>
      <td>2021-02-05 02:58:00</td>
      <td>1721</td>
      <td>창동역 2번출구</td>
      <td>2021-02-05 03:15:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>17.0</td>
      <td>1984.84</td>
      <td>2</td>
      <td>5</td>
      <td>4</td>
      <td>3</td>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>37.653015</td>
      <td>127.046997</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2021-02-10 16:11:00</td>
      <td>1721</td>
      <td>창동역 2번출구</td>
      <td>2021-02-10 16:43:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>31.0</td>
      <td>2950.00</td>
      <td>16</td>
      <td>10</td>
      <td>2</td>
      <td>16</td>
      <td>도봉구</td>
      <td>Dobong</td>
      <td>37.653015</td>
      <td>127.046997</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2021-02-03 11:50:00</td>
      <td>1332</td>
      <td>석계역 5번출구 건너편</td>
      <td>2021-02-03 12:20:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>30.0</td>
      <td>4318.45</td>
      <td>11</td>
      <td>3</td>
      <td>2</td>
      <td>12</td>
      <td>성북구</td>
      <td>Seongbuk</td>
      <td>37.613556</td>
      <td>127.066093</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2021-02-21 03:31:00</td>
      <td>1332</td>
      <td>석계역 5번출구 건너편</td>
      <td>2021-02-21 03:53:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>22.0</td>
      <td>3922.40</td>
      <td>3</td>
      <td>21</td>
      <td>6</td>
      <td>3</td>
      <td>성북구</td>
      <td>Seongbuk</td>
      <td>37.613556</td>
      <td>127.066093</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>38</th>
      <td>2021-02-02 18:58:00</td>
      <td>1663</td>
      <td>동해문화예술관앞</td>
      <td>2021-02-02 19:13:00</td>
      <td>1554</td>
      <td>번동사거리</td>
      <td>15.0</td>
      <td>2927.33</td>
      <td>18</td>
      <td>2</td>
      <td>1</td>
      <td>19</td>
      <td>노원구</td>
      <td>Nowon</td>
      <td>37.619381</td>
      <td>127.057854</td>
      <td>강북구</td>
      <td>Gangbuk</td>
      <td>37.635391</td>
      <td>127.034554</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>578390</th>
      <td>2021-02-03 13:02:00</td>
      <td>2285</td>
      <td>LH서초3단지 301동 맞은편</td>
      <td>2021-02-03 13:40:00</td>
      <td>2391</td>
      <td>구룡터널 입구(개포1단지아파트)</td>
      <td>37.0</td>
      <td>6003.14</td>
      <td>13</td>
      <td>3</td>
      <td>2</td>
      <td>13</td>
      <td>서초구</td>
      <td>Seocho</td>
      <td>37.457424</td>
      <td>127.022652</td>
      <td>강남구</td>
      <td>Gangnam</td>
      <td>37.475986</td>
      <td>127.059624</td>
    </tr>
    <tr>
      <th>578391</th>
      <td>2021-02-04 18:52:00</td>
      <td>2285</td>
      <td>LH서초3단지 301동 맞은편</td>
      <td>2021-02-04 19:25:00</td>
      <td>2391</td>
      <td>구룡터널 입구(개포1단지아파트)</td>
      <td>33.0</td>
      <td>5391.86</td>
      <td>18</td>
      <td>4</td>
      <td>3</td>
      <td>19</td>
      <td>서초구</td>
      <td>Seocho</td>
      <td>37.457424</td>
      <td>127.022652</td>
      <td>강남구</td>
      <td>Gangnam</td>
      <td>37.475986</td>
      <td>127.059624</td>
    </tr>
    <tr>
      <th>578392</th>
      <td>2021-02-10 18:44:00</td>
      <td>2285</td>
      <td>LH서초3단지 301동 맞은편</td>
      <td>2021-02-10 19:18:00</td>
      <td>2391</td>
      <td>구룡터널 입구(개포1단지아파트)</td>
      <td>34.0</td>
      <td>0.00</td>
      <td>18</td>
      <td>10</td>
      <td>2</td>
      <td>19</td>
      <td>서초구</td>
      <td>Seocho</td>
      <td>37.457424</td>
      <td>127.022652</td>
      <td>강남구</td>
      <td>Gangnam</td>
      <td>37.475986</td>
      <td>127.059624</td>
    </tr>
    <tr>
      <th>578399</th>
      <td>2021-02-12 14:34:00</td>
      <td>2350</td>
      <td>래미안그레이튼102동앞</td>
      <td>2021-02-12 15:58:00</td>
      <td>2289</td>
      <td>남태령역 2번출구</td>
      <td>84.0</td>
      <td>13774.84</td>
      <td>14</td>
      <td>12</td>
      <td>4</td>
      <td>15</td>
      <td>강남구</td>
      <td>Gangnam</td>
      <td>37.494823</td>
      <td>127.047905</td>
      <td>서초구</td>
      <td>Seocho</td>
      <td>37.464298</td>
      <td>126.988525</td>
    </tr>
    <tr>
      <th>578406</th>
      <td>2021-02-23 15:31:00</td>
      <td>2387</td>
      <td>래미안강남힐즈 사거리</td>
      <td>2021-02-23 16:12:00</td>
      <td>2287</td>
      <td>능안마을입구</td>
      <td>40.0</td>
      <td>4276.79</td>
      <td>15</td>
      <td>23</td>
      <td>1</td>
      <td>16</td>
      <td>강남구</td>
      <td>Gangnam</td>
      <td>37.472454</td>
      <td>127.096077</td>
      <td>서초구</td>
      <td>Seocho</td>
      <td>37.455620</td>
      <td>127.067101</td>
    </tr>
  </tbody>
</table>
<p>172629 rows × 20 columns</p>
</div>




```python
group_by_region.mean()
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
      <th>duration</th>
      <th>distance</th>
      <th>start_hour</th>
      <th>start_day</th>
      <th>start_dayofweek</th>
      <th>end_hour</th>
    </tr>
    <tr>
      <th>county_x</th>
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
      <th>강남구</th>
      <td>36.971170</td>
      <td>4558.229829</td>
      <td>14.688544</td>
      <td>13.345965</td>
      <td>3.025117</td>
      <td>15.080485</td>
    </tr>
    <tr>
      <th>강동구</th>
      <td>27.967941</td>
      <td>2976.452305</td>
      <td>14.571768</td>
      <td>13.374716</td>
      <td>3.030191</td>
      <td>14.838239</td>
    </tr>
    <tr>
      <th>강북구</th>
      <td>29.449734</td>
      <td>3351.564486</td>
      <td>14.307625</td>
      <td>13.408071</td>
      <td>3.076775</td>
      <td>14.530701</td>
    </tr>
    <tr>
      <th>강서구</th>
      <td>21.469900</td>
      <td>2514.249914</td>
      <td>14.577394</td>
      <td>12.940174</td>
      <td>2.822393</td>
      <td>14.788382</td>
    </tr>
    <tr>
      <th>관악구</th>
      <td>28.710733</td>
      <td>3647.366984</td>
      <td>14.418059</td>
      <td>13.329386</td>
      <td>3.079697</td>
      <td>14.660395</td>
    </tr>
    <tr>
      <th>광진구</th>
      <td>28.433886</td>
      <td>3276.029701</td>
      <td>14.651427</td>
      <td>13.398374</td>
      <td>3.105691</td>
      <td>14.926337</td>
    </tr>
    <tr>
      <th>구로구</th>
      <td>30.067854</td>
      <td>3839.468746</td>
      <td>14.537704</td>
      <td>13.077749</td>
      <td>2.966029</td>
      <td>14.870478</td>
    </tr>
    <tr>
      <th>금천구</th>
      <td>31.617827</td>
      <td>4164.356751</td>
      <td>14.484012</td>
      <td>12.723062</td>
      <td>2.789093</td>
      <td>14.790955</td>
    </tr>
    <tr>
      <th>노원구</th>
      <td>28.154819</td>
      <td>3357.213992</td>
      <td>14.723756</td>
      <td>13.270453</td>
      <td>2.992487</td>
      <td>14.985339</td>
    </tr>
    <tr>
      <th>도봉구</th>
      <td>29.755383</td>
      <td>3722.193881</td>
      <td>14.403829</td>
      <td>13.380472</td>
      <td>3.148169</td>
      <td>14.681179</td>
    </tr>
    <tr>
      <th>동대문구</th>
      <td>26.932029</td>
      <td>3137.036532</td>
      <td>14.621496</td>
      <td>13.063972</td>
      <td>2.922743</td>
      <td>14.888779</td>
    </tr>
    <tr>
      <th>동작구</th>
      <td>32.553304</td>
      <td>3936.880182</td>
      <td>14.298919</td>
      <td>13.586597</td>
      <td>3.147849</td>
      <td>14.612802</td>
    </tr>
    <tr>
      <th>마포구</th>
      <td>34.384241</td>
      <td>3837.129528</td>
      <td>14.845282</td>
      <td>13.558394</td>
      <td>3.192795</td>
      <td>15.210675</td>
    </tr>
    <tr>
      <th>서대문구</th>
      <td>31.111832</td>
      <td>3749.126858</td>
      <td>14.268249</td>
      <td>13.432487</td>
      <td>3.088235</td>
      <td>14.600201</td>
    </tr>
    <tr>
      <th>서초구</th>
      <td>34.941675</td>
      <td>4231.926488</td>
      <td>14.682283</td>
      <td>13.440823</td>
      <td>3.164709</td>
      <td>15.112795</td>
    </tr>
    <tr>
      <th>성동구</th>
      <td>34.243694</td>
      <td>4126.396393</td>
      <td>14.871425</td>
      <td>13.587728</td>
      <td>3.242669</td>
      <td>15.217807</td>
    </tr>
    <tr>
      <th>성북구</th>
      <td>28.004431</td>
      <td>3346.717005</td>
      <td>14.530938</td>
      <td>13.376542</td>
      <td>3.116705</td>
      <td>14.756927</td>
    </tr>
    <tr>
      <th>송파구</th>
      <td>29.497034</td>
      <td>3278.637469</td>
      <td>14.643503</td>
      <td>13.342247</td>
      <td>3.119628</td>
      <td>14.999246</td>
    </tr>
    <tr>
      <th>양천구</th>
      <td>25.785546</td>
      <td>2941.590943</td>
      <td>14.689385</td>
      <td>13.023426</td>
      <td>2.939941</td>
      <td>14.987656</td>
    </tr>
    <tr>
      <th>영등포구</th>
      <td>31.290870</td>
      <td>3558.056686</td>
      <td>14.649644</td>
      <td>13.361844</td>
      <td>3.038160</td>
      <td>14.973348</td>
    </tr>
    <tr>
      <th>용산구</th>
      <td>36.192710</td>
      <td>4501.900494</td>
      <td>14.783876</td>
      <td>13.833112</td>
      <td>3.415892</td>
      <td>15.194371</td>
    </tr>
    <tr>
      <th>은평구</th>
      <td>29.961431</td>
      <td>3442.589195</td>
      <td>14.337454</td>
      <td>13.265694</td>
      <td>3.063348</td>
      <td>14.589185</td>
    </tr>
    <tr>
      <th>종로구</th>
      <td>22.926037</td>
      <td>2342.861022</td>
      <td>14.581613</td>
      <td>12.706681</td>
      <td>2.740585</td>
      <td>14.838810</td>
    </tr>
    <tr>
      <th>중구</th>
      <td>25.258582</td>
      <td>2704.253222</td>
      <td>14.422472</td>
      <td>12.733599</td>
      <td>2.682149</td>
      <td>14.708265</td>
    </tr>
    <tr>
      <th>중랑구</th>
      <td>27.385646</td>
      <td>3307.660766</td>
      <td>14.620222</td>
      <td>13.369229</td>
      <td>3.063415</td>
      <td>14.879215</td>
    </tr>
  </tbody>
</table>
</div>




```python
group_by_region = df.groupby('county_x')

outflow_by_region = {}
for name, g_df in group_by_region:
    total = len(g_df)
    outflow = len(g_df[g_df['county_y'] != name])
    
    outflow_by_region[name] = float(outflow) / total * 100
    
outflow_by_region = pd.Series(data = outflow_by_region).sort_values(ascending=False)
outflow_by_region.index.name = 'county'
outflow_by_region.name = 'Outflow_Ratio'
```


```python
group_by_region = df.groupby('county_y')

inflow_by_region = {}
for name, g_df in group_by_region:
    total = len(g_df)
    inflow = len(g_df[g_df['county_x'] != name])
    
    inflow_by_region[name] = float(inflow) / total * 100
    
inflow_by_region = pd.Series(data = inflow_by_region).sort_values(ascending=False)
inflow_by_region.index.name = 'county'
inflow_by_region.name = 'Inflow_Ratio'
```


```python
bike_map = folium.plugins.DualMap(location=[37.541, 126.986], zoom_start=10, tiles='cartodbpositron', zoom_control=False)
folium.Choropleth(geo_data=geo_str,
                  data=outflow_by_region,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='Reds',
                  line_color='grey').add_to(bike_map.m1)
folium.Choropleth(geo_data=geo_str,
                  data=inflow_by_region,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='Greens',
                  line_color='grey').add_to(bike_map.m2)
bike_map
```







- Jung (located in the center) has the highest outflow and inflow ratio. Counties in the center of Seoul have both high outflow/inflow ratio.
- **Outflow ratio and inflow ratio have simliar distributions.** In other word, a county with high outflow ratio has high inflow ratio (correlation = 0.97)

## Add Time

Let's throw time into the analysis and dig in for more insights


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
group_by_region_weekday = df[df['start_dayofweek'].isin(set(range(1, 5)))].groupby('county_eng_x')

outflow_by_region_weekday = {}
for name, g_df in group_by_region_weekday:    
    total = g_df.groupby('start_hour').size() 
    g_df = g_df[g_df['county_eng_y'] != name]
    outflow = g_df.groupby('start_hour').size() / total * 100
    outflow_by_region_weekday[name] = outflow

outflow_by_region_weekday = pd.DataFrame(data = outflow_by_region_weekday)
```


```python
plt.figure(figsize=(24,13))
sns.heatmap(outflow_by_region_weekday.T, square=False, annot=True, fmt=".1f", cmap='Reds', cbar=False, vmin=0, vmax=100, linewidth=1)
plt.tight_layout()
plt.title("Hourly Outflow Ratio by County (%, Weekdays)")
plt.show()
```


    
![png](output_112_0.png)
    



```python
group_by_region_weekday = df[df['start_dayofweek'].isin(set(range(1, 5)))].groupby('county_eng_y')

inflow_by_region_weekday = {}
for name, g_df in group_by_region_weekday:    
    total = g_df.groupby('end_hour').size() 
    g_df = g_df[g_df['county_eng_x'] != name]
    inflow = g_df.groupby('end_hour').size() / total * 100
    inflow_by_region_weekday[name] = inflow

inflow_by_region_weekday = pd.DataFrame(data = inflow_by_region_weekday)
```


```python
plt.figure(figsize=(24,13))
sns.heatmap(inflow_by_region_weekday.T, square=False, annot=True, fmt=".1f", cmap='Greens', cbar=False, vmin=0, vmax=100, linewidth=1)
plt.title("Hourly Outflow Ratio by County (%, Weekdays)")
plt.tight_layout()
plt.show()
```


    
![png](output_114_0.png)
    


The outflow/inflow ratio have a little bit different patterns!
This finding triggers a question:

- **Are there counties that have simliar outflow/inflow ratio depending on time?**

Since we have to look at outflow/inflow ratio at the same time period, let's take **Measure = Inflow Ratio - Outflow Ratio**
If Measure > 0, more riders are coming to the county in that time period.
If Measure < 0, more riders are going out of the county in that time period.

Let's cluster counties with similar patterns in Measure by time period.


```python
inout_ratio = inflow_by_region_weekday- outflow_by_region_weekday
```


```python
plt.figure(figsize=(24, 24))
clustergrid = sns.clustermap(inout_ratio.corr(), cbar=True, vmin=-1, vmax=1, cmap='Blues')
plt.show()
```


    <Figure size 1728x1728 with 0 Axes>



    
![png](output_117_1.png)
    


Do you see the 3 clusters!?
I've named the clusters as A, B, C.

Now, let's plot the Measure by time.


```python
reordered_ind = clustergrid.dendrogram_row.reordered_ind
```


```python
inout_ratio = inout_ratio.T.reset_index().reindex(reordered_ind).set_index('index')
inout_ratio.index.name = 'county'
```


```python
inout_ratio
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
      <th>end_hour</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>14</th>
      <th>15</th>
      <th>16</th>
      <th>17</th>
      <th>18</th>
      <th>19</th>
      <th>20</th>
      <th>21</th>
      <th>22</th>
      <th>23</th>
    </tr>
    <tr>
      <th>county</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>Yongsan</th>
      <td>-0.597372</td>
      <td>8.914982</td>
      <td>15.898618</td>
      <td>19.187675</td>
      <td>7.076923</td>
      <td>-9.017713</td>
      <td>1.915709</td>
      <td>-3.043568</td>
      <td>-1.004252</td>
      <td>3.136908</td>
      <td>...</td>
      <td>5.596961</td>
      <td>2.214251</td>
      <td>0.311626</td>
      <td>-3.193014</td>
      <td>0.955466</td>
      <td>-0.050251</td>
      <td>7.839630</td>
      <td>-2.505828</td>
      <td>9.789280</td>
      <td>1.831502</td>
    </tr>
    <tr>
      <th>Yangcheon</th>
      <td>-0.416384</td>
      <td>4.361395</td>
      <td>0.132769</td>
      <td>-15.283401</td>
      <td>17.896866</td>
      <td>-5.879630</td>
      <td>-3.597776</td>
      <td>-11.869235</td>
      <td>-1.540075</td>
      <td>-1.344923</td>
      <td>...</td>
      <td>2.065783</td>
      <td>0.050973</td>
      <td>-4.122525</td>
      <td>2.897974</td>
      <td>6.122141</td>
      <td>8.356374</td>
      <td>0.576783</td>
      <td>0.710457</td>
      <td>1.899237</td>
      <td>1.972468</td>
    </tr>
    <tr>
      <th>Gangdong</th>
      <td>0.397255</td>
      <td>15.804848</td>
      <td>5.506073</td>
      <td>-12.581454</td>
      <td>-12.051126</td>
      <td>-7.615481</td>
      <td>-1.191072</td>
      <td>-0.039303</td>
      <td>-7.087629</td>
      <td>-11.182470</td>
      <td>...</td>
      <td>2.927805</td>
      <td>1.475683</td>
      <td>2.146255</td>
      <td>2.101343</td>
      <td>8.534957</td>
      <td>7.792132</td>
      <td>5.527270</td>
      <td>3.094648</td>
      <td>10.708617</td>
      <td>2.571429</td>
    </tr>
    <tr>
      <th>Gwangjin</th>
      <td>-0.876549</td>
      <td>-8.638065</td>
      <td>-0.175694</td>
      <td>-20.021186</td>
      <td>-9.870130</td>
      <td>-7.894737</td>
      <td>-9.585666</td>
      <td>-7.159064</td>
      <td>-20.622872</td>
      <td>-9.013087</td>
      <td>...</td>
      <td>-0.408376</td>
      <td>2.499825</td>
      <td>5.447918</td>
      <td>6.322908</td>
      <td>11.011974</td>
      <td>7.229154</td>
      <td>5.520138</td>
      <td>2.681782</td>
      <td>2.596102</td>
      <td>6.520778</td>
    </tr>
    <tr>
      <th>Dongdaemun</th>
      <td>-0.250516</td>
      <td>14.544002</td>
      <td>13.846154</td>
      <td>12.804142</td>
      <td>4.591105</td>
      <td>8.673267</td>
      <td>-6.683375</td>
      <td>-5.657233</td>
      <td>-0.520051</td>
      <td>-7.663291</td>
      <td>...</td>
      <td>-0.908656</td>
      <td>-2.898681</td>
      <td>-1.391143</td>
      <td>-1.604091</td>
      <td>1.649894</td>
      <td>6.605710</td>
      <td>-3.736619</td>
      <td>5.784929</td>
      <td>7.173059</td>
      <td>9.206731</td>
    </tr>
    <tr>
      <th>Gangseo</th>
      <td>2.731643</td>
      <td>4.107063</td>
      <td>2.618585</td>
      <td>3.952163</td>
      <td>1.686508</td>
      <td>-9.900138</td>
      <td>-4.371073</td>
      <td>-1.829930</td>
      <td>-0.248987</td>
      <td>-0.337645</td>
      <td>...</td>
      <td>-3.952672</td>
      <td>-1.962319</td>
      <td>0.475628</td>
      <td>-0.168983</td>
      <td>2.937067</td>
      <td>3.570192</td>
      <td>0.388716</td>
      <td>1.353357</td>
      <td>5.479093</td>
      <td>2.940515</td>
    </tr>
    <tr>
      <th>Dobong</th>
      <td>6.808592</td>
      <td>-9.467357</td>
      <td>-0.398804</td>
      <td>-9.677419</td>
      <td>13.903743</td>
      <td>-2.052786</td>
      <td>-11.997100</td>
      <td>-17.428883</td>
      <td>-4.327443</td>
      <td>-11.334931</td>
      <td>...</td>
      <td>-4.525421</td>
      <td>-2.126924</td>
      <td>-3.093318</td>
      <td>-2.972883</td>
      <td>3.012092</td>
      <td>11.287129</td>
      <td>11.398517</td>
      <td>4.040856</td>
      <td>9.139001</td>
      <td>2.912753</td>
    </tr>
    <tr>
      <th>Eunpyeong</th>
      <td>2.845127</td>
      <td>-3.018217</td>
      <td>13.677195</td>
      <td>1.778075</td>
      <td>18.687231</td>
      <td>-12.698413</td>
      <td>-14.626219</td>
      <td>-14.046157</td>
      <td>-17.946708</td>
      <td>-3.129151</td>
      <td>...</td>
      <td>-4.439171</td>
      <td>-2.630691</td>
      <td>-1.990442</td>
      <td>9.117192</td>
      <td>11.600919</td>
      <td>11.389064</td>
      <td>6.137961</td>
      <td>0.990624</td>
      <td>13.185634</td>
      <td>11.625387</td>
    </tr>
    <tr>
      <th>Dongjak</th>
      <td>7.029801</td>
      <td>2.324619</td>
      <td>-4.805586</td>
      <td>-6.031746</td>
      <td>-18.085106</td>
      <td>-4.019608</td>
      <td>-37.721350</td>
      <td>-25.205634</td>
      <td>-2.768439</td>
      <td>-5.395928</td>
      <td>...</td>
      <td>-7.739555</td>
      <td>-10.241758</td>
      <td>1.896445</td>
      <td>1.742989</td>
      <td>11.742451</td>
      <td>12.209644</td>
      <td>0.723387</td>
      <td>6.548039</td>
      <td>9.689696</td>
      <td>3.526889</td>
    </tr>
    <tr>
      <th>Jungnang</th>
      <td>-0.329803</td>
      <td>0.493620</td>
      <td>0.019885</td>
      <td>6.352087</td>
      <td>-1.224490</td>
      <td>8.314176</td>
      <td>-10.399791</td>
      <td>-16.659811</td>
      <td>-17.626525</td>
      <td>-10.652583</td>
      <td>...</td>
      <td>-5.304605</td>
      <td>1.932966</td>
      <td>-1.209232</td>
      <td>2.567552</td>
      <td>7.630090</td>
      <td>11.093365</td>
      <td>0.901632</td>
      <td>8.732155</td>
      <td>6.834773</td>
      <td>3.362142</td>
    </tr>
    <tr>
      <th>Seongbuk</th>
      <td>4.098108</td>
      <td>-0.560897</td>
      <td>2.934860</td>
      <td>5.314010</td>
      <td>7.459207</td>
      <td>1.111111</td>
      <td>-37.405611</td>
      <td>-26.368972</td>
      <td>-18.275662</td>
      <td>-6.406051</td>
      <td>...</td>
      <td>-12.567909</td>
      <td>-1.884575</td>
      <td>2.617378</td>
      <td>4.742868</td>
      <td>13.788263</td>
      <td>9.423582</td>
      <td>5.823885</td>
      <td>4.277100</td>
      <td>5.512821</td>
      <td>3.129240</td>
    </tr>
    <tr>
      <th>Gwanak</th>
      <td>-0.500850</td>
      <td>6.303419</td>
      <td>2.311436</td>
      <td>2.956989</td>
      <td>-4.055944</td>
      <td>-8.421424</td>
      <td>-8.446344</td>
      <td>-13.861386</td>
      <td>-20.857385</td>
      <td>-10.883399</td>
      <td>...</td>
      <td>-5.733053</td>
      <td>-2.857282</td>
      <td>-4.659698</td>
      <td>3.080020</td>
      <td>14.266019</td>
      <td>4.554859</td>
      <td>-1.395435</td>
      <td>-0.624091</td>
      <td>4.223853</td>
      <td>2.447637</td>
    </tr>
    <tr>
      <th>Seodaemun</th>
      <td>5.822499</td>
      <td>2.535547</td>
      <td>9.051517</td>
      <td>1.286262</td>
      <td>-3.347578</td>
      <td>-4.963855</td>
      <td>-18.437398</td>
      <td>-27.418463</td>
      <td>-18.333333</td>
      <td>-16.146427</td>
      <td>...</td>
      <td>-1.112154</td>
      <td>-3.693189</td>
      <td>0.158257</td>
      <td>-4.795509</td>
      <td>3.834982</td>
      <td>8.124004</td>
      <td>9.652943</td>
      <td>1.392537</td>
      <td>8.492400</td>
      <td>6.586105</td>
    </tr>
    <tr>
      <th>Jung</th>
      <td>-4.363873</td>
      <td>-8.214286</td>
      <td>-26.628151</td>
      <td>-16.599190</td>
      <td>-22.932331</td>
      <td>-13.712375</td>
      <td>8.163265</td>
      <td>22.044171</td>
      <td>11.071097</td>
      <td>10.035473</td>
      <td>...</td>
      <td>-3.899953</td>
      <td>-3.757623</td>
      <td>-1.904532</td>
      <td>-12.849046</td>
      <td>-7.855934</td>
      <td>-4.750879</td>
      <td>-10.663614</td>
      <td>-8.156924</td>
      <td>-11.849879</td>
      <td>-3.447165</td>
    </tr>
    <tr>
      <th>Gangnam</th>
      <td>-3.516857</td>
      <td>0.310078</td>
      <td>5.271739</td>
      <td>1.346389</td>
      <td>0.000000</td>
      <td>-1.602564</td>
      <td>-14.561404</td>
      <td>17.919255</td>
      <td>8.766917</td>
      <td>7.071114</td>
      <td>...</td>
      <td>0.936121</td>
      <td>-2.357163</td>
      <td>-3.269493</td>
      <td>-9.987472</td>
      <td>-8.331237</td>
      <td>-0.837348</td>
      <td>-7.404478</td>
      <td>-7.026633</td>
      <td>-7.009275</td>
      <td>-6.347222</td>
    </tr>
    <tr>
      <th>Yeongdeungpo</th>
      <td>1.721378</td>
      <td>-2.137956</td>
      <td>-5.343137</td>
      <td>5.160911</td>
      <td>-4.230377</td>
      <td>-5.952381</td>
      <td>-0.703332</td>
      <td>2.267947</td>
      <td>8.731050</td>
      <td>5.573803</td>
      <td>...</td>
      <td>0.325064</td>
      <td>0.835046</td>
      <td>0.450481</td>
      <td>-6.486038</td>
      <td>-10.063096</td>
      <td>3.200831</td>
      <td>-0.087763</td>
      <td>-4.100848</td>
      <td>1.873574</td>
      <td>-2.594204</td>
    </tr>
    <tr>
      <th>Seocho</th>
      <td>-10.326638</td>
      <td>0.187169</td>
      <td>-0.069360</td>
      <td>-2.068966</td>
      <td>16.319444</td>
      <td>-6.595745</td>
      <td>18.063872</td>
      <td>-4.660436</td>
      <td>10.023930</td>
      <td>8.870912</td>
      <td>...</td>
      <td>3.095864</td>
      <td>-1.367067</td>
      <td>-0.663163</td>
      <td>-2.423254</td>
      <td>-5.202375</td>
      <td>-1.575143</td>
      <td>1.006162</td>
      <td>-3.677833</td>
      <td>-2.951836</td>
      <td>1.115619</td>
    </tr>
    <tr>
      <th>Jongno</th>
      <td>-2.132505</td>
      <td>-7.476636</td>
      <td>8.399646</td>
      <td>-1.406926</td>
      <td>-1.960784</td>
      <td>15.051153</td>
      <td>12.577788</td>
      <td>2.987547</td>
      <td>12.179567</td>
      <td>7.300000</td>
      <td>...</td>
      <td>3.258044</td>
      <td>-3.254002</td>
      <td>-8.440345</td>
      <td>-11.427880</td>
      <td>-12.494714</td>
      <td>-6.787532</td>
      <td>-5.688016</td>
      <td>-9.311344</td>
      <td>-1.773077</td>
      <td>-9.090121</td>
    </tr>
    <tr>
      <th>Mapo</th>
      <td>-8.501848</td>
      <td>5.874587</td>
      <td>-7.759662</td>
      <td>7.948615</td>
      <td>11.709770</td>
      <td>15.942761</td>
      <td>8.681653</td>
      <td>10.518915</td>
      <td>15.019355</td>
      <td>6.745534</td>
      <td>...</td>
      <td>5.121193</td>
      <td>0.270380</td>
      <td>1.820207</td>
      <td>-4.077117</td>
      <td>-5.399222</td>
      <td>-0.353914</td>
      <td>-5.524302</td>
      <td>-6.900946</td>
      <td>-8.474644</td>
      <td>-1.882733</td>
    </tr>
    <tr>
      <th>Seongdong</th>
      <td>-11.400694</td>
      <td>3.373957</td>
      <td>-7.616812</td>
      <td>3.835616</td>
      <td>12.519272</td>
      <td>9.479055</td>
      <td>17.784195</td>
      <td>6.164420</td>
      <td>22.533866</td>
      <td>14.086676</td>
      <td>...</td>
      <td>5.465221</td>
      <td>5.140669</td>
      <td>-3.362227</td>
      <td>-6.355271</td>
      <td>-10.768087</td>
      <td>-0.301080</td>
      <td>0.072451</td>
      <td>-3.508913</td>
      <td>-9.042402</td>
      <td>-1.829986</td>
    </tr>
    <tr>
      <th>Guro</th>
      <td>-12.142577</td>
      <td>0.361562</td>
      <td>4.705882</td>
      <td>-0.539084</td>
      <td>-6.730463</td>
      <td>23.161612</td>
      <td>13.448276</td>
      <td>3.533678</td>
      <td>4.502577</td>
      <td>10.638749</td>
      <td>...</td>
      <td>-4.519450</td>
      <td>-5.375849</td>
      <td>0.136191</td>
      <td>-3.880648</td>
      <td>1.532578</td>
      <td>2.072665</td>
      <td>2.759197</td>
      <td>-0.438969</td>
      <td>3.313650</td>
      <td>-6.488169</td>
    </tr>
    <tr>
      <th>Nowon</th>
      <td>-4.666363</td>
      <td>1.058303</td>
      <td>8.213494</td>
      <td>-7.346789</td>
      <td>-16.208791</td>
      <td>14.598109</td>
      <td>1.480398</td>
      <td>-6.056139</td>
      <td>-2.920717</td>
      <td>7.442484</td>
      <td>...</td>
      <td>-1.176044</td>
      <td>2.190377</td>
      <td>-0.801996</td>
      <td>-0.621023</td>
      <td>4.260685</td>
      <td>1.153697</td>
      <td>-3.095335</td>
      <td>-3.684488</td>
      <td>-1.720064</td>
      <td>-4.080552</td>
    </tr>
    <tr>
      <th>Gangbuk</th>
      <td>0.936508</td>
      <td>5.323450</td>
      <td>2.370600</td>
      <td>22.765146</td>
      <td>6.911765</td>
      <td>22.222222</td>
      <td>-14.945339</td>
      <td>-0.580778</td>
      <td>11.529680</td>
      <td>-2.298851</td>
      <td>...</td>
      <td>-0.444764</td>
      <td>-4.150239</td>
      <td>-1.663567</td>
      <td>5.078125</td>
      <td>-4.759164</td>
      <td>6.571276</td>
      <td>-7.938316</td>
      <td>2.102426</td>
      <td>-7.342954</td>
      <td>-6.245193</td>
    </tr>
    <tr>
      <th>Geumcheon</th>
      <td>-0.082102</td>
      <td>4.065041</td>
      <td>0.877193</td>
      <td>-9.742647</td>
      <td>28.260870</td>
      <td>24.307692</td>
      <td>-13.914481</td>
      <td>3.529412</td>
      <td>17.980753</td>
      <td>2.570817</td>
      <td>...</td>
      <td>-12.754525</td>
      <td>-3.111472</td>
      <td>-2.069475</td>
      <td>-7.481859</td>
      <td>-13.244780</td>
      <td>3.305449</td>
      <td>1.429233</td>
      <td>4.844961</td>
      <td>5.973630</td>
      <td>6.167979</td>
    </tr>
    <tr>
      <th>Songpa</th>
      <td>-4.956660</td>
      <td>-18.019097</td>
      <td>3.161470</td>
      <td>6.111736</td>
      <td>7.583732</td>
      <td>14.213483</td>
      <td>-4.074645</td>
      <td>-8.411147</td>
      <td>3.612640</td>
      <td>6.692261</td>
      <td>...</td>
      <td>0.853108</td>
      <td>-3.210581</td>
      <td>2.510814</td>
      <td>0.026550</td>
      <td>-0.247471</td>
      <td>0.677801</td>
      <td>1.866905</td>
      <td>-1.673977</td>
      <td>0.369875</td>
      <td>3.491144</td>
    </tr>
  </tbody>
</table>
<p>25 rows × 24 columns</p>
</div>




```python
plt.figure(figsize=(24,13))
sns.heatmap(inout_ratio, square=False, annot=True, fmt=".1f", cmap='RdYlGn', cbar=False, vmin=-32, vmax=32)
plt.title("Hourly Inflow-Outflow Ratio by County (%, Weekdays)")
plt.ylabel("")
plt.tight_layout()
plt.show()
```


    
![png](output_122_0.png)
    


**Red = Outflow > Inflow**
**Green = Outflow < Inflow**

The clusters make sense!

**Cluster A** represents counties with high inflow in morning commute time and high outflow in evening commute time. These counties tend to have relatively more companies than other counties.

**Cluster B** is completely opposite of Cluster A. These counties are well known for residence.

**Cluster C** does not belong to either Cluster A or B. Patterns are not as clear as other clusters.

Some abnormalies(?) are that:

- Jung has significantly high outflow ratio from 2 - 4 am
- Gangbuk has very high inflow ratio at 3 am

One thing that crosses my mind is that these abnomalies may be due to distribution of bikes to counties in other stations by trucks.

## In which County do Seoul-ers Commute?

Let's visualize the heatmap 


```python
group_by_region_weekday = df[df['start_dayofweek'].isin(set(range(1, 5)))].groupby('county_x')

outflow_by_region_weekday = {}
for name, g_df in group_by_region_weekday:    
    total = g_df.groupby('start_hour').size() 
    g_df = g_df[g_df['county_y'] != name]
    outflow = g_df.groupby('start_hour').size() / total * 100
    outflow_by_region_weekday[name] = outflow

outflow_by_region_weekday = pd.DataFrame(data = outflow_by_region_weekday)

group_by_region_weekday = df[df['start_dayofweek'].isin(set(range(1, 5)))].groupby('county_y')

inflow_by_region_weekday = {}
for name, g_df in group_by_region_weekday:    
    total = g_df.groupby('end_hour').size() 
    g_df = g_df[g_df['county_x'] != name]
    inflow = g_df.groupby('end_hour').size() / total * 100
    inflow_by_region_weekday[name] = inflow

inflow_by_region_weekday = pd.DataFrame(data = inflow_by_region_weekday)

inout_ratio = inflow_by_region_weekday- outflow_by_region_weekday

reordered_ind = clustergrid.dendrogram_row.reordered_ind

inout_ratio = inout_ratio.T.reset_index().reindex(reordered_ind).set_index('index')
inout_ratio.index.name = 'county'
```


```python
mean_inout_morning = inout_ratio.iloc[:, 7:10].mean(axis=1)
mean_inout_night = inout_ratio.iloc[:, 17:20].mean(axis=1)

bike_map = bike_map = folium.plugins.DualMap(location=[37.541, 126.986], zoom_start=10, tiles='cartodbpositron', zoom_control=False)
folium.Choropleth(geo_data=geo_str,
                  data=mean_inout_morning,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='RdYlGn',
                  line_color='grey').add_to(bike_map.m1)
folium.Choropleth(geo_data=geo_str,
                  data=mean_inout_night,
                  key_on='feature.properties.SIG_KOR_NM', 
                  fill_color='RdYlGn',
                  line_color='grey').add_to(bike_map.m2)
bike_map
```








```python

```