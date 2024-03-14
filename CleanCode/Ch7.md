# Ch7

### 오류 코드보다 예외를 사용하라

- 아래의 예시 코드는 디바이스를 종료하는 로직과 오류를 처리하는 로직이 섞여있다. 뿐만 아니라 함수를 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡해진다. 예외를 던지는 게 낫다.
    
    ```java
    public class DeviceController {
    
    	...
    
    	DeviceHandle handle = getHandle(DEV1);
    	if (handle != DeviceHandle.INVALID) {
    		retrieveDeviceRecord(handle);
    		if (record.getStatus() != DEVICE_SUSPENDED) {
    			closeDevice(handle);
    		} else {
    			logger.log("Device suspended. Unable to shut down");
    		}
    		
    	} else {
    		logger.log("Invalid handle");
    	}
    
    	...
    }
    ```
    
- 다음과 같이 예외를 던져 비지니스 논리와 오류 처리가 잘 분리된 코드로 작성할 수 있다.
    
    ```java
    public class DeviceController {
    
    	...
    
    	public void sendShutDown() {
    		try {
    			tryToShutDown();
    		}
    		catch (DeviceShutDownError e) {
    			logger.log(e);
    		}
    	}
    
    	private void tryToShutDown() {
    		DeviceHandle handle = getHandle(DEV1);
    		DeviceRecord record = retrieveDeviceRecord(handle);
    
    		pauseDevice(handle);
    		clearDeviceWorkQueue(handle);
    		closeDevice(handle);
    	}
    
    	private DeviceHandle getHandle(DeviceId id) {
    		...
    		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    		...
    	}
    	
    	...
    
    }
    ```
    

### Try-Catch-Finally문부터 작성하라

- try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지하는 면이 try 블록은 트랜잭션과 비슷하다고 볼 수 있다.
- 그러므로 예외가 발생할 코드를 짤 때는 try-catch-finally문으로 시작하는 편이 낫다.
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

### 미확인 예외를 사용하라

**예외(Exception)**

- 프로그램 내에서 발생하는 예외상황으로 프로그램 내에서 처리가 가능한 것
- 배열의 크기에서 벗어난 인덱스에 접근하거나, 없는 파일을 열 때 등에서 발생

**에러(Error)**

- JVM 내에서 발생하는 에러로 프로그램 내에서 처리가 불가능한 것
- ThreadDeath나 가상머신(JVM)의 오작동 등을 말함

![Untitled](Ch7%20daf32b85eed0443e987b77edc5c562e5/Untitled.png)

**확인된 예외(checked exception)**

- 잘못된 코드가 아닌 잘못된 상황에서 발생하는 예외
- 선언부의 수정을 필요로 하기 때문에 모듈의 캡슐화를 깨버릴 수 있음
- 파일 열기와 같이 정확한 코드로 구현했음에도, 외부 환경(파일이 없는 상황 등)에 따라 발생 가능
- 예외처리를 구현하지 않으면 컴파일 에러 발생 (컴파일 시 확인해서 확인된 예외)
- RuntimeException 이외의 예외들

**미확인 예외(unchecked exception)**

- 런타임 시 잘못 구현된 코드로 인해 발생하는 예외
- 컴파일 에러가 나지 않지만 적절한 예외처리가 없을 경우 프로그램이 강제 종료
- 컴파일 시 확인하지 않기 때문에 미확인 예외
- RuntimeException에 포함된 예외들

---

안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지 않다는 사실이 분명해졌다.

최상위 함수가 아래 함수를 호출한다. 아래 함수는 그 아래 함수를 호출한다. 단계를 내려갈수록 호출하는 함수 수는 늘어난다. 최하위 함수가 오류를 확인된 오류를 던진다면 함수는 선언부에 `throws`절을 추가해야 한다.

그러면 해당 함수를 호출하는 모든 함수가 1) catch 블록에서 새로운 예외를 처리하거나 2) 선언부에 `throws`

절을 추가해야 한다.

결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다! `throws`경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화도 깨진다.

### **호출자를 고려해 예외 클래스를 정의하라**

- 아래 코드는 중복이 심하고 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하여 오류를 형편없이 분류한 예이다.

```java
ACMEPort port = new ACMEPort(12);

try{
	  port.open();
} catch (DeviceResponseException e) {
		reportPortError(e);
} catch (ATM1212UnlockedException e) {
		reportPortError(e);
} catch (GMXError e) {
		reportPortError(e);
} finally {
	...
}
```

↓ 호출하는 라이브러리의 API를 감싸면서 예외 유형을 하나 반환하여 간결하게 수정할 수 있다.

```java
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}
```

```java
// Wrapper 클래스
public class LocalPort {
  private ACMEPort innerPort;
  
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }
  
  public void open() {
    try{
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  
  ...
}
```

