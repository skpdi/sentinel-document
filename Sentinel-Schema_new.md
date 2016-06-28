# New Sentinel-Schema 설계문서 작성 manual
## Intro
* Sentinel Schema에서, 로그로 남길 데이터를 정의합니다.
* Header & Body format
  * **Header** : 모든 로그에 남는 정보(시간, device info, log version, ...) 
  * **Body** : 로그를 남기는 상황별로 달라지는 데이터의 종류(body 필드)를 기술할 수 있습니다.
    * user action, request/response, event triggered, ... 
  * 입수가 시작된 이후에도 body 에 자유롭게 필드추가가 가능해 확장성이 좋습니다.
    * 업데이트가 잦은 모바일 서비스 앱에 적합합니다. 
  * tsv 로 구분된 header 와 json string 인 body로 구성
    * example: Header A,B,C 필드, body D 필드 <br/> A_value B_value C_value {'D':'D_value'} 
    * Hive 에서 Body 필드 조회시 [get_json_object UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object)를 사용하여 값을 가져올 수 있습니다.

* SKP DIC Infra에서 제공하는 것들과 연계됩니다.
  * **RakeClient** : App/Web 단말에서 단말 로그를 직접 전송합니다.
   * 자동으로 수집되는 항목들이 있습니다. (base_time, recv_time, os_name, os_version, resolusion, ...)
  * **RakeAPI** : 운영 Server 단에서 로그를 바로 전송합니다.
  * **Shuttle** : 스키마와 연결되어 로그 값 입력을 실수없이 편리하게 할 수 있도록 합니다.
  * **암호화** : 암호화가 필요한 필드를 스키마에서 표시해두고 셔틀을 사용해 전송하면, DIC Infra에서 제공하는 암호화가 적용되어 적재됩니다.
* 입수된 로그에 대해서 간단한 검증을 할 수 있습니다.
  * 상황별로 정의한 body 필드가 모두 있는지
  * 각 필드 값에 대해, 
    * 정의한 type 대로 들어왔는지
    * 정의한 length 범위 내의 값의 길이를 갖고 있는지
    * not null 인 경우 빈값('' or JSON Null)이 아닌지
    * 기타 검증기능(date format, regex, one of defined value set, ...)을 제공합니다. 


## 필드 종류 및 Hive column 이해
* #infra 시트 #systemHeader 필드
  * 모든 서비스 로그(SKP 모든 서비스)에서 공통적으로 수집하는 필드
  * log_time, log_version, (Client Project) device_id, device_model, resolution, ...
* #dictionary 시트의 header 필드
  * 특정 서비스 로그(syrup, gifticon, ...)에서 공통적으로 수집하는 필드
  * channel(로그입수 PoC), page_id, action_id, ... 
* #dictionary 시트의 body 필드
  * 특정 서비스 로그에서 상황에 따라 다르게 수집하는 필드
  * display_order, status_code, request_param, ...
* Hive column 순서 : #infra 시트 #systemHeader 필드 작성 순서 > #dictionary 시트 header 필드 작성 순서 > 'body' column
* example : #infra 시트의 systemHeader 필드 정의(#key: A,B,C), #dictionary 시트 header 필드정의(#key: J, K) <br/>
      hive column list : A B C J K body (총 6개 column)


