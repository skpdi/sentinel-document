# Sentinel Shuttle

Shuttle 은 로그의 포맷을 담당합니다. 정의된 로그의 설계도 대로, 기록해야 하는 컬럼을 Setter 메소드를 이용해서 편리하게 남길 수 있습니다. Shuttle 은 [센티넬 홈페이지](http://sentinel.skplanet.co.kr:8080/) 에서 다운받을 수 있습니다.

Shuttle 은 포맷을 담당하고, 전송을 위해서는 Rake 를 사용해야 합니다. Rake 관련해서는 [Rake API Document](https://github.com/skpdi/rake-document/wiki) 를 참조해주세요.

**주의사항** 

이 문서는 Client 용 Shuttle 의 사용법을 다룹니다. Server 용 Shuttle 은 의존성도 있으며, 사용법도 다릅니다. DI 팀에 문의해주세요.

## Usage

### Java (Client)

Android APK 난독화 시에는, Shuttle 을 난독화 대상에서 **반드시** 제외해야 합니다. 

```
-keep class com.skplanet.pdp.sentinel.shuttle.** { *; }
```

Shuttle **jar** 파일을 클래스패스에 포함하고 아래처럼 사용할 수 있습니다. 

```java
// 주의사항: 이 코드는 클라이언트 셔틀의 사용법을 다룹니다. 
// 서버용 셔틀은 toJSONObject() 가 아니라 toString() 을 호출해주세요. toJsonString() 이 아닙니다!

// 생성
SampleSentinelShuttle shuttle = new SampleSentinelShuttle();

// 필드값 세팅
// 기록해야 하는 컬럼과 같은 이름을 가진 setter 메소드를 이용할 수 있습니다.
// 예를 들어 `page_id` 값을 기록해야 한다면 `page_id` 메소드를 이용하면 됩니다. 
shuttle.page_id("/main/card/list").action_id("tap.my_card").card_num("2012-3XXX-XXXX-XXXX").card_company("Hana SK").expired_date("MM/YY");

// 그러나 위 방법보다는 setBodyOf 메소드를 이용하는 것을 권장합니다.
// 위 처럼 사용할 경우, 실수로 메소드 호출을 누락할 수 있기 때문입니다.
// setBodyOf 메소드는 필요한 컬럼들을 모두 파라미터로 넘겨야 하기 때문에, 실수로 누락할 일이 없습니다.
// setBodyOf 메소드를 이용하면, page_id, action_id 는 자동으로 기록됩니다. 이외의 Header 값은 체이닝해서 사용해야 합니다.
shuttle.setBodyOfMain_card_list__tap_my_card("2012-3XXX-XXXX-XXXX", null /* 값을 기록하고 싶지 않은 경우 */,"MM/YY");

// session_id 헤더 값을 기록하고, 나머지 Body 값을 setBodyOf 메소드를 이용해서 기록하는 경우
shuttle.session_id("AF0EF").setBodyOfMain_card_list__tap_my_card("2012-3XXX-XXXX-XXXX", null ,"MM/YY");

// clearBody 메소드를 이용해서, 값을 비울 수 있습니다.
shuttle.clearBody();
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-Android API](https://github.com/skpdi/rake-document/wiki/1.-Rake-Android) 참고)

```java
rake.track(shuttle.toJSONObject());
```

디버깅을 위해 `toString` 메소드를 사용할 수 있습니다.

```java
logger.debug(shuttle.toString());
```

<br/>

### Objective-C (Client)

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

// clearBody 메소드를 이용해서, 값을 비울 수 있습니다.
[shuttle clearBody];
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-iPhone API](https://github.com/skpdi/rake-document/wiki/2.-Rake-iPhone) 참고)

```objective-c
[rake track:[shuttle toNSDictionary]];
```

디버깅을 위해 `toString` 메소드를 사용할 수 있습니다.

```objective-c
NSLog(@"%@",[shuttle toString]);
```
<br/>

### Javascript

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


// clearBody 메소드를 이용해서, 값을 비울 수 있습니다.
shuttle.clearBody();
```

<br/>

**Rake** 에 값을 기록한 Shuttle 을 넘겨주려면 다음처럼 사용합니다. ([Rake-Web API](https://github.com/skpdi/rake-document/wiki/3.-Rake-Web) 참고)

```javascript
mixpanel.track("", shuttle.getImmutableJSONObject());
```
