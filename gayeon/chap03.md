단위 테스트 구조
=============

# 1. 단위 테스트 구조

### 준비 - 실행 - 검증 패턴을 사용해 단위 테스트를 구성해야하는 방법

AAA 패턴이라고도 한다.(3A) 예를 들어 두 숫자의 합을 계산하는 클래스가 있다.

```
public class Caculator {
public int sum(int num1, int num2) {
        return num1 + num2;
    }
}
```

그리고 이를 3A 패턴을 기반으로 테스트를 작성해보자

```
public class CalculatorTest {
    
    @Test
    public void testSum() {
        // Arrange
        Calculator calculator = new Calculator();
        int num1 = 5;
        int num2 = 3;
        int expectedSum = 8;
        
        // Act
        int result = calculator.sum(num1, num2);
        
        // Assert
        assertEquals(expectedSum, result, "The sum of 5 and 3 should be 8");
    }
}

```

AAA 패턴을 도입할 시 모든 테스트가 단순하고 균일한 구조를 갖는 데 도움이 된다. 이러한 구조에 익숙해지면 모든 테스트를 쉽게 읽을 수 있고 이해할 수 있다. 결국 전체 테스트 스위트의 유지보수 비용이 줄어드는 셈이다.

구조를 상세히 설명하자면 이렇다.

- 준비 구절에서는 테스트 대상 시스템(SUT)과 해당 의존성을 원하는 상태로 만든다.
- 실행구절에서는 SUT에서 메서드를 호출, 준비된 의존성을 전달해 출력값을 캡처
- 검증 구절에서는 결과를 검증. 결과는 반환 값이나 SUT와 협력자의 최종 상태, SUT가 협력자에 호출한 메서드 등으로 표시될 수 있다.

(*이때 "캡처"라는 말은 이 메서드가 반환하는 값이나, 메서드 실행 후 테스트 대상 객체의 상태를 기록하고 이를 검증하는 과정을 말함)

PS) AAA와 유사한 Given-When-Then패턴이라는 것이 있다.
- Given : 준비 구절
- When : 실행 구절
- Then : 검증 구절

테스트 구성 측면에서 두 패턴의 차이는 없고 다만 Given-When-Then는 비기술자들과 공유하는 테스트에 적합하다.

TDD를 실천한다면 준비구절보단 실행구절들부터 작성하고나서, 제품코드가 준비된다음 준비구절을 작성해도 된다. 그러나 테스트 전에 제품코드가 작성되어있다면 AAA패턴을 순서대로 작성하는것이 좋다.

### 피해야할 함정

때로는 준비,실행, 검증 구절이 각각 여러개 있는 테스트를 만날 수 있다.
그러나 통합테스트가 아닌 단위테스트에서 검증과 실행구절이 반복되는건 좋지않으니 리팩토링해라.
 
물론 통합테스트에서는 의존성들을 포함해 테스트하게 될때 각 테스트들이 느려질 수 있으니 실행구절을 여러개 둬도 괜찮다. 그러나 단위테스트나 충분히 빠른 통합테스트에서는 단위테스트를 여러개로 나눠라.

#### 테스트 내 if문을 피해라

if문으로 분기를 치는 단위 테스트도 지양하자. 한번에 너무 많은것을 검증하는 구조이다. 당연히 통합 테스트에서도 분기를 쳐서 얻는 이점이 없다. 추가 유지비만 불어난다.

```
@Test
public void testCalculateDiscount() {
    Customer customer = new Customer("John Doe", 4); // 4는 고객의 등급을 의미

    if (customer.getGrade() > 3) {
        assertEquals(0.15, customer.calculateDiscount(), 0.01);
    } else {
        assertEquals(0.10, customer.calculateDiscount(), 0.01);
    }
}
```
예시로 이 코드에서도 오류가 생기면 어디서 오류가 난건지 알수 없으니 분리하라고 하는 것이다.

#### 각 구절은 얼마나 커야 하는가?

- 준비 구절이 가장 큰 경우
  - 당연히 준비 구절이 실행과 검증을 합친 만큼 코드수가 클수도 있다. 그러나 이 이상을 넘어가면 팩토리 클래스로 도출해서 줄이는 것을 추천한다. 준비 구절에서 코드 재사용에 도움이 되는 두가지 패턴으로 **오브젝트 마더**와 **테스트 데이터 빌더**가 있다.

