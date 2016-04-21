# DBSchema 작성 manual
* DBSchema 1개는 RDB에서 단일 계정으로 추출할 수 있는 여러 개의 db-data 를 정의할 수 있습니다.
    - db-data 1개는 테이블 스냅샷 인터페이스 1개를 의미합니다.
* [Sample DBSchema 파일](https://docs.google.com/spreadsheets/d/1f76XlYgiEApCe5DpzzwfWR55EN8X4whsd9YYQ2-IK78/edit?usp=sharing) 참고

##Quick Guide
1. \#define 시트를 작성합니다.
2. \#common 시트를 작성합니다.
3. \#maplist 시트를 작성합니다.
4. \#dummy 시트를 복사한 후, 시트의 이름을 {데이터 명} \#tbl 로 수정합니다.
5. 추가한 \#tbl 시트를 작성합니다.
6. 정의해야할 db-data 수만큼 4-5번을 반복합니다.

## Block 정의
* DBSchema의 모든 block은 태그를 기반으로 인식함
    * 행 지정: 모든 block의 시작 행와(\#start\_\*) 종료 행에(\#end\_\*) 태깅이 필요함
        * \#start\_\* 태그 바로 다음 행부터 각 블록의 컨텐츠가 존재해야 함
        * \#end\_\* 태그 바로 직전 행까지 각 블록의 컨텐츠가 존재해야 함
    * 열 지정: 모든 block의 첫번째 row는 헤더 row로 각 블록에서 사용할 수 있는 태깅이 필요함

#### 태그 예시
| | | | | |
|-----|-----|-----|-----|-----|
| | #start_define | | | |
| | #version    | #id   | #format | |
| | 14.02.11    | T log sample |    HM  | |
| | #end_define |  | | |
| | | | | |

## \#define 시트
### \#define 블록
스키마 문서의 버전 관리 및 문서 구조 정의합니다. 마지막 최신 버전을 가장 윗줄에 적습니다.

![Image of Define](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_define.png?raw=true)

#### 사용 태그 목록
* **\#start\_define 태그** : 시작 row 표시
* **\#end\_define 태그** : 종료 row 표시
* **\#version 태그** : 문서 (데이터) 버전 정의, 릴리즈 날짜형태(yy.mm.dd)로 작성
* **\#id 태그** : 버전별 서비스명 정의
* **\#format 태그** : 미사용. '-'로 표기

## \#common 시트
### \#extract, transform, load 블록
문서에 정의된 테이블들의 공통 속성을 적습니다. 

![Image of Common](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_common.png?raw=true)

#### 사용 태그 목록
* **\#start\_common 태그** : 시작 행 정의
* **\#end\_common 태그** : 종료 행 정의
* **\#key 태그** : properties 키
* **\#value 태그** : properties 값
* **\#description 태그** : properties 코멘트

#### property 키 목록
* **idc** : DB서버 IDC 위치 (커머스센터, 일산, 서초, 보라매, 성수)
* **dbms\_type** : RDB 종류 (oracle, mysql, postgresql, mssql)
* **jdbc\_url** : JDBC URL
* **dump\_server_host** : 테이블 스냅샷 파일을 저장하는 서버의 host
* **dump\_server\_ip** : 테이블 스냅샷 파일을 저장하는 서버의 ip
* **dump\_home** : 테이블 스냅샷 파일 저장 경로
* **system\_prefix** : 센티넬 프로젝트 명이자 테이블 접두어로 쓸 RDB 별칭

## \#maplist 시트
### \#maplist 블록
\#tbl의 \#rule에서 참고할 코드 테이블입니다.
또한 hive table로 export되어 Galleon에서 사용할 수 있습니다. (구현중)

![Image of Maplist](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_maplist.png?raw=true)

#### 사용 태그 목록
* **\#start\_ 태그** : 시작 row 정의
* **\#end\_ 태그** : 종료 row 정의
* **\#key 태그** : 코드 키
* **\#value 태그** : 코드 값
* **\#description 태그** : 코드 설명

## \#dummy 시트
\#tbl 시트를 편하게 만들기 위한 템플릿입니다. \#dummy 시트를 복사해서 \#tbl 시트를 만들어 사용하세요.

## \#tbl 시트
이 문서의 핵심입니다. 
#key, #description 은 반드시 작성해주십시오.

### \# extract, transform, load 블록

![Image of Property Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_0.png?raw=true)

#### 사용 태그 목록
* **\#start\_ 태그** : 시작 row 정의
* **\#end\_ 태그** : 종료 row 정의
* **\#key 태그** : properties 키
* **\#value 태그** : properties 값
* **\#description 태그** : properties 코멘트

#### property 키 목록
* **dump\_location**: 덤프 파일을 생성해서 전달할 경우, 덤프 파일 적재 경로
* **file\_name\_pattern**: 덤프 파일을 생성해서 전달할 경우, 파일 이름 패턴
* **extract\_period**: 덤프 주기. cron expression 사용
    * [Cron Configuration](https://en.wikipedia.org/wiki/Cron) 참고
* **extract_type**: 데이터 추출 유형
    * incremental: 증분
    * delta: 변동분
    * full: 전체
* **query**: 추출 SQL
* **null\_string**:  null 값 표현 문자열. 대소문자 구분
* **database**: 테이블 데이터베이스
* **table\_name**: 테이블 명
* **hdfs**: 테이블 경로
* **partition\_key**: 파티션 키
* **field\_delim**: 필드 구분자. 아래 값들만 사용 가능하다. 기본값은 tab
    - TAB (\t)
    - COMMA (,)
    - CONTROL_B (\002)
    - PIPE (|)
    - CONTROL_A (\001)
* **data\_description**: 데이터 설명

### \#tbl 블록
key 목록 정의, key 이름, 타입, 설명, 검증rule, 암호화여부 작성합니다.

![Image of Table Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_1.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : 테이블 필드 이름 정의. Hive 네이밍 컨벤션룰대로 작성
* **\#type 태그** : 테이블 필드 타입 정의. 
    - [Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types) 참고
* **\#description 태그** : 테이블 필드 코멘트 정의
* **\#rule 태그** : 현재 룰 변경 중
    * bypass시(검증 룰을 작성할 수 없는 경우): #bypass 태그 입력
* **\#encryptionYN 태그** : 필드의 암호화 여부. 암호화가 필요한 경우만 y
* **\#origin\_type 태그** : 원천 DB에서의 테이블 필드 타입. 테이블의 필드였던 경우만 기입
* **\#pk 태그** : 테이블 스키마 primary key 여부. PK인 경우만 y
* **\#not\_null 태그** : 테이블 스키마 not null 여부. not null인 경우만 y
* **\#default\_value 태그** : 테이블 스키마 default value 값. default value 있는 경우만 기입. 대소문자 구분.
