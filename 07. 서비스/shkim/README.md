# 서비스

서비스의 역할은 도메인 객체나 도메인 서비스라고 불리는 도메인에 일을 위임하는 공간이어야 합니다.  
이는 서비스의 역할을 크게 3가지 종류의 일을 해야 한다고 볼 수 있습니다.

1. 도메인 객체를 불러옵니다.
2. 도메인 객체나 도메인 서비스에 일을 위임합니다.
3. 도메인 객체의 변경 사항을 저장합니다.

## Manager

대부분의 개발자가 서비스라는 컴포넌트를 그렇게 많이 사용하면서도 이 컴포넌트를 왜 서비스라고 부르는지 알지 못합니다.  
그리고 정확히 무엇을 처리하는 공간인지도 모릅니다. 그래서 이 질문의 그나마 유의미한 답변은 '서비스는 비즈니스 서비스를 처리하는 곳입니다'라는 식의 답변입니다.  
이에 대한 해답을 얻으려면 스프링의 ©Service 애너테이션이 작성된 실제 코드를 찾아 가야 합니다.

<img width="608" height="435" alt="Image" src="https://github.com/user-attachments/assets/60fb1d1e-b811-49f9-b8f8-7b116b58ff02" />

이를 요약하면 다음과 같습니다.

- @Service는 에릭 에반스의 DDD에서 영감을 받아 만들어진 애너테이션이다.
- 서비스는 J2EE 패턴 중 하나인 비즈니스 서비스 파사드처럼 사용될 수 있다.

도메인(domain)이란 비즈니스 영역이자 우리가 해결하고 싶은 문제 영역(domain)입니다.  
왜냐하면 우리가 소프트웨어를 개발하는 이유가 곧 도메인에서 발생하는 문제를 해결하기 위함이기 때문입니다.  
DDD 세상의 개발자는 도메인에 대해서도 잘 알고 있어야 합니다.

다음은 DDD의 창시자 에릭 에반스가 서비스를 설명하면서 쓴 글입니다.

> 자신의 본거지를 ENTITY나 VALUE OBJECT에서 찾지 못하는 중요한 도메인 연산이 있다.  
> 이들 중 일부는 본질적으로 사물이 아닌 활동(activity)이나 행동(action)인데, 우리의 모델링 패러다임이 객체이므로 그러한 연산도 객체와 잘 어울리게끔 노력해야 한다

**이것이 바로 스프링 서비스의 정체입니다, 서비스는 도메인 객체가 처리하기 애매한 '연산' 자체를 표현하기 위한 컴포넌트입니다.**  
그런데 시스템을 개발하다 보면 분명 시스템을 구성하는 데 필요한 로직이지만 로직을 도메인 세상 속 객체에 녹이기 어려울 때가 있습니다.  
예를 들어 물건을 파는 어떤 사이트를 상상해 봅시다. 이 서비스에는 상품(Product), 쿠폰(Coupon), 사용자(User)의 마일리지(Mileage)라는 도메인이 있습니다.  
그리고 물건의 가격을 계산하기 위해 다음과 같은 계산식을 사용합니다.  

> 가격 = 상품 가격 - (상품 가격 * 쿠폰 최대 할인율) - 사용자 마일리지

’서비스에 있는 비즈니스 로직을 도메인 객체가 처리하게 만들라'라는 조언에 따라 가격 계산 로직을 도메인 객체가 처리하도록 만들어 보겠습니다.  
이 로직은 Product, Coupon, User라는 세 가지 도메인 모델 중 어디로 들어가는 것이 좋을까요?  
안타깝게도 가격을 계산하는 로직은 모든 도메인 객체가 처리하기 애매합니다. 왜냐하면 이러한 로직을 능동적인 객체에 표현하는 것 자체가 어렵기 때문입니다.  
**갖고 있는 도메인 객체에 로직을 밀어 넣을수 없다면 새로운 클래스를 만들고 그쪽으로 밀어 넣으면 됩니다.**

```java
@Service
@RequiredArgsConstructor
public class ProductService {
    private final UserJpaRepository userJpaRepository;
    private final ProductJpaRepository productJpaRepository;
    private final CouponJpaRepository couponJpaRepository;

    public int calculatePrice(long userId, long productId) {
        User user = userJpaRepository.getById(userId);
        Product product = productJpaRepository.getById(productId);
        List<Coupon> coupons = couponJpaRepository.getByUserId(userId);

        PriceManager priceManager = new PriceManager();
        return priceManager.calculate(user, product, coupons);
    }
}
```

