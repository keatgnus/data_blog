---
title: "Data_Mining_Kaggle"
author: "Seongtaek"
date: "2023-05-02"
categories: [code, data_mining, jupyter, kaggle]
image: "earthquake.png"
toc: true
---

Exercise 3 - Interactive Maps

<table align="bottom">
  <td>
    <a target="_blank" href="http://localhost:8888/notebooks/3-1%20Data_Mining/exercise3-interactive-maps.ipynb#"><img
    src = small_jup_image.jpg />Jupyter에서 실행하기</a>
  </td>
</table>

**This notebook is an exercise in the [Geospatial Analysis](https://www.kaggle.com/learn/geospatial-analysis) course.  You can reference the tutorial at [this link](https://www.kaggle.com/alexisbcook/interactive-maps).**

---


## Introduction

You are an urban safety planner in Japan, and you are analyzing which areas of Japan need extra earthquake reinforcement.  Which areas are both high in population density and prone to earthquakes?

<center>
<img src="https://storage.googleapis.com/kaggle-media/learn/images/Kuh9gPj.png" width="450"><br/>
</center>

Before you get started, run the code cell below to set everything up.

- 당신은 일본의 도시 안전 계획자이고, 당신은 일본의 어느 지역에서 추가적인 지진 보강이 필요한지 분석하고 있습니다. 인구 밀도가 높고 지진이 일어나기 쉬운 지역은 어디입니까?


```python
import pandas as pd
import geopandas as gpd

#!pip install folium
import folium
from folium import Choropleth
from folium.plugins import HeatMap
```

- 대화형 맵을 표시하기 위한 함수 embed_map()을 정의합니다. 맵을 포함하는 변수와 맵이 저장될 HTML 파일의 이름이라는 두 가지 인수를 사용할 수 있습니다.

- 이 기능을 사용하면 모든 웹 브라우저에서 지도를 볼 수 있습니다. [지도 보기](https://github.com/python-visualization/folium/issues/812).


```python
def embed_map(m, file_name):
    from IPython.display import IFrame
    m.save(file_name)
    return IFrame(file_name, width='100%', height='500px')
```

## Exercises

### 지진은 판의 경계와 일치합니까?

- 아래 코드 셀을 실행하여 전역 플레이트 경계를 표시하는 DataFrame plate_boundaries를 만듭니다. 좌표 열은 경계를 따라 위치(위도, 경도)의 목록입니다


```python
plate_boundaries = gpd.read_file("C:/Users\seong taek/Desktop/archive/Plate_Boundaries/Plate_Boundaries/Plate_Boundaries.shp")
plate_boundaries['coordinates'] = plate_boundaries.apply(lambda x: [(b,a) for (a,b) in list(x.geometry.coords)], axis='columns')
plate_boundaries.drop('geometry', axis=1, inplace=True)

plate_boundaries.head()
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
      <th>HAZ_PLATES</th>
      <th>HAZ_PLAT_1</th>
      <th>HAZ_PLAT_2</th>
      <th>Shape_Leng</th>
      <th>coordinates</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TRENCH</td>
      <td>SERAM TROUGH (ACTIVE)</td>
      <td>6722</td>
      <td>5.843467</td>
      <td>[(-5.444200361999947, 133.6808931800001), (-5....</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TRENCH</td>
      <td>WETAR THRUST</td>
      <td>6722</td>
      <td>1.829013</td>
      <td>[(-7.760600482999962, 125.47879802900002), (-7...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>TRENCH</td>
      <td>TRENCH WEST OF LUZON (MANILA TRENCH) NORTHERN ...</td>
      <td>6621</td>
      <td>6.743604</td>
      <td>[(19.817899819000047, 120.09999798800004), (19...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>TRENCH</td>
      <td>BONIN TRENCH</td>
      <td>9821</td>
      <td>8.329381</td>
      <td>[(26.175899215000072, 143.20620700100005), (26...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TRENCH</td>
      <td>NEW GUINEA TRENCH</td>
      <td>8001</td>
      <td>11.998145</td>
      <td>[(0.41880004000006466, 132.8273013480001), (0....</td>
    </tr>
  </tbody>
</table>
</div>



그런 다음 변경 없이 아래 코드 셀을 실행하여 과거 지진 데이터를 DataFrame earthquakes에 로드합니다


```python
# Load the data and print the first 5 rows
earthquakes = pd.read_csv("C:/Users\seong taek/Desktop/archive/earthquakes1970-2014.csv", parse_dates=["DateTime"])
earthquakes.head()
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
      <th>DateTime</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Depth</th>
      <th>Magnitude</th>
      <th>MagType</th>
      <th>NbStations</th>
      <th>Gap</th>
      <th>Distance</th>
      <th>RMS</th>
      <th>Source</th>
      <th>EventID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1970-01-04 17:00:40.200</td>
      <td>24.139</td>
      <td>102.503</td>
      <td>31.0</td>
      <td>7.5</td>
      <td>Ms</td>
      <td>90.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NEI</td>
      <td>1.970010e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1970-01-06 05:35:51.800</td>
      <td>-9.628</td>
      <td>151.458</td>
      <td>8.0</td>
      <td>6.2</td>
      <td>Ms</td>
      <td>85.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NEI</td>
      <td>1.970011e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1970-01-08 17:12:39.100</td>
      <td>-34.741</td>
      <td>178.568</td>
      <td>179.0</td>
      <td>6.1</td>
      <td>Mb</td>
      <td>59.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NEI</td>
      <td>1.970011e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1970-01-10 12:07:08.600</td>
      <td>6.825</td>
      <td>126.737</td>
      <td>73.0</td>
      <td>6.1</td>
      <td>Mb</td>
      <td>91.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NEI</td>
      <td>1.970011e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1970-01-16 08:05:39.000</td>
      <td>60.280</td>
      <td>-152.660</td>
      <td>85.0</td>
      <td>6.0</td>
      <td>ML</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>AK</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



아래 코드 셀은 지도에서 플레이트 경계를 시각화합니다. 모든 지진 데이터를 사용하여 동일한 지도에 열 지도를 추가하고 지진이 판 경계와 일치하는지 여부를 확인할 수 있습니다 


```python
# Create a base map with plate boundaries
m_1 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)
for i in range(len(plate_boundaries)):
    folium.PolyLine(locations=plate_boundaries.coordinates.iloc[i], weight=2, color='black').add_to(m_1)

### Your code here: Add a heatmap to the map

# 표시할 데이터로 HeatMap 레이어를 생성합니다.
data = [[row['Latitude'], row['Longitude']] for index, row in earthquakes.iterrows()]
heatmap_layer = HeatMap(data=data, radius=20)

# HeatMap 레이어를 지도에 추가합니다.
heatmap_layer.add_to(m_1)

# Show the map
embed_map(m_1, 'q_1.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_1.html"
    frameborder="0"
    allowfullscreen

></iframe>




그러면 위의 지도를 보면 지진은 판의 경계와 일치합니까?

정답 : 일치한다

### 일본에서 지진 깊이와 판 경계에 근접하는 것 사이에 관계가 있습니까?

당신은 최근에 지진의 깊이가 우리에게 말해주는 것을 읽었습니다 [important information](https://www.usgs.gov/faqs/what-depth-do-earthquakes-occur-what-significance-depth?qt-news_science_products=0#qt-news_science_products) 지구의 구조에 관하여. 여러분은 흥미로운 세계적인 패턴이 있는지 알고 싶어하고, 일본에서 깊이가 어떻게 다른지 알고 싶어합니다.



```python
# Create a base map with plate boundaries
m_2 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)
for i in range(len(plate_boundaries)):
    folium.PolyLine(locations=plate_boundaries.coordinates.iloc[i], weight=2, color='black').add_to(m_2)
    
### Your code here: Add a map to visualize earthquake depth

# 지진 발생 지점을 Marker 객체로 표시
for idx, row in earthquakes.iterrows():
    folium.Marker(location=[row['Latitude'], row['Longitude']], popup=f"Depth: {row['Depth']}").add_to(m_2)
    
# 지진 깊이를 CircleMarker 객체로 시각화
for idx, row in earthquakes.iterrows():
    folium.CircleMarker(location=[row['Latitude'], row['Longitude']],
                        radius=10,
                        popup=f"Depth: {row['Depth']}",
                        fill=True,
                        fill_opacity=0.5,
                        color='red').add_to(m_2)

# View the map
embed_map(m_2, 'q_2.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_2.html"
    frameborder="0"
    allowfullscreen

></iframe>




판 경계에 대한 근접성과 지진 깊이 사이의 관계를 감지할 수 있습니까? 이 패턴은 세계적으로 통용됩니까?

정답 : 대체적으로 판 경계와 가까울수록 깊이가 얕고 멀어질수록 깊이가 깊다

### 인구 밀도가 높은 현은 어디입니까?

다음 코드 셀을 변경하지 않고 실행하여 일본 현의 지리적 경계가 포함된 GeoDataFrame "현"을 만듭니다.


```python
# GeoDataFrame with prefecture boundaries
prefectures = gpd.read_file("C:/Users\seong taek/Desktop/archive/japan-prefecture-boundaries/japan-prefecture-boundaries/japan-prefecture-boundaries.shp")
prefectures.set_index('prefecture', inplace=True)
prefectures.head()
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
      <th>geometry</th>
    </tr>
    <tr>
      <th>prefecture</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aichi</th>
      <td>MULTIPOLYGON (((137.09523 34.65330, 137.09546 ...</td>
    </tr>
    <tr>
      <th>Akita</th>
      <td>MULTIPOLYGON (((139.55725 39.20330, 139.55765 ...</td>
    </tr>
    <tr>
      <th>Aomori</th>
      <td>MULTIPOLYGON (((141.39860 40.92472, 141.39806 ...</td>
    </tr>
    <tr>
      <th>Chiba</th>
      <td>MULTIPOLYGON (((139.82488 34.98967, 139.82434 ...</td>
    </tr>
    <tr>
      <th>Ehime</th>
      <td>MULTIPOLYGON (((132.55859 32.91224, 132.55904 ...</td>
    </tr>
  </tbody>
</table>
</div>



다음 코드 셀은 각 일본 현에 대한 인구, 면적(제곱킬로미터) 및 인구 밀도(제곱킬로미터당)를 포함하는 데이터 프레임 통계를 만듭니다. 코드 셀을 변경하지 않고 실행합니다.


```python
# DataFrame containing population of each prefecture
population = pd.read_csv("C:/Users\seong taek/Desktop/archive/japan-prefecture-population.csv")
population.set_index('prefecture', inplace=True)

# Calculate area (in square kilometers) of each prefecture
area_sqkm = pd.Series(prefectures.geometry.to_crs(epsg=32654).area / 10**6, name='area_sqkm')
stats = population.join(area_sqkm)

# Add density (per square kilometer) of each prefecture
stats['density'] = stats["population"] / stats["area_sqkm"]
stats.head()
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
      <th>population</th>
      <th>area_sqkm</th>
      <th>density</th>
    </tr>
    <tr>
      <th>prefecture</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Tokyo</th>
      <td>12868000</td>
      <td>1800.614782</td>
      <td>7146.448049</td>
    </tr>
    <tr>
      <th>Kanagawa</th>
      <td>8943000</td>
      <td>2383.038975</td>
      <td>3752.771186</td>
    </tr>
    <tr>
      <th>Osaka</th>
      <td>8801000</td>
      <td>1923.151529</td>
      <td>4576.342460</td>
    </tr>
    <tr>
      <th>Aichi</th>
      <td>7418000</td>
      <td>5164.400005</td>
      <td>1436.372085</td>
    </tr>
    <tr>
      <th>Saitama</th>
      <td>7130000</td>
      <td>3794.036890</td>
      <td>1879.264806</td>
    </tr>
  </tbody>
</table>
</div>



다음 코드 셀을 사용하여 모집단 밀도를 시각화하는 코로플레스 맵을 작성합니다.


```python
# Create a base map
m_3 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)

### Your code here: create a choropleth map to visualize population density

# Create choropleth map
Choropleth(geo_data=prefectures['geometry'],
           data=stats['density'],
           key_on="feature.id",
           fill_color='YlGnBu',
           legend_name='Population density (per square kilometer)'
          ).add_to(m_3)

# View the map
embed_map(m_3, 'q_3.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_3.html"
    frameborder="0"
    allowfullscreen

></iframe>




다른 현들보다 상대적으로 밀도가 높은 현은 어디입니까? 그들은 전국적으로 퍼져 있습니까, 아니면 모두 대략 같은 지리적 지역에 위치하고 있습니까? (일본 지리에 익숙하지 않은 경우 이 지도가 질문에 대답하는 데 유용할 수 있습니다.) [지도](https://en.wikipedia.org/wiki/Prefectures_of_Japan)

정답 : 도쿄,오사카등 중심도시

### 고밀도 현 중 규모가 큰 지진이 발생하기 쉬운 곳은 어디입니까?

지진 보강의 혜택을 받을 수 있는 한 현을 제안하는 지도를 작성합니다. 지도는 밀도와 지진 규모를 모두 시각화해야 합니다.

규모 7이상을 규모가 큰 지진으로 설정


```python
# Create a base map
m_4 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)

### Your code here: create a map

def color_producer(val):
    if val <=7:
        return 'forestgreen'
    else:
        return 'blue'
    
Choropleth(
    geo_data=prefectures['geometry'].__geo_interface__,
    data=stats['density'],
    key_on="feature.id",
    fill_color='BuPu',
    legend_name='Population density (per square kilometer)').add_to(m_4)

for i in range(0,len(earthquakes)):
    folium.Circle(
        location=[earthquakes.iloc[i]['Latitude'], earthquakes.iloc[i]['Longitude']],
        popup=("{} ({})").format(
            earthquakes.iloc[i]['Magnitude'],
            earthquakes.iloc[i]['DateTime'].year),
        radius=earthquakes.iloc[i]['Magnitude']**5.5,
        color=color_producer(earthquakes.iloc[i]['Magnitude'])).add_to(m_4)

# View the map
embed_map(m_4, 'q_4.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_4.html"
    frameborder="0"
    allowfullscreen

></iframe>




추가 지진 보강을 위해 어느 현을 추천하십니까?

Iwate현 추천

## Keep going

Learn how to convert names of places to geographic coordinates with **[geocoding](https://www.kaggle.com/alexisbcook/manipulating-geospatial-data)**.  You'll also explore special ways to join information from multiple GeoDataFrames.

---




*Have questions or comments? Visit the [course discussion forum](https://www.kaggle.com/learn/geospatial-analysis/discussion) to chat with other learners.*