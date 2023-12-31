# **음원 스트리밍 서비스 이용자의 이탈 여부 예측**

## **요약**
### **① 기본 정보**
* **팀구성 및 기여도 : 4명 / 25%**
* **담당 역할**
  * Pyspark를 이용한 대용량 로그 데이터 전처리 및 샘플링
  * EDA 및 Matplotlib, Seaborn을 이용한 데이터 시각화
  * 이탈 확률 구간별 고객 Segmentation 및 Segment별 LTV 산정

---
### **② 프로젝트 진행 배경**
* 로그 데이터, 결제 데이터 등에 대한 다각도의 EDA를 통해 **서비스 개선 Agenda를 도출**하고자 합니다.
* 고객 이탈 예측 모델을 개발, 이탈율이 높은 고객을 선별해 관리하는 **CRM**을 가능하게 하고자 합니다.

---
### **③ 결과 및 직무에 적용할 점**
* **EDA를 통해 CRM 및 퍼포먼스 마케팅에 활용할 수 있는 서비스 개선 관련 Agenda를 도출했습니다.**
  * 하나의 분석에서도 유관부서들과의 협업점을 찾고, 프로덕트/마케팅에 도움을 드릴 수 있습니다.
* **고객의 이탈율을 기반으로 Segmentation을 수행하고 Segment별 LTV를 산정하였습니다.**
  * 이탈 확률이 높은 Segment에 대해 개별적인 고객 유지 전략을 수립할 수 있습니다.
  * Segment별로 산정된 LTV를 마케팅 비용 최적화 및 로열티 프로그램에 이용할 수 있습니다.

---
### **④ 주요 액션**
<p align = 'center'><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/38c6461a-e759-4c0d-bb1f-39a34fc32b50' width = 50%></p>

---
## **상세**
### **① 데이터 로드**
* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/2af681946278faa49b00e10620ae7a6aec0137b5/data_load_sampling.ipynb)를 참고해주시기 바랍니다.
* 2017년 Kaggle에서 진행된 [KKBox's Churn Prediction Challenge](https://www.kaggle.com/competitions/kkbox-churn-prediction-challenge/overview/description)에서 제공된 대만 유명 음원 스트리밍 서비스 KKBox 유저 데이터를 사용함
* 데이터셋은 아래와 같이 크게 4가지 유형의 데이터로 구분되어 있음

<table align = "center">
 <tr>
  <td><p align = 'center'>데이터 유형</p></td>
  <td><p align = 'center'>데이터 내용</p></td>
 </tr>
 <tr>
  <td><p align = 'center'>고객 인적정보</p></td>
  <td><p align = 'left'>각 유저의 인구통계학적 정보 (성별, 나이 등)</p></td>     
 </tr>
 <tr>
  <td><p align = 'center'>서비스 사용 기록</p></td>
  <td><p align = 'left'>각 유저의 일별 스트리밍 서비스 사용 기록 (재생 곡수, 시간 등)</p></td>  
 </tr> 
 <tr>
  <td><p align = 'center'>서비스 구매 기록</p></td>
  <td><p align = 'left'>각 유저의 서비스 구매 기록 (지불 금액, 멤버십 유지 기간 등)</p></td>    
 </tr>
 <tr>
  <td><p align = 'center'>이탈 여부</p></td>
  <td><p align = 'left'>각 유저의 서비스 이탈※ 여부</p></td>     
 </tr> 
</table>  

<p align = "center" >※ 본 데이터에서는 멤버십 만료 후 30일 이내에 추가적인 서비스 구매 이력이 없는 유저를 '이탈'했다고 규정함</p>

* 데이터셋의 크기가 29GB에 달하여 데이터를 모두 메모리에 올리는 Pandas를 이용할 수 없었고, 병렬, 분산처리를 위해 Pyspark를 사용함

---

### **② 데이터 샘플링**
* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/2af681946278faa49b00e10620ae7a6aec0137b5/data_load_sampling.ipynb)를 참고해주시기 바랍니다.
* 서비스 사용 기록, 이탈 여부, 구매 기록, 인적정보를 모두 알 수 있는 고객은 전체 중 일부이며, 상당 수의 고객은 1개 이상의 정보가 누락되어 있음
* 각 데이터가 담고 있는 고객 집합을 벤 다이어그램으로 나타내면 다음과 같음
<p align = "center"><img src = "https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/dcc4821b-a0da-47ad-ac49-286d0d52cc93" width = 50% height = 50%></p>

