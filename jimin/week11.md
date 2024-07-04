# 12 메서드: 좋은 클래스에는 좋은 메서드가 있다.
## 12.1 반드시 현재 클래스의 인스턴스 변수 사용하기

메서드는 반드시 현재 클래스의 인스턴스 변수를 사용하도록 설계하자.

아래와 같이 다른 클래스의 인스턴스 변수를 변경하도록 한다면 응집도가 낮은 구조가 될 수 있다. 

```java
//code 5.14
class ActorManager {
    void shift(Location location, int shiftX, int shitfY) {
        location.x = shiftX;
        location.y = shitfY;
    }
}
```

## 12.2 불변을 활용해서 예상할 수 있는 메서드 만들기
가변 인스턴스 변수 등을 변경하는 메서드는 의도치 않게 다른 부분에 영향이 갈 수 있다. 따라서 **불변**을 활용해 예상치 못한 동작 자체를 막을 수 있게 설계해야 한다. 

## 12.3 묻지 말고 명령하라
‘다른 클래스를 확인하고 조작하는 메서드 구조’는 응집도가 낮은 구조이다.

```java
//code 5.30, 응집도 낮은 코드 
void equipArmor(int memberId, Armor newArmor) {
    if(party.members[memberId].equipments.canCahnge) {
        party.members[memberId].equipments.armor = newArmor;
    }
}
```

메서드를 호출하는 쪽에서 복잡한 처리를 하지 않고 아래와 같이 호출되는 메서드 쪽에서 복잡한 처리를 하는 형태로 만들어야 한다.

```java
//5.6.1, 묻지 말고 명령하도록 하는 코드 
public class Equipments {
    private boolean canChange;
    private Equipment head;
    private Equipment armor;
    private Equipment arm;

    public void equipArmor(Equipment newArmor) {
        if (canChange) { // 내부로 이동 
            this.armor = newArmor;
        }
    }

    public void deactivateAll() { 
        this.head = Equipment.EMPTY;
        this.armor = Equipment.EMPTY;
        this.arm = Equipment.EMPTY;
    }
```

* 저자의 말씀.. 
  * getter, setter 를 구현하기만 한다고 해서 캡슐화한 것이 아니다. 캡슐화는 **데이터와 그 데이터를 조작하는 로직을 하나의 클래스로 만들고, 필요한 로직만을 외부로 공개** 하는 것이다.

## 12.4 커맨드/쿼리 분리

* 커맨드: 상태 변경
* 쿼리: 상태 리턴
* 모디파이어: 커맨드, 쿼리를 동시에 하는 것 

```java
//모디파이어 예시
int gainAndGetPoint() {
	point += 10;
	return point;
}
```
상태 변경과 추출을 동시에 하는 모디파이어 메서드는 여러 문제의 원인이 된다. 추출만 하고 싶거나 변경만 하고 싶을 때 지원을 못한다. 따라서 분리하도록 하는 것이 좋다.


## 12.5 매개변수
* 불변 매개변수로 만들기
  * 매개변수를 변경하면 값의 의미가 바뀌어서 어떤 의미를 나타내는지 유추하기 어렵고 , 어디서 변경되었는지 찾기 어렵다.
  * final 을 붙여 불변으로 만들고, 변경하고 싶다면 지역 변수에 할당해 변경하도록 하자.
* 플래그 매개변수 사용하지 않기
  * 플래그 매개변수를 받는 메서드는 코드를 읽는 사람이 메서드가 무슨 일을 하는지 알기 어렵게 만든다. 
  * 전략 패턴 등 다른 구조로 설계를 개선하자.
* null 전달하지 않기
  * null을 활용하면 NPE 발생할 수 있으므로 매개변수로 null을 전달하지 말자.
  * null 에 의미부여하지 말고 `Equipment.EMPTY`와 같이 값을 생성하자.
* 출력 매개변수 사용하지 않기
  * 가독성 저하의 원인이 된다.  
  * ex) 5.14의 location
* 매개변수는 최대한 적게 사용하기 
  * 매개변수가 많다는 것은 메서드가 여러 기능을 한다는 것이다.
  * 매개변수가 많아질 수 밖에 없다면 별도의 클래스를 만들어보자.


## 12.6 리턴 값
* 자료형을 사용해서 리턴 값의 의도 나타내기 
  * 모두 같은 자료형을 사용하면, 매개변수 잘못 전달 등 실수가 발생할 수 있다.
    ```java
    //AS-IS 같은 자료형이지만 다른 값을 의미
    int price = productPrice.add(otherPrice); //상품 가격 합계
    int discountedPrice = calcDiscountedPrice(price); // 할인 금액
    int deliveryPrice = calcDeliveryPrice(discountedPrice); // 배송비
    ```
    ```java 
    //TO-BE 독자적인 자료형 사용
    Price price = productPrice.add(otherPrice); //상품 가격 합계
    DiscountedPrice discountedPrice = new DiscountedPrice(price); // 할인 금액
    ```
    * 기본 자료형을 사용하지 말고, 독자적인 자료형을 사용해보자

* null 리턴하지 않기
* 오류는 리턴 값으로 리턴하지 말고 예외 발생시키기 
  * 리턴한 값에 대해 오류 처리 분기를 여러군데 구현될 수 있다.
    ```java
    // AS-IS 특정 값 오류로 나타냄 
    class Location {
        Location shift(int shiftX, int shiftY) {
            int nextX = this.x + shiftX;
            int nextY = this.y + shiftY;

            if (valid(nextX, nextY)) {
                return new Location(nextX, nextY);
            }

            return new Location(-1, -1); 
        }
    }
    ```

    ```java
    // TO-BE 곧바로 예외 발생

    class Location {
        Location(final int x, final int y) {
            if (!valid(x, y)) {
                throw new IllegalArgumentException("잘못된 위치입니다.");
            }

            this.x = x;
            this.y = y;
        }
    }
    ```
    * 어떤 값으로 여러 의미를 나타내는 중의적 코드를 작성하지말고 예외를 발생해버리자. 