## Sheet 작성 가이드
* \#{tagName} 방식의 태그를 활용하여 작성, 프로그램이 읽을 수 있는 형태로 제작됩니다.
* [Sample Schema 파일](https://docs.google.com/spreadsheets/d/15U63cA3-RLa2lxUW_stpKPYeyVv0zmjYuAvjaOwg9T8/edit?usp=sharing) 참고
* 모든 sheet마다 정해진 태그 입력이 필수입니다.
* 모든 sheet의 시작 row와(\#start) 종료 row에(\#end) 태깅이 필요함
* \#start 태그 바로 다음 row부터 각 sheet의 컨텐츠가 존재해야 함
* \#end 태그 바로 직전 row까지 각 sheet의 컨텐츠가 존재해야 함
* \#start, \#end 태그로 둘러쌓인 네모 블럭의 바깥 top/left/bottom 영역은 자유롭게 사용이 가능힙니다.

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





## \#define 시트
스키마 문서의 기본 정보 정의

<img src="https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_define.png?raw=true" width="500">

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





## \#infra 시트
입수와 관련된 정보가 기록되는 시트, 상용 입수 이후에는 시트 잠금으로 관리자만 수정가능

![Image of Infra](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_infra.png?raw=true)

* 암호화 필드 설정
  * **\#start_encryptFields 태그** : 암호화 필드 블럭의 시작 row 정의
  * **\#key 태그** : 암호화가 필요한 key 이름, #infra 시트나 #dictionary 시트에 정의된 필드의 key 값이어야 합니다.
  * **\#end_encryptFields 태그** : 암호화 필드 블럭의 종료 row 정의

* 자동수집 필드(System Header) 설정
RakeClient 사용시, 자동으로 수집할 필드를 정의합니다. 
(구버젼에서 전환된 경우 dictionary 에 그대로 정의되고 자동수집 기능은 정상적으로 동작합니다.)
  * 신규 스키마 생성시 자동으로 필드리스트가 작성되어 있음
  * 필드 삭제는 불가능
  * 수집 플랫폼에 태깅이 된 경우 값을 입력하여도 자동수집값이 적용됩니다.
   * client 프로젝트의 지원가능 플랫폼 - **\#android, \#iphone, \#web**
   * server 프로젝트의 지원가능 플랫폼 - **\#java**
  * **\#start_systemHeader 태그** : 자동수집 필드 블럭의 시작 row 정의
  * 수집 플랫폼 태그 외 기타 태그는 아래 \#dictionary 시트와 동일합니다.
  * **\#start_systemHeader 태그** : 자동수집 필드 블럭의 종료 row 정의


## \#dictionary 시트
key 목록 정의, key 이름, 타입, 설명, 검증rule, 아래 나열되는 모든 태그가 존재하여야 함

![Image of Dictionary](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_dictionary.png?raw=true)

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#fieldCategory 태그** : 필드 종류에 대한 정의
  * header : 정의한 순서대로, hive column 순서가 됨. 
  * body : header 이후 작성, body 필드에 사용할 필드 정의
  * json_child: body 필드의 #type json body 필드 아래 이어서 작성가능(검증이 필요한 경우에만 작성, 암호화 미지원(지원예정))
* **\#key 태그** : key 이름, human-readable하게 정의
* **\#type 태그** : 필드 type 
  * string : 문자형
  * int : 정수형
  * float : 실수형
  * list\<TYPE\> : json list형, body에서만 사용 가능, 아래 3가지 type 이외의 type(list, object 등등)은 지원안함
    * list\<int\> : 정수형 리스트, ex)[10,20,30]
    * list\<float\> : 실수형 리스트, ex)[1.1,1.3,1.5]
    * list\<string\> :  문자형 리스트,  ex)["a","b","c","d"]
  * map\<TYPE\> : json object 형, body에서만 사용 가능, 아래 3가지 type 이외의 type(list, object 등등)은 지원안함
    * map\<int\> :  정수형 object,  ex){"a":10,"b":20,"c":30}
    * map\<float\> : 실수형 object, ex){"a":1.1,"b":1.3,"c":1.5}
    * map\<string\> :  문자형 object,  ex){"a":"q","b":"w","c":"e"}
  * json : json, body에서만 사용 가능
    * hive 및 검증기에서 json string으로 취급됩니다.
* **\#length 태그** : 값의 length 에 관해 정의가능
  * [fixed length]: 정확하게 일치
  * [min]~[max]: 범위 지정 가능
  * example: 2(일치), ~2(2이하), 2~(2이상), 1~2(1이상2이하)  
