# 테스트와 설계

테스트와 소프트웨어 설계는 긴밀한 상관관계를 맺습니다.   
이 두 요소는 소프트웨어 개발 프로세스의 핵심적인 부분이며, 서로 영향을 주고받으며 공존합니다.  
이것이 당연한 이유는 좋은 소프트웨어 설계와 테스트가 추구하는 목표가 일정 부분 같기 때문입니다.

좋은 설계는 시스템이 모듈로 분해되고 각 모듈이 독립적으로 개발될 수 있게 하는 것을 추구합니다.  
따라서 좋은 설계를 따르는 시스템은 배포 환경이나 테스트 환경을 가리지 않고 이식이 가능합니다.  
그 결과, 좋은 설계를 갖춘 코드는 대부분 테스트하기도 쉽습니다.

## 테스트와 SRP

예를 들어, UserService 컴포넌트는 회원가입만을 위한 UserRegister 컴포넌트와 로그인을 담당하는 Authenticationservice 컴포넌트 정도로 분리할 수 있을 것입니다.

- 회원가입하려는 사용자를 위한 UserRegister 컴포넌트
- 로그인하려는 사용자를 위한 Authenticationservice 컴포넌트

```java
@Service
@Builder
@RequiredArgsConstructor
public class UserRegister {

    private final UserRepository userRepository;
    private final VerificationEmailSender verificationEmailSender;

    @Transactional
    public User register(UserCreateDto userCreateDto) {
        if (userRepository.findByEmail(userCreateDto.getEmail()).isPresent()) {
            throw new EmailDuplicatedException();
        }

        User user = User.builder()
            .email(userCreateDto.getEmail())
            .nickname(userCreateDto.getNickname())
            .status(UserStatus.PENDING)
            .verificationCode(UUID.randomUUID().toString())
            .build();

        user = userRepository.save(user);
        verificationEmailSender.send(user);
        return user;
    }
}
```

```java
@Service
@Builder
@RequiredArgsConstructor
public class AuthenticationService {

    private final UserRepository userRepository;
    private final ClockHolder clockHolder;

    @Transactional
    public User login(String email) {
        User user = userRepository.getByEmail(email);
        if (user.isActiveStatus()) {
            throw new UserIsNotActiveException();
        }

        user.login(clockHolder);
        user = userRepository.save(user);
        return user;
    }
}
```

UserService 컴포넌트는 단일 책임을 위반하고 있었다고 볼 수 있습니다.  
왜나하면 UserService 컴포넌트를 사용하려는 주체가 시스템 미가입자와 시스템 가입자로 서로 다른 메시지를 보내는 두 명의 액터가 있다고 볼 수 있기 때문입니다.

```java
public class RegisterTest {

    @Test
    public void 중복된_이메일_회원가입_요청이_오면_에러가_발생한다() {
    }

    @Test
    public void 이메일_회원가입을_하면_가입_보류_상태가_된다() {
    }
}
```

```java
public class AuthenticationTest {

    @Test
    public void 존재하지_않는_사용자에_로그인하려면_에러가_발생한다() {
    }

    @Test
    public void 로그인하려는_사용자_아직_가입_보류_상태이면_에러가_발생한다() {
    }

    @Test
    public void 사용자가_로그인하면_마지막_로그인_시간이_기록된다() {
    }
}
```

## 테스트와 ISP

테스트는 인터페이스 분리를 유도합니다.
예를 들어, 회원가입 시 이메일을 보내는 부분에서 컴포넌트가 VerificationEmailSender 타입이 아닌 EmailSender라는 타입의 인터페이스를 사용하고 있었다고 가정해봅시다.  
그리고 테스트 결과. 사용자에게 전달되는 이메일은 어떻게 만들어지는지 궁금해졌다고 합시다.

```java
public class FakeEmailSender implements EmailSender {

    public Map<String, String> emails = new HashMap<String, String>();

    public void sendVerificationRequired(User user) {
        String content = VerificationEmailContentGenerator.generate(user);
        emails.put(user.getEmail(), content);
    }

    void sendWelcome(User user) { }

    void sendAdvertisement(User user) { }

    void sendCharge(User user) { }
}
```

애초에 우리는 회원가입과 관련된 테스트를 작성하고 싶을 뿐입니다.  
그런데 Fake에서 가입 축하 메일을 보내는 방법이나 광고 메일을 보내는 방법, 비용 청구 메일을 보내는 방법을 어떻게 구현해야 할지 왜 고민해야할까요 ??  
**우리는 테스트를 만들 때 '우리가 테스트하고 싶은 것'에만 관심을 두고 싶습니다. 그 외의 불필요한 의존과 인터페이스에 관심을 두고 싶지 않습니다.**  

