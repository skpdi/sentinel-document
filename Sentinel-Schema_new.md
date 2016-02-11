# New Sentinel-Schema 설계문서 작성 manual
* \#{tagName} 방식의 태그를 활용하여 작성, 프로그램이 읽을 수 있는 형태로 제작
* [Sample Schema 파일](https://docs.google.com/spreadsheets/d/1c54C-emSKnz95MnZ4RE7phEKcZ6cTF_4zuzBWChtWKQ/edit?usp=sharing) 참고

## Sheet 정의
* sentinel에서 사용되는 모든 sheet엔 태깅 필수, 아래 총 5개의 sheet가 사용됨
* 모든 sheet의 시작 row와(\#start) 종료 row에(\#end) 태깅이 필요함
* \#start 태그 바로 다음 row부터 각 sheet의 컨텐츠가 존재해야 함
* \#end 태그 바로 직전 row까지 각 sheet의 컨텐츠가 존재해야 함

#### 태그 예시(\#define)
올바른 예시<br/>

| | | | | | |
|-----|-----|-----|-----|-----|-----|
| | #start | | | | |
| |	#version	| #id	| #format | #poc	| | 
| |	14.02.11	| T log sample |	HM	| server | |
| |	#end |	| | | |
| | | | | | |

잘못된 예시
> \#start 태그 바로 다음 row에 컨텐츠가 없음

| | | | | | |
|-----|-----|-----|-----|-----|-----|
| | #start | | | | |
| | &nbsp; | | | | |
| |	#version	| #id	| #format | #poc	| | 
| |	14.02.11	| T log sample |	HM	| server | |
| |	#end |	| | | |
| | | | | | |


## \#define
스키마 문서의 기본 정보 정의

![Image of Define](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_define.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#version 태그** : 로그 버전 정의, 릴리즈 날짜형태(yy.mm.dd) 권장
* **\#id 태그** : 로그 서비스명 정의
* **\#format 태그** : "HM" (향후 확장성을 위한 태그)
* **\#poc 태그** : Log 입수 poc 구분
  * client: web/app 단말에서 rakeClient를 통해 전송되는 로그
  * server: server에서 남겨지는 로그
  * db-data: DB Schema (현재 미사용)

## \#infra
입수와 관련된 정보가 기록되는 시트, 상용 입수 이후에는 시트 잠금으로 관리자만 수정가능



## \#dictionary
key 목록 정의, key 이름, 타입, 설명, 검증rule, 아래 나열되는 모든 태그가 존재하여야 함

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_dic.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : key 이름, human-readable하게 정의
* **\#type 태그** : key type 
  * string : 가변길이 문자형
  * int : 정수형
  * float : 실수형
  * list<type> : json list형, body에서만 사용 가능, 아래 3가지 type 이외의 type(list, object 등등)은 지원안함
    * list<int> : 정수형 리스트, ex)[10,20,30]
    * list<float> : 실수형 리스트, ex)[1.1,1.3,1.5]
    * list<string> :  문자형 리스트,  ex)["a","b","c","d"]
  * map<type> : json object 형, body에서만 사용 가능, 아래 3가지 type 이외의 type(list, object 등등)은 지원안함
    * map<int> :  정수형 object,  ex){"a":10,"b":20,"c":30}
    * map<float> : 실수형 object, ex){"a":1.1,"b":1.3,"c":1.5}
    * map<string> :  문자형 object,  ex){"a":"q","b":"w","c":"e"}
  * json : json, body에서만 사용 가능
    * hive 및 검증기에서 json string으로 취급됩니다.
* **\#length 태그** : 값의 length 에 관해 정의가능
  * [fixed length]: 정확하게 일치
  * [min]~[max]: 범위 지정 가능
  * example: 2(일치), ~2(2이하), 2~(2이상), 1~2(1이상2이하)  
* **\#nullableYN 태그** : 빈값을 허용하는 필드를 태그할 수 있다.
  * Y: nullable인 경우, 빈값일때는 검증룰(#type,#length,#rule,..)을 적용하지 않고 통과 
* **\#description 태그** : key에 대한 설명
* **\#rule 태그** : key의 검증룰, 빈 값인 경우 검증하지 않음
  * UDF(user define function)
    1. **df(date_pattern)** : 시간관련 값 검증 
      - example : df(yyyyMMddHHmmssSSS)
    2. **code(key in codelist)** : #code 시트에 정의한 codemap 이내의 값 중 하나
      - example : code(page_id)
    3. **in(values with csv format)** : parameter 에 정의한 값 중 하나
      - example : in(apple,google,amazon)
    4. **regex(regular_expression)** : 정규식 검증 ([Java Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html))
      - example : regex([0-9]{2}\\.[0-9]{2}\\.[0-9]{2})
    5. Logical Syntax
      - **and(rules with csv format)** : AND 조건
      - **or(rules with csv format)** : OR 조건
      - **if(condition, rule if true, rule if false)** : 조건 분기
      - **<,<=,>,>=,=,<>** : equality
      - **@FIELD_NAME** : 다른 필드값 참조
      - example : transaction_id 필드의 #rule: if(@tech_type=ble,$nonempty,$empty)
        - tech_type 필드의 값이 ble 일 경우, transaction_id 는 빈값이 아니어야 함 
* **\#versionKey 태그** : log version을 정의하는 key, key 이름 뒤에 태깅, key 목록중에서 한 개의 version key가 필요(필수)
  * #infra 시트에 자동수집항목 필드에 정의할 경우, dictionary 에 지정할 수 없음

## \#layout
\#dictionary 에서 정의한 \#key를 활용해 로그 종류별 body field 정의

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_header_body.png?raw=true)

#### 로그 종류(action)에 대한 정의
\#logKey 아래 로그 종류를 구분할 key 나열<br/>
두 개의 key 설정 가능 <br/>
- example: 예제 그림 참조

#### Header List 정의
header는 모든 action에서 동일하게 입수할 값<br/>
server log schema 작성시 header의 첫번째 값은 log_time(YYYYMMDDHH*)을 사용(입수 시스템에서 partition 분할에 사용)

* action별 header 정의
  - header key값을 입력한 경우 : 해당 key의 rule로 검증하겠음
  - 비어있는 경우 : 해당 key를 사용하지 않겠음, 실제 로그엔 빈칸으로 기록되어야 함
  - \#bypass 태그를 입력한 경우 : 해당 key엔 어떤 값이 들어와도 상관없음. 룰 검증을 하지 않겠음
  - **header엔 list,map type의 \#key은 사용 불가**

#### Body Field 정의
로그 종류(action)별로 입수할 #key 나열, 순서와 공백은 무관
* => 구분자를 이용해 특정액션에서만 특정 룰을 정의할 수 있음

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#logKey 태그** : log 종류를 구분지을 필드를 정의
* **\#incaseHeader 태그** : action별로 header 필드 검증룰을 따로 지정하고 싶을 경우 사용
* **\#body 태그** : body시작 지점 정의


## #code
validation rule에서 사용할 key-value data를 정의, code(KEY)으로 접근 가능<br/>
MakeSentinel 시 key-value-description 그대로 hive table로 export되어 다른 통계에 사용될 수 있음<br/>

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_code_map_list.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : key, 중복가능
* **\#value 태그** : value, 동일 key에 대해서는 unique
* **\#description 태그** : value에 대한 설명 작성