* **\#nullableYN 태그** : 빈값을 허용하는 필드를 태그할 수 있다.
  * Y: nullable인 경우, 빈값일때는 검증룰(#type,#length,#rule,..)을 적용하지 않고 통과
* **\#description 태그** : 필드에 대한 설명
* **\#rule 태그** : key의 검증룰, 빈 값인 경우 검증하지 않음
  * UDF(user define function)
    1. **df(date_pattern)** : 시간관련 값 검증 ([Java DateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html))
      - example : df(yyyyMMddHHmmssSSS) 
    2. **code(key in codelist)** : #code 시트에 정의한 code map 이내의 값 중 하나
      - example : code(page_id) <br/> #code 시트에서 #key가 'page_id' 인 [#value] 들 중 하나의 값
    3. **in(values with csv format)** : parameter 에 정의한 값 중 하나
      - example : in(apple,google,amazon) <br/> 'apple', 'google', 'amazon' 중 하나의 값과 일치
    4. **regex(regular_expression)** : 정규식 검증 ([Java Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html), [Regex Tester on Web](https://regex101.com/))
      - example : regex([0-9]{2}\\.[0-9]{2}\\.[0-9]{2})
    5. Logical Syntax
      - **and(rules with csv format)** : AND 조건
      - **or(rules with csv format)** : OR 조건
      - **if(condition, rule if true, rule if false)** : 조건 분기
      - **<,<=,>,>=,=,<>** : equality
      - **@FIELD_NAME** : 다른 필드값 참조, equality 만 적용가능
      - example : transaction_id 필드의 #rule: if(@tech_type=ble,$nonempty,$empty)
        - tech_type 필드의 값이 ble 일 경우, transaction_id 는 빈값이 아니어야 함 
* **\#versionKey 태그** : log version을 정의하는 key, key 이름 뒤에 태깅, key 목록중에서 한 개의 version key가 필요(필수)
  * #infra 시트에 자동수집항목 필드에 정의할 경우, dictionary 에 지정할 수 없음
* **\#end 태그** : 종료 row 정의






## \#layout 시트
로그를 남기는 상황(로그 종류, action)별로 body 필드 리스트 정의
![Image of Layout](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_layout.png?raw=true)

<br/>\#logKey, (#incaseHeader,) \#body 태그 열은 빈 열없이 붙여서 순서에 맞게 작성하여야 함


#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#logKey 태그** : 로그 종류를 구분할 필드 정의
- example: 일반적인 client 프로젝트에서는 'page_id', 'action_id' 가 사용됨
* **\#incaseHeader 태그 (optional)** : 로그 종류별로 header 필드 검증룰을 따로 지정 가능, 정의하지 않아도 됨
  * \#infra systemHeader 와 \#dictionary 에서 header 로 정의된 필드만 사용 가능
  * 특정 액션에만 다른 검증룰 지정 가능
    * 빈 값일 경우, 각 시트에 정의한 일반 필드 #rule을 사용하여 검증
* **\#body 태그** : body시작 지점 정의
  * 로그 종류별로 입수할 #key 나열, **순서와 공백은 무관** 
  * \#dictionary에서 body로 정의된 필드만 허용, 정의된 것 이외의 필드가 들어오는 것은 확인하지 않음
  * 로그 종류별로 body field에 다른 #rule을 적용하고 싶을 경우
    * body_field( => [NEW RULE]) : () 부분을 추가로 작성
* **\#end 태그** : 종료 row 정의

#### body 작성 스타일 가이드
* 두 방식 모두 스키마를 읽어오는 데는 차이가 없음(순서와 공백 무관)
* 샘플 스키마 및 상단 이미지와 같이 왼쪽으로 모두 당겨 공백없이 작성
  * :+1: body field 가 많아져도 body 리스트를 한눈에 인식할 수 있음
* 아래 이미지처럼 필드 별로 열을 맞춰 사용할 수 있음
  * :+1: body 필드 별로 어디서 사용되는지 확인 가능

![Image of Layout](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_body_style.png?raw=true)




## #code 시트
꼭 작성할 필요는 없으며, validation rule에서 code([#key]) 로 접근 가능<br/>
MakeSentinel 시 key-value-description 그대로 hive table로 export되어 다른 통계에 활용될 수 있음<br/>

![Image of Code](https://github.com/skpdi/sentinel-document/blob/master/schema/schema_v2_code.png?raw=true)

ex) A필드의 rule 이 code(auth_type) 인 경우, A 필드가 (ipin, mobile, idpw) 중 하나의 값인지 QA 과정에서 확인가능

#### 사용 태그 목록
* **\#start 태그** : 시작 row 정의
* **\#key 태그** : key, 중복가능
* **\#value 태그** : value, 동일 key에 대해서는 unique
* **\#description 태그** : value에 대한 설명 작성
* **\#end 태그** : 종료 row 정의




