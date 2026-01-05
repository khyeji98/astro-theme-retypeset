---
title: Swift Concurrency
published: 2025-10-23
tags:
  - Swift
  - Concurrency
toc: false
lang: ko
abbrlink : concurrency
---

:::fold[Intro]
### 동시성(Concurrency)의 본질

- 여러 작업이 동시에 수행될 수 있는 구조나 특성

동시성이 나오면 "비동기, 동기, 병렬" 이런 키워드들이 나와 헷갈렸는데, 딱 대놓고 보자면 동시성은 **논리적 동시성**입니다.

여러 작업이 동시에 수행되는데에는 물리적으로 동시에 수행하는 방법(병렬)도 있고 작업 간에 왔다갔다 하면서 CPU 자원을 효율적으로 사용해서 마치 여러 작업이 동시에 수행되는 것처럼 보이도록 하는 방법(비동기)도 있는데 어떤 방법을 사용하든 여러 작업이 동시에 수행되는 것처럼 보이면 **동시성**이라는 거죠.

그래서 동시성에 대해 설명할 때 자꾸 비동기, 동기, 벙렬성 키워드가 언급되는 것입니다.

##### 그렇다면 Swift Concurrency는?

Swift Concurrency는 동시성을 구현하는 방식 중, 비동기 모델을 기반으로 한 대신 구조적이면서 안전하게 표현한 언어 차원의 시스템입니다.

🍎 **언어 레벨에서 제공하는 안전하고 구조적인 비동기식 동시성 시스템** 인거죠.

기존에 비동기 작업은 개발자들에게 꽤 골칫거리였기 때문에 고맙죠. 스레드 직접 관리하랴, 공유 자원 접근 제어하랴, 코드 복잡해지고...
:::

# Swift Concurrency

- 여러 작업이 동시에 수행되는 것처럼 설계된 비동기식 동시성 시스템
- 비동기 작업을 안전하게 구현할 수 있고, 읽기 쉽게 설계됨
  - 옛날 비동기 처리 방식 : completion handler, GCD
  - Swift Concurrency 비동기 처리 방식 : async/await, Task, Actor 등
- 특히 비동기 함수로 인해 호출부의 작업 실행이 일시 중단되면 스레드가 비동기 함수를 처리하는 작업 실행으로 넘어가고 완료되면 다시 재개하기 위해 넘어오는 과정을 개발자가 직접 제어하지 않아도, **구조적**으로 처리해주는 것이 특징 (= **Structured Concurrency**)
  - 구조적으로 처리한다 = Swift Concurrency의 비동기 코드 자체가 비동기 실행 구조를 나타낸다

```swift
// 옛날 비동기 처리 : 중첩 클로저, 에러 throw 불편, 함수가 종료돼도 async 코드블럭은 계속 실행될 수 있음
func loadData() {
    DispatchQueue.global().async {
        let data = fetchRemoteData()
        DispatchQueue.main.async {
            self.updateUI(with: data)
        }
    }
}

// Swift Concurrency : 코드 구조가 실행 구조와 같음(읽기 편함), try로 에러 throw, 부모-자식 태스크 구조(부모가 취소되면 자식도 취소됨)
func loadData() async throws -> Data {
    async let user = fetchUser()      // 자식 Task
    async let posts = fetchPosts()    // 자식 Task
    return try await process(user, posts)
}
```

## Async / Await

- `async` 키워드를 통해 비동기 함수를 정의할 수 있고, 호출할 땐 `await` 키워드를 명시함
  - `await` 키워드는 **결과가 반환될 때까지 일시 중단(suspension point)하는 것**을 의미함
  - 현재 Task가 suspend 되어 스레드가 할 일이 없어지면 다른 Task를 실행할 수 있고, 기존 Task가 실행 완료되어도 실행중인 다른 Task가 완료되거나 suspend 되어야만 다시 기존 Task를 처리할 수 있음
