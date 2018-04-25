Sentinel Schema를 통해 정의된 규격대로 로그를 남길 수 있도록 Format 을 담당하는 라이브러리입니다.
기록해야 하는 컬럼을 Setter 메소드를 이용해서 편리하게 남길 수 있습니다.

[센티넬 홈페이지](http://sentinel.skplanet.com:9090/)에서 다운 받을 수 있습니다. 
Shuttle 은 포맷만 담당하며, 전송을 위해서는 [Rake](https://github.com/skpdi/rake-document/wiki)와 같이 사용해야 합니다.

- Client용 Shuttle (Java, Objective-C, JavaScript)
- Server용 Shuttle (Java)

## 효과
- 필드 이름 오타를 방지할 수 있습니다. (IDE 자동완성)
- 필드 값 escaping 을 지원합니다. (\n, \r, 기타 이상한 아스키 코드등)
- 필드 타입 불일치 방지 (Integer 가 필요한데, String 을 넣으면 컴파일이 되지 않습니다.)
- 로그 버전 자동 생성
- 로그버전 형식 : #define시트내의 #version : shuttle version : schema version (공백없이 ":"로 concat)
- 로그 형식(JSON/TSV) 자동 생성

## 주의사항 - Client Shuttle 과 Server Shuttle 차이
- Shuttle 은 스레드 세이프 하지 **않습니다**. 멀티 스레드 이슈가 있는 위치에서는 (e.g *Controller*) 매번 생성하거나, 무거울 경우 [ThreadLocal](https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html), Spring scope 등을 이용해주세요.
- Client 용 Shuttle 은 Android, iOS, Javascript 버전을 지원하며, 의존성이 없습니다.
- Server 용 Shuttle 은 Java 버전만 제공합니다. 의존성도 있으며, Shuttle.toString 을 사용해서 로깅합니다. (*toJSONObject* 가 아닙니다)

## Schema Type - Shuttle Type

| Header / Body | Schema Type (= Hive Table Type) | Java Type (괄호 안은 기본값) | Javascript | Objective-C | Note |
| ------------- | ------------------------------- | ------------------------- | ---------- | ----------- | ---- |
| Header / Body | string                          | String (*null*) | var | NSString* (*nil*) | |
| Header / Body | int                             | Long (*null*) | var | NSNumber* (*nil*) | |
| Header / Body | float                           | Double (*null*) | var | NSNumber* (*nil*) | |
| Header Only   | fixed string(n)                 | String (*null*) | var | NSString* (*nil*) | |
| Body Only     | list\<int\>                     | List\<Long\> (*new ArrayList\<Long\>*) | var | NSMutableArray* (*nil*) | |
| Body Only     | list\<float\>                   | List\<Double\> (*new ArrayList\<Double\>*) | var | NSMutableArray* (*nil*) | |
| Body Only     | list\<string\>                    | List\<String\> (*new ArrayList\<String\>*) | var | NSMutableArray* (*nil*) | |
| Body Only     | map\<int\>                        | Map\<String, Long\> (*new LinkedHashMap\<String, Long\>*) | var | NSMutableDictionary* (*nil*) | |
| Body Only     | map\<float\>                      | Map\<String, Double\> (*new LinkedHashMap\<String, Double\>*) | var | NSMutableDictionary (*nil*) | |
| Body Only     | map\<string\>                     | Map\<String, String\> (*new LinkedHashMap\<String, String\>*) | var | NSMutableDictionary* (*nil*) | |
| Body Only     | json                            | JSONObject (*null*) | var | NSMutableDictionary* (*nil*) | 암호화 및 검증 지원 불가 |

<br/>

> Body 컬럼의 타입으로 JSON 타입을 사용하는 경우 다음과 같은 문제가 발생할 수 있습니다. 

> Body가 1depth 이기 때문에, 이 경우 2 depth 이상이 됩니다.)
> json 의 depth가 2 이상인 값에 대해 암호화를 지원하지 않습니다.
> Json 의 depth가 2 이상인 값에 대해 검증을 지원하지 않습니다.

> 위 사항에 대해서 고려하시고 사용하셔야 하며, 로그 암호화나 검증 문제는 DAS/DA 와 상의하셔야 합니다.
 
> 참고로 여러 depth의 json 값도 Hive의 get_json_object() 함수를 이용해 추출이 가능합니다.

<br/>

## Usage

### Java (Client, Server)

#### 난독화 관련

Android APK 난독화 시에는, Shuttle 을 난독화 대상에서 **반드시** 제외해야 합니다. 

```
-keep class com.skplanet.pdp.sentinel.shuttle.** { *; }
```

<br/>

#### default value (기본값), null, 빈 문자열(Empty String) 관련

| Field: Header / Body | Shuttle: Client / Server |  INPUT: none / `null` / `""` | STORAGE: Kafka, HDFS | OUTPUT | note | migration |
| -------------------- | ------------------------ | ---------------------------- | -------------------- | -------| ----- | ----- |
| Header | Client | none (default value, 아무것도 입력하지 않은 경우) | Kafka (중간단계) | `""` | 빈값을 `""` 로 표현 | 추후 `JSONObject.NULL` 로 변경예정. `JSONObject.NULL` 로 빈값을 표현하게 되면, Client 셔틀로 전송된 로그는 `""` 를 HDFS 에서 *TSV* 로 저장할 때 `''` 빈 문자로 변경됨 |
| Header | Client | `null` | Kafka (중간단계) | `""` | 빈값을 `""` 로 표현 | 추후 `JSONObject.NULL` 로 변경예정. `JSONObject.NULL` 로 빈값을 표현하게 되면, Client 셔틀로 전송된 로그는 `""` 를 HDFS 에서 *TSV* 로 저장할 때 `''` 빈 문자로 변경됨 |
| Header | Server | `null` | Kafka (중간단계) | `""` | 빈값을 `""` 로 표현 | 서버 로그는 중간 단계부터 *TSV* 포맷 |
| Header | Server | none (default value, 아무것도 입력하지 않은 경우) | Kafka (중간단계) | `""` | 빈값을 `""` 로 표현 | 서버 로그는 중간 단계부터 *TSV* 포맷 |
| Header | Header | `""`  (empty string) | Kafka (중간단계) | `""` | | |
| Header | Header | ALL | HDFS (최종단계) | `''` | *TSV* 포맷으로 로그를 저장하므로 중간단계의 `""` 가 `''` (빈 문자, 값이 없음) | |
| Body | Client | none | ALL | 키 삭제 | 바디에서는 빈 값을 표현하기 위해 해당 키를 삭제 | 추후 `JSONObject.NULL` 로 변경예정 |
| Body | Client | `null` | ALL | 키 삭제 | 바디에서는 빈 값을 표현하기 위해 해당 키를 삭제 | 추후 `JSONObject.NULL` 로 변경예정 |
| Body | Server | none | ALL | 키 삭제 | 바디에서는 빈 값을 표현하기 위해 해당 키를 삭제 | |
| Body | Server | `null` | ALL | 키 삭제 | 바디에서는 빈 값을 표현하기 위해 해당 키를 삭제 | |
| Body | ALL | `""` | ALL | `""` | | |

정리하면, 셔틀에 **빈 값** 을 표현하기 위해 `null` 값을 이용할 수 있으나, *Header* (*TSV*) 와 *Body* (*JSON*) 에서 **빈 값이 다르게 표현되고**, 저장된 위치 (Kafka 또는 HDFS) 에 따라 다르니 알고 사용하시면 됩니다.

<br/>

#### 사용법
Shuttle **jar** 파일을 클래스패스에 포함하고 아래처럼 사용할 수 있습니다. 

- 서버용 Shuttle 의 경우 암호화 관련 의존성이 있습니다. [서버용 셔틀 사용법](http://wiki.skplanet.com/pages/viewpage.action?pageId=73762456) 페이지 참조 부탁드립니다. (내부망)

```java
// 생성
SampleSentinelShuttle shuttle = new SampleSentinelShuttle();

// 헤더 또는 바디에 기록하기 위해 같은 이름을 가진 setter 메소드를 이용할 수 있습니다.
// 예를 들어 `page_id` 값을 기록해야 한다면 `page_id` 메소드를 이용하면 됩니다. 
shuttle.page_id("/main/card/list").action_id("tap.my_card").card_num("2012-3XXX-XXXX-XXXX").card_company("Hana SK").expired_date("MM/YY");

// `setBodyOf` 메소드를 이용해서 한번에 세팅할 수 있습니다. 이 경우 메소드 이름처럼 *Body* 만 세팅되고, 
// 메소드 이름에 나와있는 *KEY* 헤더값이 자동 세팅됩니다. (이 예제에서는 `page_id`, `action_id` 가 `main_card_list`, `tap_my_card` 로 세팅)
shuttle.setBodyOfMain_card_list__tap_my_card("2012-3XXX-XXXX-XXXX", null /* 값을 기록하고 싶지 않은 경우 */,"MM/YY");

// session_id 헤더 값을 기록하고, 나머지 Body 값을 setBodyOf 메소드를 이용해서 기록하는 경우
shuttle.session_id("AF0EF").setBodyOfMain_card_list__tap_my_card("2012-3XXX-XXXX-XXXX", null ,"MM/YY");

// Sentinel에서 `setBodyOf` 메소드를 사용하지 않음을 표기한 경우에는 `setBodyByJSonString` 메소드를 이용해 Body 값을 기록 할 수 있음
// (!) 사용을 권장하지 않습니다.
// 주의사항
// - 세팅한 값이 올바른 JSon 형태의 String이 아니라면, JSonObject로 parsing이 되지 않아 `toJSonObject` 메소드 이용시 빈 body가 리턴 됨
// - 정의하지 않은 key값을 넣어도 오류가 발생하지 않음
// - body에 `set` 메소드를 사용 후 `setBodyByJSonString` 메소드를 이용하면 기존 `set` 메소드로 세팅한 Body 값이 사라짐
// - `set` 메소드로 세팅 후 `setBodyByJSonString` 메소드를 이용하여 Body를 재 세팅하면 `set` 메소드로 세팅한 값을 `get` 메소드로 얻을 수는 있지만, toJSONObject() 메소드나, toString() 메소드 호출 시에는 `setBodyByJSonString`의 Body값만이 출력되게 됨.  
shuttle.setBodyByJSonString("{\"auth_no\":\"A213213\",\"card_order\":1,"\"app_version\":\"2.3.1\"}");
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-Android API](https://github.com/skpdi/rake-document/wiki/1.-Rake-Android-(%ED%95%9C%EA%B5%AD%EC%96%B4)) 참고)

```java
// 서버용 셔틀은 `toJSONObject()` 가 아니라 `toString()` 을 호출해주세요. 
// 서버용 셔틀 위키 - http://wiki.skplanet.com/pages/viewpage.action?pageId=73762456

rake.track(shuttle.toJSONObject());
```

디버깅을 위해 `toString` 메소드를 사용할 수 있습니다.

```java
logger.debug(shuttle.toString());
```

<br/>

### Objective-C (Client Only)

Shuttle 파일을 프로젝트에 포함하고 다음처럼 사용할 수 있습니다.

```objective-c
// 주의사항: 이 코드는 클라이언트 셔틀의 사용법을 다룹니다. 

// 생성
SampleSentinelShuttle *shuttle = [[SampleSentinelShuttle alloc] init];

// 필드(컬럼) 값 세팅
// 기록해야 하는 컬럼과 같은 이름을 가진 setter 메소드를 이용할 수 있습니다.
// 예를 들어 `page_id` 값을 기록해야 한다면 `page_id` 메소드를 이용하면 됩니다. 
[shuttle page_id:@"/main/card/list"];
[shuttle action_id:@"tap.my_card"];

// 그러나 위 방법보다는 setBodyOf 메소드를 이용하는 것을 권장합니다.
// 위 처럼 사용할 경우, 실수로 메소드 호출을 누락할 수 있기 때문입니다.
// setBodyOf 메소드는 필요한 컬럼들을 모두 파라미터로 넘겨야 하기 때문에, 실수로 누락할 일이 없습니다.
// setBodyOf 메소드를 이용하면, page_id, action_id 는 자동으로 기록됩니다. 이외의 Header 값은 직접 체이닝해서 사용해야 합니다.
[shuttle setBodyOf_allcoupon_brand_brandcoupon__list_tap_coupon_with_brand_id:@"123123"];

// Sentinel에서 `setBodyOf` 메소드를 사용하지 않음을 표기한 경우에는 `bodyByJSonString` 값을 세팅해 Body 값을 기록 할 수 있음
// (!) 사용을 권장하지 않습니다.
// 주의사항
// - 세팅한 값이 올바른 JSon 형태의 NSString이 아니라면, `toString` 메소드나 `toNSDictionary` 메소드 이용시 JSonObject로 parsing이 되지 않아 빈 body가 리턴 됨 `{}`
// - 정의하지 않은 key값을 넣어도 오류가 발생하지 않음
// - body에 `setter` 메소드를 사용 후 `bodyByJSonString setter` 메소드를 이용하면 기존 `setter` 메소드로 세팅한 Body 값이 사라짐
// - `setter` 메소드로 세팅 후 `bodyByJSonString setter` 메소드를 이용하여 Body를 재 세팅하면 `setter` 메소드로 세팅한 값을 `getter` 메소드로 얻을 수는 있지만, `getImmutableJSONObject` 메소드나 호출 시에는 `bodyByJSonString`의 Body값 만이 출력되게 됨.
[shuttle bodyByJSonString:@"{\"auth_no\":\"A213213\",\"card_order\":1,","\"app_version\":\"2.3.1\"}"];                     

// clearBody 메소드를 이용해서, 값을 비울 수 있습니다.
[shuttle clearBody];
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-iPhone API](https://github.com/skpdi/rake-document/wiki/2.-Rake-iPhone-(%ED%95%9C%EA%B5%AD%EC%96%B4)) 참고)

```objective-c
[rake track:[shuttle toNSDictionary]];
```

디버깅을 위해 `toString` 메소드를 사용할 수 있습니다.

```objective-c
NSLog(@"%@",[shuttle toString]);
```
<br/>

### Javascript (Client Only)

Shuttle 파일을 \<script\> 를 이용해서 로드 한 뒤에 다음처럼 사용할 수 있습니다.

```javascript
// 주의사항: 이 코드는 클라이언트 셔틀의 사용법을 다룹니다. 

// 생성
var shuttle = new SampleSentinelShuttle();

// 필드(컬럼) 값 세팅
// 기록해야 하는 컬럼과 같은 이름을 가진 setter 메소드를 이용할 수 있습니다.
// 예를 들어 `page_id` 값을 기록해야 한다면 `setPage_id` 메소드를 이용하면 됩니다. 
shuttle.setPage_id("new page").setAction_id("hello action");


// 그러나 위 방법보다는 setBodyOf 메소드를 이용하는 것을 권장합니다.
// 위 처럼 사용할 경우, 실수로 메소드 호출을 누락할 수 있기 때문입니다.
// setBodyOf 메소드는 필요한 컬럼들을 모두 파라미터로 넘겨야 하기 때문에, 실수로 누락할 일이 없습니다.
// setBodyOf 메소드를 이용하면, page_id, action_id 는 자동으로 기록됩니다.
shuttle.setBodyOf_event_purchase__event_purchase("event_name","purchase_id", purchase_amount);

// setBodyOf 메소드는 Body 값을 기록하므로 추가적인 Header 값을 기록하기 위해서는 다음처럼 체이닝해서 사용합니다.
shuttle.setSession_id("AFD0104").setBodyOf_event_purchase__event_purchase("event_name","purchase_id", purchase_amount);

// Sentinel에서 `setBodyOf` 메소드를 사용하지 않음을 표기한 경우에는 `bodyByJSonString` 값을 세팅해 Body 값을 기록 할 수 있음
// (!) 사용을 권장하지 않습니다.
// 주의사항
// - 세팅한 값이 올바른 JSon 형태의 string이 아니라면, `getImmutableJSONObject` 메소드 이용시 parsing이 되지 않아 빈 body가 리턴 됨
// - 정의하지 않은 key값을 넣어도 오류가 발생하지 않음
// - body에 `set` 메소드를 사용 후 `setBodyByJSonString` 메소드를 이용하면 기존 `set` 메소드로 세팅한 Body 값이 사라짐
// - `set` 메소드로 세팅 후 `setBodyByJSonString` 메소드를 이용하여 Body를 재 세팅하면 `getImmutableJSONObject` 메소드 호출 시에는 `setBodyByJSonString`로 세팅한 Body 값 만이 출력되게 됨.  
shuttle.setBodyByJSonString("{\"auth_no\":\"A213213\",\"card_order\":2,\"app_version\":\"2.3.1\"}");

// clearBody 메소드를 이용해서, 값을 비울 수 있습니다.
shuttle.clearBody();
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-Web API](https://github.com/skpdi/rake-document/wiki/3.-Rake-Web-(%ED%95%9C%EA%B5%AD%EC%96%B4)) 참고)

```javascript
rake.track(shuttle.getImmutableJSONObject());
```

## 관련페이지
- [LogAgent 사용자를 위한 Shuttle 포맷 가이드](http://wiki.skplanet.com/pages/viewpage.action?pageId=73762119)
- [Sentinel Shuttle Maintainer Guide](http://wiki.skplanet.com/display/DIT/Sentinel+Shuttle+Maintainer+Guide)
- [Sentinel Shuttle에서 사용 가능한 타입,예약어,코드](http://wiki.skplanet.com/pages/viewpage.action?pageId=73792210)
- [Server 용 Shuttle 사용법 - 암호화 의존성 추가](http://wiki.skplanet.com/pages/viewpage.action?pageId=73762456)



