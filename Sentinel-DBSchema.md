# DBSchema 작성 manual
* DBSchema 1개는 DB 서버에서 단일 계정으로 추출할 수 있는 여러 개의 db-data 를 정의할 수 있습니다.
    - db-data 1개는 테이블 스냅샷 인터페이스 1개를 의미합니다.
* [Sample DBSchema 파일](https://docs.google.com/spreadsheets/d/1f76XlYgiEApCe5DpzzwfWR55EN8X4whsd9YYQ2-IK78/edit?usp=sharing) 참고

##Quick Guide
1. \#define 시트를 작성합니다.
2. \#common 시트를 작성합니다.
3. \#maplist 시트를 작성합니다.
4. \#dummy 시트를 복사한 후, 시트의 이름을 {데이터 명} \#tbl 로 수정합니다.
5. 추가한 \#tbl 시트를 작성합니다.
6. 정의해야할 데이터 수만큼 4-5번을 반복합니다.

## Block 정의
* DBSchema의 모든 block은 태그를 기반으로 인식함
    * 모든 block의 시작 행와(\#start_\*) 종료 행에(\#end_\*) 태깅이 필요함
        * \#start_\* 태그 바로 다음 행부터 각 블록의 컨텐츠가 존재해야 함
        * \#end_\* 태그 바로 직전 행까지 각 블록의 컨텐츠가 존재해야 함
    * 모든 block의 첫번째 row는 헤더 row로 각 블록에서 사용할 수 있는 태깅이 필요함

#### 태그 예시
잘못된 예시 1
> \#start 태그 바로 다음 row에 헤더 row가 없음

| | | | | |
|-----|-----|-----|-----|-----|
| | #start_define | | | |
| | &nbsp; | | | |
| | #version    | #id   | #format | | 
| | 14.02.11    | T log sample |    HM  | |
| | #end_define |  | | |
| | | | | |
 
잘못된 예시 2
> \#start 태그, \#end 태그 사이에 컨텐츠가 없음

| | | | | |
|-----|-----|-----|-----|-----|
| | #start_define | | | |
| | #version    | #id   | #format | | 
| | 14.02.11    | T log sample |    HM  | |
| | &nbsp; | | | |
| | #end_define |  | | |
| | | | | |

올바른 예시<br/>

| | | | | |
|-----|-----|-----|-----|-----|
| | #start_define | | | |
| | #version    | #id   | #format | | 
| | 14.02.11    | T log sample |    HM  | |
| | #end_define |  | | |
| | | | | |

## \#define 시트
### \#define 블록
스키마 문서의 기본 정보 정의

![Image of Define](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_define.png?raw=true)

#### 사용 태그 목록
* **\#start_define 태그** : 시작 row 정의
* **\#end_define 태그** : 종료 row 정의
* **\#version 태그** : 로그 버전 정의, 릴리즈 날짜형태(yy.mm.dd) 권장
* **\#id 태그** : 로그 서비스명 정의
* **\#format 태그** : "HM" (향후 확장성을 위한 태그)



## \#common 시트
### \# extract, transform, load 블록


![Image of Common](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_common.png?raw=true)

#### 사용 태그 목록
* **\#start_common 태그** : 시작 row 정의
* **\#end_common 태그** : 종료 row 정의
* **\#key 태그** : properties 키
* **\#value 태그** : properties 값
* **\#description 태그** : properties 코멘트
  
#### properties 목록
* **idc**: DB 서버 & dump 서버 IDC
* **dbms_type**: DBMS 타입
* **jdbc_url**: 데이터베이스 jdbc url
* **dump_server_host**: 덤프 파일을 생성해서 전달할 경우, dump 서버 호스트
* **dump_server_ip**: 덤프 파일을 생성해서 전달할 경우, dump 서버 IP
* **dump_home**: 덤프 파일을 생성해서 전달할 경우, dump 서버 내 경로
* **system_prefix**: 데이터베이스 별칭

## code \#maplist
validation rule에서 사용할 key-value data를 정의, code['key']으로 접근 가능<br/>
MakeSentinel 시 key-value-description은 hive table로 export되어 다른 통계에 사용될 수 있음<br/>

![Image of Maplist](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_maplist.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : key, 중복가능
* **\#value 태그** : value, 동일 key에 대해서는 unique
* **\#description 태그** : value에 대한 설명 작성



## \#dummy 시트
\#tbl 시트의 템플릿으로 복사해서 #tbl 작성

![Image of Dummy](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_dummy.png?raw=true)

## \#tbl 시트
### \# extract, transform, load 블록


![Image of Property Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_0.png?raw=true)

#### 사용 태그 목록
* **\#start_ 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : properties 키
* **\#value 태그** : properties 값
* **\#description 태그** : properties 코멘트

#### properties 목록
* **dump_location**: 덤프 파일을 생성해서 전달할 경우, 덤프 파일 적재 경로
* **file_name_pattern**: 덤프 파일을 생성해서 전달할 경우, 파일 이름 패턴
* **extract_period**: 덤프 주기. cron expression 사용
    * [Quartz의 Cron Expression](http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/crontrigger) 참고
* **extract_type**: 데이터 추출 유형
    * incremental: 증분
    * delta: 변동분
    * full: 전체
* **query**: 추출 SQL
* **null_string**:  null 값 표현 문자열
* **data_description**: 데이터 설명

### \#tbl 블록
key 목록 정의, key 이름, 타입, 설명, 검증rule, 암호화여부 작성, 아래 나열되는 모든 태그가 존재하여야 함

![Image of Table Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_1.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : 테이블 필드 이름 정의. Hive 네이밍 컨벤션룰대로 작성
* **\#type 태그** : 테이블 필드 타입 정의. Hive 타입만 허용
    - [Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types) 참고
* **\#description 태그** : 테이블 필드 코멘트 정의
* **\#rule 태그** : key의 검증룰, groovy 문법 채용, 모든 rule이 정의되어야 함(not nullable) 
    * bypass시(룰 검증이 필요없는 경우) : #bypass 태그 입력
    * UDF(user define function)
        * dateformat(key, date_pattern) : 시간관련 key 검증 
            * example : dateformat(log_time, 'yyyyMMddHHmmssSSS')
        * regex(key, regular_expression) : 정규식 검증
            * example : regex(log_version, '[0-9]{2}\.[0-9]{2}\.[0-9]{2}')
        * list(key){value -> value 검증 룰} : list type 검증, list 내의 모든 value를 차례대로 검증 
            * example : list(product_price){value -> value >= 0}
            * product_price의 type이 list이고 value가 [10,20,30,40,50]인 경우 list내의 모든 value가 0 이상이어야 검증 통과
        * map(key){key,value -> key,value에 대한 검증 룰} : map type 검증, map 내의 모든 key, value를 차례대로 검증
            * example : map(result_message){key,value -> key.length() >= 3 && value.length() > 0}
            *  result_message의 type이 map이고 value가 {"a01":"succ","b02":"fail"}인 경우 map내의 모든 key의 길이가 3 이상, 모든 value가 0보다 커야 검증 통과
* **\#encryptionYN 태그** : 필드의 암호화 여부. 암호화가 필요한 경우만 Y
* **\#origin_type 태그** : 원천 DB에서의 테이블 필드 타입. 테이블의 필드였던 경우만 기입
* **\#pk 태그** : 테이블 스키마 primary key 여부. PK인 경우만 Y
* **\#not_null 태그** : 테이블 스키마 not null 여부. not null인 경우만 Y
* **\#default_value 태그** : 테이블 스키마 default value 값. default value 있는 경우만 기입
* **\#version_key 태그** : 레코드의 버전. 즉, 레코드가 생성 / 수정된 날짜 정보. delta / incremental에서 사용. version_key인 경우만 기입