<table align = "center">
 <tr>
  <td><p align = 'center'>집합 번호</p></td>
  <td><p align = 'center'>상세 내용</p></td>
 </tr>
 <tr>
  <td><p align = 'center'>①</p></td>
  <td><p align = 'left'> 이탈 여부, 구매기록은 알 수 있으나, 서비스 사용 기록과 인적 정보는 알 수 없음 (120,726명)</p></td>     
 </tr>
 <tr>
  <td><p align = 'center'>②</p></td>
  <td><p align = 'left'>이탈 여부, 구매 기록, 서비스 사용 기록은 알 수 있으나, 인적 정보는 알 수 없음 (33명) </p></td>  
 </tr> 
 <tr>
  <td><p align = 'center'>③</p></td>
  <td><p align = 'left'>이탈 여부, 구매 기록, 인적 정보는 알 수 있으나, 서비스 사용 기록은 알 수 없음 (6,305명) </p></td>    
 </tr>
 <tr>
  <td><p align = 'center'>④</p></td>
  <td><p align = 'left'>4개 정보를 모두 알 수 있음 (955,126명) </p></td>     
 </tr> 
</table>

* 이탈에 영향을 미치는 특성을 최대한 파악하기 위해, 4개 데이터를 모두 사용할 수 있는 ④번 그룹 (955,126명)에 한하여 프로젝트를 수행함
* 또한, 팀원 개개인의 컴퓨팅 파워(로컬 기준)를 고려하여 ④번 그룹 95만명 중 대략 47만명 가량의 데이터를 샘플링하여 사용함

---

### **③ EDA**

* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/2745414da82c974467b132cad6f9aee320595930/eda.ipynb)를 참고해주시기 바랍니다.
* EDA를 통해 확인한 주요 내용은 다음과 같음
  
  * **최근에 가입한 유저일 수록 이탈율이 높음**
    <p>경쟁 서비스의 대만 진출이 2013년에서 2016년에 집중, 소비자의 선택의 폭이 넓어짐 <br> 애플 뮤직 : <a href = 'https://www.musicbusinessworldwide.com/apple-music-launches-in-taiwan-and-turkey/'>2016년 2월</a> / 유튜브 뮤직 : <a href = 'https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%8A%9C%EB%B8%8C_%EB%AE%A4%EC%A7%81#cite_note-21'>2015년 11월</a> / 스포티파이 : <a href = 'https://thenextweb.com/news/spotify-quietly-launches-in-taiwan-its-fourth-country-in-asia'>2013년 9월</a></p>
    <p><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/01b9e505-f516-4a30-a84b-b648756f0270' width = 70%></p>

  * **가입 경로에 따라 이탈율에 큰 차이를 보임**
    <p>가입 경로가 7인 유저 그룹의 이탈율이 가장 낮았고, 가입경로가 4인 유저의 이탈율이 가장 높았음 <br> 이탈율이 낮은 가입 경로의 홍보 활동을 극대화하는 방향의 퍼포먼스 마케팅이 가능하다고 판단됨</p>
    <p><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/585bf69b-70ce-41cd-b980-a105eaef37ea' width = 70%></p>

  * **결제 방식에 따라 이탈율에 큰 차이를 보임**
    <p>적어도 5,000명 이상이 사용하는 결제 수단 중 38번의 이탈율이 40%에 달하여 높은 수준에 속함 <br> 이탈율이 높은 결제방식의 경우 파악하지 못한 불편 사항이 없는지 파악하는 CRM 마케팅이 필요함</p>
    <p><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/d4f68fc1-24ae-4ed8-ad8d-120311d25191' width = 80%></p>

