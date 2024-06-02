# 8장 결합도와 책무
- 결합도란 모듈사이의 의존도를 나타내는 지표
- 강한결합: 어떤 객체가 다른 객체에 많이 의존하고 있는 구조
- 약한결합: 낮은결합으로 만드는게 코드변경이 쉬움
  
## 8.1 결합도와 책무
- 로직의 위치에 일관성이 있어야됨
- 편의를 위해 다른 클라스의 메소드를 무리하게 횔용하면 안됨
- 책무를 고려해서 설계해야됨

### 단일 책임 원칙
- 클라스는 하나의 책무만 가져야됨

### DRY 원칙의 잘못된 사용
- 책무를 생각하면서 로직의 중복을 제거해야됨
- 같은 로직, 비슷한 로직이라도 개념이 다르면 중복을 허용해야됨

## 문제가 있는 코드
```swift
class UserManager {
    var username: String = "게스트"
    
    func login(name: String) {
        self.username = name
    }
}

// View
struct ContentView: View {
    var userManager = UserManager()  // 강한 결합, 직접 의존
    
    var body: some View {
        VStack {
            Text("어서와, \(userManager.username)")  // UserManager의 내부 상태에 직접 접근
            Button("로그인") {
                userManager.login(name: "성이름")  // UserManager의 메소드에 직접 접근
            }
        }
    }
}
```
### 문제점
- 결합도가 높음: ContentView가 UserManager에 직접 의존하여 변경에 취약.
- 로직의 일관성 부족: View에서 Model의 내부 상태와 메소드에 직접 접근.
- 단일 책임 원칙 위반: View가 UI와 비즈니스 로직 모두를 처리.

## 개선된 코드
```swift
// Model
class UserManager: ObservableObject {
    @Published var username: String = "게스트"
    
    func login(name: String) {
        self.username = name
    }
}

// ViewModel
class UserViewModel: ObservableObject {
    @Published var username: String = "게스트"
    private var userManager: UserManager
    
    private var cancellables = Set<AnyCancellable>()
    
    init(userManager: UserManager) {
        self.userManager = userManager
        self.userManager.$username
            .assign(to: \.username, on: self)
            .store(in: &cancellables)
    }
    
    func login(name: String) {
        userManager.login(name: name)
    }
}

// View
struct ContentView: View {
    @StateObject private var viewModel = UserViewModel(userManager: UserManager())  // 약한 결합, ViewModel을 통해 간접 의존
    
    var body: some View {
        VStack {
            Text("어서와, \(viewModel.username)")  // ViewModel을 통해 상태 접근
            Button("로그인") {
                viewModel.login(name: "성이름")  // ViewModel을 통해 메소드 호출
            }
        }
    }
}
```
### 개선점
- ContentView가 UserManager에 직접 의존하지 않고 UserViewModel에 의존 -> 약한 결합 (결합도가 낮음)
- 로직의 위치에 일관성이 있음 -> ViewModel을 통해 Model의 세부사항을 캡슐화
- 단일 책임 원칙 준수 -> View는 사용자 인터페이스만 관리하고, 비즈니스 로직은 ViewModel이 처리
- DRY 원칙의 잘못된 사용을 피함 -> UserManager와 ViewModel의 역할이 다르므로, 각자의 책임을 분리하고 중복을 허용
--- 

## 8.2 다양한 결합 사례와 대처방법

## 상속과 관련된 강한 결합
- 상속을 주의해서 다루지 않으면, 곧바로 강한 결합 구조를 유발할수 있음 

## 상속보다 컴포지션을 사용하자
- 슈퍼 클라스 의존으로 강한 결합을 피하려면, 상속보다 컴포지션을 이용하는거 추천
- 상속을 사용하면 서브클래스가 슈퍼클라스의 로직을 그대로 사용하게 되고, 슈퍼 클라스가 공통로직을 두는 장소로 사용됨 -> 강한 결합이 발생하기 쉬움

## 인스턴스 변수별로 분할이 가능하면 로직을 나누자
- 책임이 다른 메소드가 한 클라스안에 정의되어 있으면 문제가 발생할 수 있음
- 영향 스케치를 그리거나 해서 의존관계를 잘 확인하면서 설계 ㄱㄱ

## 특별한 이유 없이 public 사용하지 않기
- public을 사용하면 다른 클라스에서 자유롭게 접근할 수 있으니깐 나도 모르게 결합이 될수 있음
- 관계를 맺지 않았으면 하는 클라스가 결합되어 영향범위가 확대됨

## 근데 반대로 private메소드가 많다는건 책임이 많을수도 있음
- 기능이 많아질수록 클라스는 비대해지고, 여러 메서드가 정의되고, 그에따른 private 메소드도 많아질수 있음
- private메소드가 많으면 책임이 많은지 확인해보고 다른 클라스로 분리하는것이 좋음

## 높은 응집도를 오해해서 생기는 강한 결합
- 응집도가 높다는 개념을 염두에 두고, 관련이 깊다고 생각되는 로직을 한곳에 모으려하다가, 결과적으로 강한 결합 구조를 만드는 상황은 매우 자주일어남
- 결합이 느슨하고 응집도가 높은 설계라고 한덩이에 모아두는것이 아니라, 책무에 따라 적절하게 분리하는것이 중요함
  
## 거대 데이터 클라스
- 데이터를 다루는 클라스가 너무 크면, 데이터를 다루는 로직이 많아지고, 결합도가 높아질수 있음
- 데이터가 많아지다 보면 클라스를 데이터를 편리하게 운반하는 역할로 인식하고 추가하기 쉬움
- 거대 데이터 클라스는 다양한 데이터를 가지기에 수많은 유스케이스에서 사용되고 -> 결국 전역 변수와 같은 성지를 띄게 됨

