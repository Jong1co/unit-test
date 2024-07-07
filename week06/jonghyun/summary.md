# chapter 06 다ㄴ위 테스트 스타일

## 6.1 단위 테스트의 세 가지 스타일

- 출력 기반 테스트
- 상태 기반 테스트
- 통신 기반 테스트

### 6.1 출력 기반 스타일 정의

SUT에 입력을 넣고 생성되는 출력을 점검하는 방식 (반환 값만을 검증)

- 출력 기반 단위 테스트 스타일은 함수형이라고도 함
- 사이드 이펙트 없는 코드 선호를 강조하는 프로그래밍 방신인 함수형 프로그래밍에 뿌리를 두고 있음

### 6.2 상태 기반 스타일 정의

작업이 완료된 후 시스템 상태를 확인하는 것(최종 상태를 검증함)

- 상태: SUT나 협력자 중 하나, 또는 데이터베이스나 파일 시스템 등과 같은 프로세스 외부 의존성의 상태 등을 의미함

### 6.3 통신 기반 스타일 정의

목을 사용해 테스트 대상 시스템과 협력자 간의 통신을 검증함 (SUT가 협력자를 올바르게 호출하는지 검증)

## 6.2 단위 테스트 스타일 비교

세 가지 단위 테스트는 앞서 나온 좋은 단위 테스트의 4대 요소를 벗어나지 않음

- 회귀 방지
- 리팩터링 내성
- 빠른 피드백
- 유지 보수성

### 6.2.1 회귀 방지와 피드백 속도 지표로 스타일 비교하기

회귀 방지의 특성은 다음과 같음

- 테스트 중에 실행되는 코드의 양
- 코드 복잡도
- 도메인 유의성

- 어떤 스타일도 첫번째 실행되는 코드의 양 부분에서는 도움이 되지 않음(원하는 대로 테스트를 작성할 수 있기 때문)
- 코드 복잡도와 도메인 유의성 역시 마찬가지
- 통신 기반 스타일에는 예외가 하나 있음
  - (목을) 남용하면 작은 코드 조각을 검증하고 다른 것을 모두 목을 사용하는 등 피상적인 테스트가 될 수 있음
  - 하지만 이는 기술을 남용하는 극단적인 사례임

**즉, 회귀 방지에서는 어떤 스타일도 도움이 되지 않음**

피드백 속도 또한 상관관계가 거의 없음

### 6.2.2 리팩터링 내성 지표로 스타일 비교하기

리팩터링 내성은 리팩터링 중에 발생하는 거짓 양성 수에 대한 척도임

- 출력 기반 테스트: 거짓 양성 방지가 가장 우수함
- 상태 기반 테스트: 거짓 양성이 되기 쉬움
  - 이러한 테스트는 SUT 외에도 클래스 상태와 함께 작동함
  - 확률적으로 테스트와 제품 코드 간의 결합도가 클수록 유출되는 구현 세부 사항에 테스트가 얽매일 가능성이 커짐
  - 상태 기반 테스트는 큰 API 노출 영역에 의존하므로, 구현 세부 사항과 결합할 가능성도 더 높음
- 통신 기반 테스트: 허위 경보에 가장 취약함
  - 항상 스텁과 상호 작용하는 경우 이러한 상호 작용을 확인해서는 안 됨
  - **애플리케이션 경계를 넘는 상호 작용**을 확인하고 해당 상호 작용의 **사이드 이펙트가 외부 환경에 보이는 경우**에만 목이 좋음

### 6.2.3 유지 보수성 지표로 스타일 비교하기

유지 보수성 지표는 단위 테스트 스타일과 밀접한 관련이 있음

유지보수성은 다음 두 가지 특성으로 정의함

- 테스트를 이해하기 얼마나 어려운가? (테스트 크기에 대한 함수)
- 테스트를 실행하기 얼마나 어려운가? (테스트에 직접적으로 관련 있는 프로세스 외부 의존성 개수에 대한 함수)

- 출력 기반 테스트의 유지 보수성
  - 가장 유지 보수하기 용이함
  - 해당 코드는 전역 상태나 내부 상태를 변경할 리 없음(프로세스 외부 의존성을 다루지 않음)
