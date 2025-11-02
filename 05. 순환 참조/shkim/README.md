# 순환 참조

순환 참조는 두 개 이상의 객체나 컴포넌트가 서로를 참조함으로써 의존 관계에 사이클이 생기는 상황을 말합니다.  
예를 들어, 객체 A가 객체 B를 참조하고, 객체 B가 다시 객체 A를 참조하는 양방향 참조는 대표적인 순환 참조의 예입니다.  
그리고 이러한 순환 참조는 소프트웨어 설계에서 자주 볼 수 있는 대표적인 안티패턴 중 하나입니다.

```java
@Data
@NoArgsConstructor
@Entity(name = "team")
class TeamJpaEntity {

    @Id
    private String id;

    @Column
    private String name;

    @OneToMany(mappedBy = "myTeam")
    private List<MemberJpaEntity> members;
}

@Data
@NoArgsConstructor
@Entity(name = "member")
class MemberJpaEntity {

    @Id
    private String id;

    @Column
    private String name;

    @ManyToOne
    @JoinColumn(name = "my_team_id")
    private TeamJpaEntity myTeam;
}
```

> 순환 참조는 소프트웨어 설계에서 피해야 하는 잘 알려진 대표적인 안티패턴입니다.  
> 순환 참조가 발생한다는 것은 서로에게 강하게 의존한다는 의미입니다. 사실상 하나의 컴포넌트라는 의미이며, 책임이 제대로 구분돼 있지 않다는 의미입니다.  
> 따라서 순환 참조가 있는 컴포넌트는 SOLID하지도 않습니다.


## 순환 참조의 문제점

### 무한 루프

순환 참조가 있다는 말은 시스템에 무한 루프가 발생할 수 있다는 말입니다.   
객체 A가 객체 B의 메서드를 호출하고 객체 B가 객체 A의 또 어떤 메서드를 호출해서 무한 루프가 만들어지는 것입니다.  
무한 루프는 개발자가 메서드 호출 과정을 신경 쓴다고 해결할 수 있는 문제가 아닙니다.  
소프트웨어는 복잡계라서 언제, 어디서, 어떤 식으로 부작용이 발생할지 모릅니다. 그러므로 순환 참조로 인해 발생할 잠재적 위험을 안고 갈 이유가 없습니다.

### 시스템 복잡도

```java
@Data
class Team {

    private long id;
    private String name;
    private List<Member> members;
}

@Data
class Member {

    private long id;
    private String name;
    private Team myTeam;
    private int salary;

    // 팀 내 모든 팀원의 월급의 합을 반환합니다.
    public int calculateTeamMemberTotalSalary() {
        int result = 0;
        for (Member member : myTeam.getMembers()) {
            result += member.salary;
        }
        return result;
    }
}
```

순환 참조는 시스템의 복잡도를 높입니다.  
Member 클래스에 소속된 팀의 전체 월급이 얼마인지 알 수 있는 메서드가 추가됐습니다.  
**팀원이 팀 내 전체 구성원의 월급을 알 수 있는 시스템이라니 현실 세계에선 벌어질 수 없는 살짝은 이상한 상황입니다.**  
중요한 것은 잘못된 설계로 인해 '이런 코드가 만들어질 수도 있다'라는 사실입니다.  
의미상으로 그렇게 되어서는 안 되는 코드는 컴파일 타임에 아예 만들어지지 않게 해야 합니다.

순환 참조가 있으면 어떤 객체에 접근할 수 있는 접근 경로가 너무 많아집니다.  
'접근 경로가 많다'라는 말은 언뜻 보면 좋게 들릴 수도 있으나 소프트웨어 설계에서는 그렇게 좋은 말은 아닙니다.  
가능한 한 도메인 모델들에 단일 진입점을 만들어서 필요한 객체가 있을 때 단방향으로 접근하도록 만드는 것이 좋습니다.

## 순환 참조를 해결하는 방법

### 불필요한 참조 제거

불필요한 참조를 제거한다는 것은 양방향 참조가 꼭 필요한지 재고해 본다는 의미입니다.  
그래서 꼭 필요하지 않은 참조를 제거하거나 필요에 따라 관계를 표현하긴 해야 한다면 한쪽이 다른 한쪽의 식별자를 갖고 있게 해서 간접 참조 형태로 관계를 바꾸는것입니다.  
TeamJpaEntity 클래스에 있는 팀원 목록이 꼭 필요한지 고민해 봅니다.  
'List<MemberJpaEntity> members' 선언은 꼭 필요할까요? 프로젝트마다 상황이 다르겠지만 TeamJpaEntity 클래스가 팀원 목록을 모두 갖고 있는 것이 조금은 과하다는 생각이 들 수 있습니다.  

### 간접 참조 활용

