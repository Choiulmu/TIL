# ch5 번외(2). Logback

### 로깅 라이브러리 - Logback

- 로깅(logging)
    - 애플리케이션이 동작하는 동안 시스템의 상태/동작 정보를 시간순으로 기록하는 것
    - spring-boot-starter-web 라이브러리에 내장

- Logback특징
    - 5개의 로그레벨 설정(ERROR, WARN, INFO, DEBUG, TRACE)
    - 실제 운영 환경과 개발 환경에서 각각 다른 출력 레벨을 설정해서 로그 확인 가능
    - Logback 설정 파일을 일정 시간마다 스캔 - 애플리케이션 재가동 없이 설정 변경
    - 자체적으로 로그 파일 압축 가능 & 보관
    - 로그 파일 보관 기간 등 설정 관리 가능

- 5개의 로그레벨 설정
    - ERROR : 애플리케이션 작동이 불가한 정도로 심각한 문제 레벨
    - WARN : 시스템 에러 원인이 될 수 있는 경고 레벨
    - INFO : 애플리케이션 상태 변경과 같은 정보 전달을 위해 사용
    - DEBUG : 애플리케이션의 디버깅을 위한 메시지 표시하는 레벨
    - TRACE : DEBUG레벨보다 더 상세한 메시지 표현을 위한 레벨

### 6-2. Logback 설정 파일 : Appendar영역

- Logback 설정파일 전체 코드
    
    ```java
    //property영역
    <?xml version="1.0" encoding="UTF-8" ?>
    <configuration>
        <property name="LOG_PATH" value="./logs"/>
    
    //Appender영역
        <!--Appenders-->
        <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
            <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                <level>INFO</level>
            </filter>
            <encoder>
                <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
            </encoder>
        </appender>
    
        <appender name="INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <filter class="ch.qos.logback.classic.filter.ThresholdFilter" >
                <level>INFO</level>
            </filter>
            <file>${LOG_PATH}/info.log</file>
            <append>true</append>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/info_${type}.%d{yyyy-MM-dd}.gz</fileNamePattern>
                <maxHistory>30</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
            </encoder>
        </appender>
    
    //Root영역
        <!--TRAVE > DEBUG > INFO > WARN > ERROR > OFF -->
        <!-- Root Logger -->
        <root level="INFO">
            <appender-ref ref="console"/>
            <appender-ref ref="INFO_LOG"/>
        </root>
    </configuration>
    ```
    

```java
...
    <!--Appenders-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter" >
            <level>INFO</level>
        </filter>
        <file>${LOG_PATH}/info.log</file>
        <append>true</append>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/info_${type}.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
    </appender>
...
```

- 로그 형태 설정 및 로그 출력 방법 설정
- Appender라는 인터페이스 하위에 여러 구현체 존재
    
    
    ![[https://velog.io/@gehwan96/logback-설정](https://velog.io/@gehwan96/logback-%EC%84%A4%EC%A0%95)](ch5%20%E1%84%87%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%AC(2)%20Logback%20a616e1208eb64cabb1d9e94fc78a248b/Untitled.png)
    
    [https://velog.io/@gehwan96/logback-설정](https://velog.io/@gehwan96/logback-%EC%84%A4%EC%A0%95)
    
    - ConsoleAppender: 콘솔에 로그를 출력
    - FileAppender: 파일에 로그를 저장
    - RollingFileAppender: 여러개 파일을 순회하면서 로그 저장
    - SMTPAppender: 메일로 로그 전송
    - DBAppender: 데이터베이스에 로그 저장
- encoder 요소를 통해 표현되는 로그의 형식 pattern
    
    EX) patter 예시
    
    ```java
    <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
    ```
    
    - 로그 기록 시간이 yyyy-MM-dd HH:mm:ss.SSS 형태로 출력
    - 기본 로그 레벨 / 왼쪽 정렬 출력
    - (로깅 이벤트가 발생한) 현재 Thread명 출력
    - (로깅 이벤트가 발생한) 패키지 포함 클래스 정보 출력
    - 로그 메시지 출력
    - 줄바꿈 출력
    
    ---
    
    - %logger: 패키지 포함 클래스 정보
    - %logger{0}: 패키지를 제외한 클래스 이름만 출력
    - %logger{length}: Logger name을 축약할 수 있음. {length}는 최대 자리 수, ex)logger{35}
    - %-5level: 로그 레벨, -5는 출력의 고정폭 값(5글자), 로깅레벨이 Info일 경우 빈칸 하나 추가
    - ${PID:-}: 프로세스 아이디
    - %d: 로그 기록시간 출력
    - %p: 로깅 레벨 출력
    - %F: 로깅이 발생한 프로그램 파일명 출력
    - %M: 로깅일 발생한 메소드의 명 출력
    - %line: 로깅이 발생한 호출지의 라인
    - %L: 로깅이 발생한 호출지의 라인
    - %thread: 현재 Thread 명
    - %t: 로깅이 발생한 Thread 명
    - %c: 로깅이 발생한 카테고리
    - %C: 로깅이 발생한 클래스 명 (%C{2}는 somePackage.SomeClass 가 출력됨)
    - %m: 로그 메시지
    - %msg: - 로그 메시지 (=%message)
    - %n: 줄바꿈(new line)
    - %%: %를 출력
    - %r : 애플리케이션 시작 이후부터 로깅이 발생한 시점까지의 시간(ms)
    - %d{yyyy-MM-dd-HH:mm:ss:sss}: %d는 date를 의미하며 중괄호에 들어간 문자열은 dateformat을 의미. 따라서 [2021-07-12 12:42:78]과 같은 날짜가 로그에 출력됨.
    - %-4relative: %relative는 초 아래 단위 시간(밀리초)을 나타냄. -4를하면 4칸의 출력폼을 고정으로 가지고 출력. 따라서 숫자에 따라 [2021-07-12 12:42:78:232] 혹은 [2021-07-12 12:42:78:2332]와 같이 표현됨

### 6-3. Logback 설정 파일 : Root영역

```java
 <!--TRAVE > DEBUG > INFO > WARN > ERROR > OFF -->
    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="INFO_LOG"/>
    </root>
```

- 설정 파일에 정의된 Appender활용하려면 Root영역에서 Appender를 참조해서 로깅 레벨 설정
- 특정 패키지에 대해 다른 로깅 레벨 설정 → root대신 logger 사용하여 지정 가능(위 예제)
- logger 요소의 name 속성 : 패키지 단위로 로깅이 적용될 범위 지정
- level 속성 : 로그 레벨 지정
- additivity 속성: 앞에서 지정한 패키지 범위에 하위 패키지 포함 여부 결정(기본값:true - 하위 패키지 모두 포함)