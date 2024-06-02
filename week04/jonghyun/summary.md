# Chapter 04 좋은 단위 테스트의 4대 요소

**좋은 단위 테스트**

- 회귀 방지
- 리팩터링 내성
- 빠른 피드백
- 유지 보수성

### 회귀 방지

코드를 수정한 후, 기능이 의도한 대로 작동하지 않는 경우를 방지하기 위함

**고려해야 할 점**

- 테스트 중에 실행되는 코드의 양
  - 일반적으로 실행되는 코드가 많을 수록 테스트에서 회귀가 나타날 가능성이 높음
- 코드 복잡도와 도메인 유의성
  - 복잡한 비즈니스 로직을 나타내는 코드가 보일러플레이트 코드보다 훨씬 더 중요함
- 단순한 코드를 테스트하는 것은 가치가 없음
- 테스트가 라이브러리, 프레임워크, 외부 시스템을 테스트 범주에 포함시켜서 소프트웨어가 이러한 의존성에 대해 검증이 올바른지 확인해야 함
  - 내가 직접 테스트를 하라는 것이 아니라, 포함되는지 검증되도록 만들라는 의미인 거 같음
  - 예를 들어 react -> 렌더링이 잘 되는지를 테스트하는 것이 아니라, 렌더링되어 화면에 내가 의도한 바가 정확하게 출력되는지 확인하라는 뜻인듯

### 리팩터링 내성

종종 리팩터링을 진행하면 실제 기능은 제대로 동작하는데, 테스트만 실패하는 경우가 있음

-> 이를 거짓 양성이라고 함

- 이는 보통 구현을 수정하지만 식별할 수 있는 동작은 유지할 때 발생함
- 거짓 양성이 방지되는 것이 리팩터링 내성

**거짓 양성**

- 테스트가 타당한 이유가 없이 실패하면, 코드 문제에 대응하는 능력과 의지가 희석됨. 이로 인해 실패를 무시하고, 기능이 고장나도 운영 환경에 들어가게 됨
- 테스트 스위트에 대한 신뢰가 서서히 떨어지며, 이를 더 이상 믿을 만한 안정망으로 인식하지 않음

**거짓 양성의 원인**

거짓 양성의 수는 테스트 구성 방식과 관련이 있음

- 테스트와 테스트 대상 시스템의 구현 세부 사항이 많이 결합할 수록 허위 경보가 더 많이 생김
- 이를 해결하기 위해서는 구현 세부 사항에서 테스트를 분리해야 함
- 즉, 테스트는 최종 사용자의 관점에서 SUT를 검증하고, 최종 사용자에게 의미 있는 결과만 확인해야 함

```c#

//구현 코드

public class Message
{
    public string Header { get; set; }
    public string Body { get; set; }
    public string Footer { get; set; }
}

public interface IRenderer
{
    string Render(Message message);
}

public class MessageRenderer : IRenderer
{
    public IReadOnlyList<IRenderer> SubRenderers { get; }

    public MessageRenderer()
    {
        SubRenderers = new List<IRenderer>
        {
            new HeaderRenderer(),
            new BodyRenderer(),
            new FooterRenderer()
        };
    }

    public string Render(Message message)
    {
        return SubRenderers
            .Select(x => x.Render(message))
            .Aggregate("", (str1, str2) => str1 + str2);
    }
}

public class FooterRenderer : IRenderer
{
    public string Render(Message message)
    {
        return $"<i>{message.Footer}</i>";
    }
}

public class BodyRenderer : IRenderer
{
    public string Render(Message message)
    {
        return $"<b>{message.Body}</b>";
    }
}

public class HeaderRenderer : IRenderer
{
    public string Render(Message message)
    {
        return $"<h1>{message.Header}</h1>";
    }
}
```