**Wrapper 클래스로 예외 호출 라이브러리 API를 감싸면 좋은 점**

> 외부 API를 사용할 때는 Wrapper 클래스 기법이 최선이다.
> 
1. 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에 의존성이 크게 줄어든다.
2. 나중에 다른 라이브러리로 갈아타도 비용이 적다.
3. Wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주면 테스트 하기도 쉽다.
4. 특정 업체가 API를 설계한 방식에 국한되지 않는다. 프로그램이 사용하기 편리한 API를 정의할 수 있다.

### null을 반환/ 전달하지 마라

- null을 반환하는 코드는 호출자에게 책임을 떠넘겨 null 확인이 누락된 문제가 발생하기 쉽다.
    - null을 반환하고 싶다면 대신 예외를 던지거나 특수 사례 객체를 반환한다.
        
        ```java
        List<Employee> employees = getEmployees();
        for(Employee e: employees) {
        	totalPay += e.getPay();
        }
        ```
        
        - 자바에는 `Collections.emptyList()`가 있어 미리 정의된 읽기 전용 리스트를 반환할 수 있다.
        
        ```java
        public List<Employee> getEmployees(){
        	if(...직원이 없다면...){
        		return Collections.emptyList();
        }
        ```
        
        - 이렇게 코드를 변경 시 코드도 깔끔해질뿐더러 NullPointerException이 발생할 가능성도 줄어든다.
- 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는  최대한 피한다.
    - 예외를 던지거나 assert문을 사용할 순 있다.
    - 하지만 애초에 null을 전달하는 경우는 금지하는 것이 바람직하다.

### 리팩토링하며 적용하자

- 카카오 소셜 로그인을 구현하며 외부 API를 사용하였는데 예외 처리를 급하게 한 부분이 있어 리팩토링하였다.

### Before

```java
public ResponseEntity signin(String code, boolean developer) throws JsonProcessingException {
        String kakaoAccessToken = getKakaoAccessToken(code, developer);
        String userInfo = getUserInfo(kakaoAccessToken);
        ...
}
```

```java
private String getKakaoAccessToken(String code, boolean developer) throws JsonProcessingException {
        
        ...header랑 body 생성...

        // http 요청하기
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> response = restTemplate.exchange(
                "https://kauth.kakao.com/oauth/token",
                HttpMethod.POST,
                httpEntity,
                String.class
        );

        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode jsonNode = objectMapper.readTree(response.getBody()); // 예외 발생 지점

        // access_token의 값을 읽어오기
        String accessToken = jsonNode.get("access_token").asText();

        return accessToken;

}
```

### After

**수정 사항**

- 윗 메서드까지 checked 예외가 전달되지 않도록 기존 getKakaoAccessToken 메서드를 sendGetKakaoAccessToken와 tryGetKakaoAccessToken 메서드로 분리하여 비지니스 논리와 오류 처리가 잘 분리되도록 수정하였다.
- 기존에 signin 메서드에 존재하던 json parsing 로직을 sendParseKakaoAuthKey, tryParseKakaoAuthKey로 따로 빼서 비지니스 논리와 오류 처리가 잘 분리되도록 수정하였다.
- access token 요청 과정에서 404 에러가 리턴되는 경우 처리를 누락하였었는데 try-catch문으로 처리하였다.

```java
public ResponseEntity signin(String code, boolean developer) {
        String kakaoAccessToken = sendGetKakaoAccessToken(code, developer);
        String userInfo = getKakaoInfo(kakaoAccessToken);
        String authKey = sendParseKakaoAuthKey(userInfo);
        ...
}
```

```java
private String sendGetKakaoAccessToken(String code, boolean developer){
        // 오류 처리 코드
        try {
            return tryGetKakaoAccessToken(code, developer);
        } catch (JsonProcessingException e) {
            throw new CustomException(ErrorCode.JSON_PARSING_ERROR);
        }
}

private String tryGetKakaoAccessToken(String code, boolean developer) throws JsonProcessingException {
        
        ...header랑 body 생성...

        // http 요청하기
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> response;
        try {
            response = restTemplate.exchange(
                    "https://kauth.kakao.com/oauth/token",
                    HttpMethod.POST,
                    httpEntity,
                    String.class
            );
        } catch (Exception e){
            log.error("사용자의 카카오 로그인 인증 코드가 유효하지 않습니다.: " + e.toString());
            throw new CustomException(ErrorCode.INVALID_AUTHENTICATION_CODE);
        }

        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode jsonNode = objectMapper.readTree(response.getBody()); // 예외 발생 지점

        // access_token의 값을 읽어오기
        String accessToken = jsonNode.get("access_token").asText();

        log.info("Access Token: " + accessToken);
        return accessToken;

}
```