- `async let` 구문을 통해 여러 비동기 함수를 **병렬로 실행**할 수도 있음
  - 실제로 별도의 Task로 스케줄링 되어 동시에 실행되며, `async let` 구문의 호출값을 모두 받을 때까지 기다렸다가 한번에 병합해서 반환함

```swift
func loadAll() async {
    async let user = fetchUser()   // Task A 생성
    async let post = fetchPost()   // Task B 생성

    // 두 결과가 모두 완료될 때까지 loadAll() suspend
    let (u, p) = await (user, post)
}
```

## Task와 TaskGroup

- **Task** : **비동기 실행 단위**
  - 비동기 작업이 생성되면 Swift Concurrency 런타임이 Executor(실행 큐)에 등록하고, 시스템은 CPU 코어나 스레드 상태에 따라 실행 큐에서 대기중인 비동기 작업들을 실행함
  - Swift Concurrency Executor: 비동기 작업만을 다루는 큐
- **TaskGroup** : **여러 개의 자식 태스크를 묶어 별렬 실행하고, 결과를 모아 관리할 수 있는 구조**
  - 부모-자식 관계의 구조에 따라 우선순위를 상속받고 작업 취소를 수직적으로 전달받음으로써, 대표적인 Structured Concurrency를 구현함

### Cancellation

- 모든 Task는 취소 상태(Cancellation)를 전달받을 수 있음
  - `cancel()` 호출만으로 Task가 취소되지 않으며, `Task.isCancelled`을 통해 확인하거나 `try Task.checkCancellation()` 에서 에러를 던짐으로써 취소 상태를 알고 Task를 종료할 수 있음

## Structured Concurrency

- **비동기 작업의 수명과 실행 흐름을 코드 구조에 따라 안전하게 관리하는 방식**으로, Swift Concurrency의 핵심 개념
  
1. 태스크의 수명이 스코프에 의해 제어됨
    - 스코프(scope) : `{...}`
2. 부모-자식 관계로 구성되어 동시성 흐름이 명확하게 구조화됨
    - 부모의 우선순위를 상속받음
    - 부모 작업이 취소되면, 자식 작업도 취소됨

### 장점

- **리소스 누수나 race condition 가능성이 크게 줄어듦으로써, 기존의 비구조적인 GCD 기반 비동기 처리의 문제를 해결할 수 있음**
  - 리소스 누수 : 태스크 수명 관리가 구조적이지 않아 끝났어야 하는 작업이 끝나지 않음으로써 자원을 계속 점유하는 상태
  - **race condition** : 여러 스레드가 동일한 자원에 동시에 접근해 값을 수정함으로써 실행 순서에 따라 결과가 달라지는 현상

## Actor

- 내부 상태를 격리(**isolated**)하고 Serial Executor 위에서 실행됨으로써 여러 스레드가 동시에 접근할 수 없음으로써, **thread-safe를 보장하는 타입**
  - **Serial Executor** : 직렬 실행 큐로, 한 번에 하나의 Task만 실행함
- 클래스처럼 참조 타입이지만, 내부 프로퍼티나 메서드에 접근하려면 `await` 키워드를 명시해야 함
  - Actor 내부에서 self 속성에 접근할 때에는 필요없음

### Reentrancy(재진입성)

