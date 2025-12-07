# 테스트 대역

테스트 대역에서 말하는 대역은 대역폭(bandwidth) 같은 것이 아닙니다.   
테스트 대역이란 오롯이 테스트를 위해 만들어진 진짜가 아닌 가짜 객체나 컴포넌트를 가리키는 용어입니다.  
영화 속에서 대역이란 주연 배우를 대신해서 격한 스턴트 액션을 담당하거나 주연 배우가 하지 못하는 일을 대신하는 사람을 의미합니다.  
이와 마찬가지로 테스트 대역은 실제 객체를 대신해서 행동하고 실제 객체가 하지 못하는 일을 대신합니다.

```java
@Service
@Builder
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final VerificationEmailSender verificationEmailSender;

    @Transactional
    public User register(UserCreateDto userCreateDto) {
        User user = User.builder()
                .email(userCreate.getEmail())
                .nickname(userCreate.getNickname())
                .status(UserStatus.PENDING)
                .verificationCode(UUID.randomUUID().toString())
                .build();
        
        user = userRepository.save(user);
        verificationEmailSender.send(user);
        return user;
    }
}
```

UserService는 사용자 정보를 가입 보류 상태로 저장하고, 가입 인증 메일을 보냅니다.  
이 코드의 테스트를 만들어 실행하면 어떤 일이 벌어질까요 ?  
이 테스트를 잘못 구성했다가는 테스트를 실행할 때마다 실제 메일이 발송되는 대참사가 일어날 수 있습니다.  
**이럴 때 테스트 대역을 사용할 수 있습니다. 메일을 발송하는 컴포넌트인 VerificationEmailSender의 대역을 만들고 테스트할 때 UserService 컴포넌트가 이를 사용하게 만드는 것입니다.**  

테스트 대역을 이용하면 개발자가 테스트를 위한 격리되고 고정된 환경을 만들 수 있습니다.  
다시 말해, 테스트 대역을 이용하면 복잡한 시스템의 테스트 환경을 예측 가능하게 만들 수 있다는 의미입니다.

> 테스트 대역의 5가지 유형  
> 
> | 유형 | 설명 |
> | :--- | :--- |
> | Dummy | 아무런 동작을 하지 않습니다. |
> | Stub | 지정한 값만 반환합니다. |
> | Fake | 자체적인 로직이 있습니다. |
> | Mock | 아무런 동작을 하지 않습니다. 대신 어떤 행동이 호출됐는지를 기록합니다. |
> | Spy | 실제 객체와 똑같이 행동합니다. 그리고 모든 행동 호출을 기록합니다. |

## Dummy

Dummy의 역할은 아무런 동작도 하지 않는 것입니다.  
Dummy 객체는 오롯이 코드가 정상적으로 돌아가게 하기 위한 역할만 합니다. 그리고 특정 행동이 일어나지 않게 만드는 데 사용됩니다.  
Dummy는 꼭 멤버 변수에만 주입될때 사용할 수 있는 것은 아니고, 필수 매개변수가 포함된 메서드를 호출해야 하는 경우에도 사용할 수 있습니다.

```java
public class SomethingFilterTest {

    @Test
    public void 요청에_text로_달라는_요청이_있으면_응답의_콘텐츠_타입은_Text_plain이다() 
            throws ServletException, IOException {
        // given
        ServletRequest servletRequest = new MockHttpServletRequest();
        servletRequest.setAttribute("giveMe", "text");
        ServletResponse servletResponse = new MockHttpServletResponse();

        // when
        SomethingFilter somethingFilter = new SomethingFilter();
        somethingFilter.doFilter(
            servletRequest,
            servletResponse,
            new FilterChain() {
                @Override
                public void doFilter(ServletRequest req, ServletResponse res) {
                    // do nothing
                }
            });

        // then
        assertThat(servletResponse.getContentType()).isEqualTo("text/plain");
    }
}
```

테스트 코드에서 메서드 호출의 마지막 filterchain 값에 아무런 동작을 하지 않는 익명 클래스가 들어가게 했습니다.  
덕분에 SomethingFilter 클래스의 마지막 filterchain.doFilter 메서드 호출은 아무런 동작도 하지 않을 것입니다.

## Stub

Stub은 원본을 따라한 부분과 마찬가지로 실제 객체의 응답을 최대한 비슷하게 따라하는 대역입니다.  
그래서 Stub은 응답을 원본과 똑같이 반환하는 데만 집중합니다. 즉, 원본의 응답을 복제해 똑같은 응답으로 미리 준비하고 이를 바로 반환합니다.  
테스트를 작성하다 보면 어떤 객체의 메서드 호출 결과가 뻔한 것에 비해 동작이 지나치게 복잡한 경우가 있습니다.  
디스크 I/O, 네트워크 호출이 발생할 수 있는 고연산 작업 등에서 미리 준비한 값을 그대로 반환해서 고연산 작업이 실제로 실행되지 않게 할 수 있습니다.

