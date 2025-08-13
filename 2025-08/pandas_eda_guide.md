## pandas 라이브러리와 탐색적 데이터 분석

### 1. 탐색적 데이터 분석(EDA: Exploratory Data Analysis)

#### 개념 및 목적
- EDA (Exploratory Data Analysis) 라고 함
- 데이터 분석을 위해 raw data를 다양한 각도에서 관찰하여, 데이터를 이해하는 과정
- 데이터 분석 주제마다 EDA를 통해 진행하는 과정은 각양각색이므로, 정형화된 패턴은 없지만, 크게 다음과 같은 3가지 과정은 기본이 될 수 있으므로 다음 3가지 과정을 기본으로 이해

#### 기본 분석 과정

1. 데이터의 출처와 주제에 대해 이해
2. 데이터의 크기 확인
3. 데이터 구성 요소(feature)의 속성(특징) 확인
   - feature: 데이터 구성 요소를 의미함
   - 예: 어떤 초등학교에 학생 성적을 기록한 데이터가 있다면, 학생 이름, 과목별 성적등을 feature로 볼 수 있음 (가볍게 field/column 이라고 봐도 무방함)

> 존 튜키라는 미국 통계학자가 제안한 분석 방법론 <br>
> 기존 통계학이 가설을 세우고, 가설을 검정하는 방법론에 치우쳐, 데이터 본래의 정보를 파악하기 어려우므로, 본연의 데이터 탐색에 집중하자는 방법론

### 2. 실제 데이터로 pandas 라이브러리와 탐색적 분석 과정 익히기

#### 코로나 바이러스 데이터와 함께 pandas 라이브러리 익히기
- COVID-19-master 폴더 확인
  - 원본 데이터: https://github.com/CSSEGISandData/COVID-19

### pandas 라이브러리로 csv 파일 읽기
- csv 파일을 pandas dataframe 으로 읽기 위해 read_csv() 함수를 사용함
- csv 구분자는 quotechar=구분자 옵션을 넣어서 구분자가 다른 경우도 읽기 가능

```
doc = pd.read_csv("USvideos.csv", encoding='utf-8-sig', quotechar=',')
```

- 참고: 에러 나는 데이터는 항상 있을 수 있음, 해당 데이터는 생략하는 것이 일반적임
  - 관련 옵션: on_bad_lines 옵션(잘못된 행을 만났을 때 수행할 작업을 지정)<br>
    - 'error' : 잘못된 행을 만나면 에러 발생하고 중단<br>
    - 'warn' : 잘못된 행을 만나면 경고를 표시하고 해당 행을 건너뜀<br>
    - 'skip' : 잘못된 행을 만나면 에러나 경고를 표시하지 않고 건너뜀<br>
    - 옵션 미사용 시 기본 동작: 만약 on_bad_lines 옵션을 명시적으로 지정하지 않는다면, 기본값은 'error' 입니다. 즉, CSV 파일을 읽는 도중 잘못된 행을 만나면 에러를 발생시키고 전체 읽기 작업을 중단하게 됩니다.
      
```
doc = pd.read_csv("USvideos.csv", encoding='utf-8-sig', on_bad_lines='skip')
```


```python
import pandas as pd
doc = pd.read_csv("COVID-19-master/csse_covid_19_data/csse_covid_19_daily_reports/04-01-2020.csv", encoding='utf-8-sig')
```

### 엑셀 파일 읽기

#### 기본 사용법 (첫 번째 시트)
```python
pd.read_excel("파일명.xlsx")
```

#### 특정 시트 지정
```python
pd.read_excel("파일명.xlsx", sheet_name="시트명")
```

## 3. 탐색적 데이터 분석 단계별 실습

### 단계 1: 데이터 출처와 주제 이해

**COVID-19 데이터 특징**
- 국가별 코로나바이러스 일일 현황 자료
- Johns Hopkins University에서 작성
- PDF 공식 문서와 웹 크롤링을 통해 CSV로 구성

### 단계 2: 데이터 크기 확인

#### 데이터 일부 확인
```python
# 처음 5개 행 확인 (기본값)
doc.head()

# 처음 10개 행 확인
doc.head(n=10)

# 마지막 5개 행 확인
doc.tail()

# 마지막 10개 행 확인
doc.tail(n=10)
```

