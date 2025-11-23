# 모듈

모듈과 모듈 시스템이 무엇인지 알고 있다면 다음과 같은 질문에 대답할 수 있어야 합니다.

- 소프트웨어에서 말하는 모듈이나 모듈 시스템이란 무엇일까요 ?
- 자바의 패키지는 왜 모듈 시스템이 될 수 없을까요 ?
- 자바 9부터 추가된 모듈 시스템(module-info, java)은 무엇이고 왜 추가된 것일까요 ?
- 자바9부터 모듈 사스템(module-info, java)이 추가됐다면 자바9 이전에는 모듈 사스템이 없었다고 봐야 할까요 ?

## 모듈성

> 소프트웨어 관점에서 모듈이란 독립성(independence)과 은닉성(hiding)을 만족하며 연관된 코드들의 묶음입니다.  
> 1. 독립성: 모듈은 독립적이어야 한다.
> 2. 은닉성: 모듈의 사용자는 모듈의 내부 구현을 몰라도 된다. 공개된 인터페이스를 이용해 모듈과 통신한다

> 모듈 시스템이란 연관된 코드 묶음이 모듈성을 갖출 수 있게 도와주는 시스템적인 해결책입니다.  
> 모듈성을 지원하기 위해 모듈 시스템은 다음과 같은 기능을 필수적으로 지원해야 합니다.  
> 1. 의존성 관리: 모듈을 사용하기 위해 어떤 의존성이 필요한지 명시할 수 있어야 한다.
> 2. 캡슐화 관리: 모듈은 불필요한 구현을 외부로 드러내지 않아야 한다.

이를 정리하면 아래와 같습니다.

- 모듈은 연관된 코드의 묶음이 모듈성을 갖춘 경우 모듈이라고 부를 수 있다.
- 모듈성에는 독립성과 은닉성이라는 특징이 있다.
- 모듈성을 갖출 수 있게 도와주는 시스템이 모듈 시스템이다.
- 모듈 시스템은 모듈성 중 독립성을 지원하기 위해 의존성 관리 기능을 지원할 수 있어야 한다.
- 모듈 시스템은 모듈성 중 은닉성을 지원하기 위해 캡슐화 관리 기능을 지원할 수 있어야 핸다.

```javascript
// my.js
import * as log4js from "log4jsH;
        
const logger = log4js.getLogger();

function printFoobar() {
    logger.info('foobar");
)
    
export function printHelloWorld() {
    logger.info("Hello world!");
)
```

printHelloWorld 함수는 export하고, printFoobar 함수는 export하지 않습니다.  
그래서 my.js를 import하는 파일에서는 printHelloWorld 함수에 접근해 이 함수를 사용할 수 있지만 printFoobar는 사용할 수 없습니다.

따라서 모듈 수준의 의존성 관리와 캡슐화란 이처럼 개발자가 어떤 모듈을 정의하면서 해당 모듈에 필요한 의존 모듈이 무엇인지 명시할 수 있고, 해당 모듈을 배포할 때도 모든 내용을 배포하는 것이 아니라 일부만 배포할 수 있는 것을 의미합니다.  
이를 위해 모듈 시스템은 모듈이 가진 독립성과 은닉성을 보장하기 위해 의존성 관리와 캡슐화 관리를 모듈 수준에서 지원할 수 있어야 합니다.

**그러므로 자바의 패키지 시스템은 모듈 시스템이라고 볼 수 없습니다.**  
왜냐하면 자바의 패키지 시스템은 패키지 수준의 의존성 관리와 캡슐화 관리 기능을 지원하지 않기 때문입니다.  
자바의 패키지 시스템은 연관성 높은 코드를 묶는 수단에 불과합니다. 좀 더 노골적으로 말해 자바의 패키지는 폴더에 가깝습니다.


```java
module myproject.main {
    requires org.apache.httpcomponents.httpclient;
    exports com.myorg.myproject.model;
}
```