PriceManager라는 매니저 클래스를 만들고 비즈니스 로직을 이 클래스 안으로 옮겼습니다, 코드를 매니저라는 클래스로 위임한 것입니다.  
여기서 2개의 서비스가 있다고 볼 수 있는데, 하나는 스프링 컴포넌트를 이용해서 만든 ProductService이고 또 다른 하나는 가격 계산 로직을 표현하기 위해 만든 PriceManager입니다.  
서비스가 서비스를 실행시키는 것은 좋은데, 이 둘을 구분할 필요가 있어 보입니다. 양쪽 모두 어떤 도메인 객체로 표현하기 애매한 연산 로직을 모아둔 클래스인 것은 맞지만 그 성격이 조금씩 다르기 때문입니다.  
**PriceManager•는 도메인 시스템을 구축하기 위해 존재합니다. 그리고 가격을 계산한다는 점과 가격을 계산하는 비즈니스 업무 규칙을 갖고 있으므로 '도메인'에 가까운 로직입니다.**  
**한편 ProductService는 다릅니다. 스프링에서 사용하는 ©Service 애너테이션으로 만들어진 ProductService는 도메인에 필요한 비즈니스 업무 규칙을 갖고 있다기보다 애플리케이션이 돌아가는 데 필요한 연산을 갖고 있는 서비스입니다.**  
이에 따라, ProductService 같은 서비스는 '애플리케이션 서비스'라고 부르고, PriceManager 같은 서비스를 보고 '도메인 서비스'라고 부릅니다.

| 분류 | 역할 | 주요 행동 | 예시 |
| :--- | :--- | :--- | :--- |
| **도메인** | 비즈니스 로직을 처리 | * 도메인 역할을 수행한다. <br> * 다른 도메인과 협력한다. | User, Product, Coupon |
| **도메인 서비스** | 비즈니스 '연산' 로직을 처리 | * 도메인 협력을 중재한다. <br> * 도메인 객체에 기술할 수 없는 연산 로직을 처리한다. | PriceManager |
| **애플리케이션 서비스** | 애플리케이션 '연산' 로직을 처리 | * 도메인을 저장소에서 불러온다. <br> * 도메인 서비스를 실행한다. <br> * 도메인을 실행한다. | ProductManager |

## 서비스보다 도메인 모델

가격 계산 로직을 옮길 만한 적절한 도메인을 찾지 못해 PriceManager라는 클래스를 만들었습니다.  
하지만 사실 이 로직은 도메인 객체로 옮길 수 있습니다.  
기존의 도메인 객체로 비즈니스 연산 로직을 옮길 수 없다면 새로운 도메인을 만들면 됩니다.

```java
@Service
@RequiredArgsConstructor
public class ProductService {
    private final UserJpaRepository userJpaRepository;
    private final ProductJpaRepository productJpaRepository;
    private final CouponJpaRepository couponJpaRepository;

    public int calculatePrice(long userId, long productId) {
        User user = userJpaRepository.getById(userId);
        Product product = productJpaRepository.getById(productId);
        List<Coupon> coupons = couponJpaRepository.getByUserId(userId);

        Cashier cashier = new Cashier();
        return cashier.calculate(user, product, coupons);
    }
}
```


Cashier라는 이름에서 알 수 있듯 이 객체는 점원의 역할을 수행하며 능동적으로 일할 것 같습니다.  
PriceManager라는 이름을 들으면 PriceManager가 가격과 관련된 연산 로직만 갖고 있을 것처럼 느껴집니다.  
그래서 이 클래스로 만들어진 객체는 객체지향이라는 역할극에 어떤 유의미한 인물로 나오는 것이 아니라 그저 계산식을 여러 개 갖고 있는 장치처럼 사용될 것 같습니다.

더불어 PriceManager 클래스는 앞으로 시스템이 성장하면서 가격과 관련된 모든 연산 로직이 모이는 곳이 될 확률이 높습니다.  
그래서 모든 동작이 수동적인 객체들한테 값을 가져와 연산 로직을 수행하는 방식으로 동작할 것입니다.

**도메인과 도메인 서비스는 이름으로 결정되는 것이 아니라는 것입니다.**  
도메인과 도메인 서비스를 구분 짓는 것은 행동으로 결정됩니다.

## 작은 기계

- 서비스는 한번 생성하면 여러 번 사용하지만 그 자신은 바꿀 수 없다.
- 서비스는 작은기계처럼 영원히 실행할 수 있다

이 말은 서비스는 불변해야 한다는 것을 가리킵니다.  
서비스의 상태가 변경된다면 서비스는 영원히 같은 일을 할 수 없습니다.  
서비스는 불변성을 유지하고 예측할 수 있는 컴포넌트가 돼야 합니다. 같은 입력에는 항상 같은 결과만 나와야 합니다.

필드 주입과 수정자 주입을 사용하는 서비스는 클래스를 불변으로 만들지 못합니다.  
그래서 원론적으로 서비스를 정의할 때 이 두 주입 방식을 이용해 클래스를 만들지 않는 것이 옳습니다.



