- 상태 기반 테스트의 유지 보수성
  - 출력 기반 테스트보다 유지보수가 쉽지 않음
  - 출력 검증보다 더 많은 공간을 차지하기 때문

```c#
[Fact]
public void Adding_a_comment_to_an_article()
{
  var sut = new Article();
  var text = "comment text";
  var author = "John Doe";
  var now = new DateTime(2019, 4, 1);

  sut.AddComment(text, author, now);

// 글의 상태를 검증
  Assert.Equal(1, sut.Comments.Count);
  Assert.Equal(text, sut.Comments[0].Text);
  Assert.Equal(author, sut.Comments[0].Author);
  Assert.Equal(now, sut.Comments[0].DateCreated);
}
```

위 테스트는 단순하고 댓글이 하나 있지만, 검증부는 네 줄에 걸쳐 있다.

- 헬퍼 메서드를 사용하여 해결할 수 있지만, 이러한 메서드를 작성하고 유지하는 데 상당한 노력이 필요함
- 또는 검증 대상 클래스의 동등 멤버를 정의할 수 있음 (테스트만을 위한 코드가 제품 코드에 들어가는 것이 아닐까?)
  - 필자 또한 본질적으로 클래스가 값에 해당하고 값 객체로 변환할 수 있을 때만 효과적이라고 함
  - 그렇지 않으면 코드 오염으로 이어짐
  - 근데 이러한 상태는 제품 코드에서도 유용하게 사용될 거 같음

```c#
[Fact]
public void Adding_a_comment_to_an_article()
{
  var sut = new Article();
  var text = "comment text";
  var author = "John Doe";
  var now = new DateTime(2019, 4, 1);

  sut.AddComment(text, author, now);

  // 헬퍼 메서드
  sut.ShouldContainNumberOfComments(1)
    .WithComment(text, author, now);
}
```

위와 같은 기술을 사용하더라도 출력 기반 테스트보다 공간을 더 많이 차지하므로 유지 보수성이 떨어짐

- 통신 기반 테스트의 유지 보수성

통신 기반 테스트는 sut와 상호 작용 검증을 설정해야 하며, 이는 공간을 더 많이 차지하기 때문에 위 두 개의 스타일보다 유지 보수하기가 어려워짐

### 6.2.4 스타일 비교하기: 결론

|                                  | 출력 기반 | 상태 기반 | 통신 기반 |
| -------------------------------- | --------- | --------- | --------- |
| 리팩터링 내성을 지키기 위한 노력 | 낮음      | 중간      | 중간      |
| 유지비                           | 낮음      | 중간      | 높음      |

- 상태와 통신 기반 테스트는 두 지표 모두 좋지 않음
- 그러므로 출력 기반 테스트를 선호해야 함
- 하지만 대부분 객체지향 프로그래밍 언어에는 해당하지 않음(함수형으로 작성된 코드에만 적용되기 때문)
- 그렇지만 테스트를 출력 기반 스타일로 변경하는 기법은 있음 (코드를 순수 함수로 만들면 됨)

## 6.3 함수형 아키텍처 이해

### 6.3.1 함수형 프로그래밍

함수형 프로그래밍이란 수학적 함수를 사용한 프로그래밍임

- 수학적 함수는 숨은 입출력이 없는 함수로 첫 번째 집합의 각 요소에 대해 두 번째 집합에서 정확히 하나의 요소를 찾는 두 집합 사이의 관계
- 입출력을 명시한 수학적 함수는 이에 따르는 테스트가 짧고 간결하며 이해하고 유지보수하기 쉬우므로 테스트하기가 매우 쉬움
- 반면에 숨은 입출력은 코드를 테스트하기 힘들게 한다
  - 숨은 입출력의 종류
  - 사이드 이펙트: 메서드 시그니처에 표시되지 않은 출력
  - 예외: 메서드가 예외를 던지면, 프로그램 흐름에 메서드 시그니처에 설정된 계약을 우회하는 경로를 만듬
  - 내외부 상태에 대한 참조: DateTime.now와 같이 정적 속성을 사용해 현재 날짜와 시간을 가져오는 메서드가 있을 수 있음