**대신 자바에는 더 확실한 모듈 시스템이 존재합니다. 바로 module-info. java라고 하는 모듈 디스크립터입니다.**  

### 독립성

**'모듈은 독립적이어야 한다'라는 의미는 모듈이 다른 모듈이나 컴포넌트에 강하게 의존하지 않고 각 모듈을 개별적으로 수정하거나 교체할 수 있어야 한다는 뜻입니다.**  
그리고 모듈이 독립적이어야 하는 이유는 유지보수를 용이하게 하고, 확장성을 높이고, 코드의 재사용성을 높이기 위함입니다.  
독립적인 모듈은 개발 과정에서의 효율성을 높이고 시스템 전체의 안정성을 유지하는 데 도움이 됩니다. 또한 코드를 테스트하기 쉽게 만들 수 있어 전체 시스템의 품질을 높이는 데 기여합니다.

독립성을 이해하기에 앞서 이 말을 가장 먼저 이해해야 합니다. 어떤 시스템이 독립적이어야 한다라는 말은 대상이 외부 시스템과 완전히 격리돼야 한다는 말이 아닙니다.  
독립적이다라는 말은 외부에 의존하는 상황이 생기는 것 자체를 부정하지 않습니다! 외부에 의존할 때 강한 의존이 생기는 것을 피하라는 의미일 뿐입니다.

- 최대한 내부에서 해결하라.
- 외부에는 강하게 의존하지 마라.
- 외부 시스템을 사용한다면 외부 시스템의 사용을 명시하라.

### 은닉성

모듈이 은닉성을 추구해야 한다는 말은 클래스가 은닉성을 추구하는 것처럼 모듈 수준의 캡슐화가 가능해야 한다는 말입니다.  
즉, 우리는 모듈을 외부에 공유하더라도 공개된 인터페이스 이외에 불필요한 정보를 숨길 수 있길 원합니다.

> API 사용자가 충분히 많다면 계약을 어떻게 했는지는 크게 중요하지 않습니다. 시스템의 모든 관측 가능한 행동은 사용자에 의해 결정될 것입니다.  
> ㅡ 하이럼 라이트(Hyrum Wright)

이 법칙은 'API를 사용하는 사용자가 충분히 많다면 개발자의 설계 의도는 더 이상 중요하지 않습니다.' 라고 의역할 수 있습니다.  
문제는 이렇게 생긴 암묵적인 책임과 부담이 고스란히 API 개발자의 몫이라는 것입니다.    
라이브러리 사용자가 라이브러리의 모든 사용 권한을 갖게 되는 것은 마냥 좋게 느껴질 수도 있지만 실은 그렇지 않습니다.  
사용자의 행동은 예측할 수 없습니다. 하물며 모듈의 사용자도 마찬가지입니다.   


## 패키지 구조

일반적으로 스프링을 기반으로 하는 프로젝트를 보면 크게 두 가지 방식으로 패키지 구조를 구성하는 것을 볼 수 있습니다.

- 계층 기반 구조: 계층 이름이 먼저 나오는 패키지 구성 방식
- 도메인 기반 구조: 도메인 이름이 먼저 나오는 패키지 구성 방식


### 계층 기반 구조

```
project
└─ src/main/java
   └─ com.demo.myapp
      ├─ presentation
      │  ├─ UserController.java
      │  ├─ CafeController.java
      │  ├─ BoardController.java
      │  └─ PostController.java
      ├─ business
      │  ├─ UserService.java
      │  ├─ CafeService.java
      │  ├─ BoardService.java
      │  ├─ PostService.java
      │  └─ repository
      │     ├─ UserRepository.java
      │     ├─ CafeRepository.java
      │     ├─ BoardRepository.java
      │     └─ PostRepository.java
      ├─ domain
      │  ├─ User.java
      │  ├─ Cafe.java
      │  ├─ Board.java
      │  └─ Post.java
      └─ infrastructure
         ├─ UserRepositoryImpl.java
         ├─ UserJpaEntity.java
         ├─ UserJpaRepository.java
         ├─ CafeRepositoryImpl.java
         ├─ CafeJpaEntity.java
         ├─ CafeJpaRepository.java
         ├─ BoardRepositoryImpl.java
         ├─ BoardJpaEntity.java
         ├─ BoardJpaRepository.java
         ├─ PostRepositoryImpl.java
         ├─ PostJpaEntity.java
         └─ PostJpaRepository.java
```

