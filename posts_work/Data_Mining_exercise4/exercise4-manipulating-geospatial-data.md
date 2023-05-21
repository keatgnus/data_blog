---
title: "Data_Mining_Kaggle"
author: "Seongtaek"
date: "2023-05-02"
categories: [code, data_mining, jupyter, kaggle]
image: "sb.png"
toc: true
---

Exercise 4 - Manipulating Geospatial Data

<table align="bottom">
  <td>
    <a target="_blank" href="http://localhost:8888/notebooks/3-1%20Data_Mining/exercise4-manipulating-geospatial-data.ipynb#"><img
    src = small_jup_image.jpg />Jupyter에서 실행하기</a>
  </td>
</table>

**This notebook is an exercise in the [Geospatial Analysis](https://www.kaggle.com/learn/geospatial-analysis) course.  You can reference the tutorial at [this link](https://www.kaggle.com/alexisbcook/manipulating-geospatial-data).**

---


## Introduction

You are a Starbucks big data analyst ([that’s a real job!](https://www.forbes.com/sites/bernardmarr/2018/05/28/starbucks-using-big-data-analytics-and-artificial-intelligence-to-boost-performance/#130c7d765cdc)) looking to find the next store into a [Starbucks Reserve Roastery](https://www.businessinsider.com/starbucks-reserve-roastery-compared-regular-starbucks-2018-12#also-on-the-first-floor-was-the-main-coffee-bar-five-hourglass-like-units-hold-the-freshly-roasted-coffee-beans-that-are-used-in-each-order-the-selection-rotates-seasonally-5).  These roasteries are much larger than a typical Starbucks store and have several additional features, including various food and wine options, along with upscale lounge areas.  You'll investigate the demographics of various counties in the state of California, to determine potentially suitable locations.

<center>
<img src="https://storage.googleapis.com/kaggle-media/learn/images/BIyE6kR.png" width="450"><br/><br/>
</center>

Before you get started, run the code cell below to set everything up.

- 당신은 스타벅스의 빅데이터 분석가입니다. (그것은 진정한 직업입니다!) 스타벅스 리저브 로스터리에서 다음 매장을 찾고 있습니다. 이 로스터리는 일반적인 스타벅스 매장보다 훨씬 크고 고급 라운지 공간과 함께 다양한 음식과 와인 옵션을 포함한 몇 가지 추가 기능을 갖추고 있습니다. 캘리포니아 주의 다양한 카운티의 인구 통계를 조사하여 잠재적으로 적합한 위치를 결정합니다.
- 시작하기 전에 아래의 코드 셀을 실행하여 모든 설정을 수행합니다


```python
#!pip install geopy
import math
import pandas as pd
import geopandas as gpd
from geopy.geocoders import Nominatim
import folium 
from folium import Marker
from folium.plugins import MarkerCluster
```

이전 연습의 embed_map() 함수를 사용하여 지도를 시각화합니다.


```python
def embed_map(m, file_name):
    from IPython.display import IFrame
    m.save(file_name)
    return IFrame(file_name, width='100%', height='500px')
```

## Exercises

### 누락된 위치를 지오코드합니다

다음 코드 셀을 실행하여 캘리포니아 주에 있는 스타벅스 위치가 포함된 데이터 프레임 스타벅스를 만듭니다.


```python
# Load and preview Starbucks locations in California
starbucks = pd.read_csv("C:/Users\seong taek/Desktop/archive/starbucks_locations.csv")
starbucks.head()
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
      <th>Store Number</th>
      <th>Store Name</th>
      <th>Address</th>
      <th>City</th>
      <th>Longitude</th>
      <th>Latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10429-100710</td>
      <td>Palmdale &amp; Hwy 395</td>
      <td>14136 US Hwy 395 Adelanto CA</td>
      <td>Adelanto</td>
      <td>-117.40</td>
      <td>34.51</td>
    </tr>
    <tr>
      <th>1</th>
      <td>635-352</td>
      <td>Kanan &amp; Thousand Oaks</td>
      <td>5827 Kanan Road Agoura CA</td>
      <td>Agoura</td>
      <td>-118.76</td>
      <td>34.16</td>
    </tr>
    <tr>
      <th>2</th>
      <td>74510-27669</td>
      <td>Vons-Agoura Hills #2001</td>
      <td>5671 Kanan Rd. Agoura Hills CA</td>
      <td>Agoura Hills</td>
      <td>-118.76</td>
      <td>34.15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29839-255026</td>
      <td>Target Anaheim T-0677</td>
      <td>8148 E SANTA ANA CANYON ROAD AHAHEIM CA</td>
      <td>AHAHEIM</td>
      <td>-117.75</td>
      <td>33.87</td>
    </tr>
    <tr>
      <th>4</th>
      <td>23463-230284</td>
      <td>Safeway - Alameda 3281</td>
      <td>2600 5th Street Alameda CA</td>
      <td>Alameda</td>
      <td>-122.28</td>
      <td>37.79</td>
    </tr>
  </tbody>
</table>
</div>



대부분의 상점은 알려진 위치(위도, 경도)를 가지고 있습니다. 하지만, 버클리 시의 모든 장소가 사라졌습니다


```python
# How many rows in each column have missing values?
print(starbucks.isnull().sum())

# View rows with missing locations
rows_with_missing = starbucks[starbucks["City"]=="Berkeley"]
rows_with_missing
```

    Store Number    0
    Store Name      0
    Address         0
    City            0
    Longitude       5
    Latitude        5
    dtype: int64
    




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
      <th>Store Number</th>
      <th>Store Name</th>
      <th>Address</th>
      <th>City</th>
      <th>Longitude</th>
      <th>Latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>153</th>
      <td>5406-945</td>
      <td>2224 Shattuck - Berkeley</td>
      <td>2224 Shattuck Avenue Berkeley CA</td>
      <td>Berkeley</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>154</th>
      <td>570-512</td>
      <td>Solano Ave</td>
      <td>1799 Solano Avenue Berkeley CA</td>
      <td>Berkeley</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>155</th>
      <td>17877-164526</td>
      <td>Safeway - Berkeley #691</td>
      <td>1444 Shattuck Place Berkeley CA</td>
      <td>Berkeley</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>156</th>
      <td>19864-202264</td>
      <td>Telegraph &amp; Ashby</td>
      <td>3001 Telegraph Avenue Berkeley CA</td>
      <td>Berkeley</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>157</th>
      <td>9217-9253</td>
      <td>2128 Oxford St.</td>
      <td>2128 Oxford Street Berkeley CA</td>
      <td>Berkeley</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



아래 코드 셀을 사용하여 이러한 값을 Nominatim 지오코더로 채웁니다.

튜토리얼에서 Nominatim()(geopy.geocoders에서)을 사용하여 값을 지오코딩했으며, 이는 본 과정 이외의 프로젝트에서 사용할 수 있는 것입니다.

이 연습에서는 (learn tools.geospatic에서) 약간 다른 함수 Nominatim()을 사용합니다.도구). 이 기능은 노트북 상단에 가져온 것으로 GeoPandas의 기능과 동일하게 작동합니다.

즉, 다음과 같은 경우에 한합니다:

노트북 상단에 있는 가져오기 문을 변경하지 않습니다
당신은 아래의 코드 셀에서 지오코딩 함수를 지오코딩이라고 부릅니다,
코드가 의도한 대로 작동합니다!


```python
# Create the geocoder
geolocator = Nominatim(user_agent="kaggle_learn")

# Your code here
def my_geocoder(row):
    point = geolocator.geocode(row).point
    return pd.Series({'Latitude': point.latitude, 'Longitude': point.longitude})

berkeley_locations = rows_with_missing.apply(lambda x: my_geocoder(x['Address']), axis=1)
starbucks.update(berkeley_locations)

starbucks
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
      <th>Store Number</th>
      <th>Store Name</th>
      <th>Address</th>
      <th>City</th>
      <th>Longitude</th>
      <th>Latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10429-100710</td>
      <td>Palmdale &amp; Hwy 395</td>
      <td>14136 US Hwy 395 Adelanto CA</td>
      <td>Adelanto</td>
      <td>-117.40</td>
      <td>34.51</td>
    </tr>
    <tr>
      <th>1</th>
      <td>635-352</td>
      <td>Kanan &amp; Thousand Oaks</td>
      <td>5827 Kanan Road Agoura CA</td>
      <td>Agoura</td>
      <td>-118.76</td>
      <td>34.16</td>
    </tr>
    <tr>
      <th>2</th>
      <td>74510-27669</td>
      <td>Vons-Agoura Hills #2001</td>
      <td>5671 Kanan Rd. Agoura Hills CA</td>
      <td>Agoura Hills</td>
      <td>-118.76</td>
      <td>34.15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29839-255026</td>
      <td>Target Anaheim T-0677</td>
      <td>8148 E SANTA ANA CANYON ROAD AHAHEIM CA</td>
      <td>AHAHEIM</td>
      <td>-117.75</td>
      <td>33.87</td>
    </tr>
    <tr>
      <th>4</th>
      <td>23463-230284</td>
      <td>Safeway - Alameda 3281</td>
      <td>2600 5th Street Alameda CA</td>
      <td>Alameda</td>
      <td>-122.28</td>
      <td>37.79</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2816</th>
      <td>14071-108147</td>
      <td>Hwy 20 &amp; Tharp - Yuba City</td>
      <td>1615 Colusa Hwy, Ste 100 Yuba City CA</td>
      <td>Yuba City</td>
      <td>-121.64</td>
      <td>39.14</td>
    </tr>
    <tr>
      <th>2817</th>
      <td>9974-98559</td>
      <td>Yucaipa &amp; Hampton, Yucaipa</td>
      <td>31364 Yucaipa Blvd., A Yucaipa CA</td>
      <td>Yucaipa</td>
      <td>-117.12</td>
      <td>34.03</td>
    </tr>
    <tr>
      <th>2818</th>
      <td>79654-108478</td>
      <td>Vons - Yucaipa #1796</td>
      <td>33644 YUCAIPA BLVD YUCAIPA CA</td>
      <td>YUCAIPA</td>
      <td>-117.07</td>
      <td>34.04</td>
    </tr>
    <tr>
      <th>2819</th>
      <td>6438-245084</td>
      <td>Yucaipa &amp; 6th</td>
      <td>34050 Yucaipa Blvd., 200 Yucaipa CA</td>
      <td>Yucaipa</td>
      <td>-117.06</td>
      <td>34.03</td>
    </tr>
    <tr>
      <th>2820</th>
      <td>6829-82142</td>
      <td>Highway 62 &amp; Warren Vista</td>
      <td>57744  29 Palms Highway Yucca Valley CA</td>
      <td>Yucca Valley</td>
      <td>-116.40</td>
      <td>34.13</td>
    </tr>
  </tbody>
</table>
<p>2821 rows × 6 columns</p>
</div>



### 2) Berkeley 위치를 봅니다.
방금 찾은 위치를 살펴보겠습니다. OpenStreetMap 스타일로 Berkeley의 (위도, 경도) 위치를 시각화합니다.


```python
# Create a base map
m_2 = folium.Map(location=[37.88,-122.26], zoom_start=13)

# Your code here: Add a marker for each Berkeley location
for idx, row in starbucks[starbucks["City"]=='Berkeley'].iterrows():
    Marker([row['Latitude'], row['Longitude']]).add_to(m_2)

# Show the map
embed_map(m_2, 'q_2.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_2.html"
    frameborder="0"
    allowfullscreen

></iframe>




Considering only the five locations in Berkeley, how many of the (latitude, longitude) locations seem potentially correct (are located in the correct city)?

### 3) 데이터를 통합합니다.

아래 코드를 실행하여 캘리포니아 주의 각 카운티에 대한 이름, 면적(제곱킬로미터) 및 고유 ID("GEOID" 열)가 포함된 GeoDataFrame "CA_counties"를 로드합니다. 지오메트리 열에는 카운티 경계가 있는 폴리곤이 포함되어 있습니다.


```python
CA_counties = gpd.read_file("C:/Users\seong taek/Desktop/archive/CA_county_boundaries/CA_county_boundaries/CA_county_boundaries.shp")
CA_counties.crs = {'init': 'epsg:4326'}
CA_counties.head()
```

    C:\Users\seong taek\anaconda3\lib\site-packages\pyproj\crs\crs.py:141: FutureWarning: '+init=<authority>:<code>' syntax is deprecated. '<authority>:<code>' is the preferred initialization method. When making the change, be mindful of axis order changes: https://pyproj4.github.io/pyproj/stable/gotchas.html#axis-order-changes-in-proj-6
      in_crs_string = _prepare_from_proj_string(in_crs_string)
    




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
      <th>GEOID</th>
      <th>name</th>
      <th>area_sqkm</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6091</td>
      <td>Sierra County</td>
      <td>2491.995494</td>
      <td>POLYGON ((-120.65560 39.69357, -120.65554 39.6...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6067</td>
      <td>Sacramento County</td>
      <td>2575.258262</td>
      <td>POLYGON ((-121.18858 38.71431, -121.18732 38.7...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6083</td>
      <td>Santa Barbara County</td>
      <td>9813.817958</td>
      <td>MULTIPOLYGON (((-120.58191 34.09856, -120.5822...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6009</td>
      <td>Calaveras County</td>
      <td>2685.626726</td>
      <td>POLYGON ((-120.63095 38.34111, -120.63058 38.3...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6111</td>
      <td>Ventura County</td>
      <td>5719.321379</td>
      <td>MULTIPOLYGON (((-119.63631 33.27304, -119.6360...</td>
    </tr>
  </tbody>
</table>
</div>



다음으로 세 가지 데이터 프레임을 만듭니다:

- CA_pop에는 각 카운티의 인구 추정치가 포함되어 있습니다.
- CA_high_earner에는 연간 $150,000 이상의 소득을 가진 가구 수가 포함되어 있습니다.
- CA_median_age에는 각 카운티의 중위수 연령이 포함됩니다.


```python
CA_pop = pd.read_csv("C:/Users\seong taek/Desktop/archive/CA_county_population.csv", index_col="GEOID")
CA_high_earners = pd.read_csv("C:/Users\seong taek/Desktop/archive/CA_county_high_earners.csv", index_col="GEOID")
CA_median_age = pd.read_csv("C:/Users\seong taek/Desktop/archive/CA_county_median_age.csv", index_col="GEOID")
```

다음 코드 셀을 사용하여 CA_pop, CA_high_earners 및 CA_median_age와 함께 CA_counties GeoDataFrame에 join 합니다.

결과 GeoDataFrame CA_stats의 이름을 지정하고 "GEOID", "name", "area_sqkm", "geometry", "population", "high_earners" 및 "median_age"의 8개 열이 있는지 확인합니다.


```python
# Your code here
cols_to_add = CA_pop.join([CA_high_earners, CA_median_age]).reset_index()
CA_stats = CA_counties.merge(cols_to_add, on="GEOID")

CA_stats.head()
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
      <th>GEOID</th>
      <th>name</th>
      <th>area_sqkm</th>
      <th>geometry</th>
      <th>population</th>
      <th>high_earners</th>
      <th>median_age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6091</td>
      <td>Sierra County</td>
      <td>2491.995494</td>
      <td>POLYGON ((-120.65560 39.69357, -120.65554 39.6...</td>
      <td>2987</td>
      <td>111</td>
      <td>55.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6067</td>
      <td>Sacramento County</td>
      <td>2575.258262</td>
      <td>POLYGON ((-121.18858 38.71431, -121.18732 38.7...</td>
      <td>1540975</td>
      <td>65768</td>
      <td>35.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6083</td>
      <td>Santa Barbara County</td>
      <td>9813.817958</td>
      <td>MULTIPOLYGON (((-120.58191 34.09856, -120.5822...</td>
      <td>446527</td>
      <td>25231</td>
      <td>33.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6009</td>
      <td>Calaveras County</td>
      <td>2685.626726</td>
      <td>POLYGON ((-120.63095 38.34111, -120.63058 38.3...</td>
      <td>45602</td>
      <td>2046</td>
      <td>51.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6111</td>
      <td>Ventura County</td>
      <td>5719.321379</td>
      <td>MULTIPOLYGON (((-119.63631 33.27304, -119.6360...</td>
      <td>850967</td>
      <td>57121</td>
      <td>37.5</td>
    </tr>
  </tbody>
</table>
</div>



이제 모든 데이터가 한 곳에 있으므로 열 조합을 사용하는 통계량을 계산하는 것이 훨씬 쉬워졌습니다. 다음 코드 셀을 실행하여 모집단 밀도가 있는 "밀도" 열을 만듭니다.


```python
CA_stats["density"] = CA_stats["population"] / CA_stats["area_sqkm"]
```

### 4) 어느 카운티가 유망해 보이나요?
모든 정보를 단일 GeoDataFrame으로 통합하면 특정 기준을 충족하는 카운티를 훨씬 쉽게 선택할 수 있습니다.

다음 코드 셀을 사용하여 CA_stats GeoDataFrame에서 행의 하위 집합(및 모든 열)을 포함하는 GeoDataFramesel_counties를 만듭니다. 특히 다음과 같은 국가를 선택해야 합니다:

- 매년 15만 달러를 버는 최소 10만 가구가 있습니다,
- 중위연령은 38.5세 미만이고
- 거주자의 밀도는 최소 285(제곱킬로미터당)입니다.

또한 선택된 카운티는 다음 기준 중 하나 이상을 충족해야 합니다:

- 매년 15만 달러를 버는 최소 50만 가구가 있습니다,
- 중위연령이 35.5세 미만인 경우
- 거주자의 밀도는 적어도 1400(평방킬로미터당)입니다.


```python
# Your code here
# Your code here
sel_counties = CA_stats[((CA_stats.high_earners > 100000) &
                         (CA_stats.median_age < 38.5) &
                         (CA_stats.density > 285) &
                         ((CA_stats.median_age < 35.5) |
                         (CA_stats.density > 1400) |
                         (CA_stats.high_earners > 500000)))]

sel_counties.head()
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
      <th>GEOID</th>
      <th>name</th>
      <th>area_sqkm</th>
      <th>geometry</th>
      <th>population</th>
      <th>high_earners</th>
      <th>median_age</th>
      <th>density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>6037</td>
      <td>Los Angeles County</td>
      <td>12305.376879</td>
      <td>MULTIPOLYGON (((-118.66761 33.47749, -118.6682...</td>
      <td>10105518</td>
      <td>501413</td>
      <td>36.0</td>
      <td>821.227834</td>
    </tr>
    <tr>
      <th>8</th>
      <td>6073</td>
      <td>San Diego County</td>
      <td>11721.342229</td>
      <td>POLYGON ((-117.43744 33.17953, -117.44955 33.1...</td>
      <td>3343364</td>
      <td>194676</td>
      <td>35.4</td>
      <td>285.237299</td>
    </tr>
    <tr>
      <th>10</th>
      <td>6075</td>
      <td>San Francisco County</td>
      <td>600.588247</td>
      <td>MULTIPOLYGON (((-122.60025 37.80249, -122.6123...</td>
      <td>883305</td>
      <td>114989</td>
      <td>38.3</td>
      <td>1470.733077</td>
    </tr>
  </tbody>
</table>
</div>



### 5) 당신은 몇 개의 상점을 확인했습니까?
다음 스타벅스 리저브 로스터리 위치를 찾을 때는 선택한 카운티 내의 모든 매장을 고려해야 합니다. 그렇다면, 선택된 카운티 내에 몇 개의 상점이 있을까요?

이 질문에 대한 답변을 준비하려면 다음 코드 셀을 실행하여 모든 스타벅스 위치와 함께 GeoDataFrame stabs_gdf를 만듭니다.


```python
starbucks_gdf = gpd.GeoDataFrame(starbucks, geometry=gpd.points_from_xy(starbucks.Longitude, starbucks.Latitude))
starbucks_gdf.crs = {'init': 'epsg:4326'}
```

    C:\Users\seong taek\anaconda3\lib\site-packages\pyproj\crs\crs.py:141: FutureWarning: '+init=<authority>:<code>' syntax is deprecated. '<authority>:<code>' is the preferred initialization method. When making the change, be mindful of axis order changes: https://pyproj4.github.io/pyproj/stable/gotchas.html#axis-order-changes-in-proj-6
      in_crs_string = _prepare_from_proj_string(in_crs_string)
    

그렇다면, 당신이 선택한 county에는 몇 개의 가게가 있나요?


```python
# Fill in your answer
locations_of_interest = gpd.sjoin(starbucks_gdf, sel_counties)
num_stores = len(locations_of_interest)
num_stores
```




    1043



1043개

### 6) 저장소 위치를 시각화합니다.
이전 질문에서 식별한 상점의 위치를 보여주는 맵을 만듭니다.


```python
# Create a base map
m_6 = folium.Map(location=[37,-120], zoom_start=6)

# Your code here: show selected store locations
mc = MarkerCluster()

locations_of_interest = gpd.sjoin(starbucks_gdf, sel_counties)
for idx, row in locations_of_interest.iterrows():
    if not math.isnan(row['Longitude']) and not math.isnan(row['Latitude']):
        mc.add_child(folium.Marker([row['Latitude'], row['Longitude']]))
m_6.add_child(mc)

# Show the map
embed_map(m_6, 'q_6.html')
```





<iframe
    width="100%"
    height="500px"
    src="q_6.html"
    frameborder="0"
    allowfullscreen

></iframe>




## Keep going

Learn about how **[proximity analysis](https://www.kaggle.com/alexisbcook/proximity-analysis)** can help you to understand the relationships between points on a map.

---




*Have questions or comments? Visit the [course discussion forum](https://www.kaggle.com/learn/geospatial-analysis/discussion) to chat with other learners.*
