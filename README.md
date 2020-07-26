# Google-Earth-Python-Utils


## 개요 

Google Earth에서 Satellite Imagery 데이터셋을 보다 쉽게 만들 수 있도록 해주는 코드입니다.

우선 초기 버전은 `EasyEarth`라는 클래스로 제작되었습니다.

Google Earth Engine에 대한 인증이 상대적으로 간편한 [Google Colab](https://colab.research.google.com/) 환경에서 실행되었습니다.

### 특장점
- 위도, 경도 기반 km x km 설정을 통한 데이터 추출
- 날짜 필터링 가능
- 구름양 검출 및 해당 데이터셋 중에서 가장 구름이 적은 이미지 획득 가능
- 손쉬운 resampling을 통한 화질 개선

 

### 이슈

- 특정 지역의 경우 범위 밖으로 인식되어 빈 값들만 나온다. (region 설정 문제)
- RGB 설정 문제


## Usage

### 1. 데이터 소스 (인공위성) 지정

Google Earth Data Catalog [[link]](https://developers.google.com/earth-engine/datasets)

```python
earth = EasyEarth("COPERNICUS/S2")
# earth = EasyEarth("LANDSAT/LC08/C01/T1_SR")
```

### 2. Area of Interest(AOI) 및 기타 조건 설정 

전체 데이터셋 중에서 관심이 있는 지역을 설정해야 합니다. 

`select_AOI` 함수는 위도, 경도를 중심으로 하여 입력된 **k** (default 10) 만큼의 반경을 가지는 AOI를 생성합니다.

추가적으로 날짜도 필터링 가능합니다.

```python
lat = 37.370474
lon = 128.3899769 

earth.select_AOI(lat,lon, 5, ("2016-01-01", "2016-12-31")) 
print(earth.AOI_size) ## 필터링된 조건에 부합하는 데이터의 수
```

### 3. 이미지 획득

이제 마지막으로 Google Earth Engine에서 좋은 품질의 이미지를 획득하기 위해, 우선 해당 위성에 대한 데이터 카탈로그의 정보를 기반으로하여 

`sort_by` 함수로 구름양을 기준으로 데이터셋을 정렬합니다 (오름차순).

그 다음 `get_collection_at` 함수로 정렬된 데이터 중에서 첫번째 사진을 선택하고, 동일하게 데이터 카탈로그를 기반으로 한 `parameters`를 생성합니다.

여기까지 준비되었다면,  `plot_img` 함수를 통해 기존 `ee.Image`를 `PIL` 모듈의 이미지 형태로 반환합니다. 


```python

cloud_coverage = "CLOUDY_PIXEL_PERCENTAGE"

earth.sort_by(cloud_coverage) # 사진의 구름양을 기반으로 정렬 (오름차순)
least_cloudy_img = earth.get_collections_at(0) # 정렬된 이미지 중 첫번째

parameters = {
  'min': 0.0,
  'max': 2500,
  'bands': ['B4', 'B3', 'B2'],
  'dimensions': 512,
  'region': earth.AOI
}

# Resampling 기법 적용 및 파라미터를 입력하여 사진 획득 (PIL.Image 포맷)
result = earth.plot_image(least_cloudy_img.resample('bicubic'), parameters, cloud_coverage) 
result

```