- **Actor에서 실행중인 비동기 작업이 suspend 되면, 같은 Actor의 Executor에 대기중이던 다른 비동기 작업이 실행되는 것**
- reentrancy를 통해 deadlock을 피할 수 있지만, 공유 상태를 변경하는 로직에서는 주의할 필요가 있음
  - **Deadlock 회피** : Actor의 비동기 작업이 suspend 됐을 때 다른 비동기 작업이 진입할 수 없으면 Actor의 실행 흐름은 suspend된 비동기 작업이 완료될 때까지 잠긴 상태를 유지함으로써 Deadlock이 발생할 수 있어, **재진입성을 통해 Actor의 실행 흐름이 멈추는 것을 방지할 수 있음**
    - Deadlock(교착상태) : 여러 작업이 동일한 리소스 점유를 하기 위해 서로 기다리는 무한 대기 현상
  - **공유 상태 불일치 위험** : suspend 된 비동기 작업과 suspend 됨으로써 실행되는 비동기 작업이 같은 상태를 공유하게 되면 각 작업에서 얻는 상태 값이 달라 **상태 일관성을 보장할 수 없으므로, 공유 상태를 변경하는 작업들이 같은 실행 흐름에서 실행될 여지가 있는지 주의해야 함**

> "**재진입성**"이라는 키워드는, Actor의 실행이 완전히 끝나지 않았는데 **같은 Actor의 다른 실행이 재진입한다**는 의미라고 생각할 수 있습니다!

### GlobalActor

- 매크로를 통해 특정 Actor의 Serial Executor에서 실행됨을 보장할 수 있는 Actor로, **전역 단일 실행 컨텍스트를 정의한다**고 정의하기도 함
  - 대표적으로 `@MainActor`가 있는데 Serial Executor를 통해 항상 메인 스레드에서만 실행됨을 보장하는 Actor로, UI 업데이트와 같은 작업에 사용함

#### CustomActor와 차이점

- CustomActor : 직접 인스턴스를 생성해야 함
- GlobalActor : **인스턴스 생성없이, 매크로를 통해 앱 전역에서 해당 Actor의 실행 흐름에 작업을 등록할 수 있음**

```swift
// GlobalActor 정의
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

// GlobalActor 실행 흐름에 작업 등록
// 두 함수 모두 같은 Serial Executor 위에서 순차 실행됨
@DatabaseActor
func loadUserData() { ... }
@DatabaseActor
func saveUserData() { ... }
```

### Nonisolated

- `nonisolated`를 통해 특정 메서드나 프로퍼티를 격리에서 제외할 수 있음
  - Actor 내부에서 사용하며, 타입 자체는 Actor의 격리를 유지하면서도 비동기 컨텍스트 없이 접근 가능한 예외를 허용함

## Sendable

- 비동기 작업 간에 전달될 때 안전하게 전달될 수 있도록 보장하는 프로토콜
  - **비동기 작업에서 값이 안전함을 보장할 수 있으나, actor처럼 내부 동작의 안전한 실행을 보장하진 않음**
- actor의 경우 당연히 Sendable이고, struct/enum과 같은 값 타입의 경우 내부 모든 저장 프로퍼티가 Sendable이면 자동 Sendable
  - 컴파일러가 Sendable 여부를 판단할 수 없는 경우 직접 Sendable 채택을 명시해야 함
  - 클로저가 비동기로 실행됨으로써 안전성을 보장해야 한다면 `@Sendable`을 명시해야 함  
- 검증 우회(bypass) : 컴파일러가 클래스가 비동기 작업에서 안전한지 증명할 수 없는 경우 경고를 띄우는데, 이때 개발자가 직접 안전함을 보장한다는 의미로 `@unchecked Sendable`을 명시할 수 있음 (= 개발자가 수동 보장 선언)

## Unstructured Concurrency

- `Task {...}`를 명시적으로 생성해 structured context 밖에서 실행하는 것도 가능함
  - 이렇게 생성된 비동기 작업은 부모-자식 관계에 해당되지 않기 때문에 **수명관리와 취소 처리를 직접 해야함**

## AsyncSequences

