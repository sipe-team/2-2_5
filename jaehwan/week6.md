# 7장: 컬렉션: 중첩을 제거하는 구조화 테크닉

## 7.1. 이미 존재하는 기능을 다시 구현하지 말기

- 내가 구현할 기능이 표준 라이브러리에 있는 메서드인지 확인하기
- 바퀴의 재발명 = 이미 있는 거 또 구현한 것
- 네모난 바퀴의 재발명 = 이미 있는 거를 원본보다 못하게 구현한 것

## 7.2. 반복 처리 내부의 조건 분기 중첩

- 6장에서 설명했던 조기 리턴이랑 비슷하게 조기 컨틴뉴를 활용해라
- 조기 브레이크를 활용해라

## 7.3. 응집도가 낮은 컬렉션 처리

- 컬렉션 처리를 캡슐화하기
    - 원래 클래스의 구성요소는?
        - 인스턴스 변수
        - 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드
    - 일급 컬렉션 패턴(퍼스트 클래스 컬렉션)의 구성요소는?
        - 컬렉션 자료형의 인스턴스 변수, 예제에선 Member
        - 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드, 예제에선 Party
- 외부로 전달할 때 컬렉션의 변경 막기
    - 인스턴스 변수를 그대로 외부에 전달하면 클래스 외부에서 마음대로 멤버를 추가하고 제거할 수 있음
    - 컬렉션의 변경을 막자.

```python
# Party가 멤버를 관리하고 다 사용하는 형태
class Party:
    MAX_MEMBER_COUNT = 4

    def __init__(self, members=None):
        if members is None:
            self._members = []
        else:
            self._members = members

    def add(self, new_member):
        if self.exists(new_member):
            raise RuntimeError("이미 파티 참가")
        if self.is_full():
            raise RuntimeError("더 이상 추가 불가")
        adding = self._members.copy()
        adding.append(new_member)
        return Party(adding)

    def is_alive(self):
        return any(member.is_alive() for member in self._members)

    def exists(self, member):
        return any(each.id == member.id for each in self._members)

    def is_full(self):
        return len(self._members) == self.MAX_MEMBER_COUNT

    def members(self):
        return tuple(self._members)
	
```