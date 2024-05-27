# 7장 컬렉션 : 중첩을 제거하는 구조화 테크닉

## 이미 존재하는 기능을 다시 구현하지 말기

다음과 같이 for 반복문 내부에 if 조건문이 중첩되어 있으면 가독성이 좋지 않습니다.

```java
boolean hasPrisonKey = false;
for (Item item : items) {
    if (item.name.equals("감옥 열쇠")) {
        hasPrisonKey = true;
        break;
    }
}
```

같은 기능을 자바 표준 라이브러리에 있는 컬렉션 전용 메서드인 `anyMatch` 메서드로 구현할 수 있습니다.

```java
boolean hasPrisonKey = items.stream().anyMatch(
        item -> item.name.equals("감옥 열쇠")
);
```

이처럼 이미 존재하는 기능을 다시 구현하지 않고 사용하는 것이 구현 실수에 의한 버그 가능성이나 코드의 복잡도를 낮출 수 있어 좋습니다.

## 반복 처리 내부의 조건 분기 중첩

다음과 같이 for 문 내에 if 문이 여러 개 중첩된 코드는 가독성이 좋지 않습니다.

```java
for 조건문 {
    if 조건문 {
        if 조건문 {
            if 조건문 {
                // ...
            }
        }
    }
}
```

이러한 조기 분기 중첩을 제거하기 위해서는 다음과 같은 방법을 사용할 수 있습니다.

1. 조기 continue 사용 : 조건을 만족하지 않는 경우, continue 로 다음 반복으로 넘어간다.
2. 조기 break 사용 : break 를 통해 처리를 중단하고 반복문 전체를 벗어난다.

## 응집도가 낮은 컬렉션 처리

컬렉션과 관련된 응집도가 낮아지는 문제는 **일급 컬렉션 패턴**을 사용해 해결할 수 있습니다.

**일급 컬렉션(First Class Collection)** 이란 컬렉션과 관련된 로직을 캡슐화하는 디자인 패턴으로, 다음과 같은 요소로 구성됩니다.

- 컬렉션 자료형의 인스턴스 변수
- 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드

그러나 다음과 같이 사용하면, members의 요소가 변화하는 부수 효과가 발생할 수 있습니다.

```java
class Party {
    private final List<Member> members;
    
    // ...
    
    void add(final Member member) {
        members.add(member);
    }
}
```

이를 막기 위해 새로운 리스트를 생성하고 해당 리스트에 요소를 추가하는 형태로 구현합니다.
이러면 원래 members를 변화시키지 않아 부수 효과를 막을 수 있습니다.

```java
import java.util.ArrayList;

class Party {
    // ...

    Party add(final Member member) {
        List<Member> adding = new ArrayList<>(members);
        adding.add(member);
        return new Party(adding);
    }
}
```

~~실제로 게임 업계에서 파티에 멤버를 추가할 때 어떻게 처리할까?~~

컬렉션 요소를 **외부에 전달할 때는 변경하지 못하게 막아 두는 것**이 좋습니다.
이 때는 `unmodifiableList` 메서드를 사용합니다.

```java
class Party {
    // ...
    List<Member> members() {
        reutrn members.unmodifiableList();
    }
}
```

`unmodifiableList`로 리턴되는 컬렉션은 요소를 추가하거나 제거할 수 없습니다.
