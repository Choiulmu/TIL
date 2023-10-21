# ch5. API를 작성하는 다양한 방법

### 1. GET API 만들기

- GET API : 웹 애플리케이션 서버에서 값을 가져올 때 사용하는 API

### 1) `@PathVariable` 을 활용한 GET 메서드 구현

```java
package com.springboot.api.controller;

import...

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {

//1. 매개변수 없는 버전(기본)
    @GetMapping("/name")
    public String getName(){
        return "Flature";
    }

//2. 매개변수 - 이름 동일 버전
    @GetMapping("/variable1/{variable}")
    public String getVariable(@PathVariable String variable){
        return variable;
    }

//3. 매개변수 - 이름 다른 버전
    @GetMapping(value = "/variable2/{variable}")
    public String getVariable2(@PathVariable("varibale") String var) {
        return var;
    }

}
```

- URL 경로에 값(변수)을 담아 요청을 보내는 방법
- @GetMapping 값으로 URL입력시 받을 값의 위치를 표시하고자 중괄호({}) 활용
- @GetMapping 어노테이션과 @PathVariable에 지정된 변수 이름 동일
- 일치하지 않는 경우 : 별도의 매핑 필요

### 2) `@RequestParam` 을 활용한 GET 메서드 구현

```java
package com.springboot.api.controller;

import...

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {

//1. 기본 
	@GetMapping(value = "/request1")
	    public String getRequestParam1(
	            @RequestParam String name,
	            @RequestParam String email,
	            @RequestParam String organization
	    ) {
	        return name + " " + email + " " + organization;
	    }
	
//2. 들어올 매개변수가 뭔지 모르는 경우 	
	    @GetMapping(value = "/request2")
	    public String getRequestParam2(@RequestParam Map<String, String> param){
	        StringBuilder sb = new StringBuilder();
	
	        param.entrySet().forEach(map -> {
	            sb.append(map.getKey() + " : " + map.getValue() + "\n");
	        });
	        return sb.toString();
	    }
}
```

- 쿼리 형식으로 파라미터에 값(변수)을 넣어 전달하는 방법
- ?{키}={값}
- 주관식 / 선택 기입 항목 → Map 객체로 받는 것이 효율적

### 3) DTO객체를 활용한 GET 메서드 구현

```java
package com.springboot.api.dto;

public class MemberDto {
    private String name;
    private String email;
    private String organization;

    //getter&setter 메소드..

    @Override
    public String toString(){
        return "MemberDto{" +
                "name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", organization='" + organization +'\'' +
                '}';
    }
}
```

```java
package com.springboot.api.controller;

//import문..

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {

    @GetMapping(value = "/request3")
    public String getRequestParam3(MemberDto memberDto){
        return memberDto.toString();
    }
}
```

- DTO(Data Transfer Object)
    - 다른 레이어 간 데이터 교환을 위한 매개변수로 활용되는 데이터 객체
    - 별도 로직X
- VO(Value Object)
    - 데이터 그 자체로 의미있는 객체
    - 읽기 전용 - 값 변경 못하도록 설계 - 데이터 신뢰성 유지

### 2. POST API 만들기

- POST API: 웹 애플리케이션을 통해 데이터베이스 등 저장소에 리소스 저장할 때 사용되는 API
- 저장하고자 하는 리소스/값을 HTTP 바디(body)에 담아 서버에 전달

### 1) `@RequestMapping` 을 활용한 POST메서드 구현

```java
package com.springboot.api.controller;

//import문..

@RestController
@RequestMapping("/api/v1/post-api")
public class PostController {

    @PostMapping(value = "/domain")
    public String postExample(){
        return "Hello Post API";
    }
}
```

### 2) `@RequestBody` 를 활용한 POST 메서드 구현

```java
package com.springboot.api.controller;

//import문..

@RestController
@RequestMapping("/api/v1/post-api")
public class PostController {

// 1. 기본
    @PostMapping(value = "/member")
    public String postMember(@RequestBody Map<String, Object> postData){
        StringBuilder sb = new StringBuilder();

        postData.entrySet().forEach(map -> {
            sb.append(map.getKey() + " : " + map.getValue() +"\n");
        });

        return sb.toString();
    }

// 2.MemberDto 활용
    @PostMapping(value = "/member2")
    public String postMemberDto(@RequestBody MemberDto memberDto){
        return memberDto.toString();
    }
}
```

- POST 요청에는 리소스 담기 위해 HTTP Body에 값을 넣어 전송
- Body영역에 작성되는 값 : JSON 형식

### 3. PUT API 만들기

- PUT API : 웹 애플리케이션 서버를 통해 데이터베이스 같은 저장소에 존재하는 리소스 값 업데이트하는 API
- controller 구현 자체는 POST API와 유사

### 1) `@RequestBody`를 활용한 PUT 메서드 구현

