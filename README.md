# MQL 데이터 기반 B2B 영업기회 창출 예측 모델 개발  
> 2024.01.02 ~ 2024.02.26 

**LG Aimers 4기 해커톤 프로젝트**  


📄 [LG AImers 해커톤 수료증](certificates/lg_aimers_hackathon_certificate.pdf)  
📄 [LG AImers AI 실습 교육 이수증](certificates/lg_aimers_ai_practice_completion.pdf)

## 1. 프로젝트 개요

B2B Marketing은 **기업 고객을 대상으로 영업 기회를 발굴하고 지속적인 매출을 창출하는 것**을 목표로 한다.  

MQL(Marketing Qualified Lead) 고객을 기반으로 영업사원을 할당하고, **최종 구매로 이어지게 하는 개인화 마케팅 활동**이 진행되는데, **할당 가능한 영업 인력은 한정적**이다.

따라서 본 프로젝트는 MQL 고객 정보를 활용해 **영업 전환 성공 여부를 예측하는 AI 모델**을 개발하고, 이를 통해 **영업사원을 효율적으로 배치**하는 것이 목적이다.

> 🏅 **해커톤 결과**: 844명 중 상위 6.6%의 성과 달성  


### 사용 기술 스택
[![My Skills](https://skillicons.dev/icons?i=py,sklearn,)](https://skillicons.dev)

---

## 2. 데이터셋 설명  

| bant_submit | customer_country        | business_unit | com_reg_ver_win_rate | customer_idx | customer_type        | enterprise | historical_existing_cnt | id_strategic_ver | it_strategic_ver | ...  | response_corporate | expected_timeline     | ver_cus | ver_pro | ver_win_rate_x | ver_win_ratio_per_bu | business_area       | business_subarea | lead_owner | is_converted |
|-------------|-------------------------|---------------|---------------------|--------------|----------------------|------------|------------------------|------------------|------------------|------|--------------------|-----------------------|---------|---------|----------------|-----------------------|---------------------|------------------|------------|--------------|
| 1.0         | /Quezon City/Philippines | AS            | 0.066667            | 32160        | End-Customer          | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEPH              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Engineering      | 0          | True         |
| 1.0         | /PH-00/Philippines       | AS            | 0.066667            | 23122        | End-Customer          | Enterprise | 12.0                   | NaN              | NaN              | ...  | LGEPH              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Advertising      | 1          | True         |
| 1.0         | /Kolkata /India          | AS            | 0.088889            | 1755         | End-Customer          | Enterprise | 144.0                  | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Construction     | 2          | True         |
| 1.0         | /Bhubaneswar/India       | AS            | 0.088889            | 4919         | End-Customer          | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | IT/Software      | 3          | True         |
| 1.0         | /Hyderabad/India         | AS            | 0.088889            | 17126        | Specifier/ Influencer | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 0       | 0       | 0.003079       | 0.026846              | corporate / office   | NaN              | 4          | True         |

- 전체 약 **60,000건**의 MQL 데이터로 구성  
- 주요 컬럼:
  - **고객 정보**
    - `customer_country`: 고객 위치
    - `customer_type`: 고객 유형 (End-Customer, Influencer 등)
    - `enterprise`: 기업 고객 여부
    - `business_area`, `business_subarea`: 산업군 및 세부 산업군

  - **제품 및 전략 정보**
    - `business_unit`: 관심 제품군
    - `com_reg_ver_win_rate`, `ver_win_rate_x`, `ver_win_ratio_per_bu`: 과거 유사 사례의 수주 성공률 관련 변수
    - `id_strategic_ver`, `it_strategic_ver`: 전략적 제품 관련 여부

  - **문의 정보**
    - `lead_owner`: 리드 담당자
    - `expected_timeline`: 예상 구매 시기
    - `response_corporate`: 담당 법인
    - `lead_description`: 고객 문의 텍스트  
  
  - **기타**
    - `customer_idx`: 고객 고유 식별자
    - `historical_existing_cnt`: 과거 거래 수 등 히스토리
  
  - **타겟 변수**
    - `is_converted`  
      - `True`: 영업 전환 성공  
      - `False`: 영업 전환 실패  

---

## 3. EDA (탐색적 데이터 분석)

#### **1. Lead Description 텍스트 길이와 전환율 관계 분석**  
고객이 작성한 `lead_description` 텍스트의 총 길이를 나타내는 `lead_desc_length` 변수를 생성하였다.  
`lead_desc_length`가 길수록 영업 전환(`is_converted`) 성공률이 높을 것이라는 가설을 세우고 시각화한 결과,  
실제로 전환율이 길이가 긴 리드에서 더 높게 나타나는 경향을 확인하였다.  
![lead_desc_length 전환율](images/eda1.png)


#### **2. 고객 문의 유형별 전환율 분석**  
고객의 문의 유형에 따른 전환율을 분석한 결과, `quotation/purchase` 카테고리에서 전환율이 특히 높게 나타났다.  
이를 통해 기존 범주형 변수의 각 카테고리별 전환율을 파생변수로 추가하면 예측 모델 성능 향상에 도움이 될 것이라는 인사이트를 얻었다.  
![문의 유형별 전환율](images/eda2.png)


#### **3. ‘lead_desc_length’ 변수 분포 및 변환**  
`lead_desc_length`의 분포가 오른쪽으로 치우쳐 있는(skewed) 것을 확인하였고, 이를 보정하기 위해 Box-Cox 변환을 수행하였다.  
변환 후에는 분포가 보다 정규분포에 가까워졌으며, 모델 학습에 더 적합한 형태가 되었다.
- 변환 전 분포  
  ![Box-Cox 이전](images/eda3.png)

- Box-Cox 변환 후 분포  
  ![Box-Cox 이후](images/eda3-1.png)


#### **4. 파생변수와 타겟 변수 간 상관관계 분석**  
각 범주형 변수의 카테고리별 전환율을 계산하여 파생변수로 생성한 뒤, 이 변수들과 타겟 변수 `is_converted` 간의 상관관계를 히트맵으로 시각화하였다.  
특히 `customer_idx_converted_rate` 변수가 `is_converted`와 0.97의 매우 강한 양의 상관관계를 보였으며,  
이는 고객별 과거 전환률이 현 전환 예측에 매우 중요한 변수임을 시사하였다.
![상관관계 히트맵](images/eda4.png)

---

## 4. 데이터 전처리 (Data Preprocessing)
모델 성능 향상 및 과적합 방지를 위해 아래와 같이 데이터를 전처리하였다.

- 불필요한 컬럼 제거    
- 결측값 처리  
- `lead_desc_length` 변수 Box-Cox 변환  
- **Google Maps API**를 활용한 국가 정보 보완 및 대륙 파생변수 생성  
  - 일부 국가명이 잘못 입력되어 있거나 누락된 경우를 보완하기 위해 Google Maps API를 활용하여 `customer_country` 정보 정제
  - 이후 국가 정보를 기반으로 대륙 파생변수를 생성하여 지역적 특성 반영
  ```python
  import googlemaps
  gmaps = googlemaps.Client(key='')
  
  # 주요 국가 매칭
  for country in primary_countries:
      df.loc[df['customer_country'].str.contains(country, na=False), 'customer_country'] = country
  
  # 나머지는 Geocoding API 활용
  geocode_result = gmaps.geocode(loc)
  if geocode_result:
      country_name = geocode_result[0]['address_components'][0]['long_name']
      df.at[index, 'customer_country'] = country_name
  
  # 대륙 파생 변수
  def get_continent(country):
      if country in asia_list:
          return 'Asia'
      elif country in africa_list:
          return 'Africa'
      # ...
      return 'Others'
  
  df['customer_continent'] = df['customer_country'].apply(get_continent)
  ```

- 정제되지 않은 컬럼들은 **유사한 카테고리끼리 그룹화**하여 범주화 처리 

- `converted_rate` **파생변수 생성**   
  - 범주형 컬럼에 대해 각 카테고리별 전환율을 계산하여 `~_converted_rate` 변수들을 생성 
  - 이 변수들은 고객군별 전환 패턴을 반영하므로 모델에 중요한 신호로 작용


- **LightGBM 모델**을 활용한 결측값 처리
  - 대상 컬럼: ‘ver_win_rate_x’, ‘ver_win_ratio_per_bu’ (분류), ‘com_reg_ver_win_rate’ (회귀)
  - 처리 절차:
      1. `ver_win_rate_x`의 결측값을 포함한 train 데이터를 이용해 분류 모델 학습 후, 결측값이 있는 데이터를 예측하여 새로운 train 데이터셋 생성
      2. 위에서 생성한 train 데이터를 사용해 test 데이터의 `ver_win_rate_x` 결측값을 예측하고, 새로운 test 데이터셋 생성
      3. 동일한 방법으로 ‘ver_win_ratio_per_bu’와 ‘com_reg_ver_win_rate’ 컬럼의 결측값도 채움


- **특정 범주 파생변수 생성**
  - 범주형 컬럼별 카테고리별 전환율 계산
  - 전환율이 높은 범주에 가중치를 부여하는 파생변수 생성
  - 최종적으로 84개의 컬럼으로 확장됨

- 라벨인코딩
  - 데이터셋 내 모든 범주형 변수에 대해 Label Encoding 수행
  - 테스트 데이터에 train에 없는 새로운 범주가 있을 경우, ‘other’ 범주로 처리하여 **인코딩 안정성 확보**

--- 

## 5. 모델링 및 성능 평가

- **모델 성능 비교**
  - 단일 모델:
    - LightGBM: F1-score `0.67`
    - AutoGluon: F1-score `0.68`
  - 앙상블 모델:
    - Soft Voting 앙상블 적용 후 성능이 F1-score `0.72`로 크게 향상됨
    - **임계값 조정까지 포함한 최종 모델 성능은 F1-score 기준 약 `0.76`에 도달함**

- **상관관계 기반 변수 제거**
  - EDA 결과 `customer_idx_converted_rate` 컬럼과 타겟 변수 사이의 상관계수가 0.97로 매우 높았음
  - 해당 컬럼을 제거한 후 모델 성능이 개선되어, 과적합 가능성을 줄이고 일반화 성능을 향상시킴

- **하이퍼파라미터 튜닝**
  - `Bayesian Optimization` 기법을 통해 각 모델별 최적의 하이퍼파라미터 도출
  - LightGBM, XGBoost, CatBoost 모델에 각각 적용함

- **Soft Voting 앙상블**
  - `VotingClassifier`를 사용하여 LightGBM, XGBoost, CatBoost를 soft voting 방식으로 결합
  - 개별 모델이 예측한 **probability 평균을 기준으로 최종 class 결정**
  - 각 모델 가중치는 동일하게 `[1, 1, 1]`로 설정

- **클래스 불균형 처리**
  - `class_weight` 설정을 통해 불균형 보정:
    - LightGBM: `{0: 1, 1: 2}`
    - XGBoost: `scale_pos_weight=2`
    - CatBoost: `class_weights = [1, 2.5]`

  - 추가적으로, **예측 확률 기반 임계값(threshold)을 조정**하여 민감도 향상 도모
    - 기본적으로 0.5였던 threshold를 **0.058**로 낮춤
    - 이를 통해 불균형 클래스(양성 클래스) 탐지율을 증가시킴

- **모델 성능 평가**  
  - 최종 모델은 검증 데이터셋을 기준으로 Accuracy, Precision, Recall, F1-score, AUC 등의 지표를 활용하여 성능을 평가함
--- 

## 6. 결과 요약 및 회고
- **리더보드 성과**
<table>
  <tr>
    <td align="center">
      <b>🏅 Public LB</b><br/>
      전체 844명 중 <b>38등</b> (상위 4.5%)<br/>
      <img src="images/public.png" alt="public score" width="500"/>
    </td>
    <td align="center">
      <b>🏅 Private LB</b><br/>
      전체 844명 중 <b>56등</b> (상위 6.6%)<br/>
      <img src="images/private.png" alt="private score" width="500"/>
    </td>
  </tr>
</table>  

- 주요 학습 내용
  - **전처리 함수 모듈화**: Box-Cox 변환, 결측값 처리, 범주화 등의 작업을 각각 함수로 분리하여 모듈화함으로써 재사용성과 유지보수성 향상
  - **의미 있는 파생변수 생성 경험**: `is_converted`와 관련된 범주별 전환율 기반 파생변수를 생성하여 모델 성능 향상 및 도메인 기반 변수 설계 중요성 체감
  - **도메인 지식의 중요성 인식**: MQL, BANT, Lead Description 등의 용어 학습을 통해 데이터 해석 및 모델 설계의 이해도 제고
  - **효율적인 하이퍼파라미터 튜닝**: 베이지안 최적화를 통한 탐색 효율 개선과 성능 향상
  - **시드 고정 반복 실험**: 500회 반복 실험으로 모델 성능의 안정성 검증 및 결과 신뢰도 확보
  - **정교한 결측값 처리 방식**: 단순 삭제나 일괄 치환이 아닌, 머신러닝 기반 예측으로 결측값을 대체하여 데이터 손실 최소화 및 성능 저하 방지

- 기대효과
  - 해당 모델을 통해 기업이 잠재 고객을 효과적으로 식별하여 맞춤형 마케팅 전략을 수립하는 데 기여할 수 있음
  - 고객 행동 예측력을 높여 교차 판매 및 업셀링 전략 수립에 활용 가능
  - 마케팅 비용 최적화와 전환율 향상을 동시에 달성할 수 있음
  - 고객 만족도 향상 및 리스크 관리 체계 강화에 도움을 제공함

- 아쉬운 점
  - 데이터 불균형이 심해 모델의 민감도와 특이도 균형 맞추는 데 한계가 있었음  
  - 일부 중요한 도메인 변수에 대한 추가적인 이해와 데이터 확보가 부족했음
  - 텍스트 데이터에 포함된 고객 문의 내용이나 피드백을 효과적으로 분석하지 못해, 잠재적인 의미 있는 패턴과 인사이트 발굴에 한계가 있었음





