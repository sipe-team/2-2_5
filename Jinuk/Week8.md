# 9. 설계의 건전성을 해치는 여러 악마

1. 데드 코드
    - 절대로 실행되지 않는 조건 내부에 있는 코드
      - 데드 코드 또는 도달 불가능한 코드라고 함
    - 코드의 가독성을 떨어뜨림
    - 사양 변경에 의해 도달 가능한 코드로 변경될 수 있기 때문에, 버그가 될 가능성이 있음

2. YAGNI 원칙
    - You Aren't Gonna Need It
    - 지금 필요 없는 기능을 만들지 말라
    - 요구사항을 미리 예측하고 구현해도, 대부분 맞지 않게 됨
    - 데드 코드가 될 수 있음

3. 매직 넘버
    ``` java
    class ComicManager {
        boolean isOk(final int value) {
            return 60 <= value;
        }

        void tryConsume() {
            int tmp = value - 60;
            if (tmp < 0) {
                throw new RuntimeException();
            }
            value = tmp;
        }
    }
    ```
    - 60이라는 특정 숫자가 자꾸 등장함
    - 설명이 없다면 해당 숫자의 의도를 알기 힘듬
    - 사양이 변경된다면 해당 매직 넘버를 하나씩 찾으며 변경해 줘야 함
    - 상수로 정의 필요
    ``` java
    class ComicManager {
        private static final int MIN = 0;
        private static final int TRIAL_READING_POINT;

        boolean isOk(final int value) {
            return TRIAL_READING_POINT <= value;
        }

        void tryConsume() {
            int tmp = value - TRIAL_READING_POINT;
            if (tmp < MIN) {
                throw new RuntimeException();
            }
            value = tmp;
        }
    }
    ```


4. 문자열 자료형에 대한 집착
    - 의미가 다른 여러 개의 값을 String 변수에 무리하게 넣으면, 의미를 알기 어려움
    - 로직이 쓸데 없이 복잡해질 수 있고, 가독성이 저하됨
    - 의미가 다른 값은 다른 변수에 저장

5. 전역 변수
    - 자바 언어 사양에는 전역 변수가 없지만, `public static`으로 설정한다면 모든 곳에서 접근할 수 있음
    - 여러 로직에서 전역 변수를 참조하고 값을 변경하면, 어디에서 어떤 시점에 값을 변경했는지 파악하기 어려움
    - 동기화가 필요한 경우에도 문제 발생
    - 거대 데이터 클래스도 전역 변수와 같은 성질을 띰
    - 전역 변수를 직접적으로 사용하지 않더라도, 전역 변수와 같은 개념을 알게 모르게 사용하는 케이스가 많음
    - 영향 범위가 최소화되도록 설게하기
      - 호출할 수 있는 위치가 적고 국소적일수록 로직을 이해하고 구현하기 쉬움
      - 전역변수가 정말로 필요한지 판단하기

6. null 문제
    - null이 있다고 전제하고 로직을 짜면, 모든 곳에서 null check를 해야 함
    - 무언가를 갖고 있지 않은 상태 또는 무언가 설정되지 않은 상태를 null로 활용하면 문제가 발생할 가능성이 큼
    - null을 리턴하거나 전달하지 않는 설계 하기
    ``` java
    class Equipment {
        static final Equipment EMPTY = new Equipment("장비 없음", 0, 0, 0);

        final String name;
        final int price;
        final int defence;
        final int magicDefence;

        Equipment(final String name, final int price, final int defence, final int magicDefence) {
            if(name.isEmpty()) {
                throw new IllegalArgumentException();
            }

            this.name = name;
            this.price = price;
            this.defence = defence;
            this.magicDefence = magicDefence;
        }
    }

    void takeOffAllEquipment() {
        head = Equipment.EMPTY;
        body = Equipment.EMPTY;
        arm = Equipment.EMPTY;
    }
    ```
    - 이처럼 장비하지 않은 상태를 만들어 주면, null 체크가 필요하지 않음
    - null safe
      - null에 의한 오류가 아예 발생하지 않게 만드는 구조
      - 코틀린은 기본적으로 null 안전 자료형

7. 예외를 catch하고 무시하는 코드
    - try-catch로 예외를 catch하고 별다른 처리를 하지 않는 경우 원인 분석을 어렵게 만듬
    - 문제가 발생했다면 소리치기
      - 최소한 로그로 기록하고, 상위 레이어 클래스로 오류 통지 필요
      - 가드가 있는 생성자도 좋은 방법

8. 설계 질서를 파괴하는 메타 프로그래밍
    - 프로그램 구조 자체를 제어하는 것 - 메타 프로그래밍
       - 자바에서는 리플렉션 API가 대표적
    - 리플렉션으로 인한 클래스 구조와 값 변경 문제
      - 리플렉션을 사용함으로써 final 변수의 값도 변경할 수 있고, private 변수도 접근할 수 있으며, static value도 변경할 수 있다.
      - 일종의 뒷문이라고 볼 수 있음
    - 자료형의 장점을 살리지 못하는 하드 코딩
      - 클래스의 인스턴스를 만들때는 일반적으로 new를 사용하지만, 리플렉션을 이용하면 메타 정보 기반으로 인스턴스를 생성할 수 있음
      - ide 등을 통해 변경할 때, 문자열로 전달한 부분은 변경되지 않을 수 있음
      - IDE의 정적 분석 기능과 같은 개발 도구의 지원을 받기가 어려워짐
    - 단점을 이해하고 용도를 한정해서 사용하기
      - 시스템 분석 용도로 한정하거나, 작은 범위에서만 사용하는 등 리스크를 최소화해야 함

9. 기술 중심 패키징
    - 구조에 따라 폴더와 패키지를 나누는 방법
    - mvc로 나누는 폴더 구성이 대표적
    - 관련된 비즈니스 개념을 기준으로 폴더를 구분하는 것이 좋음

10. 샘플 코드 복사해서 붙여넣기
    - 샘플 코드를 그대로 복사 붙여넣기 하면, 설계 측면에서 좋지 않은 구조가 되기 쉬움
    - 샘플 코드는 참고만 하고, 클래스 구조를 잘 설계하자

11. 은 탄환
    - 현실에서 발생하는 문제는 특정 기술 하나로 해결할 수 없는 경우가 많음
    - 어던 문제를 해결하는 비장의 무기, 묘책을 silver bullet이라고 하는데, 개발에서는 silver bullet이 존재하지 않음
    - 어떤 문제가 있을 때, 어떤 방법이 효과적일지, 비용이 더 들지 않을지 판단하는 자세 중요, 적절한 기술을 선택할 수 있도록 노력할 것.
    - Best는 없다. Better가 목표일 뿐이다.