- 수학적 함수인지 판별하는 가장 좋은 방법은 메서드에 대한 호출을 반환 값으로 대체할 수 있는 것 (참조 투명성)

### 6.3.2 함수형 아키텍처란?

어떠한 사이드 이펙트도 일으키지 않는 애플리케이션을 만들 수는 없음

- 함수형 프로그래밍의 목표는 비즈니스 로직을 처리하는 코드와 사이드 이펙트를 일으키는 코드를 분리하는 것임
  - 아래 두 가지 코드 유형을 구분하여 비즈니스 로직과 사이드 이펙트를 분리할 수 있음
  - 결정을 내리는 코드(함수형 코어): 순수 함수
  - 해당 결정에 따라 작용하는 코드(가변 셸): 모든 결정을 가시적인 부분으로 변환함

함수형 코어와 가변 곌은 다음과 같은 방식으로 협력함

- 가변 셸은 모든 입력을 수집
- 함수형 코어는 결정을 생성
- 셸은 결정을 사이드 이펙트로 변환

### 6.3.3 함수형 아키텍처와 육각형 아키텍처 비교

함수형 아키텍처와 육각형 아키텍처는 관심사 분리라는 아이디어를 기반으로 한다. (비슷한 점이 있음)

**유사한 점**

- 관심사 분리

  - 육각형 아키텍처는 도메인 계층과 애플리케이션 서비스 계층으로 구별함
  - 도메인 계층은 비즈니스 로직에 책임이 있음
  - 애플리케이션 서비스 계층은 외부 애플리케이션과의 통신에 책임이 있음

- 의존성 간의 단방향 흐름
  - 도메인 계층이 애플리케이션 서비스 계층에 의존하지 않음
  - 함수형 또한 코어는 가변 셸에 의존하지 않음

**차이점**

- 사이드 이펙트에 대한 처리
  - 함수형은 모든 사이드 이펙트를 비즈니스 연산 가장자리로 밀어냄(이 가장자리는 가변 셸이 처리함)
    - 비즈니스 연산 가장자리로 밀어낸다는 것이 가변 셸로 밀어낸다는 것인지 코어의 가장자리로 밀어낸다는 것인지 잘 모르겠음
  - 육각형 아키텍처는 도메인 계층으로 인한 사이드 이펙트도 문제 없음 (모든 수정 사항은 도메인 계층 내에 있어야 하며, 계층의 경계를 넘어서는 안 됨)

## 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환

두 가지 리팩터링 단계를 거쳐 전환할 수 있음

- 프로세스 외부 의존성에서 목으로 변경
  목에서 함수형 아키텍처로 변경

### 6.4.1 감사 시스템 소개

조직의 모든 방문자를 추적하는 감사 시스템

- 텍스트 파일을 기반 저장소로 사용
- 가장 최근 파일의 마지막 줄에 방문자의 이름과 방문 시간을 추가
- 파일당 최대 항목 수에 도달하면 인덱스를 증가시켜 새 파일을 작성함

```c#
public class AuditManager {
    private readonly int _maxEntriesPerFile;
    private readonly string _directoryName;

    public AuditManager(int maxEntriesPerFile, string directoryName) {
        _maxEntriesPerFile = maxEntriesPerFile;
        _directoryName = directoryName;
    }

    public void AddRecord(string visitorName, DateTime timeOfVisit) {
        //파일 목록을 검색
        string[] filePaths = Directory.GetFiles(_directoryName);
        //인덱스 별로 정렬
        (int index, string path)[] sorted = SortByIndex(filePaths);

        string newRecord = visitorName + ';' + timeOfVisit;

        // 파일이 없으면 단일 레코드로 첫 번째 파일을 생성
        if (sorted.Length == 0) {
            string newFile = Path.Combine(_directoryName, "audit_1.txt");
            File.WriteAllText(newFile, newRecord);
            return;
        }

        // 파일이 있으면 최신 파일을 가져옴
        (int currentFileIndex, string currentFilePath) = sorted.Last();
        List<string> lines = File.ReadAllLines(currentFilePath).ToList();


        if (lines.Count < _maxEntriesPerFile) {
            // 한계에 도달하지 않았다면 레코드 추가
            lines.Add(newRecord);
            string newContent = string.Join("\r\n", lines);
            File.WriteAllText(currentFilePath, newContent);
        } else {
            // 한계에 도달했다면 새로운 파일 생성
            int newIndex = currentFileIndex + 1;
            string newName = $"audit_{newIndex}.txt";
            string newFile = Path.Combine(_directoryName, newName);
            File.WriteAllText(newFile, newRecord);
        }
    }
}
```

