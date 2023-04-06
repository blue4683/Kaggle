# KAGGLE 경진대회

## 개요
- 기간 : 2022.07.29 ~ 2022.08.05
- 언어 : `Python`
- 제출 : [사이트](https://www.kaggle.com/competitions/regression220718/overview)

## 진행
### 각 컬럼별 고유값 개수 확인
```
for col in df.columns:
    print(col, df[col].nunique(), '', sep = '\n')

> ID # 각 집의 고유번호
33656

ADDRESS # 집주소(범주형 데이터)
33566

SUBURB # 동네 이름(범주형 데이터)
321

PRICE # 가격(타겟값)
1806

BEDROOMS # 침실의 개수
10

BATHROOMS # 욕실의 개수
8

GARAGE # 차고의 수
25

LAND_AREA # 토지 면적
4372

FLOOR_AREA # 건물 면적
528

BUILD_YEAR # 건축 년도
124

CBD_DIST # Central business district까지의 거리
595

NEAREST_STN # 근처 역 정보(범주형 데이터)
68

NEAREST_STN_DIST # 근처 역까지 거리
1189

DATE_SOLD # 판매된 날짜(범주형 데이터 ex: 09-2015\r)
350

POSTCODE # 우편번호
114

LATITUDE # 위도
29707

LONGITUDE # 경도
28557

NEAREST_SCH # 근교의 학교(범주형 데이터)
160

NEAREST_SCH_DIST # 근교의 학교까지의 거리
33318

NEAREST_SCH_RANK # 근교의 학교의 랭킹
103
```

### 데이터 전처리 및 모델링 시도

1. 실패(점수 하락)
- NEAREST_STN 원핫인코딩(처음에 테스트 데이터와 같이 fit시켜서 테스트 데이터에만 있는 STN으로 결과 오류)
    - 재시도했으나 점수 하락
- NEAREST_SCH_RANK 학교별 집값 평균을 통해 대학 랭킹 재산정
- GARAGE 결측치가 있는 행 지우기
- GARAGE 결측치를 2로 채우기(차고가 2개인 집이 2만채 이상)
- POSTCODE 이상치 제거
    - POSTCODE는 우편번호로 이 특징으로 집들의 지역을 유추할 수 있어 점수가 하락한 것으로 보임(이상치가 아님)
- LAND_AREA 이상치 실제 데이터를 기반으로 재입력(999999와 같은 쓰레기 값이 존재)
- NEAREST_STN이 Midland Station인지 아닌지의 특징 생성
    - 생각보다 Midland Station 근처 집이 많아서 시도했으나 의미 없었음
- GARAGE 이상치 제거
    - 다른 집들에 비해 GARAGE가 많아서 이상치라고 생각했으나 주차장인 것으로 추측
- (BEDROOMS + BATHROOMS + GARAGE) / FLOOR_AREA 특징 생성
    - 면적당 방 개수로 외국인들은 방의 개수가 많은 집을 선호하는 기사를 통해 예측
- NEAREST_SCH_DIST가 10.5보다 큰 값들을 10.5로 설정
    - 10.5 이상의 값들이 많지 않아 최빈값으로 설정했으나 의미는 없었음
- NEAREST_SCH_RANK 실제 데이터를 탐색하여 재산정
    - 랭크가 없는 학교도 많고, 실제와 많이 달라 점수 하락


2. 성공(점수 상승 요인)
- LAND_AREA가 1200보다 작으면 SMLLAND 크거나 같으면 BIGLAND로 특징 생성
    - Public 점수에서는 감소했으나 Private 점수에서 상승함
- GARAGE의 결측치만 0으로 채우고 나머지 결측치들은 평균값으로 대치
- DATE_SOLD를 SOLD_YEAR와 SOLD_MONTH로 나눈 후에 SOLD_YEAR만을 사용
    - 연도에 따라 시세가 달라졌을 것이라 판단
- PRESOLD = SOLD_YEAR < BUILD_YEAR 특징 생성
    - 지어지기 전에 팔린 건물들만의 특징이 있을것이라 판단
- CatBoostRegressor 모델 사용
    - 기본적으로 좋은 성능을 보인다고 하여 사용
- Polynomial 사용
    - 특징을 조합하여 새로운 특징을 생성
    - numerical features를 직접 제곱, 세제곱하여 데이터 프레임에 concat했을 때 성능이 좋아짐
    - Polynomial 사용시 다른 특징들의 곱들로 특징을 생성해내는데 이 부분에서 과대적합이 발생한 것으로 보임

## 참고
[탐색적 데이터 분석(EDA)](https://www.kaggle.com/code/shtrausslearning/eda-perth-housing-price-prediction) <br/>
[모델 및 특징 생성 참고](https://www.kaggle.com/code/shtrausslearning/perth-housing-price-prediction-models?scriptVersionId=117296536)