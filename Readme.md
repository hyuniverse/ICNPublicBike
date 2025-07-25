## 1. Introdoction
본 프로젝트는 서울시 공공자전거 '따릉이'의 성공 요인을 계량적으로 분석하고, 이를 인천시에 적용하여 신규 대여소의 최적 입지를 도출하는 공간 분석(Geospatial Analysis) 모델을 개발하는 것을 목표로 합니다.

핵심 기술은 이종(Heterogeneous)의 GIS 데이터—벡터(Point, Polygon)와 래스터(Raster)—를 파이썬 생태계 안에서 융합하여 처리하고, 가설을 검증하며, 최종적으로 실행 가능한 비즈니스 인사이트(Actionable Insight)를 제공하는 데 있습니다.
모든 분석 과정은 **재현성(Reproducibility)과 확장성(Scalability)** 을 고려하여 Python, geopandas, rasterio 기반의 코드로 구현되었습니다.

## 2. Stack
### Primary Language:
Python (3.x)

### Core Libraries:

**geopandas & shapely**: 벡터 데이터(SHP, GeoJSON)의 I/O, 좌표계 변환(CRS), 공간 연산 및 기하 객체(Geometry) 처리를 위한 핵심 라이브러리.

**rasterio & richdem**: DEM(Digital Elevation Model)과 같은 래스터 데이터의 I/O, 특정 좌표의 값 추출 및 경사도(Slope)와 같은 지형 속성 계산에 사용.

**pandas**: 정형 데이터(CSV) 처리 및 통계 분석.

**pyplot & matplotlib & seaborn**: 분석 결과 시각화.

## 3. Data Analysis Pipeline
### Step 1: Data Ingestion & CRS Unification
Challenge: 서울시와 인천시에서 수집된 데이터(SHP, CSV)는 각각 다른 좌표계(CRS)를 사용하거나 지리 정보가 누락되어 있어 공간 연산이 불가능했습니다.

Solution:

pandas로 CSV를 읽고, 위경도 정보를 shapely.geometry.Point 객체로 변환하여 GeoDataFrame을 생성했습니다.

모든 벡터 및 래스터 데이터의 좌표계를 EPSG:5179 (Korea 2000 / Unified CS) 로 통일하여 공간 연산의 정합성을 확보했습니다. 이 과정에서 geopandas.GeoDataFrame.to_crs() 함수를 핵심적으로 사용했습니다.

### Step 2: Feature Engineering via Geospatial Operations
Objective: 대여소의 '수요'와 '주변 환경' 간의 상관관계를 밝히기 위해 공간적 피처를 생성합니다.

Techniques:

Buffer Analysis: 각 대여소(Point)를 중심으로 GeoDataFrame.buffer(100)를 적용하여 반경 100m의 영향권(Polygon)을 생성했습니다. 이는 주변 환경을 분석하기 위한 Micro-region을 정의하는 과정입니다.

Spatial Join (geopandas.sjoin): 생성된 버퍼와 건물(Polygon) 데이터를 predicate='within' 옵션을 사용하여 공간 조인했습니다. 이를 통해 각 대여소의 영향권 내에 포함된 건물의 용도별 개수를 정확히 집계, 이를 주요 피처로 활용했습니다.

Raster Data Extraction:

rasterio.open()으로 DEM 파일을 로드하고, 각 대여소의 좌표(Point)가 래스터의 어떤 픽셀에 해당하는지 식별했습니다.

richdem.rd.TerrainAttribute(dem, attrib='slope_riserun')을 사용하여 DEM 전체의 경사도 맵을 계산하고, 대여소 위치의 정확한 경사도 값을 추출하여 지형적 제약 요소를 피처에 추가했습니다.

### Step 3: Hypothesis Testing & Model Formulation
Hypothesis: "서울시에서 성공한 대여소는 특정 건물 용도(예: 근린생활시설, 업무시설)가 밀집되어 있고, 경사도가 낮을 것이다."

Validation:

대여 건수 기준 상위/하위 100개소 그룹을 설정하고, 위에서 생성한 공간 피처(건물 용도별 개수, 경사도)에 대해 T-검정(T-test) 또는 평균 비교를 수행하여 가설의 유효성을 통계적으로 검증했습니다.

검증된 핵심 요인들을 조합하여 **"최적 입지 스코어링(Scoring) 모델"** 의 기준을 수립했습니다. (예: Score = w1 * (근생시설 수) + w2 * (업무시설 수) - w3 * (경사도))

### Step 4: Application to Target Region (Incheon)
서울시 분석과 동일한 파이프라인을 인천시 데이터에 적용했습니다.

앞서 수립한 스코어링 모델을 활용하여 인천시 내 모든 잠재 후보 지역의 점수를 계산하고, 우선순위가 높은 최종 추천 입지를 선정했습니다. 이는 분석 모델의 확장성과 일반화 가능성을 보여줍니다.

## 4. Key Technical Achievements
**End-to-End Geospatial Pipeline:** 데이터 수집부터 전처리, 분석, 모델링, 최종 제안까지 모든 과정을 Python 코드로 자동화하여 재현 가능한(Reproducible) 연구를 수행했습니다.

**Heterogeneous Data Integration:** 정형(tabular), 벡터(vector), 래스터(raster) 데이터를 완벽하게 통합하여 단일 분석 환경에서 처리하는 기술적 역량을 확보했습니다.

**Quantitative Problem Solving:** '입지가 좋다'는 정성적 개념을 '버퍼 내 건물 수', '경사도' 등 측정 가능한 정량적 지표로 변환하여 문제에 접근하고 해결했습니다.
