# DBSchema 작성 manual
* DBSchema 한 개는 한 개의 인스턴스로부터 추출할 수 있는 여러 개의 db-data 를 정의할 수 있습니다.
* [Sample DBSchema 파일](https://docs.google.com/spreadsheets/d/1f76XlYgiEApCe5DpzzwfWR55EN8X4whsd9YYQ2-IK78/edit?usp=sharing) 참고

##Quick Guide
0. \#define 시트를 작성합니다.
1. \#common 시트를 작성합니다.
2. \#maplist 시트를 작성합니다.
3. \#dummy 시트를 복사한 후, 시트의 이름을 {데이터 명} \#tbl 로 수정합니다.
4. 추가한 \#tbl 시트를 작성합니다.
5. 정의해야할 데이터 수만큼 2-3번을 반복합니다.

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
* **\#key 태그** : 
* **\#value 태그** : 
* **\#description 태그** : 
  


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
\#tbl 시트의 템플릿

![Image of Dummy](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_dummy.png?raw=true)

## \#tbl 시트
### \# extract, transform, load 블록


![Image of Property Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_0.png?raw=true)

#### 사용 태그 목록
* **\#start_ 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : 
* **\#value 태그** : 
* **\#description 태그** : 

### \#tbl 블록
key 목록 정의, key 이름, 타입, 설명, 검증rule, 암호화여부 작성, 아래 나열되는 모든 태그가 존재하여야 함

![Image of Table Block](https://github.com/skpdi/sentinel-document/blob/master/schema/db_schema_tbl_1.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : 
* **\#type 태그** : 
* **\#description 태그** : 
* **\#rule 태그** : 
* **\#encryptionYN 태그** : 
* **\#origin_type 태그** : 
* **\#pk 태그** : 
* **\#not_null 태그** : 
* * **\#default_value 태그** : 



