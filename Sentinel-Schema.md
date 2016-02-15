# OUT-DATED
[신규 스키마 바로가기](https://github.com/skpdi/sentinel-document/blob/master/Sentinel-Schema_new.md)

# Sentinel-Schema 설계문서 작성 manual
* \#{tagName} 방식의 태그로 작성하여, Sentinel이 읽을 수 있는 형태로 제작
* [Sample Schema 파일](https://docs.google.com/spreadsheets/d/1c54C-emSKnz95MnZ4RE7phEKcZ6cTF_4zuzBWChtWKQ/edit?usp=sharing) 참고

## Sheet 정의
* sentinel에서 사용되는 모든 sheet엔 태깅 필수, 아래 총 4개의 sheet가 사용됨
* 모든 sheet의 시작 row와(\#start) 종료 row에(\#end) 태깅이 필요함
* \#start 태그 바로 다음 row부터 각 sheet의 컨텐츠가 존재해야 함
* \#end 태그 바로 직전 row까지 각 sheet의 컨텐츠가 존재해야 함

#### 태그 예시(\#define)
올바른 예시<br/>

| | | | | |
|-----|-----|-----|-----|-----|
| | #start | | | |
| |	#version	| #id	| #format |	| 
| |	14.02.11	| T log sample |	HM	| |
| |	#end |	| | |
| | | | | |

잘못된 예시
> \#start 태그 바로 다음 row에 컨텐츠가 없음

| | | | | |
|-----|-----|-----|-----|-----|
| | #start | | | |
| | &nbsp; | | | |
| |	#version	| #id	| #format |	| 
| |	14.02.11	| T log sample |	HM	| |
| |	#end |	| | |
| | | | | |
 


## \#define
스키마 문서의 기본 정보 정의

![Image of Define](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_define.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#version 태그** : 로그 버전 정의, 릴리즈 날짜형태(yy.mm.dd) 권장
* **\#id 태그** : 로그 서비스명 정의
* **\#format 태그** : "HM" (향후 확장성을 위한 태그)



## \#dictionary
key 목록 정의, key 이름, 타입, 설명, 검증rule, 암호화여부 작성, 아래 나열되는 모든 태그가 존재하여야 함

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_dic.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : key 이름, human-readable하게 정의
* **\#type 태그** : key type 
  * string : 가변길이 문자형
  * fixed string(n) : 고정길이 문자형, ex)fixed string(10) : 10자리 문자형
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
    * Body가 1depth 이기 때문에, 이 경우 2 depth 이상이 됩니다. json 의 depth가 2 이상인 값에 대해 암호화/검증기능을 지원하지 않습니다.
* **\#description 태그** : key에 대한 설명
* **\#rule 태그** : key의 검증룰, groovy 문법 채용, 모든 rule이 정의되어야 함(not nullable)
  * bypass시(룰 검증이 필요없는 경우) : \#bypass 태그 입력
  * UDF(user define function)
    1. *dateformat(key, date_pattern)* : 시간관련 key 검증 
      - example : dateformat(log_time, 'yyyyMMddHHmmssSSS')
    2. *regex(key, regular_expression)* : 정규식 검증
      - example : regex(log_version, '[0-9]{2}\\.[0-9]{2}\\.[0-9]{2}')
    3. *list(key){value -> value 검증 룰}* : list type 검증, list 내의 모든 value를 차례대로 검증
      - example : list(product_price){value -> value >= 0}<br/>
        product_price의 type이 list<int>이고 value가 [10,20,30,40,50]인 경우<br/>
        list내의 모든 value가 0 이상이어야 검증 통과
    4. *map(key){key,value -> key,value에 대한 검증 룰}* : map type 검증, map 내의 모든 key, value를 차례대로 검증
      - example : map(result_message){key,value -> key.length() >= 3 && value.length() > 0}<br/>
        result_message의 type이 map<string>이고 value가 {"a01":"succ","b02":"fail"}인 경우<br/>
        map내의 모든 key의 길이가 3 이상, 모든 value가 0보다 커야 검증 통과<br/>
* **\#encryptionYN 태그** : key 저장시 암호화 여부, 암호화가 필요한 경우 Y 필요없으면 null
* **\#action_key 태그** : action을 정의하는 key, key 이름 뒤에 태깅, key 목록중에서 한 개의 action key가 필요(optional)
* **\#version_key 태그** : log version을 정의하는 key, key 이름 뒤에 태깅, key 목록중에서 한 개의 version key가 필요(필수)


## \#layout
\#dictionary 에서 정의한 \#key를 활용해 header list 및 로그 종류별 body field 정의

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_header_body.png?raw=true)

#### 로그 종류(action)에 대한 정의
\#action 아래 action key로 사용할 key 정의<br/>
두 개의 key 조합 설정 가능 <br/>
- example: page_id:action_id 

#### Header List 정의
header는 모든 action에서 동일하게 입수할 값<br/>
server log schema 작성시 header의 첫번째 값은 log_time(YYYYMMDDHH*)을 사용(입수 시스템에서 partition 분할에 사용)

* action별 header 정의
  - header key값을 입력한 경우 : 해당 key의 rule로 검증하겠음
  - 비어있는 경우 : 해당 key를 사용하지 않겠음, 실제 로그엔 빈칸으로 기록되어야 함
  - \#bypass 태그를 입력한 경우 : 해당 key엔 어떤 값이 들어와도 상관없음. 룰 검증을 하지 않겠음
  - **header엔 list,map type의 \#key은 사용 불가**

#### Body Field 정의
로그 종류(action)별로 header list 외에 입수할 #key 나열


#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#action 태그** : 액션명
* **\#header 태그** : header시작 지점 정의
* **\#body 태그** : body시작 지점 정의


## code \#maplist
validation rule에서 사용할 key-value data를 정의, code['key']으로 접근 가능<br/>
MakeSentinel 시 key-value-description은 hive table로 export되어 다른 통계에 사용될 수 있음<br/>

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_code_map_list.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#end 태그** : 종료 row 정의
* **\#key 태그** : key, 중복가능
* **\#value 태그** : value, 동일 key에 대해서는 unique
* **\#description 태그** : value에 대한 설명 작성