- 실행 구절이 한 줄 이상인 경우를 경계
  - 실행 구절은 보통 코드 한줄이다. 실행구절이 두줄 이상인 경우 호출중인 api가 문제가 있는 셈이다.

    ```
    //준비
        var store = new Store();
        store.AddInventory(Product.shampoo, 10);
        var customer = new Customer();
        
        //실행
        bool success = customer.Purchase(store, Product.Shampoo, 15);
        store.RemoveInventory(store, Product.Shampoo, 15);
        
        //검증 (실패 했기에 수량 변화가 없음)
        Assert.False(success);
        Assert.Equals(10, store.GetInventory(Product.Shampoo));
    ```

    이렇게 실행 구절이 두줄인 경우를 보자. 구매를 한 뒤 두번째 메서드를 호출해야하므로 캡슐화가 깨진거다. 

    비즈니스 관점에서 구매가 정상적으로 이뤄지면 고객의 제품 획득(customer.Purchase)과 매장재고감소(store.RemoveInventory) 두가지가 동시에 일어나 단일 호출로 이루어져야 맞다.

    하나의 기능인데 각각 호출해야한다면 둘중 하나를 누락했을때 논리적 모순이 생긴다. 이러한 모순을 **불변 위반** 이라고 하며 잠재적 모순으로부터 코드를 보호하는 행위를 **캡슐화**라고 한다.

    테스트코드란 클라이언트 입장에서 어떻게 호출하느냐에 대한 이야기기도하기때문에 클라이언트 코드에 의존하지 않도록(클라이언트가 어떻게 잘짰는지에 대해 의존하지 않도록) 불변 위반을 초래할 수 있는 잠재적인 행동을 제거해야 한다.

- 테스트당 하나의 검증을 가져야하는가?
  - 단위 테스트는 동작의 단위 이므로 단일 동작으로 인해서도 여러 결과가 나올 수 있으니, 하나의 테스트로 모든 결과를 평가하는 것이 좋다. 단,검증 구절이 너무 커지는 것은 경계하자.
  예를 들어 SUT에 반환된 객체 내에서 모든 속성을 검증하지 말고 적절한 동등 멤버를 정의하자. (= 그러니까 a클래스의 요소1과 요소2와 요소3등등등 전부 검사하지말고 어떤걸 검사해야할지 정의하고 비교하라는 얘기)

- 주석 제거하라
  - AAA 패턴을 따르고 준비 및 검증 구절에 빈줄을 추가하지 않아도 되는 테스트라면 구절 주석을 제거하자
  - 아니라면 주석을 유지해라

### 테스트를 가능한 한 읽기 쉽게 만드는 방법

#### 테스트 간 테스트 픽스쳐 재사용

테스트 픽스쳐는 테스트 실행 대상 객체다. 하나의 테스트 클래스내에서 각각의 테스트 메서드마다 준비 구절이 동일 한 경우 해당 클래스의 생성자로 미리 추출해 준비구절을 제거하는 패턴이다.

```
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class BankAccountTest {
    private BankAccount account;

    // 테스트 픽스처 설정
    @BeforeEach
    public void setUp() {
        account = new BankAccount(1000.0);  // 모든 테스트는 $1,000의 초기 잔액으로 시작합니다.
    }

    // 입금 테스트
    @Test
    public void testDeposit() {
        account.deposit(500);
        assertEquals(1500, account.getBalance(), "Balance should be $1,500 after depositing $500.");
    }

    // 출금 테스트
    @Test
    public void testWithdraw() {
        account.withdraw(200);
        assertEquals(800, account.getBalance(), "Balance should be $800 after withdrawing $200.");
    }

    // 잘못된 금액 입금 테스트
    @Test
    public void testDepositNegativeAmount() {
        account.deposit(-300);
        assertEquals(1000, account.getBalance(), "Balance should remain the same after attempting to deposit a negative amount.");
    }
}
```

@BeforeEach로 테스트에 필요한 것들을 미리 준비해놓고 테스트 로직에서는 실행,검증부분만 작성하는 것이다.

다만 이러한 방법들은

- 테스트 간 결합도가 높아지고
- 가독성이 떨어진다.

#### 테스트 간의 높은 결합도는 안티 패턴이다.

```
@BeforeEach
    public void setUp() {
        account = new BankAccount(1000.0);  // 모든 테스트는 $1,000의 초기 잔액으로 시작합니다.
    }
```
만약 준비 로직에서 1000이었던것을 100으로 수정해보자. 그러면 testWithdraw의 테스트는 원래 성공했어야하지만 빠그라지게 된다.

원칙은 테스트를 수정해도 다른 테스트에 영향을 주어서는 안되는데 이 경우는 그걸 위반하고 있다.

이 지침을 따르려면 테스트 클래스에 공유 상태를 두지말아야한다. C#에서 제공하는 테스트 픽스쳐의 경우 이러한 공유 상태의 문제가 생기지만 자바에서 제공하는 @BeforeEach의 경우 각 테스트메서드가 실행될때마다 각각의 @BeforeEach가 동작하기때문에 이 문제에 대한 걱정은 없다.

#### 가독성을 떨어뜨리는 생성자 사용

테스트 픽스처를 재사용할때 생성자 사용만이 최선은 아니다. 비공개 팩토리 메서드를 사용하는 법도 있다.

```
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class BankAccountTest {
    // 비공개 팩토리 메서드로 테스트 픽스처 생성
    private BankAccount createAccountWithBalance(double initialBalance) {
        return new BankAccount(initialBalance);
    }

    @Test
    public void testDeposit() {
        BankAccount account = createAccountWithBalance(1000.0);
        account.deposit(500);
        assertEquals(1500, account.getBalance(), "Balance should be $1,500 after depositing $500.");
    }

    @Test
    public void testWithdraw() {
        BankAccount account = createAccountWithBalance(1000.0);
        account.withdraw(200);
        assertEquals(800, account.getBalance(), "Balance should be $800 after withdrawing $200.");
    }

    @Test
    public void testWithdrawMoreThanBalance() {
        BankAccount account = createAccountWithBalance(500.0);
        account.withdraw(600);
        assertEquals(500, account.getBalance(), "Balance should remain $500 after attempting to withdraw more than the balance.");
    }
}

```

