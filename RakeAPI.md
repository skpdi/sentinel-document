# RakeAPI-Java
RakeAPI는 Java 서버 로그 수집을 도와주는 API입니다.
Java 서버에서 발생한 로그를 Kafka Broker로 전송합니다.

## 장점
1. Kafka에 대한 별도의 지식 없이 Kafka Broker로 메시지 전송이 가능합니다.
2. 내부적으로 FileQueue를 사용하기 때문에 메시지 가능성을 줄여줍니다.
3. FileQueue Write와 Kafka Send부분을 비동기로 구성하여 서버의 delay를 줄였습니다.
4. 내부적으로 Monitoring 로그를 전송하기 때문에 입수 현황을 체크할 수 있습니다.

## 환경
1. 로컬 환경과 SK planet의 개발 및 상용 서버로 전송이 가능합니다.
2. 개발 Broker로 로그 전송은 SK planet OA망 및 IVPN환경에서 전송이 가능합니다.
3. 상용 Broker로 로그 전송은 SK planet 커머스 존에서 전송이 가능합니다.
4. Kafka Producer API 0.9.0.1을 사용합니다.
5. FileQueue는 Netflix의 Suro 프로젝트에도 사용했던 BigQueue 라이브러리를 사용합니다.
  https://github.com/bulldog2011/bigqueue

## 사용법
### 사전 작업
#### 1. RakeAPI 서버는 호스트 이름을 기준으로 연결되므로, 아래 호스트에 접근할 수 없는 경우, 호스트 파일에 등록해 주어야 합니다.
##### 호스트 이름이 등록되어 있는지 확인은 Linux command창에서 아래의 명령어로 확인이 가능합니다.
```
개발 Broker : nslookup dici-devbroker01~05.is.skp
```
```
상용 Broker : nslookup dicc-broker01~05.is.skp
```
##### 호스트 이름이 등록되어 있지 않는 경우 /etc/hosts 파일에 아래와 같이 등록합니다.
* 라이브 Brokers
```
172.22.212.21 dicc-broker01-172.cm.skp
172.22.212.22 dicc-broker02-172.cm.skp
172.22.212.23 dicc-broker03-172.cm.skp
172.22.212.24 dicc-broker04-172.cm.skp
172.22.212.25 dicc-broker05-172.cm.skp
172.22.212.26 dicc-broker06-172.cm.skp
172.22.212.27 dicc-broker07-172.cm.skp
172.22.212.28 dicc-broker08-172.cm.skp
172.22.212.29 dicc-broker09-172.cm.skp
172.22.212.30 dicc-broker10-172.cm.skp
```

* 개발 Brokers
```
172.21.0.101 dici-devbroker01.is.skp
172.21.0.102 dici-devbroker02.is.skp
172.21.0.103 dici-devbroker03.is.skp
172.21.0.104 dici-devbroker04.is.skp
172.21.0.105 dici-devbroker05.is.skp
```

#### 2. 전송하려는 서버와 Kafka Broker간의 방화벽이 열려있어야 합니다.
방화벽 신청 : http://secuportal.skplanet.com/request/FDS
* 상용 Brokers
```
IP : 172.22.212.21~30 , PORT : 9092
```

* 개발 Brokers
```
IP : 172.21.0.101~105 , PORT : 9092
```
#### 3. 라이브러리 설정
* Maven
~~~~
...
<dependencies>
...
    <dependency>
        <groupId>com.rake.rakeapi</groupId>
        <artifactId>rake-core</artifactId>
        <version>0.2.0</version>
    </dependency>
...
</dependencies>
...
<repositories>
...
    <repository>
        <id>skp-public</id>
        <name>skplanet public repository</name>
        <url>http://mvn.skplanet.com/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
...
</repositories>
~~~~

* Gradle
~~~~
repositories {
    maven { url "http://mvn.skplanet.com/content/groups/public" }
}

dependencies {
    compile('com.rake.rakeapi:rake-core:0.2.0')
}
~~~~

#### 4. 클라이언트 사용
일반적인 설정 값들은 모두 Properties를 통해서 넘겨줍니다.
LoggerAPI는 싱글톤으로 프로세스당 하나만 생성되며, 이것을 통해 데이터를 전송하게 됩니다.
예제 코드는 아래와 같고, 상용 로그 수집의 경우, topic(Kafka Topic)을 사전에 협의하여 사용하여야 수집이 가능합니다.
(topic의 경우, 변경될 가능성이 있으므로, 설정으로 빼기를 권장) 테스트의 경우, 어떤 topic도 사용할 수 있으며, ConsoleSender를 사용하여, 전송되는 데이터를 콘솔로 확인할 수 있습니다.

> properties의 "api.type" 과 "service.id"는 필수로 넣어줘야 하는 설정 값입니다.
>> api.type
- "proto" - 로컬 콘솔 출력
- "dev" - 개발 (OA,IVPN) Broker로 전송 
- "live" - 상용 (커머스) Broker로 전송 
- 세 가지 중 하나를 선택하여 입력합니다.

>> service.id
- service.id 서버의 인스턴스 명을 넣으시면 됩니다. 
- service.id 예시 : Test-Camera서비스를 3개의 톰캣 인스턴스로 구성할 때, ```Test-Camera01, Test-Camera02, Test-Camera03``` 처럼 같은 서비스 명을 가져도 3개의 인스턴스가 서로 구분 되도록 네이밍을 하는 것이 바람직합니다.

~~~~
Properties properties = new java.util.Properties();
properties.setProperty("api.type", "live"); // 필수 설정 값입니다. proto, dev, live 중 한 가지를 입력해야 합니다.
properties.setProperty("service.id", "rakeapi-live01"); // 필수 설정 값 입니다. service중인 서버 이름을 넣어야 합니다.
LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
loggerAPI.log("topic", "log");
~~~~