- **시간에 따라 비동기적으로 생성되는 값들을 순차적으로 비동기 처리할 수 있는 프로토콜**
  - 일반적인 Sequence는 이미 메모리에 존재함으로써 즉시 순회할 수 있으나, **AsyncSequence는 아직 존재하지 않는 값이 시간이 지남으로써 하나씩 생성됨**으로써 생성될 때마다 `await` 를 통해 suspend 되면서 순차적으로 값을 처리하는 구조
  - Combine의 Publisher처럼 비동기적으로 이벤트를 전달하는 스트림을 다루지만, **Combine은 이벤트 발생 즉시 구독자에게 값을 push하는 구조이고 AsyncSequence는 다음 값을 요청할 때까지 suspend 됐다가 호출함으로써 값을 pull하는 구조로 다름**

---

:::fold[면접 대비]
##### "🙋‍♂️ Swift Concurrency에 대해 설명해주세요!"

Swift Concurrency는 여러 작업이 동시에 수행되는 것처럼 설계된 비동기 기반의 동시성 시스템으로, 비동기 코드의 수명과 실행 흐름을 구조적으로 제어할 수 있습니다.  
  
비동기 함수를 정의하고 호출함으로써 코드 흐름에 따라 비동기 작업의 구조와 흐름을 명확하게 알 수 있고, `async` 키워드를 통해 정의하며 `await` 키워드를 통해 호출합니다.  
`await` 키워드는 비동기 작업이 완료될 때까지 현재 Task를 일시 중단함을 나타내며, 작업이 완료되면 해당 suspended point부터 작업을 재개합니다.  
  
모든 비동기 작업은 Task 단위로 실행되고, 각 Task는 비동기 작업을 다루는 executor에서 대기하고 스케줄러에 의해 실행됩니다.  
비동기 함수 또한 하나의 Task로, 스코프 내에서 실행되는 또다른 비동기 작업은 자식 작업으로 부모-자식 관계의 구조적인 비동기 처리를 할 수 있습니다.  
  
이렇게 부모-자식 관계의 구조적인 비동기 처리를 Structured Concurrency라고 하며, Task의 수명과 실행 흐름을 코드 구조에 따라 표현하고 제어하는 모델입니다.  
Swift Concurrency는 구조적인 비동기 처리를 통해 기존 GCD 기반 비동기 처리에서 발생하던 리소스 누수나 race condition과 같은 문제를 해결할 수 있습니다.  
race condition은 여러 스레드에서 같은 자원에 동시에 접근함으로써 실행 순서에 따라 다른 값을 받게 되는 현상을 말하는데, Actor를 통해 해결할 수 있습니다.  
  
Actor는 isolation을 통해 내부 작업을 격리하고 serial executor 위에서 실행됨으로써, 여러 작업이 동시에 접근할 수 없도록 thread-safe를 보장하는 타입입니다.  
Actor는 CustomActor와 GlobalActor로 나눌 수 있는데, CustomActor는 클래스처럼 `actor` 키워드를 통해 Actor 타입으로 정의하고 인스턴스를 생성하고 GlobalActor는 매크로를 통해 앱 전역에서 해당 Actor의 실행흐름에 비동기 작업을 등록함으로써 인스턴스없이 접근할 수 있는 Actor입니다.   
GlobalActor의 대표적인 예로는 MainActor가 있습니다. MainActor는 작업이 항상 메인 스레드에서 실행됨을 보장하는 GlobalActor로, UI 업데이트와 같이 메인 스레드에서 동작해야 하는 작업에 `@MainActor` 키워드를 통해 메인 스레드에서의 비동기 작업을 등록할 수 있습니다.   
  
Actor는 클래스처럼 참조타입이기 때문에 클래스의 thread-safe 보장은 Actor로 정의함에 따라 가능하지만, 구조체와 같은 값타입의 경우 Sendable 프로토콜을 채택함을써 비동기 작업 간의 tread-safe를 보장할 수 있습니다.  
  
이렇듯 Swift Concurrency를 통해 **안전성, 예측 가능성, 가독성**을 모두 확보할 수 있습니다.  
:::

---

##### 참고 문헌

- [Apple Inc. (2023). The Swift Programming Language (Swift 5.9): Concurrency.](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency)