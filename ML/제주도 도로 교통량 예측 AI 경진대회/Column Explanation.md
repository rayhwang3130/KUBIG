# Data Preprocessing & EDA

**FEATURE OVERVIEW**

- id : 아이디 -> **drop**

---
*날짜 관련*

- base_date : 날짜  
    - date parsing
        - 년/월/일 or 년/월_일 or 년_월_일 로 나눠보기
        - 그대로 두고 date type으로
    - 요일과 상관성이 높지 않을까 -> 공휴일 파악하기 위함으로 두고 나중에 drop해도 될수도

- day_of_week : 요일  
    - 요일에 따라 순서대로 labeling
        - 0을 일요일으로, 1-6 : 월-토

- base_hour : 시간대  
    - 0~23로 구분
        - 등하교, 출퇴근 시간 고려 : 아마 모델링에서 해당 수치들에 가중치가 붙지 않을까?
        - target이 평균 속도인만큼, 차량이 많다 = 차가 막힌다 = 속도 저하 : 추측

---
*도로 및 차량 관련*

- lane_count : 차로수  
    - 차로가 많다 = 많은 양의 차가 한번에 통과 가능하다 = 차의 유동성이 높다 = 속도 높게 이동 가능 : 추측

**필요한 정보와 관련된 표**

<img width="317" alt="column_road_related_explanation" src="https://user-images.githubusercontent.com/111053011/221128235-fc2f35f1-7275-4e02-830a-028d0fd08e82.png">

출처 :  
[<지능형교통체계 표준 노드·링크 구축기준 [시행 2015. 10. 16.] [국토교통부고시 제2015-756호, 2015. 10. 16., 일부개정]>](https://www.law.go.kr/LSW/admRulInfoP.do?admRulSeq=2100000029728)


- road_rating : 도로등급  
    - `train.road_rating.unique()`와 `test.road_rating.unique()` 돌려본 결과 
        - 103 : 일반 국도, 106 : 지방도, 107 : 시도, 군도 밖에 안나옴
    - 고속도로는 *없음*

- multi_linked : 중용구간 여부  
    - 중용구간이란? 2개 이상의 노선이 도로의 일정 구간을 공동으로 사용하는 구간 [(출처)](https://www.semanticscholar.org/paper/%EC%9D%BC%EB%B0%98%EA%B5%AD%EB%8F%84-%EC%A4%91%EC%9A%A9%EA%B5%AC%EA%B0%84%EC%9D%98-%EC%8B%9C%EA%B0%84%EC%A0%81-%EC%A0%95%EB%B3%B4%EA%B4%80%EB%A6%AC%EC%97%90-%EB%8C%80%ED%95%9C-%EC%97%B0%EA%B5%AC-%EC%84%9C%EC%9E%AC%ED%99%94-%EC%84%B1%EC%A0%95%EA%B3%A4/5c905f83ce13879d65c10fc1c92b1ef20d3b402f)
    - 0이 아님, 1이 맞음으로 추정

- connect_code : 연결로 코드  
    - 아마 해당 도로가 연결되는 점이 어디인가 인듯
    - connect_code 103이 있길래 road_rating과 같은지 확인 위하여 코드로 확인해본 결과 '일반국도12호선', '일반국도95호선'만 나옴
        - 해당 값들에 대해 좀 리서치 해야할듯
        - 코드: `train.loc[train['road_rating']==103].loc[train['connect_code']==103].road_name.unique()`

- maximum_speed_limit : 최고속도제한  
    - 어린이 제한 구역?
    - 속도 제한이 낮다 = 차들이 느리게 갈 수 밖에 없다 = 속도 저하
    - target과 가공 없는 상태에서 봤을때 가장 관련성 자체는 높아 보이는 feature

- weight_restricted : 통과제한하중  
    - `train.weight_restricted.unique()`와 `test.weight_restricted.unique()`로 검증 결과
    - 32400., 0., 43200., 50000. : 4가지의 값들만 존재
    - 이는 제한 하중의 유무로 그냥 0과 1로 나누어도 될수도

- height_restricted : 통과제한높이  
    - train, test에서 모두 0 -> **drop**

- road_type : 도로유형  
    - `unique()` 돌려본 결과 0, 3 밖에 없음
    - 0 : 일반도로, 3 : 교량

- start_latitude : 시작지점의 위도 & start_longitude : 시작지점의 경도  
    - concat하여 start_point 로 파생변수 생성 가능할 듯

- end_latitude : 도착지점의 위도 & end_longitude : 도착지점의 경도  
    - concat하여 end_point 로 파생변수 생성 가능할 듯

- start_turn_restricted : 시작 지점의 회전제한 유무 & end_turn_restricted : 도작지점의 회전제한 유무
    - 회전제한유무 : 좌회전금지, 우회전금지, U-Turn 등 교차로에서의 회전제한 (출처는 표와 동일)
    - target과 어떤 연관성이 있을지는 의문

- road_name : 도로명  
    - 도로별로 묶어서 분석? 어떻게 전처리해야할지 논의해봐야함 (**예은 주목**)

- start_node_name : 시작지점명 & end_node_name : 도착지점명  
    - start_point, end_point (위도 경도)와 엮을 수 있을 듯

- vehicle_restricted : 통과제한차량  
    - 전 데이터에서 0 -> **drop**

---

외부 데이터?  
- 기후 데이터 (비오는 날이나 더운 날) -> 기상청에서 데이터 [여기서](https://data.kma.go.kr/climate/RankState/selectRankStatisticsDivisionList.do?pgmNo=179) 옵션 조절 후 다운로드 가능
- 휴일에 관한 데이터 (휴일 시 1, 아닐 시 0으로 인코딩 가능)
    - [2021년 공휴일 데이터](https://search.naver.com/search.naver?sm=tab_hty.top&where=nexearch&query=2021+%EA%B3%B5%ED%9C%B4%EC%9D%BC&oquery=2022+%EA%B3%B5%ED%9C%B4%EC%9D%BC&tqi=h2u%2FLdprvh8ssMTI%2BEwssssssud-119076)
    - [2022년 공휴일 데이터](https://search.naver.com/search.naver?sm=tab_hty.top&where=nexearch&query=2022+%EA%B3%B5%ED%9C%B4%EC%9D%BC&oquery=2021+%EA%B3%B5%ED%9C%B4%EC%9D%BC&tqi=h3Lx0sprvh8ssPXKm8hssssss98-520113)
- [날짜별 관광객 입도 현황](https://www.jeju.go.kr/open/open/iopenboard.htm?category=1035)
- 공항과의 거리 -> 어떻게 계산할 것인가