#### 5. 코드 예제
~~~~
package com.rake.rakeapi;


import org.junit.Test;
import java.util.Properties;
import static org.junit.Assert.assertEquals;

// Daemon으로 항상 띄워놓고 데이터를 전송할 때, 예제
public class LoggerAPIDaemonExample {

    // For Proto
    public void SyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "proto");
        properties.setProperty("service.id", "rakeapi-proto01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // proto_topic의 경우, 콘솔로만 출력되므로, 임의로 지정하여도 됨
        // test_log가 실제 저장되는 데이터. 센티넬을 사용할 경우, sentinel.toString()을 사용하면 됨
        // 콘솔로 출력될 때, {"service_id":"proto_topic","log":"test_log"} 로 보이는데 이것은 디버깅 용이므로, test_log부분만 정확하면 됨
        loggerAPI.log("proto_topic", "test_log");
    }

    // For Dev
    public void AsyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "dev");
        properties.setProperty("service.id", "rakeapi-dev01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // dev_topic의 경우, 기존의 아이디와 겹치지만 않으면 되지만, 다른 팀과 겹칠 수 있으므로, 별도 협의 필요
        // dev_log가 실제 저장되는 데이터. 센티넬을 사용할 경우, sentinel.toString()을 사용하면 됨
        loggerAPI.log("dev_topic", "dev_log");
    }
    // For Live
    public void AsyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "live");
        properties.setProperty("service.id", "rakeapi-live01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // live_topic의 경우, 별도 협의 필요
        // live_log가 실제 저장되는 데이터. Sentinel-Shuttle 을 사용할 경우, sentinel.toString()을 사용하면 됨
        // 서버용 Sentinel-Shuttle 사용법은 http://wiki.skplanet.com/pages/viewpage.action?pageId=73762456 참고
        loggerAPI.log("live_topic", "live_log");
    }
}

// Batch로 데이터를 처리하여 보낼 경우, 예제
public class LoggerAPIBatchExample {

    // For Proto
    public void SyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "proto");
        properties.setProperty("service.id", "rakeapi-proto01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // proto_topic의 경우, 콘솔로만 출력되므로, 임의로 지정하여도 됨
        // test_log가 실제 저장되는 데이터. 센티넬을 사용할 경우, sentinel.toString()을 사용하면 됨
        // 콘솔로 출력될 때, {"service_id":"proto_topic","log":"test_log"} 로 보이는데 이것은 디버깅 용이므로, test_log부분만 정확하면 됨

        // ... 파일을 열거나, reader를 통해서 한 줄 씩 읽으면서 아래 코드를 호출
        loggerAPI.log("proto_topic", "test_log");

        // ... 보내고자 하는 데이터를 전부 보낸 뒤에 아래 코드를 호출하면, 모두 전송될때까지 기다림
        loggerAPI.close(true);

    }

    // For Dev
    public void AsyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "dev");
        properties.setProperty("service.id", "rakeapi-dev01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // dev_topic의 경우, 기존의 아이디와 겹치지만 않으면 되지만, 다른 팀과 겹칠 수 있으므로, 별도 협의 필요
        // dev_log가 실제 저장되는 데이터. 센티넬을 사용할 경우, sentinel.toString()을 사용하면 됨

        // ... 파일을 열거나, reader를 통해서 한 줄 씩 읽으면서 아래 코드를 호출
        loggerAPI.log("dev_topic", "test_log");

        // ... 보내고자 하는 데이터를 전부 보낸 뒤에 아래 코드를 호출하면, 모두 전송될때까지 기다림
        loggerAPI.close(true);
    }

    // For Live
    public void AsyncLoggerAPITest() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("api.type", "live");
        properties.setProperty("service.id", "rakeapi-live01");
        private LoggerAPI loggerAPI = LoggerAPI.getInstance(properties);
        // live_topic의 경우, 별도 협의 필요
        // live_log가 실제 저장되는 데이터. Sentinel-Shuttle 을 사용할 경우, sentinel.toString()을 사용하면 됨
        // 서버용 Sentinel-Shuttle 사용법은 http://wiki.skplanet.com/pages/viewpage.action?pageId=73762456 참고
        // ... 파일을 열거나, reader를 통해서 한 줄 씩 읽으면서 아래 코드를 호출
        loggerAPI.log("live_topic", "test_log");

        // ... 보내고자 하는 데이터를 전부 보낸 뒤에 아래 코드를 호출하면, 모두 전송될때까지 기다림
        loggerAPI.close(true);
   }
}
~~~~

## Exception
아래의 예외 상황은 RakeAPI 사용시 발생할 수 있는 예외들 입니다.
* IllegalArgumentException :
Properties가 잘못 됐을 때 발생합니다.
필수 property인 오타나 api.type과 service.id를 명시하지 않았을 때 발생합니다.

* InvocationException :
RakeAPI는 Queue와 Sender 클래스를 동적으로 로딩합니다.
동적 로딩이 실패했을 때 발생하는 예외입니다.
동적 로딩이 실패하는 경우는 보통 Sender에서 발생하고, Sender생성시 전송해야 하는 호스트 네임을 찾지 못할 때 발생합니다.
위 목록에서 '사전 작업'을 참조하여 전송할 Kafka Broker의 호스트 네임을 등록해야 합니다.

* ClassNotFoundException :
동적 로딩 시 Queue또는 Sender 클래스를 찾지 못할 때 발생합니다.
정상적으로 배포됐을 시 위 예외가 발생하지 않지만, 만일 ClassNotFoundException가 발생했다면 DI팀에 문의 하셔야 합니다.
http://jira.skplanet.com/browse/DI/?selectedTab=com.atlassian.jira.jira-projects-plugin:summary-panel