```java
@Data
@NoArgsConstructor
@Entity(name = "team")
class TeamJpaEntity {

    @Id
    private String id;

    @Column
    private String name;

    @OneToMany(mappedBy = "myTeam")
    private List<MemberJpaEntity> members;
}

@Data
@NoArgsConstructor
@Entity(name = "member")
class MemberJpaEntity {

    @Id
    private String id;

    @Column
    private String name;

    @Column(name = "my_team_id")
    private long myTeamId;
}
```

MemberJpaEntity 클래스가 갖고 있던 TeamJpaEntity 클래스로의 참조를 없애고 myTeamld 변수를 뒀습니다.  
그리고 이 같은 코드에서 팀원은 팀이 필요할 때 TeamJpaRepository.findByld(teamld) 같은 메서드를 호출해 팀 정보를 불러올 수 있습니다.  
이처럼 간접 참조를 활용한다는 의미는 기존에 직접 참조하던 것을 참조 객체의 식별값을 이용해 참조하도록 바꾼다는 의미입니다.

### 공통 컴포넌트 분리

만약 서비스 같은 컴포넌트에 순환 참조가 있고, 그것이 각 컴포넌트의 설정상 필수적이라면 어떻게 이문제를 해결할 수 있을까요?  
가장 간단하고 효과적인 방법으로 다음과 같이 공통 컴포넌트를 분리하는 방법이 있습니다.

<img width="758" height="260" alt="Image" src="https://github.com/user-attachments/assets/ef948ee0-af8d-4fb5-8286-73c4944c6ccc" />

즉, 양쪽 서비스에 있던 공통 기능을 하나의 컴포넌트로 분리하는 것입니다. 그러고 나서 양쪽 서비스가 공통 컴포넌트에 의존하도록 바꾸면 순환 참조가 없어집니다.  
이 방법의 또 다른 장점으로는 공통 기능을 분리하는 과정에서 책임 분배가 적절하게 재조정된다는 점입니다.

### 이벤트 기반 시스템 사용

서비스를 공통 컴포넌트로도 분리할 수 없다면 이벤트 기반 프로그래밍을 시스템에 적용할 수 있습니다.  

1. 시스템에서 사용할 중앙 큐를 만듭니다
2. 필요에 따라 컴포넌트들이 중앙 큐를 구독하게 합니다.
3. 컴포넌트들은 자신의 역할을 수행하던 중 다른 컴포넌트에 시켜야 할 일이 있다면 큐에 이벤트를 발행합니다
4. 이벤트가 발행되면 큐를 구독하고 있는 컴포넌트들이 반응합니다.
5. 컴포넌트들은 이벤트를 확인하고 자신이 처리해야 하는 이벤트라면 이를 읽어 처리합니다.
6. 컴포넌트들은 자신이 처리하지 않아도 되는 이벤트라면 무시합니다.

이 구조에서 서비스는 더 이상 서로를 상호 참조하지 않습니다. 대신 이벤트와 이벤트 큐에 의존합니다.  
이벤트와 이벤트 큐가 인터페이스이자 곧 메시지가 되는 것입니다. 이벤트 기반 시스템은 객체 간의 통신을 이벤트로 이뤄지게 해서 결합을 느슨하게 만들어 순환 참조를 피할 수 있게 도와줍니다.

## 양방향 매핑

**양방향 매핑은 도메인 설계를 하다가 '어쩔 수 없이' 나오는 순환 참조 문제에 사용하는 것이 바람직하다.**  

양방향 매핑을 사용하지 않아도 얼마든지 개발이 가능합니다. jpa는 수단일 뿐입니다.  
따라서 수단인 JPA로 인해 시스템 설계가 영향을 받아서는 안 됩니다. 심지어 영향을 받아 만들어진 설계 결과물이 안좋은 방향이라면 더더욱 받아들여서는 안 됩니다.  
우리는 순환 참조가 없는 도메인 먼저 구성해야 합니다. 그리고 그다음에 JPA를 연동하는 방식으로 개발해야 합니다.

## 상위 수준의 순환 참조

순환 참조는 객체뿐만 아니라 패키지 사이나 시스템 수준에서도 발생할 수 있는 문제입니다.  
그리고 이러한 순환 참조 문제가 패키지나 시스템 수준에서 발생한다면 이는 객체 간 순환 참조보다 더 큰 문제를 야기할 수 있습니다.  
예를 들어, 서로 다른 회사에서 만든 시스템이 양방향으로 API 호출을 주고받는 상황을 상상해 봅시다.  
그런데 이 두 시스템이 인터페이스 같은 추상 계층 없이 직접 의존하는 형태로 코드가 작성돼 있으면 어떨까요?  
어느 날 갑자기 그 회사가 경영 악화로 시스템 서비스를 중단하겠다고 선언해버리면 우리 시스템도 돌연 서비스를 중단해야 할지 모릅니다.


