```c#

//테스트 1
public void MessageRenderer_uses_correct_sub_renderers()
{
    var sut = new MessageRenderer();

    IReadOnlyList<IRenderer> renderers = sut.SubRenderers;

    Assert.Equal(3, renderers.Count);
    Assert.IsAssignableFrom<HeaderRenderer>(renderers[0]);
    Assert.IsAssignableFrom<BodyRenderer>(renderers[1]);
    Assert.IsAssignableFrom<FooterRenderer>(renderers[2]);
}
```

- 테스트 1번과 같이 테스트 코드를 작성한다면 하위 렌더링 클래스를 재배열하거나 그 중 하나를 새 것으로 교체하면 테스트가 실패함 (거짓 양성)
- 또는 하위 렌더링 클래스를 제거하고 직접 렌더링을 구현할 때, 테스트가 실패함 (거짓 양성)

-> 이유는 구현 세부 사항과 테스트가 결합됐기 때문임

```c#
public void Rendering_a_message()
{
    var sut = new MessageRenderer();
    var message = new Message
    {
        Header = "h",
        Body = "b",
        Footer = "f"
    };

    string html = sut.Render(message);

    Assert.Equal("<h1>h</h1><b>b</b><i>f</i>", html);
}
```

- 구현 세부 사항이 아닌 결과값을 검증하면 내부적으로 구현이 바뀐다고 하더라도, 테스트가 실패하는 거짓 양성이 발생하지 않음

### 테스트 정확도 극대화

테스트 결과를 넓은 관점으로 살펴봤을 때, 4가지 결과가 있을 수 있음

- 테스트 통과 - 작동 : 올바른 추론
- 테스트 실패 - 고장 : 올바른 추론
- 테스트 실패 - 작동 : 1종 오류(거짓 양성)
  - 거짓 양성을 피하는 데 리팩터링 내성이 도움이 됨
- 테스트 성공 - 고장 : 2종 오류(거짓 음성)
  - 회귀 방지가 훌륭한 테스트는 2종 오류인 거짓 음성의 수를 최소화하는 데 도움이 됨

거짓 양성과 거짓 음성을 생각해보는 방법으로 소음 대비 신호 비율 측면에서 볼 수 있음

테스트 정확도 = 신호(발견된 버그 수) / 소음(허위 경보 발생 수)

- 신호(발견된 버그 수)를 증가시키거나 : 회귀를 더 잘 찾아내는 테스트로 개선하는 것
- 소음(허위 경보 발생 수)를 줄이는 것: 허위 경보를 발생시키지 않는 테스트로 개선하는 것
  - 리팩터링 내성이 좋은 테스트로 변경하는 것

### 빠른 피드백과 유지 보수성

- 빠른 피드백
  - 테스트가 빠르게 실행되면 코드에 결함이 생기자마자 버그에 대해 경고하기 시작할 정도로 피드백 루프를 대폭 줄여, 버그를 수정하는 비용을 거의 0까지 줄일 수 있음
- 유지 보수성 (유지비를 평가함)
  - 아래 두 요소로 결정됨
  - 테스트가 얼마나 이해하기 어려운가
    - 테스트 코드를 일급 시민으로 취급하라 -> 무슨 뜻이지,,?
  - 테스트가 얼마나 실행하기 어려운가

### 이상적인 테스트를 찾아서

- 회귀 방지
- 리팩터링 내성
- 빠른 피드백
- 유지 보수성

위 네 가지 특성을 곱하여 테스트의 가치가 결정되는데, 이는 어떤 특성이 0이라면 전체 가치가 0이 된다는 뜻

**이상적인 테스트**

네 가지 특성 모두 최고 점수인 1을 얻는 것이 가장 이상적이지만, 그런 테스트는 불가능하다.

- 회귀 방지, 리팩터링 내성, 빠른 피드백은 상호 배타적이기 때문임
- 따라서 특성 중 어떤 것도 크게 줄이지 않는 방식으로 최대한 크게 해야 함

아래는 두 특성을 최대로 하는 것을 목표로 해서 한 가지 특성을 희생해 결국 가치가 0에 가까워진 테스트 예시이다.