`AuditManager` 클래스는 파일 시스템과 밀접하게 연결돼 있어 테스트가 어려움

- 테스트 전에 파일을 올바른 위치에 배치하고, 테스트가 끝나면 해당 파일을 읽고 내용을 확인한 후 삭제해야 하기 때문
- 병목 지점은 파일 시스템이고, 이는 테스트 실행 흐름을 방해할 수 있는 공유 의존성임
- 파일 시스템은 테스트를 느리게 함
- 로컬 시스템과 빌드 서버 모두 작업 디렉터리가 있고 테스트할 수 있어야 하므로 유지 보수성도 저하됨

### 6.4.2 테스트 파일 시스템에서 분리하기 위한 목 사용

```c#
public class AuditManager {
    private readonly int _maxEntriesPerFile;
    private readonly string _directoryName;
    private readonly IFileSystem _fileSystem;

    public AuditManager(int maxEntriesPerFile, string directoryName, IFileSystem fileSystem) {
        _maxEntriesPerFile = maxEntriesPerFile;
        _directoryName = directoryName;
        _fileSystem = fileSystem
    }
}
```

위와 같이 `AditManager` 에 `IFileSystem` 을 주입하면 파일 시스템을 목으로 대체가 가능하다.

- 파일 시스템이 분리되므로, 공유 의존성이 사라지고 테스트를 독립적으로 실행할 수 있음

```c#
[Fact]
public void A_new_file_is_created_when_the_current_file_overflows() {
    var fileSystemMock = new Mock<IFileSystem>();
    fileSystemMock
        .Setup(x => x.GetFiles("audits"))
        .Returns(new string[]
        {
            @"audits\audit_1.txt",
            @"audits\audit_2.txt"
        });

    fileSystemMock
        .Setup(x => x.ReadAllLines(@"audits\audit_2.txt"))
        .Returns(new List<string>
        {
            "Peter; 2019-04-06T16:30:00",
            "Jane; 2019-04-06T16:40:00",
        });
    var sut = new AuditManager(3, "audits", fileSystemMock.Object);

    sut.AddRecord("Alice", DateTime.Parse("2019-04-06T18:00:00"));

    fileSystemMock.Verify(x => x.WriteAllText(
        @"audits\audit_3.txt",
        "Alice;2019-04-06T18:00:00"));
}
```

위와 같이 파일 시스템과의 통신과 이러한 통신의 사이드 이펙트(파일 변경)은 애플리케이션의 식별할 수 있는 동작이다.

### 6.4.3 함수형 아키텍처로 리팩터링하기

인터페이스 뒤로 사이드 이펙트를 숨기고 해당 인터페이스를 주입하는 대신, 사이드 이펙트를 클래스 외부로 완전히 이동할 수 있음

