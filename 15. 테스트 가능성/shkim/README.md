# 테스트 가능성

개발자는 테스트를 어떻게 하면 쉽게 작성할 수 있을지 고민함으로써 코드의 품질을 높일 수 있습니다.  
이를 알고 있는 똑똑한 개발자들은 테스트를 사용하는 목적을 '회귀 버그 방지'에만 두지 않습니다.  
테스트를 '좋은 설계를 얻기 위한 수단'으로 보고 좋은 설계를 얻기 위해 테스트를 사용합니다.

테스트를 회귀 버그 방지를 위한 수단으로만 바라보면 테스트가 주는 가치를 온전히 누릴 수 없습니다.  
반면 테스트를 좋은 설계를 갖춘 시스템을 얻기 위한 도구로 본다면 테스트가 주는 가치를 더 누릴 수 있게 됩니다.

## 테스트를 어렵게 만드는 요소

**테스트는 테스트하려는 대상의 입력을 쉽게 변경할 수 있고, 출력을 쉽게 검증할 수 있을 때 작성하기 쉽습니다.**  
반면 테스트하려는 대상에 숨겨진 입력이 존재하거나 숨겨진 출력이 있을 때 테스트를 검증하기가 어려워집니다.

### 숨겨진 입력

```java
@Getter
@Builder
public class User {

    private String email;
    private long lastLoginTimestamp;

    public void login() {
        // ...
        this.lastLoginTimestamp = Clock.systemUTC().millis();
    }
}
```

```java
public class UserTest {

    @Test
    public void 로그인을_호출할_경우_사용자의_마지막_로그인_시각이_갱신된다() {
        // given
        User user = User.builder()
            .email("foobar@localhost.com")
            .build();

        // when
        user.login();

        // then
        long expected = Clock.systemUTC().millis();
        assertThat(user.getLastLoginTimestamp()).isEqualTo(expected);
    }
}
```

이 테스트는 비결정적으로 동작합니다. when 절에서 login 메서드를 실행하는 시점의 현재 시각과 then 절에서 마지막 로그인 시간을 검증하기 위해 불러온 현재 시각은 다를 수 있으니까요.  
login 메서드를 실행하는 사용자 입장에서 이 메서드는 필요한 의존성이 없는 것처럼 보입니다. 하지만 내부 구현은 마지막 로그인 시각을 기록하기 위해 현재 시각을 알 수 있어야 합니다.  
그러한 까닭에 Clock 클래스의 전역 메서드에 의존하고 있습니다.  
**숨겨진 입력은 외부 사용자가 코드를 사용할 때 코드가 어떤 식으로 동작할지 예상할 수 없게 만듭니다.**  
명시된 입력에 같은 값을 넣어 같은 코드를 실행해도 다른 결과가 나오기 때문입니다.

<br>

```java
@Getter
@Builder
public class User {

    private String email;
    private long lastLoginTimestamp;

    public void login(long currentTimestamp) {
        // ...
        this.lastLoginTimestamp = Clock.systemUTC().millis();
    }
}
```

```java
public class UserTest {

    @Test
    public void 로그인을_호출할_경우_사용자의_마지막_로그인_시각이_갱신된다() {
        // given
        User user = User.builder()
            .email("foobar@localhost.com")
            .build();

        // when
        long currentTimestamp = Clock.systemUTC().millis();
        user.login(currentTimestamp);

        // then
        assertThat(user.getLastLoginTimestamp()).isEqualTo(currentTimestamp);
    }
}
```

우리는 어떻게 해야 테스트를 쉽게 할 수 있을지를 고민했더니 숨겨진 입력을 외부로 드러내게 되었습니다.  
그 결과, 코드는 테스트하기 쉬워졌으며 더 유연하고 명료해졌습니다.


### 숨겨진 출력

```java
@Getter
@Builder
public class User {
    
    private String email;
    private long lastLoginTimestamp;
    
    public void login(ClockHolder clockHolder) {
        // ...
        this.lastLoginTimestamp = clockHolder.now();
        System.out.println(this.lastLoginTimestamp);
    }
}
```

테스트 검증 단계에서는 이러한 부수적인 출력을 확인할 길이 없습니다. 로그 출력 결과는 표준 출력(System.out)을 확인해야 하기 때문입니다.  
그런데 시스템 출력은 테스트 환경 밖에서 벌어지는 일입니다. 그래서 이러한 출력은 자연스럽지 않습니다.  
우리는 인터페이스를 정의하면서 입력(매개변수), 출력(반환값), 시그니처(메서드 이름)만을 사용해 메서드를 정의합니다.  
그래서 개발자가 메서드를 보고 알 수 있는 것도 이 세 가지가 전부입니다. 개발자는 입력, 출력, 시그니처만 가지고 메서드의 동작과 호출 결과가 어떨지 추론해야 한다는 것입니다.  
**그러니 메서드 호출의 출력 결과는 반환값을 통해 드러내는 것이 좋습니다.**

```java
@Getter
@Builder
public class User {
    
    private String email;
    private long lastLoginTimestamp;
    
    public LoginSuccess login(ClockHolder clockHolder) {
        // ...
        this.lastLoginTimestamp = clockHolder.now();
        return LoginSuccess.builder()
                .auditMessage("User(" + email + ") login!")
                .build();
    }
}
```

## 테스트가 보내는 신호

테스트를 작성하는 방법을 고민하다 보면 개발자가 느낄 수 있는 테스트가 보내는 몇 가지 신호들이 있습니다.

- 테스트의 입출력을 확인할 수 없는데 어떻게 할지
- private 메서드는 어떻게 테스트할지
- 서비스 컴포넌트의 간단한 메서드를 테스트하고 싶을 뿐인데, 이를 위해 필요도 없는 객체를 너무 많이 주입해야할떄 어떻게 할지
- 메서드의 코드 커버리지를 100% 달성하려면 테스트해야 할 케이스가 너무 많아지는데 어떻게 할지

이러한 모든 생각들이 바로 테스트가 보내는 신호입니다. 여러분이 테스트를 작성하는 방법을 고민하면서 위와 같은 생각을 했다면 테스트가 보내는 신호를 포착한 것입니다.  
위와 같은 신호들은 모두 **설계가 잘못되었을 확률이 높으니 좋은 설계로 변경해봐**라고 말합니다.  
테스트를 작성하면서 이러한 신호를 포착할 수 있는 이유는 '코드 작성자’ 입장에서 코드를 바라보는 것이 아니라 '코드 사용자' 입장에서 바라볼 수 있게 되기 때문입니다.  
이러한 시점의 변화 덕분에 개발자는 코드를 알고리즘이 아닌 요구사항 위주로 바라볼 수 있게 됩니다.