**1. 엔드 투 엔드 테스트**

- 한 번에 많은 코드를 실행하기 때문에 회귀 방지를 훌륭히 해냄
- 거짓 양성에 면역이 돼 리팩터링 내성도 우수함
  - 구현 세부 사항이 아닌, 실제 사용자 관점에서 테스트를 진행하기 때문에 우수함
- 하지만 느린 속도로 인해, 피드백을 빨리 받기가 어려움

**2. 간단한 테스트**

- 빠르게 실행되고, 거짓 양성이 생길 가능성이 낮기 때문에 리팩터링 내성 또한 우수함
- 하지만 이러한 테스트는 항상 통과하거나 검증이 무의미하기 때문에 어떤 것도 테스트한다고 할 수 없음
  - 회귀 방지가 없음

**3. 깨지기 쉬운 테스트**

```c#

[Fact]
public void GetById_executes_correct_SQL_code()
{
  var sut = new UserRepository();

  User user = sut.GetById(5);

  Assert.Equal(
    "SELECT * FROM dbo.[User] WHERE UserId = 5",
    sut.LastExecutedSqlStatement
  )
}

```

- 위 테스트는 개발자가 SQL 코드 생성을 이상하게 하거나 userId 대신 id로 잘못 사용할 수 있으므로 테스트가 실패해서 이를 지적함
- 하지만 SQL문은 다르게 사용하여도 동일하게 동작할 수 있음
  - SELECT \* FROM dob.User WHERE UserID = 5라고 작성하여도 올바르게 동작함
  - 무엇이 아닌 어떻게에 초점을 맞춘 테스트로 내부 구현 세부 사항이 결합되어 거짓 양성을 만듬

**4. 결론**

- 회귀 방지, 리팩터링 내성, 빠르 피드백은 상호 배타적이기 때문에 좋은 테스트를 만들기 위해서는 절충안을 마련하여 전략적으로 희생해야 함
- 리팩터링 내성을 최대한 마련하는 것이 좋음
  - 리팩터링 내성은 이진 선택이기 때문 (있거나 없거나)

### 테스트 피라미드

테스트 스위트에서 테스트 유형 간의 일정한 비율을 일컫는 개념

- 단위테스트에서 e2e로 갈 수록 사용자 경험에 가장 가깝게 흉내 내는 것을 의미함
- 그렇기 때문에 위에서 말한 빠른 피드백과 회귀 방지 사이에서 선택을 해야 함
  - 피라미드 상단의 테스트는 회귀 방지에 유리하고, 하단은 실행 속도를 강조함
  - 하지만 각각의 계층 또한 리팩터링 내성을 포기해선 안 됨
- 간단한 CRUD -> 단위 테스트와 통합 테스트의 수가 같고 엔드 투 엔드가 없을 수 있음

// 여기 조금 더 정리

### 블랙박스 테스트와 화이트박스 테스트 간의 선택

- 블랙박스 테스트: 내부 구조를 몰라도 시스템 기능을 검사할 수 있는 소프트웨어 테스트 방법
  - 어떻게가 아닌 무엇을 해야 하는지를 중심으로 구축됨
  - 회귀 방지는 나쁘지만 리팩터링 내성이 좋음
- 화이트박스 테스트: 애플리케이션 내부 작업을 검증하는 테스트 방식이며, 테스트는 요구 사항이나 명세가 아닌 소스 코드에서 파생됨
  - 회귀 방지는 좋지만 리팩터링 내성이 나쁨
  - 구현과 결합되어 있기 때문에 깨지기 쉬움
- 따라서 테스트를 작성할 때는 블랙박스 테스트가 바람직하지만, 테스트를 분석할 때에는 화이트박스 방법을 사용할 수 있음
  - 커버리지 도구를 사용해 분기 확인한 후, 처리되지 않은 분기에 대해 내부 구조에 대해 전혀 모르는 것 처럼 테스트하라 !