```c#
public class AuditManager {
    private readonly int _maxEntriesPerFile;

    public AuditManager(int maxEntriesPerFile) {
        _maxEntriesPerFile = maxEntriesPerFile;
    }

    public FileUpdate AddRecord(
        FileContent[] files, // file path 대신 file content 배열을 받음
        string visitorName,
        DateTime timeOfVisit) {

        (int index, FileContent file)[] sorted = SortByIndex(files);
        string newRecord = visitorName + ';' + timeOfVisit;
        if (sorted.Length == 0) {
            return new FileUpdate( Returns an update "audit_1.txt", newRecord); // Returns an update instruction
        }

        (int currentFileIndex, FileContent currentFile) = sorted.Last();
        List<string> lines = currentFile.Lines.ToList();

        if (lines.Count < _maxEntriesPerFile) {
            lines.Add(newRecord);
            string newContent = string.Join("\r\n", lines);
            return new FileUpdate(currentFile.FileName, newContent); // Returns an update instruction
        } else {
            int newIndex = currentFileIndex + 1;
            string newName = $"audit_{newIndex}.txt";
            return new FileUpdate(newName, newRecord);
        }
    }
}

// 가변 셸
public class Persister {
    public FileContent[] ReadDirectory(string directoryName) {
        return Directory
            .GetFiles(directoryName)
            .Select(x => new FileContent(
                Path.GetFileName(x),
                File.ReadAllLines(x)
                ))
                .ToArray();
    }

    public void ApplyUpdate(string directoryName, FileUpdate update) {
        string filePath = Path.Combine(directoryName, update.FileName);
        File.WriteAllText(filePath, update.NewContent);
    }
}
```

위와 같이 작성하면

1. Persister 클래스에서는 작업 디렉터리 내용을 읽고
2. AuditManager에서 받은 업데이트 명령을 작업 디렉터리에서 다시 수행하기만 하면 된다.

이렇게 작성한다면 모든 복잡도는 AuditManager 클래스에 있다.

**AuditManager와 Persister를 붙이려면 애플리케이션 서비스라는 또 다른 클래스가 필요하다**<br/>
(가변셸 !== 애플리케이션 서비스)

```c#
public class ApplicationService {
    private readonly string _directoryName;
    private readonly AuditManager _auditManager;
    private readonly Persister _persister;

    public ApplicationService(string directoryName, int maxEntriesPerFile) {
        _directoryName = directoryName;
        _auditManager = new AuditManager(maxEntriesPerFile);
        _persister = new Persister();
    }

    public void AddRecord(string visitorName, DateTime timeOfVisit) {
        FileContent[] files = _persister.ReadDirectory(_directoryName);
        FileUpdate update = _auditManager.AddRecord(files, visitorName, timeOfVisit);
        _persister.ApplyUpdate(_directoryName, update);
    }
}
```

함수형 코어와 가변 셸을 붙이면서 애플리케이션 서비스가 외부 클라이언트를 위한 시스템의 진입점을 제공함

```c#
[Fact]
public void A_new_file_is_created_when_the_current_file_overflows() {
    var sut = new AuditManager(3);
    var files = new FileContent[] {
        new FileContent("audit_1.txt", new string[0]),
        new FileContent("audit_2.txt", new string[] {
            "Peter; 2019-04-06T16:30:00",
            "Jane; 2019-04-06T16:40:00",
            "Jack; 2019-04-06T17:00:00"
        })
    };

    FileUpdate update = sut.AddRecord(files, "Alice", DateTime.Parse("2019-04-06T18:00:00"));

    Assert.Equal("audit_3.txt", update.FileName);
    Assert.Equal("Alice;2019-04-06T18:00:00", update.NewContent);
}
```

위와 같이 이제 검증은 AuditManager가 내린 결정을 검증만 하면 된다.

## 6.5 함수형 아키텍처의 단점

- 프로세스 외부 의존성을 더 많이 호출하여 성능이 떨어짐
  - 하지만 성능 영향이 그다지 눈에 띄지 않는 일부 시스템에서는 유지 보수성을 향상시키는 편이 좋음
- 코드베이스 크기 증가
  - 모든 프로젝트에 초기 투자가 타당할 만큼 복잡도가 높은 것이 아님
- **함수형 방식에서 순수성에 많은 비용이 든다면 순수성을 따르지 말라.**
  - 대부분 프로젝트에서는 모든 도메인 모델을 불변으로 할 수 없기 때문에 출력 기반 테스트에만 의존할 수 없음
  - 대부분의 경우 출력 기반 스타일과 상태 기반 스타일을 조합하며, 통신 기반 스타일을 섞어도 괜찮음