* 기타 발견 사항은 아래와 같음
  * 인사이트 관련
    
   <table align = 'center'>
    <tr>
        <th>데이터 유형</th>
        <th>EDA 내용</th>
    <tr>
        <td rowspan = '1' valign = 'center' align = 'center'>인적 정보</td>
        <td>10대 후반 ~ 20대 초반의 사용자가 타 연령대에 비해 이탈율이 높음</td>
    </tr>
    <tr>
        <td rowspan = '2' valign = 'center' align = 'center'>사용 기록</td>
        <td>사용 기록이 많은 고객일 수록 이탈율이 낮음</td>
    </tr>
    <tr>
        <!--<td>사용 기록</td>-->
        <td>이탈 유저의 경우 장기간 미접속 경험 비율이 높음</td>
    </tr>
    <tr>
        <td rowspan = '4'  valign = 'center' align = 'center'>구매 기록</td>
        <td>고객이 선택하는 결제 수단에 따라 이탈율의 편차가 큼</td>
    </tr>
    <tr>
        <!--<td>구매 기록</td>-->
        <td>이탈 유저는 구독 자동 갱신 횟수가 적음</td>
    </tr>
    <tr>
        <!--<td>구매 기록</td>-->
        <td>구독 유지 기간이 길수록 이탈율은 낮아짐</td>
    </tr>
    <tr>
        <!--<td>구매 기록</td>-->
        <td>가입 이후 결제 방식을 변경한 고객의 이탈율이 더 높음</td>
    </tr>
   </table>

  * 전처리 관련

   <table align = 'center'>
       <tr>
           <th>데이터 유형</th>
           <th>EDA 내용</th>
       <tr>
           <td valign = 'center' align = 'center'>공통</td>
           <td >이탈 잔존 유저의 구성 비율이 불균형함</td>
       </tr>
       <tr>
           <td valign = 'center' align = 'center'>인적 정보</td>
           <td>성별, 나이 데이터에 결측치, 이상치가 다수 존재함</td>
       </tr>
       <tr>
           <td valign = 'center' align = 'center'>사용 기록</td>
           <td>음원 재생 관련 데이터에 이상치가 다수 존재함</td>
       </tr>
       <tr>
           <td valign = 'center' align = 'center'>구매 기록</td>
           <td>거래 일자와 구독 만료일 데이터에 이상치가 발견됨</td>
       </tr>
       </tr>
   </table>

---

### **④ 데이터 전처리**
* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/5bb3c61ed55963a907f89dcf9ddb306c84d9154f/preprocessing_modeling.ipynb)를 참고해주시기 바랍니다.
* EDA 결과에 따라 아래와 같이 전처리를 수행함
  * **구매 기록**
    * 거래 일자가 중복되는 데이터는 가장 마지막만 남김
    * 멤버십 만료일자가 KKBox 서비스 시작연도 이전이거나 <br>데이터 공개시점인 2017년 3월 30일로부터 1년 1개월※이후인 경우는 제외 <br>(최대 1년 단위 구독 가능 + 30일 무료 체험기간)
  * **인적 정보**
    * 아래의 사유로 인해 정보의 신뢰도가 낮다고 판단되는 성별, 나이 데이터는 제외
      * 결측치 혹은 이상치의 비중이 높음
      * 고객이 직접 입력할 뿐 별도의 인증절차가 요구되지 않음
  * **사용 기록**
    * 일일 총 재생시간이 음수이거나 하루(24시간 * 60분 * 60초)를 넘어서는 데이터는 삭제
    * Boxplot 전개 시 확인되는 재생 음원 수의 이상치 데이터 삭제 (IQR 활용)

---

