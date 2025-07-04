# [공모전] 데이터 기반으로 '사용자 중심' 전기차 충전 인프라 입지 선정 

## 0. 대회 개요

- 주최/주관 : 한국문화관광연구원

- 기간 : 2023.03.20 ~ 2023.05.04

- 주제 : **광진구 도시 현안 관련 자유주제**
    - 교통/주차, 도시기반시설, 치안/방범, 일자리 분야 등

- 평가 : (1차심사) 5개 항목 총점 100점, 3배수 선정 -> (2차심사) 분석과제 관련 부서 정책 반영 가능 여부 검토, 6개팀 선정 -> (본선심사) 발표 평가 후 순위 결정 및 시상

- 참가 : 3인 1팀 구성으로 대회 참가

## 1. 프로젝트 설명

- **프로젝트 기간**: 2023.04.01 ~ 2023.05.04

- **프로젝트 개요**:

    - (배경) 친환경 추세에 따라 전기차 수요와 공급의 폭발적인 성장이 예상됨
        - [ 정부 ] '2050 탄소중립 시나리오'에 따른 전기차 `수요`의 빠른 증가

        - [ 기업 ] 현대차의 '탄소중립 로드맵' 실행에 따른 전기차 `보급`의 가속화

        > => 광진구 내 전기차 등록 대수의 증가 예상 (\`22->\`25 사이 165%+ 예상)

    - (문제) 광진구의 전기차 충전 인프라 수는 더딘 상황이고 전기차 이용자 증가 추이를 고려했을 때, 더 많은 인프라가 요구됨
    
        - (현황) 서울시 구별 `전기차 1대당 충전기 비율` 비교 결과, 광진구는 전체 25개구 중 17위 기록 (중하위 수준) 
    
    - (해결) 데이터 기반 최적의 전기차 충전소 입지를 도출하여, 2025년까지 3000천기 이상 확보를 계획한 광진구의 정책 추진을 지원 
    
    - (기대효과) `사용자 중심 충전 인프라`로 충전기 사용자인 국민들의 만족도 증진 및 정책 효과성 극대화
    
- **분석 툴** : Python
    - 지리정보: QGIS, Folium
    - ML 모델링 : Scikit-learn, XGBoost, LightGBM, Catboost
    - 데이터 시각화: Matplotlib, Seaborn

- **데이터 소스** :
    - 국토정보플랫폼 : <인구통계지도>
    - 서울열린데이터광장 : <버스승하차인원정보>, <지하철승하차인원정보>
    - 공공데이터포털 : <서울시 광진구 공공시설 기본정보>
    - 소상공인시장진흥공단 : <상가(상권) 정보>
    - 국토정보플랫폼 : <건물통계지도>
    - 환경부 : <전기차충전소모니터링>

## **2. 주요 역할 및 기여** : 데이터 수집 분석 및 ML 모델링 담당

### **(#1) 지리정보 데이터 처리 통해 머신러닝 학습이 가능한 데이터셋 확보**

- **[문제]** 전기차 충전소 입지 예측에 설명변수로 관여될 수 있는 공공데이터를 수집하였지만, 이를 적절한 학습 데이터로 가공이 필요한 상황

- **[해결]** GQIS을 통해 100m*100m 그리드에 27개의 각 X_feature 정보와 Y_target(충전기수)를 매핑하여 1933개 그리드를 관측치로 하는 학습 데이터 구축

    - 상권 데이터 내 상가 개수가 0인 지역들을 제거하여 충전기 설치가 불가하거나 불필요한 지역 제외 (1933개->1095개)

    - Stratified K-Fold(K=3)으로 OOF 데이터에 대한 예측을 진행하는 형태로 Train Set과 Test Set 구성

    - Oversampling 통해 Positive 개수를 증대하여 클래스 불균형 문제 해소 (12%->50%)

- **[성과]** 머신러닝 모델링이 가능한 데이터셋 구축에 성공 -> 398개의 1차 후보군 도출 가능

### **(#2) ML 모델링을 역으로 활용하여 입지 후보군 도출**

- **[문제]** 일반적인 ML 모델 학습의 제약 발견

    - 기존 ML 분류 모델은 충전소가 이미 설치된 지역을 중심으로 패턴을 학습하기 때문에, 신규 충전소가 필요한 '미설치 지역'에 대한 후보지 추천에 한계가 존재

    - 특히, 신규 입지로 검토해야 할 충전소 미설치 지역을 모델이 자동으로 '부정(0)'으로만 분류하는 구조적 문제 발생

    - 이에 따라 신규 충전소 입지 발굴 과정에서 모델 예측 결과를 직접적으로 활용하기 어려운 상황

- **[해결]** Confusion Matrix의 FP 부분을 신규 후보지로 도출

    - 모델의 False Positive(FP) 영역을 역으로 활용하는 전략을 수립 (기 연구된 프로젝트 레퍼런스 참고)

    - 충전소가 없지만 모델이 '충전소가 있을 가능성이 높다'고 예측한 FP 영역을 유망한 입지 후보군으로 재해석
    
    - LightGBM, XGBoost, RandomForest, CatBoost 등 4개 모델을 비교 분석하여, accuracy 및 f1-score에서 우수한 성능을 보인 LightGBM을 최종 선정

    - 모델의 예측 확률 P(y=1)에 대해 Threshold를 최적화하여 Accuracy 손실과 FN 비율을 최소화하면서도 FP를 적정 수준 확보하는 방향으로 모델링을 수행

- **[성과]** 기 설치된 충전소 입지 지역의 패턴 학습이 반영된 입지 1차 후보지 도출

    - 총 1,095개의 분석 대상 그리드 중 398개의 잠재적 입지 후보지 도출

    - 이후 실증적 데이터 분석과 결합하여 후보지 축소 진행

## 3. 주요 성과

- **데이터를 기반으로 최적의 입지 후보군 도출**

    - 모델링의 예측 확률(probability)를 통해 `타당성` 확보
        - 예측 확률이 높을수록 데이터 상 가장 유력한 후보 지역임을 파악 가능
        - 사람이 정성적으로 파악할 수 없는 데이터의 패턴이 반영된 결과

    - 급속과 완속 입지를 구분함으로서 `현실성` 확보 
        > - (급속) 상업시설이 많은 지역(주간 인구 > 야간 인구)은 급속충전기 후보지로 분류
        >    - 급속 충전기의 경우 주로 주간 시간대에 사용이 집중되기 때문
        
        > - (완속) 주거 시설이 많은 지역과 주요 핵심 지역은 완속충전기 후보지로 분류
        >    - 완속 충전기의 경우 야간 시간대에 사용이 집중되기 때문

    -  최종적으로 급속충전기 후보 33곳, 완속충전기 후보 55곳 도출

## 4. 프로젝트 회고

- **GIS 분석 방법론 및 툴 활용의 중요성 인식**

    - 공공데이터 분석에서 지리정보(GIS)의 비중이 매우 높음을 체감

    - 파이썬만으로는 공간 데이터를 다루는 데 한계가 있어 QGIS와 같은 전문 GIS 툴의 필요성을 실감

    - 향후 분석에서도 GIS 기반 데이터 처리 및 시각화 역량 강화가 필요함을 느낌

- **분석의 단순화 필요성 인식**

    - 전기차 충전소 입지 선정 과정을 ① 모델링 기반 1차 후보 도출 → ② 실증 자료 기반 2차 후보군 정제 → ③ 최종 후보 선정의 3단계로 구조화

    - 분석 구조는 체계적이었지만, 분석 흐름이 다소 복잡하게 느껴졌음

    - 데이터 분석 결과가 정책 결정 및 비전문가 대상 보고서로 활용되려면, 분석 결과뿐만 아니라 과정 역시 직관적이고 명료해야 함을 깨달음

    - 향후에는 누구나 이해할 수 있는 단순명료한 분석 설계의 중요성을 반영할 계획

## 5. 프로젝트 결과물

- gwangjingu 디렉토리
- [최종 발표 자료](gwangjingu\[알파브레인_제출본]사용자중심전기차충전기입지분석결과서.pdf)