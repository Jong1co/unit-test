# Chapter 02 단위 테스트란 무엇인가

## 2.1 단위 테스트의 정의

단위 테스트는 가장 중요한 세 가지 속성이 있음

- 작은 코드 조각을 검증
- 빠르게 수행
- 격리된 방식으로 처리하는 자동화된 테스트

세번째 **격리된 방식으로 처리하는 자동화된 테스트** 를 두고 격리가 정확히 무엇인지에 대한 의견 차이로 `고전파` 와 `런던파` 의 분파가 갈리게 됨

### 2.1.1 격리 문제에 대한 런던파의 접근

런던파는 코드 조각을 격리된 방식으로 검증함

- 테스트 대상 시스템을 협력자에게서 격리하는 것
- 모든 의존성을 테스트 대역(목)으로 대체해야 함

**런던파의 특징**

- 테스트가 실패하면 코드베이스의 어느 부분이 고장 났는지 확실히 알 수 있음
- 객체 그래프를 분할할 수 있음
  - 직접적인 의존성을 갖고 있으며, 그래프가 상당히 복잡해질 수 있는데 런던파는 모든 의존성을 테스트 대역으로 대체하기에 편하게 테스트 코드를 작성할 수 있음
- 테스트 단위에 대한 고민이 줄 수 있음(ex. 클래스가 있으면 클래스에 대한 단위 테스트 클래스를 생성하라!)

```c#
// 고전파

public void Purchase_succeeds_when_enough_inventory()
{
  //arrange
  var store = new Store(); // -> 협력자
  store.AddInventory(Product.Shampoo, 10);
  var customer = new Customer(); // -> SUT(테스트 대상 시스템)

  //act -> 검증하고자 하는 동작을 수행
  bool success = customer.Purchase(store, Product.Shampoo, 5);

  //assert
  Assert.True(success);
  Assert.Equal(5, store.GetInventory(Product.Shampoo))
}

```

- 테스트 대상 메서드를 컴파일 하려면 Store 인스턴스가 필요하기 때문에 Store = 협력자
- 위 코드는 고전적인 방식의 자연스러운 결과로, Customer만이 아닌 Store도 함께 검증함
- 두 클래스는 격리되어 있지 않기 때문에 Store에 오류가 발생하면 단위 테스트에 실패할 수 있음

```c#

//런던파
public void Purchase_succeeds_when_enough_inventory()
{
  //arrange
  var storeMock = new Mock<IStore>(); // -> 협력자
  storeMock
    .Setup(x => x.HasEnoughInventory(Product.Shampoo, 10))
    .Returns(true);
  var customer = new Customer(); // -> SUT(테스트 대상 시스템)

  //act -> 검증하고자 하는 동작을 수행
  bool success = customer.Purchase(storeMock.Object, Product.Shampoo, 5);

  //assert
  Assert.True(success);
  storeMock.Vertify(
    x => x.RemoveInventory(Product.Shampoo, 5),
    Times.Once);
}
```

- Store의 실제 인스턴스를 생성하지 않고, Mock으로 대체. 즉, Store를 사용하지 않음
- 이전에는 Store의 상태를 검증했다면, 지금은 Customer와 Store의 상호작용을 검증함. 즉, RemoveInventory가 한 번 동작하는지를 검증함.
  - 결과가 아니라 상호작용을 검증하는구나.

### 2.1.2 격리 문제에 대한 고전파의 접근

- 런던파와 다른 점은 무엇이 작은 코드 조각 단위에 해당하는가
- 고전파의 방식은 코드를 꼭 격리하는 방식으로 테스트해야 하는 것은 아니다.
- 공유 의존성이 없는 한 여러 클래스를 묶어서 테스트할 수 있음
- 하지만 단위 테스트는 서로 격리하여 실행해야 한다.
  - 그럼 런던파의 테스트는 격리하여 실행하지 않아도 되는 것인가?
  - 왜 ? -> 외부 의존성을 항상 모킹하는 방식으로 하기에 격리할 필요가 없음
- 테스트 간에 공유 상태를 일으키는 의존성에 대해서만 테스트 대역을 사용함
  - 파일 시스템과 데이터베이스가 이에 해당됨
  - 서로간의 간섭으로 인해 오류가 발생할 수 있기 때문
  - 또한 실행 속도를 높이는 데 필요함 (단위 테스트의 속성 중 빨리 실행해야 하는 필요성이 있기에)
    - 공유 의존성을 포함하는 테스트는 통합 테스트에 포함됨

**의존성**

- 공유 의존성
  - 테스트 간에 공유되고 서로의 결과에 영향을 미칠 수 있는 수단을 제공하는 의존성
  - ex) 정적 가변 필드
  - TODO: 정적 가변 필드가 무엇인가? 전역 상태와 같은 것인가?
- 휘발성 의존성
  - 비결정적 동작을 포함 (난수 생성기와 현재 날짜를 반환하는 클래스 등)
- 비공개 의존성
  - 공유하지 않는 의존성
- 프로세스 외부 의존성
  - 애플리케이션 실행 프로세스 외부에서 실행되는 의존성
  - db는 프로세스 외부이면서 공유 의존성임
  - 하지만 각 테스트 전에 컨테이너로 실행하면 동일한 인스턴스가 아니기에 공유하지 않는 의존성임
