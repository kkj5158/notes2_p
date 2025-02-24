#### 참조문서
[주니어 개발자의 클린 아키텍쳐 적용기](https://techblog.woowahan.com/2647/)
[헥사고날 아키텍쳐를 향하여(By TDD)](https://blogshine.tistory.com/689)

# 문제 의식
![[클린 아키텍쳐.png]]

클린아키텍쳐에서는 각 계층마다의 독립적인 DATA 모델을 가지게 된다. 

..
# What data crosses the boundaries 
첫 번째로는 경계 간의 데이터를 전달할 때 무엇을 전달해야 하는가에 대한 이야기입니다. **의존성 규칙을 지키기 위해서는 우리가 사용하는 단순하고, 고립된 형태의 데이터 구조를 사용하는 것을 추천합니다.** 만약 DB의 형식의 데이터 구조 또는 Framework에 종속적인 데이터 구조가 사용되게 된다면, 이러한 저수준의 데이터 형식을 고수준에서 알아야 하기 때문에 의존성 규칙을 위반하게 됩니다.

저희 팀에서도 이러한 경계 간의 데이터를 전달할 때 간단한 구조의 Data Transfer Objects(DTO)를 이용하고 있는데, 이를 만들면서 몇 가지 생각이 들게 되었습니다.

### DTO 간의 중복 코드

업무 로직을 구현하다 보면 클라이언트로부터 전달되는 데이터를 DTO를 통해 받아서 처리합니다. 일반적으로 요청은 크게 등록, 수정, 조회, 삭제의 형태로 오게 되는데, 저는 요청에 따라 DTO를 어떻게 만들어야 할지에 대한 고민을 많이 했습니다. 초반에 로직을 구현하다 보면 생성과 수정 요청은 거의 비슷한 형식의 데이터, 검증 로직을 필요로 하게 됩니다. 때문에 이를 통합하여 중복 코드를 제거할지, 기능에 따라 분리할지에 대한 고민을 많이 했습니다.

```java
public class CreateRequest {
    private String name;
    private LocalDate startDate;
    private LocalDate endDate;
    ...
}

public class UpdateRequest {
    private String name;
    private LocalDate startDate;
    private LocalDate endDate;
    ...
}
```

<클린 아키텍처> 책에서 이에 대한 좋은 내용을 언급하는데, 그 내용은 중복에도 종류가 있다는 것이다.

1. 진짜 중복
    - 한 인스턴스가 변경되면, 동일한 변경을 그 인스턴스의 모든 복사본에 반드시 적용해야한다.
2. 우발적 중복(거짓된 중복)
    - 중복으로 보이는 두 코드의 영역이 각자의 경로로 발전한다면, 즉 서로 다른 속도와 다른 이유로 변경된다면 이 두 코드는 진짜 중복이 아니다.

저는 저장과 수정 사이의 DTO의 중복은 우발적 중복에 속한다고 생각합니다. 초반에는 코드가 비슷할지 모르지만 각 기능은 서로 다른 이유와 속도로 변경될 가능성이 높기 때문입니다.

간단한 예로 스케줄을 저장해야 하는 기능을 개발한다고 생각해봅시다. 스케줄을 처음 저장할 때는 시작 일자가 과거 시간이 될 필요가 없기 때문에 이에 대한 유효성 검사를 진행합니다.

```java
public class CreateRequest {
    public void assertValidation() {
        if(startDate.isBefore(LocalDate.now())) {
            throw new IllegalArgumentException();
        }
    }
}
```

하지만 수정을 하는 경우에는 이미 해당 데이터가 과거 날짜일 수 있기 때문에 StartDate에 대한 유효성 검사가 필요 없습니다.

```java
public class UpdateRequest {
    public void assertValidation() {
        if(startDate.isBefore(LocalDate.now())) { ---> 불필요한 체크
            throw new IllegalArgumentException();
        }
    }   
}
```

**즉 이렇게 서로 다른 이유와 속도로 변경이 될 수 있는 상황에서는 DTO를 분리하는 것이 좋다고 생각합니다.**

### Entity가 DTO를 직접 참조하는 문제

가끔씩 코드를 보면 Entity를 생성할 때 Controller로 부터 생성된 Request(DTO) 객체를 이용하는 경우를 보았습니다.

```java
public class Entity {
    ...
    public static Entity of(Request request) {
        ...
    }
}
```

이러한 코드를 보고 Web으로 오는 데이터를 직접 참조하지 않고 DTO를 통해 다른 경계로 전달하기 때문에 규칙을 지켰다고 생각할 수 있습니다. 하지만 제가 생각하기에 의존성 규칙의 목적은 저수준의 개념이 고수준의 개념에 영향을 주지 않게 하기 위함이라고 생각합니다.

위의 코드와 같이 Entity가 직접 DTO를 참조하게 된다면 DTO의 변경에 의해 Entity도 변경될 수 있다는것을 알려줍니다. 따라서 Entity를 생성하는 UseCase 계층에서 DTO의 데이터를 꺼내서 Entity를 생성하는것이 DTO의 변경에 의해 Entity가 변경할 가능성을 낮출수 있다고 생각합니다.