이러한 계층 기반 구조를 사용할 때 가장 큰 장점은 이해하기 쉽고 사용하기 쉽다는 점입니다.  
패키지 구조가 레이어드 아키텍처의 인지 모델을 그대로 따르고 있습니다. 그래서 각 레이어에 관한 약간의 이해와 이에 대응하는 스프링 컴포넌트가 무엇인지만 알면 누구나 쉽게 개발할 수 있습니다.

반면 단점은 도메인이 눈에 들어오지 않는다는 점입니다.  
패키지 구조를 봐도 이 애플리케이션이 어떤 애플리케이션인지 알 수 없습니다. 더불어 애플리케이션에 어떤 도메인이 사용되는지 파악하려면 모든 계층을 열어보고 정리해야만 합니다.  
그래서 도메인 관점의 응집도가 떨어지고, 그 결과 비즈니스 코드를 한곳에 모아 볼 수 없습니다.  

### 도메인 기반 구조

```
project
└─ src/main/java
   └─ com.demo.myapp
      ├─ user
      │  ├─ UserController.java
      │  ├─ UserService.java
      │  ├─ UserRepository.java
      │  ├─ User.java
      │  ├─ UserRepositoryImpl.java
      │  ├─ UserJpaEntity.java
      │  └─ UserJpaRepository.java
      ├─ cafe
      │  ├─ CafeController.java
      │  ├─ CafeService.java
      │  ├─ CafeRepository.java
      │  ├─ Cafe.java
      │  ├─ CafeRepositoryImpl.java
      │  ├─ CafeJpaEntity.java
      │  └─ CafeJpaRepository.java
      ├─ board
      │  ├─ BoardController.java
      │  ├─ BoardService.java
      │  ├─ BoardRepository.java
      │  ├─ Board.java
      │  ├─ BoardRepositoryImpl.java
      │  ├─ BoardJpaEntity.java
      │  └─ BoardJpaRepository.java
      └─ post
         ├─ PostController.java
         ├─ PostService.java
         ├─ PostRepository.java
         ├─ Post.java
         ├─ PostRepositoryImpl.java
         ├─ PostJpaEntity.java
         └─ PostJpaRepository.java
```

이러한 구성의 가장 큰 장점은 도메인 코드를 한곳으로 모음으로써 비즈니스 코드가 여기저기 산재되지 않게 됐다는 것입니다.  
더불어 프로젝트의 패키지 최상단에 도메인을 드러낸 덕분에 패키지 구조만 보고도 이 프로젝트가 어떤 도메인을 사용하고 있는지 알 수 있게 됐습니다.

하지만 이런 구조로는 계층이 눈에 들어오지 않게 됩니다. 이로 인해 계층별로 대응되는 스프링 컴포넌트가 어디에 있는지도 눈에 안 들어옵니다. 이것은 분명 좋은 현상이 아닙니다.

## 패키지와 모듈

모듈이 추구하는 가치인 독립성과 은닉성은 소프트웨어 개발의 기본 원리입니다.  
더불어 시스템의 복잡도를 낮추고 확장성을 높일 수 있다는 것이 검증된 확실한 전략입니다.  
그러니 어떤 대상을 모듈화하겠다라는 생각으로 접근하는 것보다 그냥 언제 어디서든 독립성과 은닉성을 습관적으로 추구하는 것이 좋습니다.











