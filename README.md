<img src = "https://github.com/TAEJIN-AHN/Churn-Prediction/assets/125945387/55cb1525-0ea2-4d54-8086-a07b97dd7e01)">

## **A. 목차**
* A. 목차
* B. 프로젝트 진행
  * B.1. 문제 정의
  * B.2. 주요 액션
    * B.2.1. 데이터 로드 - 관련 코드
    * B.2.2. 데이터 샘플링 - 관련 코드
    * B.2.3. EDA - 관련 코드
    * B.2.4. 데이터 전처리 - 관련 코드
    * B.2.5. 모델링 - 관련 코드
    * B.2.6. 과적합 개선 - 관련 코드
  * B.3. 결과
* C. Deck
* D. Methods Used
---
## **B. 프로젝트 진행**
### **B.1. 문제 정의**
* 멜론, 스포티파이와 같은 음원 스트리밍 서비스는 고객을 유지하는 것이 안정적인 매출 발생과 연결됨
* 본 프로젝트에서는 고객의 이탈 여부를 예측하는 분류 모델을 개발하여 고객의 이탈 방지를 위한 선제적 대응을 가능하게 하고자 함

### **B.2. 주요 액션**
#### **B.2.1. 데이터 로드**
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

<p align = "center" >※ 본 데이터에서는 멤버십 만료 후 30일 이내에 추가적인 서비스 구매 이력이 없는 유저를 '이탈'했다고 규정함</p><br>

* 데이터셋의 크기가 29GB에 달하여 데이터를 모두 메모리에 올리는 Pandas를 이용할 수 없었고, 병렬, 분산처리를 위해 Pyspark를 사용함

#### **B.2.2. 데이터 샘플링**

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
* 또한, 팀원 개개인의 컴퓨팅 파워(로컬 기준)를 고려하여 ④번 그룹 95만명 중 대략 10만명 가량의 데이터를 샘플링하여 사용함