Stub은 아무런 동작도 하지 않았던 Dummy와는 다르게 실제 구현체의 응답을 흉내 냅니다. 그렇게 해서 테스트 환경을 마음대로 조작합니다.  
덕분에 Stub은 외부 연동을 하는 컴포넌트나 클라이언트를 대체하는 데 자주 사용됩니다.

## Fake

Fake는 테스트를 위한 자체적인 논리를 갖고 있습니다.  
테스트를 위해 매번 Stub을 달리 해줘야 해서 테스트의 중요한 부분을 가리고, 그로 인해 테스트의 목적이 무엇인지 한눈에 파악할 수 없게 되는 것은 그다지 좋은 현상이 아닙니다.  
Stub 대신 이런 대역 객체가 있다면 어떨까요? UserRepository 역할의 대역으로 사용될 객체인데, 데이터 저장을 위한 간단한 메모리 변수를 갖고 있게 하는 것입니다.  
그리고 UserRepository 인터페이스에 읽기/쓰기 요청이 왔을 때 이 요청을 메모리 변수에 쓰고 불러오게 합니다.  
그렇게 한다면 데이터베이스의 동작을 메모리 수준에서 흉내 낼 수 있을 것입니다.

## Mock

Mock은 메서드 호출이 발생했는지를 검증하기 위해 만들어지는 테스트 대역에 해당합니다.  
조금 더 자세히 설명하자면 Mock은 '메서드 호출 및 상호 작용을 기록하고, 실제로 상호 작용이 일어났는지. 어떻게 상호 작용이 일어났는지를 확인하는 데 사용되는 객체'를 말합니다.

### 상태 기반 검증

상태 기반 검증(state-based verification)은 테스트의 검증 동작에 상태를 사용하는 것을 의미합니다.  
즉, 상태 기반 검증으로 동작하는 테스트에서는 테스트를 실행한 후 테스트 대상의 상태가 어떻게 변화됐는지를 보고 테스트 실행 결과를 판단합니다.

### 행위 기반 검증

행위 기반 검증(behaviour-based verification)은 테스트의 검증 동작에 메서드 호줄 여부를 보게 하는 것을 의미합니다.  
즉, 행위 기반 검증으로 동작하는 테스트에서는 테스트 대상이나 협력 객체, 협력 시스템의 메서드 호출 여부를 봅니다.

### 상태 기반 vs 행위 기반

행위 기반 검증을 이용해서 테스트를 작성하는 것은 그렇게 좋은 전략이 아니라는 것을 금방 알 수 있을 것입니다.  
왜냐하면 '상호 작용 테스트', '행위 기반 검증' 같은 말로 포장했지만 이는 사실상 알고리즘을 테스트하는 것과 같기 때문입니다.  
따라서 상호 작용 테스트가 많아지면 시스템 코드가 전체적으로 경직될 수 있습니다. **가급적이면 테스트는 상태 기반 검증으로 작성하는 편이 좋습니다.**  
그렇다면 코드 수준에서 Mock을 어떻게 구현하면 좋을까요 ?

```java
public class MockVerificationEmailSender implements VerificationEmailSender {

    public boolean isSendCalled = false;

    @Override
    public void send(User user) {
        this.isSendCalled = true;
    }
}
```

```java
@Test
public void 이메일_회원가입을_하면_가입_보류_상태가_된다() {
    // given
    UserCreateDto userCreateDto = UserCreateDto.builder()
            .email("foobar@localhost.com")
            .nickname("foobar")
            .build();
    
    MockVerificationEmailSender verificationEmailSender = new MockVerificationEmailSender();

    // when
    UserService userService = UserService.builder()
            .verificationEmailSender(verificationEmailSender)
            .userRepository(new FakeUserRepository())
            .build();
    
    User user = userService.register(userCreateDto);

    // then
    assertThat(user.isPending()).isTrue();
    assertThat(verificationEmailSender.isSendCalled).isTrue();
}
```

맨 마지막에 VerificationEmailSender 객체의 isSendCalled 값이 true로 변경됐는지 확인합니다. 그렇게 해서 send 메서드 호출 여부를 판단합니다.  
그래서 이렇게 작성된 테스트 대역인 MockVerificationEmailSender를 가리켜 Mock이라고 부릅니다.

## Spy

Spy는 실제 객체 대신 사용돼서 만약 실제 객체였다면 어떤 메서드가 호출되고 이벤트가 발생했는지 등을 기록하고 감시합니다.  
더불어 메서드가 몇 번 호출됐는지, 메서드는 어떤 매개변수로 호출됐는지, 메서드 호출 순서는 어떤지 등 모든 것을 기록합니다.  
따라서 개념적으로 Spy는 상호 작용을 검증하는 데 주로 사용됩니다.  
**Mock과 Spy의 차이점은, 내부 구현이 진짜 구현체인가, 가짜 구현체인가 입니다.**

Mock으로 만들어진 객체는 기본적으로 모든 메서드 호출이 Dummy 또는 Slub처럼 동작합니다.  
반면, Spy로 만들어진 객체는 기본적인 동작이 실제 객체의 코드와 같습니다. 즉. Spy는 실제 객체와 구분할 수 없습니다. 


























