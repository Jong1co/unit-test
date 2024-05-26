# Chapter03 단위 테스트 구조

## 3.1 단위 테스트를 구성하는 방법

### 3.1.1 AAA패턴

준비, 실행, 검증이라는 세 부분으로 나눌 수 있음

```c#

public class Calculator{
  public double Sum(double first, double second){
    return first + second;
  }
}

public class CalculatorTests{
  public void Sum_of_two_numbers(){
    //준비
    double first = 10;
    double second = 20;
    var calculator = new Calculator();

    //실행
    double result = calculator.Sum(first, second);

    //검증
    Assert.Equal(30, result)
  }
}
```

- 스위트 내 모든 테스트가 단순하고 균일한 구조를 갖는 데 도움이 됨
  - 준비: 테스트 대상 시스템(SUT) 과 해당 의존성을 원하는 상태로 만듬
  - 실행: SUT에서 메서드를 호출하고 준비된 의존성을 전달하며, 출력 값을 캡쳐(변수에 저장을 말하는듯)
  - 검증: SUT와 협력자의 최종 상태, 반환값, SUT가 협력자에 호출한 메서드 등을 검증
- 비슷한 패턴으로 Given-When-Then 패턴이 있음
  - 두 패턴 사이에 차이는 없으나 유일한 차이점은 프로그래머가 아닌 사람에게 더 읽기 쉽다는 것(왜?)
  - 비기술자들과 공유하는 테스트에 더 적합함

**TDD**

- 준비 구절부터 실행하는 것이 자연스럽지만, 검증 구절부터 시작할 수 있음
  - TDD를 실천할 때, 기능이 어떻게 동작할지 충분히 알지 못함
  - 그래서 먼저 기대하는 동작으로 윤곽을 잡은 후(검증 구절부터 작성한 후) 시스템을 어떻게 개발할지 아는 것이 중요함
  - 직관적이지는 않지만, 이 것은 문제를 해결하는 방식인데, 특정 동작이 무엇을 해야하는지에 대한 목표를 생각하면서 시작함
  - 다른 것을 하기 전에 검증문을 작성하는 것은 단지 사고 과정의 형식이다.
  - TDD를 실천할 때에만 적용될 수 있음.

### 3.1.2 여러 개의 준비, 실행, 검증 구절 피하기

때때로 준비 -> 실행 -> 검증 -> 실행 -> 다시 검증 과 같이<br/>
여러 개의 실행 구절을 볼 수 있음

- 이는 단위 테스트가 아닌 통합 테스트임(단위 테스트에서 이러한 구조는 피하는 것이 좋음)
- 통합 테스트에서는 실행 구절을 여러 개 두는 것이 괜찮을 때도 있음
  - 통합 테스트는 느릴 수 있는데, 실행을 여러 개 두면 테스트의 속도를 높일 수 있기 때문
  - 실행이 동시에 후속 실행을 위한 준비로 제공될 때 유용함

### 3.1.3 테스트 내 if문 피하기

- 단위 테스트든 통합 테스트든 테스트는 분기가 없는 간단한 일련의 단계여야 한다.
- if문의 존재는 테스트가 한 번에 너무 많은 것을 검증한다는 표시.
  - 테스트에 분기가 있어서 얻는 이점은 없음 (추가 유지비만 불어날 뿐임)
  - if문은 테스트를 읽고 이해하는 것을 어렵게 만듬.

### 3.1.4 각 구절은 얼마나 커야 하는가?

- 일반적으로 준비 구절이 가장 크며, 실행과 검증을 합친 만큼 클 수도 있음
- 그러나 훨씬 크다면 비공개 메서드 또는 별도의 팩토리 클래스로 도출하는 것이 좋음(테스트만을 위한 객체를 생성하라는 뜻)
- 오브젝트 마더와 테스트 데이터 빌더가 있음
  - 오브젝트 마더:
    - 테스트에 사용되는 여러 example objects를 생성하는데 도움을 주는 클래스
    - 특정 상태를 가진 객체가 필요한 여러 테스트 코드가 있다면 아래와 같이 오브젝트 마더 클래스에 정의하고 얻는 것이 좋음

