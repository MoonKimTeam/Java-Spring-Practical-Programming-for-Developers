# 도메인

## 소프트웨어 개발의 시작

비즈니스는 현실 세상에서 벌어지는 문제에서 출발합니다. 즉, 사용자가 겪는 문제를 해결해주는 것이 비즈니스입니다.  
소프트웨어는 그러한 해결책 중 하나로 선택될 뿐입니다.  
전통적인 사업가든 린 방식을 추구하는 사업가든 오늘날 대부분의 사업은 고객의 문제에서 출발합니다.  
이들은 고객이 겪는 문제를 파악하고, 요구사항을 분석하고, 요구사항을 정리해 솔루션을 만듭니다. 솔루션은 하드웨어 제품이 될 수도 있고 소프트웨어 제품이 될 수도 있습니다.

**사용자들이 겪는 문제 영역이 바로 도메인입니다.**  
그리고 문제 영역이 곧 비즈니스 영역이므로 도메인은 비즈니스 영역을 의미하기도 합니다. 따라서 도메인은 문제 영역이자 비즈니스 영역입니다.  
우리가 개발해야 하는 것은 그냥 애플리케이션이 아닙니다. ’도메인 애플리케이션입니다. 그러므로 도메인이 무엇인지를 잘 파악하고 있어야 합니다.  
도메인을 제대로 이해하지 못하면 프로덕트의 품질은 전체적으로 낮아질 수밖에 없습니다.

## 애플리케이션의 본질

신규 프로젝트를 시작할땐 도메인을 분석하고 도메인의 요구사항을 정리하는 것이 먼저여야 합니다.  
시스템의 설계와 패턴, 세부 구현은 분석한 도메인을 바탕으로 선택해야 합니다.  
우리의 역할은 도메인 애플리케이션을 만드는 것입니다. 그러니 도메인이 제일 중요합니다.  
프로젝트 설계자 입장에서 프로젝트에 사용될 구현이나 도구는 선택사항이어야 합니다. 이를 전제로 설계에 임해야 유연한 결과물을 얻을 수 있습니다.

## 도메인 모델과 영속성 객체

도메인 모델에 관해 또 한가지 이야기할 것은, '도메인 모델과 영속성 객체는 구분해야 하는가?' 입니다.  

### 통합하기 전략

통합하기 전략은 클래스 하나에 도메인 모델과 영속성 객체의 역할을 모두 몰아넣겠다는 의미입니다.

```java
@Data
@Entity(name = "account")
public class Account {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String email;

    @Column
    private String nickname;

    public void changeNickname(String nickname) {
        this.nickname = nickname;
    }
}
```

개발은 결국 비용 싸움입니다. 그리고 개발자의 시간은 비용입니다.  
이는 같은 코드를 여러 벌 만드느라 개발 속도가 5% 정도만 지연되더라도 조직 차원에서 굉장한 비용을 낭비하는 결과로 이어질 수 있게 된다는 말입니다.  
통합하기 전략에서는 이러한 문제가 발생하지 않습니다. 하나의 클래스만 잘 관리하면 됩니다.

다만 이 전략을 사용할 경우 클래스의 책임이 제대로 눈에 들어오지 않는다는 단점이 있습니다.  
도메인 모델에 영속성 객체와 관련된 코드가 들어 있으면 개발자는 데이터베이스 위주의 사고를 하기 쉽습니다.

### 구분하기 전략

```java
@Builder
public class Account {

    private Long id;
    private String email;
    private String nickname;

    public void changeNickname(String nickname) {
        this.nickname = nickname;
    }
}
```

```java
@Data
@Entity(name = "account")
public class AccountJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String email;

    @Column
    private String nickname;

    public static AccountJpaEntity from(Account account) {
        AccountJpaEntity result = new AccountJpaEntity();
        result.id = account.getId();
        result.email = account.getEmail();
        result.nickname = account.getNickname();
        return result;
    }

    public Account toModel() {
        return Account.builder()
                .id(this.id)
                .email(this.email)
                .nickname(this.nickname)
                .build();
    }
}
```

구분하기 전략을 사용할 것이라면 굳이 ORM을 사용할 이유가 없습니다. 그런데 그것이 구분하기 전략이 추구하는 바입니다.  
더 나아가 구분하기 전략에서는 애플리케이션이 관계형 데이터베이스에도 의존하지 않습니다.  
클래스를 구분했던 이유는 이처럼 도메인의 책임과 데이터 영속의 책임을 구분해 유연함을 얻기 위해서였습니다.

다만 작성해야 하는 코드가 많아진다는 점은 확실한 단점입니다.  
더불어 이렇게 되면 ORM이 갖고 있는 다양한 혜택을 누리기가 어려워집니다.















