# 8장. 경계

> 우리 코드와 외부 코드를 어떻게 구분지을까?
> 

## 🧹 경계

<aside>
❓ 우리 코드와 외부 코드 사이에 있는 경계

</aside>

- 시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드묾
    - 때로는 패키지를 구매하고, 때로는 오픈 소스를 이용하며, 때로는 사내 다른 팀이 제공하는 컴포넌트를 사용함
        
        → 우리가 만든 코드에 외부에서 들어온 코드를 병합해야 함
        
- 우리 코드와 외부 코드를 깔끔하게 통합시키기 위해 경계를 잘 지어야 함
---

## 🧹 경계 짓기(1) - 우리 코드를 보호하기

<aside>
🤔 외부에서 사용되는 Sensor를 관리해야 한다면?

</aside>

```java
Map sensors = new HashMap();

Sensor s = (Sensor)sensors.get(sensorId);
```

- Map이 제공하는 다양한 기능을 클라이언트가 사용할 수 있음
    
    → `clear()`를 사용하여 누구나 Map 내용을 지울 권한이 있음
    
- Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 클라이언트에게 있음
    - 해당 코드가 의도를 분명하게 드러내지도 못함

```java
Map<String, Sensor> sensors = new HashMap<String, Sensor>();
...
Sensor s = sensors.get(sensorId);
```

- 제네릭을 사용하면 코드 가독성이 크게 높아지지만, 사용자에게 필요하지 않은 기능까지 제공한다는 문제는 해결하지 못함
    - Map 인터페이스가 변할 경우, 수정할 코드가 상당히 많아짐

```java
public class Sensors {
		private Map sensors = new HashMap();

		public Sensor getById(String id) {
				return (Sensor) sensors.get(id);
		}
		
		...
}
```

- 경계 인터페이스인 Map을 Sensors 안으로 숨김 → Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않음
- Sensors 클래스는 프로그램에 필요한 인터페이스만 제공하여 코드는 이해하기 쉽지만, 오용을 줄일 수 있음
- 나머지 프로그램이 설계 규칙과 비즈니스 규칙을 따르도록 강제할 수 있음
- Map 클래스를 사용할 때마다 캡슐화하라는 것은 아님
    
    → Map과 같은 경계 인터페이스를 사용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의해야 함
    
    - Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않음

---
## 🧹 경계 짓기(2) - 외부 코드와 호환하기

<aside>
🔌 외부 코드를 호출할 때 우리가 원하는 방식으로 사용하고 싶다!

</aside>

- 어댑터를 통해 우리가 원하는 방식과 외부에서 제공하는 방식이 맞아 떨어지도록 호환을 시키자!

> 외부 코드를 호출할 때, 우리가 정의한 인터페이스대로 호출하기 위해 사용하는 패턴 → [어댑터 패턴](https://github.com/YAPP-19th/Design-Pattern-Study/blob/main/Adapter-Pattern/7장_한지희_어댑터%20패턴.md)
> 

### 🤔  외부에서 가져온 패키지를 사용하고 싶다면 어디서 어떻게 시작해야 할까?

외부 패키지 테스트가 우리 책임은 아니지만, 우리 자신을 위해 우리가 사용할 코드를 테스트 하는 편이 바람직함

- 타사 라이브러리를 가져왔으나 사용법이 분명하지 않을 때,
    - 문서를 읽으며 사용법을 결정
        
        → 우리 쪽 코드를 작성해 라이브러리가 예상대로 동작하는지 확인
        
        → 때로는 우리 버그인지 라이브러리 버그인지 찾아내느라 오랜 디버깅을 수행
        

곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신, 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히자! ← 학습 테스트

```java
public class LogTest {
		private Logger logger;

    @Before
    public void initialize() {
        logger = Logger.getLogger("logger");
        logger.removeAllAppenders();
        Logger.getRootLogger().removeAllAppenders();
    }

    @Test
    public void basicLogger() {
        BasicConfigurator.configure();
        logger.info("basicLogger");
    }

    @Test
    public void addAppenderWithStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
        logger.info("addAppenderWithStream");
    }

    @Test
    public void addAppenderWithoutStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n")));
        logger.info("addAppenderWithoutStream");
    }
}
```

- 학습 테스트는 이해도를 높여주는 정확한 실험
- 패키지 새 버전이 나온다면, 학습 테스트를 돌려 차이가 있는지 확인 → 학습 테스트는 패키지가 예상대로 작동하는지 검증
    - 새 버전이 우리 코드와 호환되지 않으면 학습 테스트가 이 사실을 곧바로 밝혀냄
        
        → 이런 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워짐
        

### ☁️  아직 존재하지 않는 코드를 사용하기

경계의 또 다른 유형 중 하나는, 아는 코드와 모르는 코드를 분리하는 경계

- 자체적인 인터페이스 정의하여 사용하기
    - 우리가 바라는 대로 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제할 수 있음
    - 코드 가독성이 높아지고, 코드의 의도도 분명해짐

외부에서 API를 정의한 후에는 Adapter를 구현하여 사용

→ API 사용을 캡슐화하여 API가 바뀔 때 수정할 코드를 한 곳으로 모을 수 있음