### **⑤ 모델링**
* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/5bb3c61ed55963a907f89dcf9ddb306c84d9154f/preprocessing_modeling.ipynb)를 참고해주시기 바랍니다.
* 모델링에 사용한 입력 변수는 다음과 같음<br>
<table align = 'center'>
  <tr>
    <th valign = 'center' align = 'center'>종류</th>
    <th valign = 'center' align = 'center'>변수명</th>
    <th valign = 'center' align = 'center'>내용</th>
  </tr>
  <tr>
    <td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>
    <td>in_membership_days</td>
    <td>멤버십을 유지한 기간</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>-->
    <td>is_cancel</td>
    <td>구독을 취소한 경험의 유무</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>-->
    <td>is_auto_renew</td>
    <td>자동 구독 이용 여부 <br> (전체 구매 중에서 자동 구독의 비율이 50% 이상인 경우 참)</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>-->
    <td>discount</td>
    <td>할인 받은 구독 거래의 횟수</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>-->
    <td>is_method_change</td>
    <td>가입 후 결제수단을 변경한 이력의 유무</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>구매 기록</td>-->
    <td>pri_payment_method</td>
    <td>가입 후 가장 자주 사용한 결제수단</td>
  </tr>
  <tr>
    <td rowspan = '4' valign = 'center' align = 'center'>인적 정보</td>
    <td>city</td>
    <td>이용자 거주 도시</td>
  </tr>
  <tr>
    <!--<td rowspan = '4' valign = 'center' align = 'center'>인적 정보</td>-->
    <td>register_via</td>
    <td>서비스 가입 경로</td>
  </tr>
  <tr>
    <!--<td rowspan = '4' valign = 'center' align = 'center'>인적 정보</td>-->
    <td>register_init_time</td>
    <td>최초 가입 연도</td>
  </tr>
  <tr>
    <!--<td rowspan = '4' valign = 'center' align = 'center'>인적 정보</td>-->
    <td>after_regit_to_buy</td>
    <td>가입 후 최초 구입까지 걸린 시간 (일)</td>
  </tr>
  <tr>
    <td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>
    <td>per_25</td>
    <td>25% 이하 감상한 노래의 비율</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>-->
    <td>per_25_75</td>
    <td>25~75% 감상한 노래의 비율</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>-->
    <td>per_100</td>
    <td>75% 이상 감상한 노래의 비율</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>-->
    <td>seconds_per_song</td>
    <td>총 재생시간 / 사용자가 재생한 음악의 수</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>-->
    <td>mean_seconds</td>
    <td>평균 노래 재생시간</td>
  </tr>
  <tr>
    <!--<td rowspan = '6' valign = 'center' align = 'center'>사용 기록</td>-->
    <td>max_log_term</td>
    <td>최장 미접속 기간</td>
  </tr>
</table>

* 분류 알고리즘, 스케일링※ 여부, 리샘플링 사용 등 경우의 수를 72가지로 나누어 각각 모델링을 수행하고 평가함 (※ Standard Scaler 사용)
* **Precision의 과적합 수준이 가장 낮은 모델을 채택**하여 하이퍼 파라미터 튜닝에 사용함

<table align = 'center'>
 <tr>
  <th>채택</th>
  <th>스케일링</th>
  <th>변수선택 및 추출</th>
  <th>리샘플링</th>
  <th>알고리즘</th>
  <th>Test Recall</th>
  <th>Test Precision</th>
  <th>과적합※</th>
 </tr>
 <tr>
  <td align = 'center'>★</td>
  <td align = 'center'>사용함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>LGBM</td>
  <td align = 'center'>0.4449</td>
  <td align = 'center'>0.7496</td>
  <td align = 'center'>0.0136</td>
 </tr>
 <tr>
  <td align = 'center'>-</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>LGBM</td>
  <td align = 'center'>0.4453</td>
  <td align = 'center'>0.7487</td>
  <td align = 'center'>0.0158</td>
 </tr>
 <tr>
  <td align = 'center'>-</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>CATBOOST</td>
  <td align = 'center'>0.4877</td>
  <td align = 'center'>0.7350</td>
  <td align = 'center'>0.0621</td>
 </tr>
 <tr>
  <td align = 'center'>-</td>
  <td align = 'center'>사용함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>XGBOOST</td>
  <td align = 'center'>0.4748</td>
  <td align = 'center'>0.7321</td>
  <td align = 'center'>0.0542</td>
 </tr>
 <tr>
  <td align = 'center'>-</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>사용안함</td>
  <td align = 'center'>XGBOOST</td>
  <td align = 'center'>0.4748</td>
  <td align = 'center'>0.7321</td>
  <td align = 'center'>0.0542</td>
 </tr>
</table>

<p align = 'center'>※ 과적합 : Test Precision과 Train Precision의 차</p>

* Average-Precision을 기준하여 모델을 선정, 아래와 같은 성능 변화가 확인됨
  * 하이퍼 파라미터
    * learning_rate : 0.1,
    * min_child_samples : 20
    * n_estimators : 50,
    * num_leaves : 248
  * **최종 성능**
    * Test Recall : 0.4449 -> **0.4912** (+ 0.0463, 기존 대비 약 10% 상승)
    * Test Precision : 0.7496 -> **0.7389** (- 0.0107, 기존 대비 약 1% 하락)

---

