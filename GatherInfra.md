### DIC 입수 Infra 구성

![Image of GatherInfra](https://github.com/skpdi/sentinel-document/blob/master/infra/GatherInfra.png?raw=true)


### Header & Body Model

* Header: 모든 로그에 공통으로 남길 데이터 field
* Body: 로그 종류별로 달라지는 데이터 field
* example: DIC Header & Body Model
  * header list: tsv
  * body: maximum 2-depth json

| log_time | log_id | header_key1 | header_key2 | body|
|----|----|----|----|----|
| 201511041803123 | action1 | header_value1 | header_value2 | {"body1":'sample',"body2":1} |

### Header & Body Model in DIC
* newline(\n)기호로 구분된 로그 한 줄이 tsv 형태로 hdfs에 저장되어 있음
* 주로 [Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)가 데이터 처리에 사용되고 있음(~ 2015.11 현재)
  * Hive: hdfs에서 SQL과 유사한 분석 환경 제공
  * Hive는 hdfs에 저장된 string 그대로 읽어와 구분자(tab)로 table 인식
    * 초기 테이블이 생성된 이후 header list의 변경이 어려움
      * 추가된 header 값 처리 곤란
      * 초기 설계시 header list에 header 특성 field를 최대한 추가, 이후에는 body field로 추가
  * body json 탐색시 UDF(get_json_object)사용, Query의 complexity를 낮추기 위해 maximum 2-depth json 으로 한정
