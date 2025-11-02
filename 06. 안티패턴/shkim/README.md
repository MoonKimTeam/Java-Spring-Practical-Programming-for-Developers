# 안티 패턴

## 스마트 UI

스마트 UI 패턴은 에릭 에반스의 저서 **도메인 주도 설계**에서 소개돼 유명해진 안티패턴입니다.  
에릭 에반스가 말하는 스마트 UI는 다음과 같은 특징을 가진 코드를 말합니다.

1. 스마트 UI는 데이터 입출력을 UI 레벨에서 처리합니다.
2. 스마트 UI는 비즈니스 로직도 UI 레벨에서 처리합니다.
3. 스마트 UI는 데이터베이스와 통신하는 코드도 UI 레벨에서 처리합니다.

즉, 스마트 UI란 시스템의 UI 레벨에서 너무 많은 업무를 처리하고 있는 경우를 의미합니다.  
백엔드 개발자에게 API는 UI입니다. 그리고 컨트롤러는 API를 만드는 컴포넌트입니다. 그렇다면 컨트롤러는 스프링에서 UI를 만드는 도구라고 볼 수 있습니다.  
**스마트 UI는 컨트롤러의 핸들러 메서드에 지나치게 많은 로직이 들어있는 경우를 뜻하겠네요.**  

> 스마트 UI 방식으로 개발된 애플리케이션은 설계에 구조라고 부를 만한 것이 존재하지 않습니다. 모든 코드가 오롯이 기능이 동작하게 만드는 데만 초점을 맞춰 작성됩니다.  
> 그러한 탓에 사실상 모든 API는 어떤 스크립트를 실행하고 응답하는 수준에 그칩니다. 그래서 이러한 코드는 당연히 확장성이 떨어지고 유지보수성도 떨어집니다.

컨트롤러의 가장 큰 역할은 엔드포인트를 정의하고 API 사용자의 요청을 받아 그 결과를 응답 포맷에 맞춰 반환하는 것입니다.  
컨트롤러에 비즈니스 로직이 있어서는 안 됩니다. 데이터베이스 관련 로직이 있어도 안 됩니다.

## 양방향 레이어드 아키텍처

레이어드 아키텍처는 소프트웨어 시스템을 설계하는 방식 중 하나로, 이름에서도 알 수 있듯이 '레이어'라고 불리는 분류 체계를 사용합니다.  

1. presentation layer:  사용자와의 상호작용을 처리하고 결과를 표시하는 역할을 담당합니다.
2. business layer:  애플리케이션의 비즈니스 로직을 처리하는 역할을 합니다. 그래서 데이터의 유효성 검사, 데이터 가공 비즈니스 규칙 적용 등의 일이 이 레이어에서 이뤄집니다.
3. infrastructure layer: 외부 시스템과의 상호작용을 담당합니다. 예를 들어, 대표적인 외부 시스템으로 데이터베이스가 있습니다. 

그렇다면 레이어드 아키텍처를 이렇게 구성할 때 얻을 수 있는 장점은 무엇일까요?  
가장 큰 장점은 '단순하고 직관적인 구조'라는 것입니다.  
즉, 어떤 컴포넌트를 개발하거나 찾아야 할 때 컴포넌트를 어디에 위치시켜야 할지 고민할 필요가 없습니다.

**양방향 레이어드 아키텍처는 레이어드 아키텍처를 지향해 개발했지만, 레이어드 아키텍처가 반드시 지켜야 할 가장 기초적인 제약을 위반할 때를 지칭하는 말입니다.**  
그리고 여기서 가장 기초적인 제약이란 레이어 간 의존 방향은 단방향을 유지해야 한다'라는 것입니다.

```java
@Service
@RequiredArgsConstructor
public class PostService {

    private final CafeMemberJpaRepository cafeMemberJpaRepository;
    private final BoardJpaRepository boardJpaRepository;
    private final PostJpaRepository postJpaRepository;

    @Transactional
    public Post create(
        long cafeId,
        long boardId,
        long writerId,
        PostCreateRequest postCreateRequest // API 요청을 받는 모델인데 비즈니스 레이어에서 사용함
    ) {
        long currentTimestamp = Instant.now().toEpochMilli();
        CafeMember cafeMember = cafeMemberJpaRepository
            .findByCafeIdAndUserId(cafeId, writerId);
        
        // ...
        // ...
    }
}
```