따라서 이번에는 테스트가 인터페이스를 분리하라는 신호를 보내고 있는 것입니다.  
VerificationEmailSender 인터페이스일 때에는 이런 고민을 할 필요가 없었습니다.  
이런 고민을 시작하게 된 것은 EmailSender라는 통합된 인터페이스를 사용하고 나서부터입니다.  
**그러니 이러한 고민을 하고 싶지 않다면 처음부터 인터페이스는 상세히 분리하는 것이 좋습니다.**

## 테스트와 OCP, DIP

좋은 설계를 갖춘 시스템은 유연합니다. 여기서 '유연하다'의 의미는 시스템이 변화에 효과적으로 대응할수 있다는 의미입니다.  
그래서 좋은 설계로 개발된 시스템이라면 외부 요구사항이 변경돼 코드를 수정해야 할 때도 해당 코드 변경으로 인한 영향을 최소화할 수 있어야 합니다.  
**개방 폐쇄 원칙을 따르는 시스템은 새로운 기능으로 확장해야 할 때 언제든 새로운 기능을 맞이할 준비가 돼 있습니다.**   

**그런데 유연한 설계를 따르고 있는지 아닌지는 어떻게 판단하면 좋을까요 ??**  
이 같은 상황에서 활용할 수 있는 것이 있으니 바로 테스트입니다. 테스트를 이용하면 코드의 유연성과 확장성이 어떤지를 판단할 수 있습니다.  
테스트를 작성하는 개발자는 시스템을 개발할 때 배포 환경과 테스트 환경을 둘 다 고려해야 합니다. 그렇게 해서 시스템이 배포 환경과 테스트 환경에서도 원활하게 실행될 수 있게 만들어야 합니다.  
그 결과. 코드는 여러 환경에서도 실행 가능한 코드가 됩니다. 유연해지는 것입니다.

조금만 더 나아가서, 의존성 역전 원칙을 생각해봅시다.  
의존성 역전 원칙이 추상화를 뜻하는 것은 아니지만 추상화를 통해서 유연성을 추구할 수 있습니다.  
결국 의존성 역전 원칙을 추구하는 것 역시 시스템의 유연성을 높이는 일이라 볼 수 있습니다. 그런데 이런 의존성 역전 원칙도 테스트를 고민하다 보면 자연스럽게 따라옵니다.

## 테스트와 LSP

그렇다면 어떤것을 테스트해야 할까요 ???

- Right-BICEP
  - Right: 결과가 올바른지 확인해 봐야 합니다. 
  - Boundary: 경계 조건에서 코드가 정상적으로 동작하는지 확인해 봐야 합니다. 
  - Inverse: 역함수가 있다면 이를 실행해 입력과 일치하는지 확인해 봐야 합니다. 
  - Cross-Check: 검증에 사용할 다른 수단이 있다면 이를 비교해 봐야 합니다. 
  - Error Conditions: 오류 상황에서도 프로그램이 의도한 동작을 하는지 확인해 봐야 합니다. 
  - Performance: 프로그램이 예상한 성능 수준을 유지하는지 확인해 봐야 합니다.
- CORRECT 
  - Conformance(적합성): 데이터 포맷이 제대로 처리되는지 확인해 봐야 합니다. 
  - Ordering(정렬): 출력에 순서가 보장돼야 한다면 이를 확인해 봐야 합니다. 
  - Range(범위): 입력에 양 끝점이 있다면 양 끝점이 들어갈 때 정상 동작하는지 확인해 봐야 합니다. 
  - Reference(참조): 협력 객체의 상태에 따라 어떻게 동작하는지 확인해 봐야 합니다. 
  - Existence(존재): null, blank 같은 값이 입력될 때 어떻게 반응하는지 확인해 봐야 합니다. 
  - Cardinality(원소 개수): 입력의 개수가 0, 1, 2, ..., n일 때 어떻게 동작하는지 확인해 봐야 합니다. 
  - Time(시간): 병렬 처리를 한다면 순서가 보장되는지 확인해 봐야 합니다.

또한, 어떤 메서드나 시스템의 실행 결과를 미리 작성하고 유지했으면 하는 시스템의 모든 상태를 테스트로 작성해야 합니다.  
테스트는 시스템의 상태를 검증하는 수단입니다. 그러므로 개발자는 유지하고 싶은 상태가 있다면 역할과 책임 관점에서 개발자의 모든 의도를 테스트로 작성해둬야 합니다.  
그렇게 해서 코드베이스가 변경됐을 때 잘못된 상태 변경이 있으면 테스트가 이 오류를 찾아낼 수 있어야 합니다.

