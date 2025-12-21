# 테스트와 개발 방법론

TDD와 BDD는 코드의 안정성과 유연성을 높여 소프트웨어의 품질을 향상시킬 수 있는 가장 현대적인 개발 방법론입니다.  
그리고 많은 기업과 개발팀에 채택돼 성공적으로 프로덕션 환경까지 적용된 검증된 방법론입니다.

## TDD

프로젝트에서 TDD를 이용하기로 한 경우 개발자는 Red, Green, Refactor라고 하는 세 단계를 거쳐 소프트웨어를 개발하게 됩니다.

- Red 단계: 첫 번째 단계인 Red 단계에서는 아직 구현되지 않은 기능을 테스트하는 케이스를 작성합니다.
- Green 단계: 두 번째 단계인 Green 단계에서는 테스트를 통과시키기 위한 최소한의 코드를 작성합니다.
- Refactor 단계: 세 번째 단계인 Refactor 단계에서는 Green 단계에서 작성한 코드를 리팩터링합니다. Green 단계에서 기능을 구현하는 데만 집중했다면 Refactor 단계에서는 코드의 가독성과 유지보수성, 성능을 높이는 데 집중합니다.

TDD에서는 개발 단계를 Red-Green-Refactor 단계로 나누고 이 단계를 반복합니다. '테스트를 먼저 작성한다', '기능을 구현한다', '리팩터링 한다' 가 TDD의 전부입니다.  
그리고 테스트를 중요시하는 개발 방법론이기 때문에 테스트의 장점이 곧 TDD의 장점이 됩니다.

TDD도 단점은 분명히 있으니, TDD를 적용하려는 개발자들이 미리 알고 유의해야 할 사항이 하나 있습니다. 바로 TDD를 적용하는 것이 어렵다라는 점입니다.  
TDD를 팀에 적용하기 위해서는 시스템을 개발하는 모든 팀원이 테스트 코드의 필요성을 느끼고 테스트가 필요한 이유에 대해 팀 내 문화적 공감대가 형성돼야 합니다.  
더불어 팀원들 모두 테스트 코드 작성에 어느 정도 숙련된 상태여야 합니다.

## BDD

BDD는 TDD에서 파생된 소프트웨어 개발 방법론입니다. BDD는 TDD에 '사용자 행동'이라는 가치를 덧붙이고 이를 강조합니다.  
그래서 BDD에서는 사용자 행동을 '행동 명세' 같은 요구사항으로 먼저 만듭니다. コ・리고 이것이 테스트로 표현될 수 있게 만듭니다.  
즉, 테스트가 요구사항 문서이자 기획 문서가 될 수 있게 만드는 것입니다. BDD에서는 테스트의 단위가 사용자의 행동이며, 이에 맞춰 애플리케이션을 설계할 것을 강조합니다.

```java
public class AccountTest {

    @Test
    public void 새로운_계좌가_생성되면_초기_잔액이_정확히_일치해야_한다() {
        // given
        Account account = new Account("12345", 100);

        // then
        assertThat(account.getAccountNumber()).isEqualTo("12345");
        assertThat(account.getBalance()).isEqualTo(100);
    }

    @Test
    public void 잔액이_100인_계좌에_50을_입금하면_잔액은_150이어야_한다() {
        // given
        Account account = new Account("12345", 100);

        // when
        account.deposit(50);

        // then
        assertThat(account.getBalance()).isEqualTo(150);
    }

    @Test
    public void 잔액이_100인_계좌에서_30을_출금하면_잔액은_70이어야_한다() {
        // given
        Account account = new Account("12345", 100);

        // when
        account.withdraw(30);

        // then
        assertThat(account.getBalance()).isEqualTo(70);
    }

    @Test
    public void 잔액을_넘어서는_출금_요청을_하면_예외가_발생한다() {
        // given
        Account account = new Account("12345", 100);

        // then
        assertThatThrownBy(() -> {
            // when
            account.withdraw(150);
        }).isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    public void 음수_금액을_입금하면_예외가_발생한다() {
        // given
        Account account = new Account("12345", 100);

        // then
        assertThatThrownBy(() -> {
            // when
            account.deposit(-50);
        }).isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    public void 음수_금액을_출금하면_예외가_발생한다() {
        // given
        Account account = new Account("12345", 100);

        // then
        assertThatThrownBy(() -> {
            // when
            account.withdraw(-50);
        }).isInstanceOf(IllegalArgumentException.class);
    }
}
```

테스트를 분리하고 Given-When-Then 주석을 추가한다고 해서 TDD가 곧바로 BDD가 되는 것은 아닙니다.  
Given-When-Then 형식으로 테스트를 작성하는 것은 BDD를 실천하는 여러 가지 방법 중 하나일 뿐입니다.

> 객체지향적인 프로그램을 만들고 싶을 때 TDD와 DDD는 상호보완적입니다.  
> TDD는 기능을 테스트하고 구축함으로써 안정성과 유연성을 확보하는 데 중점을 두는 반면, 객체지향을 보장하지 않습니다.  
> DDD는 도메인 모델을 중심으로 비즈니스 요구사항을 이해하고 설계하는 데 초점을 두어 객체지향을 추구할 수 있는 반면, 소프트웨어의 안정성을 확보하기 위한 시스템적인 해결책은 아닙니다.   
> TDD에 DDD의 이론을 차용하면 TDD가 고려하지 못하는 맥락적이고 서술적인 설계 내용을 보완할 수 있습니다.  
> 이러한 배경에서 BDD가 탄생했습니다. TDD에 DDD를 끼얹은 것이 BDD입니다.




