PostCreateRequest 클래스는 API 레이어의 모델입니다. 
API로 들어오는 요청을 ©RequestBody 애너테이션을 이용해 매핑하려고 만든 객체인데, 하위 레이어에 존재하는 서비스 컴포넌트로 전달 해 서비스에서 이를 사용하고 있는 상황입니다.  
비즈니스 레이어에 위치한 서비스 컴포넌트가 프레젠테이션 레이어에 위치한 객체에 의존하는 바람에 두 레이어 간에 양방향 의존 관계가 생겼습니다,  
이처럼 레이어 간에 양방향 의존성이 생긴 상황을 가리켜 '양방향 레이어드 아키텍처'라고 부릅니다.

## 완화된 레이어드 아키텍처

컨트롤러가 리포지터리를 사용하는 것은 괜찮을까요?  
이 질문의 답을 먼저 이야기하자면 '컨트롤러가 리포지터리를 사용할 수 있게 하는 것은 좋지 못하다'입니다.  
왜냐하면 일반적으로 이처럼 2개 이상의 레이어를 건너뛰어 통신하는 구조도 안티패턴으로 분류하기 때문입니다.  
그래서 이처럼 상위 레이어에 모든 하위 레이어에 접근할 수 있는 권한을 주는 구조를 가리켜 완화된 레이어드 아키텍처라고 부릅니다.

## 트랜잭션 스크립트

트랜잭션 스크립트는 비즈니스 레이어에 위치하는 서비스 컴포넌트에서 발생하는 안티패턴입니다.  
트랜잭션 스크립트는 서비스 컴포넌트의 구현이 사실상 어떤 '트랜잭션이 걸려있는 스크립트'를 실행하는 것처럼 보일 때를 말합니다.

```java
@Service
@RequiredArgsConstructor
public class PostService {

    private final CafeMemberJpaRepository cafeMemberJpaRepository;
    private final BoardJpaRepository boardJpaRepository;
    private final PostJpaRepository postJpaRepository;

    @Transactional
    public Post create(
        long cafeId,
        long boardId,
        PostCreateCommand postCreateCommand
    ) {
        long currentTimestamp = Instant.now().toEpochMilli();

        CafeMember cafeMember = cafeMemberJpaRepository
            .findByCafeIdAndUserId(cafeId, postCreateCommand.getWriterId())
            .orElseThrow(() -> new ForbiddenAccessException());

        User writer = userJpaRepository
            .findById(postCreateCommand.getWriterId())
            .orElseThrow(() -> new UserNotFoundException());

        Cafe cafe = cafeMember.getCafe();

        Board board = boardJpaRepository
            .findById(boardId)
            .orElseThrow(() -> new BoardNotFoundException());

        Post post = new Post();
        post.setTitle(postCreateCommand.getTitle());
        post.setContent(postCreateCommand.getContent());
        post.setCafe(cafe);
        post.setBoard(board);
        post.setWriter(writer);
        post.setCreatedTimestamp(currentTimestamp);
        post.setModifiedTimestamp(currentTimestamp);

        post = postJpaRepository.save(post);
        cafe.setNewPostTimestamp(currentTimestamp);
        cafe = cafeJpaRepository.save(cafe);

        return post;
    }
}
```

서비스 컴포넌트의 동작이 사실상 트랜잭션이 걸려있는 거대한 스크립트를 실행하는 것처럼 보입니다.  
어떻게 보면 스마트 UI와 비슷한 맥락으로 트랜잭션 스크립트는 '스마트 서비스'라고 부를 수도 있습니다.  
객체지향보다 절차지향에 가까운 사례이기 때문에 절차지향의 문제점을 그대로 가집니다. 변경에 취약하고 확장에 취약하며 업무가 병렬 처리되기 어렵습니다.

그래서 이 같은 패턴을 피하려면 서비스의 역할이 무엇인지 재고해야 합니다.  
서비스란 무엇이고 서비스의 역할이 어떤 것인지 이해해야 이 패턴을 피할 수 있습니다.  

**비즈니스 로직은 도메인 모델에 위치해야 합니다.**  
도메인 모델이라는 말이 아직 익숙하지 않다면 Cafe, Post. Board, User 같은 객체들을 떠올리면 됩니다. 비즈니스 로직은 이러한 객체들이 갖고 있어야 합니다.
비즈니스 로직이 처리되는 '주(main)' 영역은 도메인 모델이어야 합니다. 서비스 컴포넌트가 아닙니다.  
서비스는 도메인을 불러와서 도메인에 일을 시키는 정도의 역할만 해야 합니다.






