```java
package com.springboot.api.controller;

// import문..

@RestController
@RequestMapping("api/v1/put-api")
public class PutController {

// 1. 기본
    @PutMapping(value = "/member")
    public String postMember(@RequestBody Map<String, Object> putData) {
        StringBuilder sb = new StringBuilder();

        putData.entrySet().forEach(map -> {
            sb.append(map.getKey() + " : " + map.getValue() +"\n");
        });
        return sb.toString();
    }

// 2. MemberDto를 매개변수로 활용/ String 타입을 return
    @PutMapping(value = "/member1")
    public String postMemberDto1(@RequestBody MemberDto memberDto){
        return memberDto.toString();
    }

// 3. MemberDto를 매개변수로 활용/ MemberDto 타입을 return
    @PutMapping(value = "/member2")
    public MemberDto postMemberDto2(@RequestBody MemberDto memberDto){
        return memberDto;
    }
}
```

### 2) `@ResponseEntity`를 활용한 PUT 메서드 구현

```java
package com.springboot.api.controller;

// import문..

@RestController
@RequestMapping("api/v1/put-api")
public class PutController {
    
		@PutMapping(value = "/member3")
    public ResponseEntity<MemberDto> postMemberDto3(@RequestBody MemberDto memberDto){
        return  ResponseEntity
                .status(HttpStatus.ACCEPTED)
                .body(memberDto);
    }
}
```

- ResponseEntity
    - 서버에 들어온 요청에 대해 응답 데이터 구성해서 전달
    - RequestEntity와 ResponseEntity는 HttpEntity를 상속받아 구현한 클래스
    - HttpEntity로부터 HttpHeaders와 Body를 가지고 자체적으로 HttpStatus구현

**[HttpEntity클래스]**

```java
public class HttpEntity<T> {
	private final HttpHeaders headers;
	
	@Nullable
	private final T body;
	...
}
```

- 스프링 프레임워크 클래스
- Header와 Body로 구성된 HTTP 요청과 응답 구성하는 역할

**[ResponseEntity클래스]**

```java
public class ResponseEntity<T> extends HttpEntity<T> {
	private final Object status;
	..생략..
}
```

- 응답코드 변경 / Header와 Body 쉽게 구성

### 4. DELETE API 만들기

```java
package com.springboot.api.controller;

//import문..

@RestController("/api/v1/delete-api")
public class DeleteController {

    @DeleteMapping(value = "/{variable}")
    public String DeleteVariable(@PathVariable String variable){
        return variable;
    }

    @DeleteMapping(value = "/request1")
    public String getRequestParam1(@RequestParam String email){
        return "email :" + email;
    }
}
```

- Delete API : 웹 애플리케이션 서버를 거쳐 저장소(데이터베이스 등)에 있는 리소스 삭제할 때 사용하는 API
- 서버는 클라이언트로부터 리소스 식별값을 받아 데이터베이스/캐시 리소스 조회 → 삭제
- URI에 값(리소스 식별값)을 넣어 요청 받는 형식으로 구현

### 5. Swagger 활용하기

- Spring Boot 3.0.0이상부터 springfox가 아닌 springdoc-openapi-ui 라이브러리 사용

참고 문헌) [https://resilient-923.tistory.com/414](https://resilient-923.tistory.com/414) 

[https://adjh54.tistory.com/72#3. Config 파일 구성-1](https://adjh54.tistory.com/72#3.%20Config%20%ED%8C%8C%EC%9D%BC%20%EA%B5%AC%EC%84%B1-1)

### 6. 로깅 라이브러리 - Logback

- 로깅(logging)
    - 애플리케이션이 동작하는 동안 시스템의 상태/동작 정보를 시간순으로 기록하는 것
    - spring-boot-starter-web 라이브러리에 내장

- Logback특징
    - 5개의 로그레벨 설정(ERROR, WARN, INFO, DEBUG, TRACE)
        - ERROR : 애플리케이션 작동이 불가한 정도로 심각한 문제 레벨
        - WARN : 시스템 에러 원인이 될 수 있는 경고 레벨
        - INFO : 애플리케이션 상태 변경과 같은 정보 전달을 위해 사용
        - DEBUG : 애플리케이션의 디버깅을 위한 메시지 표시하는 레벨
        - TRACE : DEBUG레벨보다 더 상세한 메시지 표현을 위한 레벨
    - 실제 운영 환경과 개발 환경에서 각각 다른 출력 레벨을 설정해서 로그 확인 가능
    - Logback 설정 파일을 일정 시간마다 스캔 - 애플리케이션 재가동 없이 설정 변경
    - 자체적으로 로그 파일 압축 가능 & 보관
    - 로그 파일 보관 기간 등 설정 관리 가능

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

- Appender 영역
    - 로그 형태 설정 및 로그 출력 방법 설정
    - Appender라는 인터페이스 하위에 여러 구현체 존재

### URI와 URL의 차이

- URL : 웹 주소 / 리소스가 어디에 있는지 알려주기 위한 경로
- URI : 특정 리소를 식별할 수 있는 식별자
- URL을 활용해 서버에 접근하여 리소스 접근하려면 URI 필요