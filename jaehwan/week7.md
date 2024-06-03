# 8장 - 강한 결합: 복잡하게 얽혀서 풀 수 없는 구조

- 결합도: 모듈(클래스) 사이의 의존도를 나타내는 지표
- 강한 결합: 어떤 클래스가 다른 클래스에 많이 의존하고 있는 구조
- 느슨한 결합: 결합도가 낮은 구조, 이런 구조로 가야한다.

## 8.1. 결합도와 책무

- 단일 책임 원칙
    - 클래스가 담당하는 책임은 하나로 제한해야 한다. 안 지키면 하나 수정할 때 코드 다 바꿔야 한다.
    - 로직의 위치에 일관성있게 설계하자
    - 계산은 계산하는 곳에서, 가격과 관련된 책임은 가격만 책임지는 클래스에서 하도록
    - 근데 이러면 너무 중복이 많아질 수 있지 않나? DRY 원칙 안 지키냐! 비슷한 로직들도 있는데, 이유가 다 있다.
- DRY 원칙 ( Don’t Repeat Yourself)
    - 모든 지식은 시스템 내에서 단 한 번만, 애매하지 않고, 권위 있게 표현되어야 한다.
    - 지식 = 비즈니스 지식 (e.g., 게임에서 공격력, 매직 포인트, 매직 포인트 등)
    - 같은 로직, 비슷한 로직이라도 개념이 다르면 **중복을 허용** (일반 할인과 여름 할인은 개념이 다른 것이다.)
    - 너무 일반화하지 말아라, 비슷해보여도 개념이 다르면 분리하는 게 맞다.

```python
# 잘못된 코드의 예시, 너무 많은 책임을 가지고 있다.
class Product:
    def __init__(self, id, name, price, can_discount):
        self.id = id
        self.name = name
        self.price = price
        self.can_discount = can_discount

class ProductDiscount:
    def __init__(self, id, can_discount):
        self.id = id
        self.can_discount = can_discount

class DiscountManager:
    def __init__(self):
        self.discount_products = []
        self.total_price = 0

    def add(self, product, product_discount):
        if product.id < 0:
            raise ValueError("0보다 작을 수 없다")
        if not product.name:
            raise ValueError("비어있을 수 없다")
        if product.price < 0:
            raise ValueError("0보다 작을 수 없다")
        if product.id != product_discount.id:
            raise ValueError("두 개가 같아야 한다")
        
        discount_price = self.get_discount_price(product.price)
        
        if product_discount.can_discount:
            tmp = self.total_price + discount_price
        else:
            tmp = self.total_price + product.price
        
        if tmp <= 200000:
            self.total_price = tmp
            self.discount_products.append(product)
            return True
        else:
            return False

    @staticmethod
    def get_discount_price(price):
        discount_price = price - 3000
        if discount_price < 0:
            discount_price = 0
        return discount_price
        
        
class SummerDiscountManager:
    def __init__(self, discount_manager):
        self.discount_manager = discount_manager

    def add(self, product):
        if product.id < 0:
            raise ValueError("0 ㄴㄴ")
        if not product.name:
            raise ValueError("엠티 ㄴㄴ")
        
        discount_price = DiscountManager.get_discount_price(product.price)
        
        if product.can_discount:
            tmp = self.discount_manager.total_price + DiscountManager.get_discount_price(product.price)
        else:
            tmp = self.discount_manager.total_price + product.price
        
        if tmp <= 300000:
            self.discount_manager.total_price = tmp
            self.discount_manager.discount_products.append(product)
            return True
        else:
            return False
```

현재 책무를 고려하지 않은 설계가 되어있다.

1. 가드가 이상한 곳에 있다.
2. 하나 수정하면 여러 곳이 수정되어야 한다. (e.g., get_discount_price가 4000 빼게 수정하면? 로직이 엮여있어서 다 수정해야 함)
3. Manager가 너무 많은 일을 담당한다.

→ 책임이 하나가 되게 클래스를 설계하자.

상품의 가격을 나타내는 RegularPrice. 유효성 검사도 한다.

일반 할인 가격, 여름 할인 가격과 관련된 내용도 개별적으로 책임지는 별도의 클래스로 만들자.

```python
# 개선한 예시
class RegularPrice:
    MIN_AMOUNT = 0

    def __init__(self, amount):
        if amount < self.MIN_AMOUNT:
            raise ValueError("Amount must be 0 or higher")
        self.amount = amount

class RegularDiscountedPrice:
    MIN_AMOUNT = 0
    DISCOUNT_AMOUNT = 4000

    def __init__(self, price: RegularPrice):
        discounted_amount = price.amount - self.DISCOUNT_AMOUNT
        if discounted_amount < self.MIN_AMOUNT:
            discounted_amount = self.MIN_AMOUNT
        self.amount = discounted_amount

class SummerDiscountedPrice:
    MIN_AMOUNT = 0
    DISCOUNT_AMOUNT = 3000

    def __init__(self, price: RegularPrice):
        discounted_amount = price.amount - self.DISCOUNT_AMOUNT
        if discounted_amount < self.MIN_AMOUNT:
            discounted_amount = self.MIN_AMOUNT
        self.amount = discounted_amount
```

## 8.2. 다양한 강한 결합 사례와 대처 방법

### 강한 결합을 유발하는 것들

- 상속
    - 서브 클래스가 슈퍼 클래스에 굉장히 크게 의존한다, 슈퍼 클래스가 공통 로직을 두는 장소로 사용됨
    - 물론 상속도 잘 쓰면 좋다. 템플릿 메서드 패턴 같은 거 잘 쓰면 ㄱㅊㄱㅊ
    - 그래도 컴포지션을 사용하는 것이 좋다.
    - 컴포지션: 사용하고 싶은 클래스를 private 인스턴스 변수로 갖고 사용하는 것
        - 변경으로 인한 영향이 적다.
        - 컴포지션 관계에 있어도, 각 메서드가 어떤 인스턴스 변수를 활용하고 있는지 잘 파악해서 메서드끼리 관계없으면 클래스를 또 나누자.
- public 접근 지정자
    - 특별한 이유없이 남발하지 말아라. private 써라.
- 클래스 하나에 너무 많은 private 메서드
    - 클래스 하나에 책임이 많다는 거다. 단일 책임인지 의심해봐라.
    - 주문을 담당하는 클래스에서 할인 적용, 주문 내역 조회에 대한 메서드가 있을 수도 있지만, 주문 담당 클래스의 책임은 아니지.
    - 높은 응집도를 생각하다가 결합도가 높아졌는지 확인해라. 클래스를 분할해서 값 객체로 설계해라
- 거대 데이터 클래스
    - 전역변수처럼 될 가능성이 높다. 그냥 데이터 클래스보다 더 악마를 불러올 수 있다.
- 트랜잭션 스크립트 패턴
    - 트랙잭션 스크립트 패턴: 메서드 내부에 일련의 처리가 하나하나 길게 작성되어 있는 구조, 심해지면 갓 클래스가 됨
    - 갓 클래스: 하나의 클래스 내부에 수천에서 수만 줄의 로직을 담고 있어서 수많은 책임을 지는 클래스
    

### 해결 방법

- 지금까지의 객체 지향 설계와 단일 책임 원칙에 따라 제대로 설계하자
    - 책임별로 클래스를 분할하면 보통 많아도 200줄, 일반적으로 100줄임
    - 조기 리턴, 전략 패턴, 일급 컬렉션, 목적 중심 이름 설계 같은 걸 활용해보도록 하자.
