# 알아두면 유용한 스프링 활용법

## 타입 기반 주입

스프링에서 @Autowired 애너테이션을 이용한 의존성 주입은 타입을 기반으로 동작합니다.  
의존성 주입이 필요할 경우 스프링 컨테이너는 타입을 기반으로 빈(bean)을 찾는다는 말입니다.  
아래의 경우 Notificationservice 컴포넌트의 멤버 변수 notificationchannel에는 EmailNotificationChannel 빈이 들어갑니다.

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final NotificationChannel notificationChannel;

}
```

```java
public interface NotificationChannel {

    void notify(Account account, String message);
}
```

```java
@Component
public class EmailNotificationChannel implements NotificationChannel {

    @Override
    public void notify(Account account, String message) {
    }
}
```

<br>

이를 응용하여 다음과 같이 사용할 수 있습니다.  
푸시 알림을 위한 새로운 컴포넌트를 작성하더라도, OCP를 만족합니다.  
스프링의 타입 기반 주입을 활용하면 SOLID에서 말하는 OCP를 프레임워크 수준에서도 적용할 수 있게 됩니다.

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final List<NotificationChannel> notificationChannels;

    public void notify(Account account, String message) {
        for (NotificationChannel notificationChannel : notificationChannels) {
            if (notificationChannel.supports(account)) {
                notificationChannel.notify(account, message);
            }
        }
    }
}
```


## 자가 호출

자가 호출(self invocation)은 어떤 객체가 메서드를 처리하는 와중에 자신이 갖고 있는 다른 메서드를 호출하는 상황을 의미합니다.  
스프링의 빈 메서드에서 발생하는 자가 호출은 개발자의 의도에서 벗어나는 결과를 만들 수 있습니다.  
특히 자가 호출되는 메서드에 AOP 애너테이션이 지정돼 있을 경우 문제가 됩니다.  
왜냐하면 자가 호출이 발생하면 호출되는 메서드에 적용된 AOP 애너테이션이 동작하지 않기 때문입니다.

```java
@Service
class Something {
    
    public void doSomething() {
        doSomethingElse();
    }
    
    @Transactional
    public void doSomethingElse() {
        
    }
}
```

이것은 스프링의 AOP가 프록시를 기반으로 동작하기 때문에 발생하는 현상입니다.   
스프링 AOP는 프록시 객체를 만들어 추가 동작을 삽입하는 방식으로 AOP의 부가 기능이 동작하게 합니다.  
그래서 메서드에 지정된 AOP 애너테이션이 수행되려면 반드시 이 프록시 객체를 통해 메서드가 실행돼야 합니다.


