### **⑥ 이탈 확률 기반 Segmentation 및 LTV 산정** 
* 자세한 내용과 코드는 [링크](https://github.com/TAEJIN-AHN/Churn-Prediction/blob/3113a2361365c466caebec09f3c7c67ff66ae9e2/LTV_calculation.ipynb)를 참고해주시기 바랍니다.
* 채택된 모델을 적용하여 각 유저별 이탈 확률을 구하고 10% 간격으로 묶어 실제 이탈유저의 비율과 LTV를 계산함
  * ARPU는 아래와 같이 계산됨
    * 본 데이터셋의 구매 기록은 2015년 1월 1일부터 2017년 3월 31일사이를 대상으로 함 (27개월)
  
  $$총\\;매출 = 대상\\;기간\\;동안의\\;그룹별\\;매출$$
  $$ARPU = \frac{총\\;매출 / 27개월}{그룹별\\;전체\\;고객\\;수}$$
  
  * LTV는 아래와 같이 계산됨
  
  $$LTV = \frac{ARPU}{그룹별\\;실제\\;이탈유저의\\;비율}$$

<br>
<table align = 'center'>
 <tr>
  <th>이탈 확률(%)</th>
  <th>총 유저수(명)</th>
  <th>이탈 유저의 비율(%)</th>
  <th>ARPU(NTD)</th>
  <th>LTV(NTD)</th>
 </tr>
 <tr>
  <td align = 'center'>0 ~ 10</td>
  <td align = 'center'>337,884</td>
  <td align = 'center'>1.01</td>
  <td align = 'center'>88</td>
  <td align = 'center'>8713</td>
 </tr>
 <tr>
  <td align = 'center'>10 ~ 20</td>
  <td align = 'center'>27,182</td>
  <td align = 'center'>13.25</td>
  <td align = 'center'>111</td>
  <td align = 'center'>838</td>
 </tr>
 <tr>
  <td align = 'center'>20 ~ 30</td>
  <td align = 'center'>29,607</td>
  <td align = 'center'>21.86</td>
  <td align = 'center'>90</td>
  <td align = 'center'>412</td>
 </tr>
 <tr>
  <td align = 'center'>30 ~ 40</td>
  <td align = 'center'>25,022</td>
  <td align = 'center'>34.06</td>
  <td align = 'center'>74</td>
  <td align = 'center'>217</td>
 </tr>
 <tr>
  <td align = 'center'>40 ~ 50</td>
  <td align = 'center'>15,577</td>
  <td align = 'center'>48.85</td>
  <td align = 'center'>71</td>
  <td align = 'center'>145</td>
 </tr>
 <tr>
  <td align = 'center'>50 ~ 60</td>
  <td align = 'center'>10,909</td>
  <td align = 'center'>60.78</td>
  <td align = 'center'>83</td>
  <td align = 'center'>137</td>
 </tr>
 <tr>
  <td align = 'center'>60 ~ 70</td>
  <td align = 'center'>10,040</td>
  <td align = 'center'>71.44</td>
  <td align = 'center'>87</td>
  <td align = 'center'>122</td>
 </tr>
 <tr>
  <td align = 'center'>70 ~ 80</td>
  <td align = 'center'>8,501</td>
  <td align = 'center'>83.46</td>
  <td align = 'center'>88</td>
  <td align = 'center'>105</td>
 </tr>
 <tr>
  <td align = 'center'>80 ~ 90</td>
  <td align = 'center'>7,023</td>
  <td align = 'center'>91.98</td>
  <td align = 'center'>51</td>
  <td align = 'center'>55</td>
 </tr>
 <tr>
  <td align = 'center'>90 ~ 100</td>
  <td align = 'center'>4,726</td>
  <td align = 'center'>96.42</td>
  <td align = 'center'>61</td>
  <td align = 'center'>63</td>
 </tr>
</table>

---

### **⑦ 결과 및 기대효과**
* **EDA를 통해 CRM 및 퍼포먼스 마케팅에 활용할 수 있는 Agenda 도출**
  <p align = 'center'><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/11d25b1e-410b-481b-8201-bda773c197f3' width = 80%></p>

* **이탈율에 기반한 Semgentation과 LTV 산정으로 고객 관리 방안 수립, 마케팅 비용 최적화 등이 가능함**
  <p align = 'center'><img src = 'https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/5a7f837d-97c7-41e5-8789-2f811a4beba8' width = 80%></p>
