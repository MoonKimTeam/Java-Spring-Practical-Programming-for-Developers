# 절차지향과 비교하기

**자바는 객체지향 언어인데, 절차지향적인 코드가 나올 수 있습니다.**  
절차지향 프로그래밍은 함수를 만들어서 프로그램을 만드는 방식입니다. 복잡한 문제를 개별 함수'로 분해하고, 여러 함수를 이용해 문제를 해결하는 방식입니다.  
그러므로 여러분이 자바나 코틀린 같은 언어를 쓰고 있더라도 함수 위주의 사고 방식으로 프로그램을 만든다면 여전히 절차지향 패러다임으로 개발하고 있는 셈입니다.

```java
class RestaurantChain {

    private List<Store> stores;

}

@Getter
class Store {

    private List<Order> orders;
    private long rentalFee; // 임대료

}

@Getter
class Order {

    private List<Food> foods;
    private double transactionFeePercent = 0.03; // 결제 수수료 3%

}

@Getter
class Food {

    private long price;
    private long originCost; // 원가

}
```

```java
class RestaurantChain {

    private List<Store> stores;

    // 매출을 계산하는 함수
    public long calculateRevenue() {
        long revenue = 0;

        for (Store store : stores) {
            for (Order order : store.getOrders()) {
                for (Food food : order.getFoods()) {
                    revenue += food.getPrice();
                }
            }
        }

        return revenue;
    }

    // 손익을 계산하는 함수
    public long calculateProfit() {
        long cost = 0;

        for (Store store : stores) {
            for (Order order : store.getOrders()) {
                long orderPrice = 0;

                for (Food food : order.getFoods()) {
                    orderPrice += food.getPrice();
                    cost += food.getOriginCost();
                }

                // 결제 금액의 3%를 비용으로 잡는다.
                cost += orderPrice * order.getTransactionFeePercent();
            }

            cost += store.getRentalFee();
        }

        return calculateRevenue() - cost;
    }
}

```

Restaurantchain 클래스에 매출과 순이익을 계산하는 코드를 추가한 것입니다. 이를 객체지향적으로 작성된 코드라고 할 수 있나요?  
안타깝게도 calculateRevenue, calculateProfit 같은 코드는 모두 절차지향적인 코드입니다.  
클래스와 객체들이 Restaurantchain 클래스에 있는 calculateRevenue, calculateProfit 함수를 실행하기 위한 데이터로서 존재할 뿐이기 때문입니다.  
Store, Order, Food를 클래스로 표현했지만 이 클래스에는 아무런 책임이 존재하지 않습니다.  

```java
// 코드 1.5 절차지향처럼 동작하는 RestaurantChainService

@Service
@RequiredArgsConstructor
public class RestaurantChainService {

    private final StoreRepository storeRepository;

    public long calculateRevenue(long restaurantId) {
        List<Store> stores = storeRepository.findByRestaurantId(restaurantId);
        long revenue = 0;

        for (Store store : stores) {
            for (Order order : store.getOrders()) {
                for (Food food : order.getFoods()) {
                    revenue += food.getPrice();
                }
            }
        }

        return revenue;
    }

    public long calculateProfit(long restaurantId) {
        List<Store> stores = storeRepository.findByRestaurantId(restaurantId);
        long cost = 0;

        for (Store store : stores) {
            for (Order order : store.getOrders()) {
                long orderPrice = 0;

                for (Food food : order.getFoods()) {
                    orderPrice += food.getPrice();
                    cost += food.getOriginCost();
                }

                // 결제 금액의 3%를 비용으로 잡는다.
                cost += orderPrice * order.getTransactionFeePercent();
            }

            cost += store.getRentalFee();
        }

        return calculateRevenue(restaurantId) - cost;
    }
}
```

위 코드도 앞에서 설명한 것과 같은 이유로 절차지향적인 코드입니다.  
대부분의 프로젝트들은 레이어드 아키텍처라는 미명하에 절차지향적인 코드에서 벗어나지 못하는 경우가 많았습니다.  
서비스에 모든 비즈니스 로직이 들어가고, 클래스는 그저 데이터를 저장하는 용도로만 사용되고 있었습니다.

이를 객체지향적으로 바꾸려면, 비즈니스 로직을 객체가 처리하도록 변경해야합니다.  
각 객체는 매출과 순이익을 계산해달라는 요청이 들어왔을 때 어떻게 처리해야 할지 알아야 합니다.  
어떤 요청이 들어왔을 때, 어떤 일을 책임지고 처리한다라는 책임이 생겨야 합니다.  
정리하면 다음과 같습니다.

- 객체에 어떤 메시지를 전달할 수 있게 됐다.
- 객체가 어떤 책임을 지게 됐다.
- 객체는 어떤 책임을 처리하는 방법을 스스로 알고 있다.


## 책임과 역할

그런데 절차지향이라서 책임을 제대로 구분할 수 없는 것일까요?  
그렇지 않습니다. 절차지향적인 코드를 통해서도 책임을 분할할 수 있습니다. 절차지향에서는 함수 단위로 책임을 지면 됩니다.  
Store, Order, Food에 '책임이 없다라는 말은 객체지향 관점에서 '객체에 책임이 없다'는 의미인 것입니다.  
**절차지향에서는 책임을 프로시저로 나누고 프로시저에 할당합니다.**  
**객체지향에서는 책임을 객체로 나누고 객체에 할당합니다. 이것이 객체지향에서 말하는 책임입니다.**

> 엄밀히 말하자면 객체지향에서는 책임을 객체에 할당하지 않습니다. 객체를 추상화한 역할에 책임을 할당합니다.  
>  그리고 이는 분명히 c 언어 같은 언어에서는 지원하지 못하는 기능입니다.  
> C 언어의 구조체는 추상의 개념을 지원하지 못합니다. 그러므로 C 언어는 절차지향 언어인 것입니다.

## TDA 원칙

그렇다면 어떻게 해야 절차지향적 사고에서 벗어나 객체지향적인 사고 방식을 가질 수 있을까요?  
개발자들이 객체지향적인 사고를 하도록 만들 수 있는 가장 쉬운 방법은 TDA 원칙을 지켜가며 개발하게 하는 것입니다.  
TDA 원칙이란 'Tell, Don't Ask'의 줄임말입니다. 말 그대로 '물어보지 말고 시켜라'라는 원칙입니다.

```java
public class Shop {

    public void sell(Account account, Product product) {
        if (account.canAfford(product.getPrice())) {
            account.withdraw(product.getPrice());
            System.out.println(product.getName() + "를 구매했습니다.");
        } else {
            System.out.println("잔액이 부족합니다.");
        }
    }
}

class Account {

    private long money;

    // TDA 원칙에 따라 잔액이 물건의 가격보다 큰지 확인
    public boolean canAfford(long amount) {
        return money >= amount;
    }

    public void withdraw(long amount) {
        money -= amount;
    }
}

class Product {

    private String name;
    private long price;

    public long getPrice() {
        return price;
    }

    public String getName() {
        return name;
    }
}
```

단편적으로 이야기하자면 TDA 원칙은 무분별하게 사용되는 getter, setter를 줄이라는 의미로 해석될수도 있습니다.  
그리고 실제로 게터와 세터는 개발자가 객체지향적인 사고를 못하게 하는 방해 요인 중 하나이자 절차지향적인 사고를 하게 만드는 대표적인 원인이기도 합니다.





