#### 데이터 정보 확인
```python
# 행과 열 개수 확인
doc.shape

# 컬럼별 데이터 타입과 실제 데이터 개수 확인
doc.info()
```

### 단계 3: 데이터 구성 요소(feature) 속성 확인

#### 컬럼 이해
```python
# 모든 컬럼명 확인
doc.columns
```

**주요 컬럼 설명**
- `Country_Region`: 국가
- `Lat/Long`: 위도/경도
- `Confirmed`: 확진자 수
- `Deaths`: 사망자 수
- `Recovered`: 회복자 수
- `Active`: 활성 확진자 수 (확진자 - 사망자 - 회복자)

#### 숫자 데이터 기본 통계
```python
# 기본 통계치 확인
doc.describe()
```

**통계치 설명**
- `count`: 데이터 개수
- `mean`: 평균
- `std`: 표준편차
- `min/max`: 최솟값/최댓값
- `25%/50%/75%`: 사분위수 (1사분위/중위수/3사분위)

#### 상관관계 분석

##### 상관계수 계산
```python
# 숫자 컬럼에 대한 피어슨 상관계수 계산
doc.corr(numeric_only=True)
```

**피어슨 상관계수 해석**
- `+1`에 가까움: 강한 양의 상관관계 (한 변수 증가 → 다른 변수 증가)
- `0`에 가까움: 상관관계 없음
- `-1`에 가까움: 강한 음의 상관관계 (한 변수 증가 → 다른 변수 감소)

##### 피어슨 상관계수 공식
두 변수 X, Y에 대해:

$$
r_{XY} = \frac{\sum_{i}^{n} (X_{i} - \bar{X}) \cdot (Y_{i} - \bar{Y})}{\sqrt{\sum_{i}^{n} (X_{i} - \bar{X})^2} \cdot \sqrt{\sum_{i}^{n} (Y_{i} - \bar{Y})^2}}
$$

여기서 $\bar{X}$, $\bar{Y}$는 각각 X, Y의 평균이다.

## 4. 데이터 시각화

### 시각화의 중요성
방대한 데이터를 숫자로만 파악하는 것보다 시각화를 통해 보다 명확하고 직관적으로 이해할 수 있다.

### 주요 시각화 라이브러리

#### matplotlib
- 파이썬 기본 시각화 라이브러리
- 가장 기초적이고 널리 사용되는 라이브러리

#### seaborn
- matplotlib 기반으로 개발
- 다양한 통계 차트와 색상 테마 제공
- matplotlib의 한계를 보완한 고급 시각화 기능

#### plotly
- 최신 인터랙티브 데이터 시각화 라이브러리
- 웹 기반 상호작용 그래프 제공
- 확대/축소, 마우스 오버 등 인터랙티브 기능 기본 제공

### plotly를 활용한 상관계수 시각화

#### 라이브러리 설치 및 임포트
```python
# 설치 (필요한 경우)
!pip install plotly

# 임포트
import plotly.express as px
```

#### 상관계수 히트맵 생성
```python
# 상관계수 행렬 계산
corr = doc.corr(numeric_only=True)

# 히트맵 시각화
fig = px.imshow(
    corr,                           # 상관계수 데이터
    text_auto=True,                 # 셀에 상관계수 값 표시
    color_continuous_scale='Blues', # 색상 스케일
    zmin=-1, zmax=1,                # 상관계수 범위
    width=800, height=600           # 그래프 크기
)
fig.show()
```

#### 주피터 노트북 설정 (그래프가 표시되지 않을 경우)
```python
import plotly.graph_objects as go
import plotly.offline as pyo
pyo.init_notebook_mode()
```

## 요약

탐색적 데이터 분석은 데이터 분석의 첫 단계로, 다음 과정을 통해 체계적으로 접근한다:

1. **데이터 출처와 주제 파악** → 데이터의 신뢰성과 맥락 이해
2. **데이터 크기 확인** → 분석 가능한 데이터 양과 구조 파악
3. **feature 속성 분석** → 각 변수의 특성과 상호 관계 이해

pandas와 plotly를 활용하면 효율적이고 직관적인 데이터 탐색이 가능하며, 이를 통해 후속 분석의 방향성을 설정할 수 있다.