## 트랜잭션 스크립트 패턴
- 데이터를 보유하고 있는 클라스와 데이트를 처리하는 클라스를 나누어 분리해야됨\

## 갓 클라스
- 클라스 내부에서 수천에서 수만줄으리 로직을 담고 있으면 수많은 책임이 담당하는 로직이 얽혀있는 클라스 (어.. 난가?)
- 갓클라스는 책무가 많아서 결합도가 높아지고, 응집도가 낮아짐

## 문제가 있는 코드
```swift
import SwiftUI

// 상속으로 만들어본 강한 결합
class Item {
    var name: String
    var quantity: Int
    var price: Double
    
    init(name: String, quantity: Int, price: Double) {
        self.name = name
        self.quantity = quantity
        self.price = price
    }
}

class ShoppingCart: Item {
    var items: [Item] = []
    
    func addItem(_ item: Item) {
        items.append(item)
    }
    
    func totalPrice() -> Double {
        return items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }

    public func removeItem(at index: Int) {
        items.remove(at: index)
    }
    
    private func validateItem(_ item: Item) -> Bool {
        return item.price > 0 && item.quantity > 0
    }
}


struct ContentView: View {
    var cart = ShoppingCart(name: "", quantity: 0, price: 0.0) 
    
    var body: some View {
        VStack {
            List(cart.items, id: \.name) { item in
                Text("\(item.name): \(item.quantity) x $\(item.price)")
            }
            Button("추가하기") {
                let newItem = Item(name: "Apple", quantity: 3, price: 1.0)
                cart.addItem(newItem)  // 강한 결합
            }
            Button("제거하기") {
                cart.removeItem(at: 0)  // 강한 결합
            }
            Text("총합: \(cart.totalPrice())")
        }
    }
}

```

### 문제점
- 상속과 관련된 강한 결합: ShoppingCart가 Item을 상속
- 인스턴스 변수별로 분할하지 않음: ShoppingCart가 여러 책임을 가짐 -> 거대 데이터 클래스 위험 증가
- 특별한 이유 없이 public 사용: removeItem 메서드가 public으로 외부 접근 허용 -> 결합도 증가
- private 메서드가 많음: validateItem 메서드가 클래스 내부에 많아짐 -> 책임이 많음
- 거대 데이터 클래스: ShoppingCart 클래스가 너무 많은 데이터를 다룸
- 데이터 보유와 처리 로직이 분리되지 않음
- 갓 클래스: ShoppingCart 클래스가 너무 많은 책임을 가짐
  
## 개선된 코드

```swift
// 데이터 클라스
class Item: Identifiable, ObservableObject {
    var id = UUID()
    @Published var name: String
    @Published var quantity: Int
    @Published var price: Double
    
    init(name: String, quantity: Int, price: Double) {
        self.name = name
        self.quantity = quantity
        self.price = price
    }
}

// 데이터 처리 클래스 분리, 상속 대신 컴포지션 사용
class ShoppingCart {
    @Published var items: [Item] = []
    
    func addItem(_ item: Item) {
        if validateItem(item) {
            items.append(item)
        }
    }
    
    func removeItem(at index: Int) {
        items.remove(at: index)
    }
    
    func totalPrice() -> Double {
        return items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }
    
    private func validateItem(_ item: Item) -> Bool {
        return item.price > 0 && item.quantity > 0
    }
}

class ShoppingCartViewModel: ObservableObject {
    @Published var cart = ShoppingCart()
    
    func addItem(name: String, quantity: Int, price: Double) {
        let newItem = Item(name: name, quantity: quantity, price: price)
        cart.addItem(newItem)
    }
    
    func removeItem(at index: Int) {
        cart.removeItem(at: index)
    }
    
    func totalPrice() -> Double {
        return cart.totalPrice()
    }
}

struct ContentView: View {
    @StateObject private var viewModel = ShoppingCartViewModel()
    
    var body: some View {
        VStack {
            List(viewModel.cart.items) { item in
                Text("\(item.name): \(item.quantity) x $\(item.price)")
            }
            Button("추가하기") {
                viewModel.addItem(name: "Apple", quantity: 3, price: 1.0)  // 약한 결합, ViewModel을 통해 간접 의존
            }
            Button("제거하기") {
                viewModel.removeItem(at: 0)  // 약한 결합, ViewModel을 통해 간접 의존
            }
            Text("총합: \(viewModel.totalPrice())")
        }
    }
}
```

## 개선점
- 상속보다 컴포지션 사용: ShoppingCart가 Item을 상속하지 않고, Item을 포함하여 사용 -> 약한 결합
    - ShoppingCart 클래스가 Item을 상속하지 않고, items 배열을 통해 Item 객체를 포함함으로써 상속에 따른 강한 결합을 피함.
- 인스턴스 변수별로 분할: Item과 ShoppingCart로 책임을 분리 -> 단일 책임 원칙 준수
    - Item 클래스는 데이터 보유 역할, ShoppingCart 클래스는 데이터 처리 역할을 분리하여 단일 책임 원칙을 준수.
- 특별한 이유 없이 public 사용하지 않음: 필요한 메서드만 외부에 공개하여 결합도 감소
- private 메서드가 줄임: 내부적으로만 필요한 메서드만 private으로 유지
- 높은 응집도의 올바른 이해: 관련 로직을 한 곳에 모으지 않고, 책무에 따라 분리
    - 관련 로직을 ShoppingCart와 Item으로 분리하여 응집도를 높이고 결합도를 낮춤.
- 거대 데이터 클래스 방지: Item과 ShoppingCart로 역할을 나눠서 데이터 처리
- 갓 클래스 방지: ShoppingCart와 ShoppingCartViewModel로 책임을 분산하여 결합도를 낮추고 응집도를 높임