위 코드를 보면 createAccountWithBalance에서 initialBalance를 파라미터로 받아서 객체 초기 잔액을 생성한다.

공통 초기화 코드를 비공개 팩토리 메서드로 추출하면 (비공개 메서드를 잘 일반화한다면)
1. 테스트코드를 짧게 만드는 동시에
2. 여러 테스트에서 해당 메소드를 사용하기에 재사용성을 올리고
3. 동시에 테스트 진행 상황에 대한 전체 맥락을 유지할 수 있다.


# 2. 좋은 단위 테스트 명명법

이러한 네이밍이 유명하지만 필자는 추천하지 않는다.

[테스트 대상 메서드] _ [시나리오] _ [예상 결과]

동작 대신 구현 세부 사항에 집중하게끔 부추기기 때문에 도움이 되지 않는다고 본다.
이보다는 좀 더 간단하고 명확한 문구로 동작을 설명하도록 하자.

예를 들어 위와 같은 규칙을 따라 이름을 짓는다면, 덧셈 기능을 하는 테스트는
**Sum_TwoNumbers_ReturnSum()** 이 되는데 이보단 단순히 **Sum_of_two_numbers()** 가 더 알아보기 쉽다.

물론 이에 대한 지침도 있다.

- 엄격한 명명 정책을 따르지 말자. 복잡한 동작에 대한 설명을 이름에 넣는건 한계가 있으니 표현의 자유를 허용하자.
- 비개발자들에게 시나리오를 설명하듯 테스트 이름을 짓자.(비개발자란 도메인 전문가를 얘기한다.)
- 단어를 언더스코어(_)로 구분하자.

지금까지는 테스트 클래스 이름을 제품클래스이름+Tests 패턴을 이용해 작성했지만
테스트는 해당 클래스 검증용이라기보단 동작의 단위로써 사용하자. 동작의 단위로 사용하다보면 클래스 단위가 아닌 여러 클래스에 걸쳐 있을수도 있기 때문이다. 

PS) 메소드와 테스트명이 매핑되게 하지말자.

테스트 이름에 SUT의 메서드 이름을 포함하지 말자. 이유는 코드를 테스트하는게 아니라 애플리케이션 동작을 테스트하는 것이 핵심이기 때문이다. **SUT는 단지 진입점, 동작을 호출하는 수단일 뿐이다.** 동작 대신 코드를 목표로하면 해당 코드의 구현 세부 사항과 테스트 간의 결합도가 높아진다.

# 3. 매개변수화된 테스트 작성 

매개변수에 따라 다른 테스트를 매번 작성하지말고 하나의 테스트로 묶어서 작성하자.

이건 이전에 작성했던 369 게임에 대해 특정 룰을 적용하면 제대로 return이 오는지 검사하는 로직이다.
```
@ParameterizedTest
    @CsvSource({
            "3, clap",
            "6, clap",
            "9, clap",
            "369, clap",
            "1, '1'",
            "10, '10'",
            "29, 'clap'",
            "30, clap",
            "39, clap",
            "40, '40'"
    })
    void doSeoul369(int number, String expected) {
        ClapRule rule = new Seoul369();
        assertEquals(expected, rule.do369(number));
    }
```
이를 파라미터를 사용하지 않고 일일히 케이스를 나눠 작성했다면 재사용성도 낮아지고 똑같은 코드만 늘어났겠지만 매개변수화된 테스트로 작성함에 따라 테스트 코드의 양을 줄였다.

# 4. 검증문 라이브러리를 사용한 가독성 향상

```
@Test
    public void testDeposit() {
        BankAccount account = createAccountWithBalance(1000.0);
        account.deposit(500);
        assertEquals(1500, account.getBalance(), "Balance should be $1,500 after depositing $500.");
    }
```

**assertEquals(500, account.getBalance());** -> 단순히 이런식으로 특정값이 다른값과 같냐고 검사하는 것도 무슨말인지 알아 먹을 수 있으나
우리는 이야기의 형태로 정보를 흡수하는 것을 선호한다.

주어 + 행동 + 목적어

이러한 패턴을 추천한다. (Java의 경우 AssertJ를 사용해 이를 적용할 수 있다.)

```
@Test
    public void testWithdrawMoreThanBalance() {
        BankAccount account = createAccountWithBalance(500.0);
        account.withdraw(600);
        assertThat(account.getBalance()).isEqualTo(500);  // 원하는 상세한 메시지를 체이닝으로 추가 가능
    }
```

해당 코드에선 (account.getBalance()).isEqualTo(500) 이기 때문에 account가 비교대상이고 getBalance를 가져와야하고 isEqualTo가 우리의 목적인게 조금 더 읽기 쉬워진다.