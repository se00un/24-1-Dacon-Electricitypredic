# 24-1-Dacon-Electricitypredic

자세한 분석 과정은 블로그에 정리되어 있습니다.
👉 자세한 분석 내용: https://blog.naver.com/develop0420/224081739201

대회 설명: DACON 건물 전력 사용량 예측 AI 경진대회
건물 정보와 시공간 정보를 활용하여 특정 시점의 전력 사용량을 예측하는 AI 모델을 개발하는 대회

📅 프로젝트 기간: 2024.03.26 – 2024.04.08

### Project Overview
건물의 특성 정보와 날씨 및 시간 정보를 활용하여 특정 시점의 전력 사용량을 예측하는 회귀 모델을 구축하는 프로젝트이다.데이터 분석 파이프라인을 처음부터 경험하며 다음 과정을 수행하였다.
- Feature Engineering
- Exploratory Data Analysis (EDA)
- Clustering을 통한 전력 소비 패턴 분석
- 결측치 처리
- AutoML 기반 모델 비교 및 성능 평가


### Feature Engineering
전력 사용량에 영향을 줄 수 있는 시간 및 날씨 관련 파생 변수를 생성하였다.

*Time Feature*
- holiday: 주말 여부 변수 생성
- sin_time, cos_time: 시간의 주기성(cyclical feature) 반영

*Weather Feature*
- THI (Temperature Humidity Index)
- 기온과 습도를 결합한 불쾌지수
- PT (Perceived Temperature)
- 풍속을 반영한 체감온도


### Exploratory Data Analysis
1. 건물 유형별 전력 사용 요인 분석
건물 유형이 전력 사용량에 영향을 줄 것이라는 가설을 세우고 변수 간 상관관계를 분석하였다.
주요 결과:
대부분의 건물 유형에서 습도와 THI가 전력 사용량과 높은 상관관계를 보였다.
체감온도(PT)보다 **기온(C)**이 전력 사용량과 더 높은 관계를 보였다.
대학교 건물 유형은 모든 건물에 태양광 설비가 존재하는 특징을 확인하였다.

2. Feature Importance
XGBoost 모델을 활용하여 변수 중요도를 분석하였다.이를 통해 전력 사용량에 가장 큰 영향을 미치는 요소는 건물 규모라는 것을 확인하였다.

3. Power Consumption Pattern Clustering
전력 소비 패턴을 분석하기 위해 클러스터링을 수행하였다. 요일과 시간대별 전력 소비 패턴을 기반으로 4개의 클러스터가 형성되었다. 각 클러스터는 서로 다른 건물 유형 특성을 보였다.

예시:
Cluster 0
백화점, 아울렛, 호텔
주말 사용량 높음
Cluster 1
데이터센터, 대학교
주중 사용량 높음
Cluster 2
공공기관, 병원
전형적인 업무시간 패턴
Cluster 3
아파트, 상업시설
특정 시간대 사용량 급증

이를 통해 건물 유형에 따라 전력 소비 패턴이 다르게 나타남을 확인하였다.


### Missing Value Handling
결측치는 변수 특성에 따라 다음과 같이 처리하였다.
- 태양광용량, ESS용량, PCS용량 → 0으로 대체
- 강수량 → 0으로 대체
- 풍속 및 습도 → 동일 건물 + 동일 월 + 동일 시간 기준 평균값으로 대체


### Model
PyCaret을 활용한 **AutoML 기반 모델 비교**를 진행하였다.

| Model | RMSE | R² | Notes |
|------|------|------|------|
| XGBoost | 348.17 | 0.98 | 높은 R²로 인해 overfitting 가능성 |
| LightGBM | 243.68 | - | K-fold 교차검증 평균 RMSE |
| CatBoost | - | - | LightGBM보다 낮은 RMSE로 가장 안정적인 성능 |
| KNN Regressor | 478.66 | 0.962 | RMSE는 높지만 R²는 높은 편 |

모델 성능 비교 결과 CatBoost가 가장 안정적인 성능을 보였다.


### What I Learned
이 프로젝트는 첫 데이터 분석 프로젝트로 데이터 분석의 전체 파이프라인을 경험할 수 있었다.
특히, 
- Feature Engineering의 중요성
- 결측치 처리 방식에 따른 분석 결과 변화
- 모델이 어떤 변수를 중요하게 사용하는지 확인하는 Feature Importance 개념
- 데이터 분석 과정에서 분석가의 의사결정이 매우 중요하다는 점
- 또한 데이터 분석은 단순히 모델을 학습시키는 것이 아니라 데이터를 어떻게 전처리할지, 어떤 변수를 생성할지, 어떤 모델을 사용할지
등 판단이 지속적으로 필요한 과정이라는 것을 깨닫게 되었다.