```javascript
class User {
  constructor(name, age, email) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
}

class ObjectMother {
  static createDefaultUser() {
    return new User("John Doe", 30, "john.doe@example.com");
  }

  static createTeenagerUser() {
    return new User("Teen Doe", 15, "teen.doe@example.com");
  }
}
```

- 테스트 데이터 빌더:
  - 테스트 데이터 빌더 패턴은 테스트 데이터를 생성하는 코드를 캡슐화하는 방법

```javascript
class UserBuilder {
  constructor() {
    this.user = new User();
  }

  withName(name) {
    this.user.name = name;
    return this;
  }

  withAge(age) {
    this.user.age = age;
    return this;
  }

  withEmail(email) {
    this.user.email = email;
    return this;
  }

  build() {
    return this.user;
  }
}

const user = new UserBuilder()
  .withName("John Doe")
  .withAge(30)
  .withEmail("john.doe@example.com")
  .build();
```

실행 구절이 한 줄 이상인 경우를 경계하라

- 실행 구절은 보통 한줄임
- 두 줄 이상인 경우 SUT의 공개 API에 문제가 있을 수 있음

```c#
public void Purchase_succeeds_when_enough_inventory()
{
	var store = new Store();
  store.AddInventory(Product.Shampoo, 10);
  var customer = new Customer();

  bool success = customer.Purchase(store, Product.Shampoo, 5);
  store.RemoveInventory(success, Product.Shampoo, 5);

  Assert.True(success);
  Assert.Equal(5, store.GetInventory(Product.Shampoo));
}
```

1. 첫 번째 줄에서 고객이 상점에서 샴푸 다섯 개를 얻으려고 한다.
2. 두 번째 줄에서는 재고가 감소되는데, Purchase 호출이 성공을 반환하는 경우에만 수행한다.

**이 코드의 문제점은 단일 작업을 수행하는 데 두 개의 메서드 호출이 필요하다는 것**

- 비즈니스 관점에서 구매가 정상적으로 이뤄지면 고객의 제품 획득과 매장 재고 감소라는 두 가지 결과가 만들어진다.
- 이 결과는 함께 만들어져야 하고, 이는 다시 단일한 공개 메서드가 있어야 한다는 뜻
- 첫번째만 호출하고 두번째를 호출하지 않을 때 모순이 발생함 (불변 위반이라고 한다.)
- **이러한 잠재적 모순으로부터 코드를 보호하는 행위를 캡슐화 라고 한다.** (멋진 말..)

### 3.1.5 검증 구절에는 검증문이 얼마나 있어야 하는가

- 단위 테스트의 단위는 동작의 단위이지, 코드의 단위가 아님
- 검증 구절이 너무 커지는 것 또한 추상화가 누락됐을 수 있기에 경계해야 함
- 예를 들어 SUT에 반환된 모든 속성을 검증하는 대신 객체 클래스 내 적절한 동등 멤버를 정의하는 것이 좋음
  - 동등 멤버란: "=="와 같이 동등 비교가 가능한 멤버 메서드 또는 연산자를 통칭함
  - 자바스크립트에서는 toEqual메서드로 비교하는 방법이 있을듯(메모리 주소로 비교하기에 실제 속성 값이 같더라도 서로 다른 객체로 인식하기 때문)

### 3.1.7 테스트 대상 시스템 구별하기

- SUT와 의존성을 구분하기 위해 이름을 sut로 하면 구분하기 쉬움
- ex) sut.Sum(first, second)로 실행하면 구분하기 쉬움!

### 3.1.8 준비, 실행, 검증 주석 제거하기

- 단위테스트에서는 빈 줄로 구절을 구분하고, 그 것이 아닌 복잡한 설정을 포함하는 경우에는 구절 주석을 유지하는 것이 좋음

**단위 테스트를 작성할 때 사고방식**

- 각 테스트는 이야기가 있어야 한다.
- 이 이야기는 문제 영역에 대한 개별적이고 원자적인 사실이나 시나리오이며, 테스트가 통과하는 것은 이 사실 또는 시나리오가 실제 사실이라는 증거임
- 테스트가 제품 코드의 기능을 무조건 나열하면 안 됨
- 오히려 애플리케이션 동작에 대한 고수준의 명세가 있어야 함
- 비즈니스 담당자에게도 의미가 있어야 함
