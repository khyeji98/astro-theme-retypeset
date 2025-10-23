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

- 비동기 함수 호출시, `await` 키워드를 명시함으로써 해당 함수의 결과가 반환될 때까지 **일시 중단**(suspension point)하는 것을 표현함
  - 프로그램 전체가 멈추는 것이 아니며, 다른 비동기 작업이 동시에 실행될 수 있음
- `async let` 구문을 통해 여러 비동기 함수를 **병렬로 실행**할 수도 있음
  - 실제로 여러 태스크를 동시에 스케줄링하며, 결과는 `await` 시점에 한번에 병합되어 값에 접근할 수 있음

## Task와 TaskGroup

- **Task** : 비동기 컨텍스트 내에서 독립적으로 실행되는 작업 단위
  - 새로운 비동기 작업을 스폰(spawn)하며, 현재 스코프의 취소나 우선순위를 상속받을 수 있음
- **TaskGroup** : 여러 개의 자식 태스크를 그룹화하여 결과를 모으거나 관리할 수 있는 구조 (= Structured Concurrency의 핵심)
  - 모든 하위 태스크는 부모 태스크가 종료될 때 자동으로 취소됨

## Structured Concurrency

- 기존 GCD 기반 코드는 비구조적이었기 때문에, 어느 시점에 어떤 비동기 작업이 끝나는지를 명시적으로 추적해야 했음
- Swift의 Structured Concurrency는
  - 태스크의 수명이 스코프에 의해 제어되고
  - 부모-자식 관계로 구성되어
  - 프로그램의 동시성 흐름이 명확하게 구조화된다.
- 이를 통해 리소스 누수나 race condition 가능성이 크게 줄어듬

## Cancellation

- 모든 Task는 cancellation을 전달받을 수 있음
  - `Task.isCancelled`을 통해 확인하거나, `try Task.checkCancellation()` 에서 에러를 던짐으로써 알 수 있음
- 태스크 스스로 취소 상태를 체크해야 실제로 중단됨

## Actor

- 상태값들을 격리(isolated)하여 안전하게 공유 자원을 관리하기 위한 타입
- 클래스처럼 참조 타입이지만, 외부에서 동시에 접근할 수 없도록 Swift 런타임이 보호함
  - 내부 프로퍼티나 메서드에 접근하려면 `await` 키워드를 통해 격리를 유지함 (Actor 내부에서 self 속성에 접근할 때에는 필요없음)

## Global Actor

- 전역 단일 실행 컨텍스트를 정의함
- 대표적으로 `@MainActor`
  - UI 업데이트와 같이 메인 스레드에서만 실행해야 하는 작업을 지정할 때 사용함
  - `@MainActor`로 정의한 함수는 항상 메인 스레드에서 실행됨
  - SwiftUI, UIKit 등 UI 관련 API와 안전하게 연동할 수 있음

## Sendable

- 동시성 환경에서는 데이터를 여러 Task 간에 안전하게 전달할 필요가 있는데, 이때 Sendable 프로토콜을 통해 안전한 전달을 보장함
  - Sendable을 채택한 구조체의 경우, 여러 스레드 간에 안전하게 복사될 수 있음을 보장함
  - 클래스의 경우, `@unchecked Sendable`을 수동으로 명시해야 할 수도 있음

## Nonisolated 및 Actor Reentrancy

- `nonisolated`를 통해 특정 메서드나 프로퍼티를 격리에서 제외할 수 있음
  - Actor 내부에서 사용하며, 타입 자체는 Actor의 격리를 유지하면서도 비동기 컨텍스트 없이 접근 가능한 예외를 허용함
- Actor는 기본적으로 재진입성(Reentrant)
  - 재진입성 : `await` 로 일시 중단된 사이에 다른 Task가 Actor의 메서드를 호출할 수 있음
  - 이를 통해 deadlock을 피할 수 있지만, 공유 상태를 변경하는 로직에서는 주의해야 함

## Unstructured Concurrency

- `Task`를 명시적으로 생성해 structured context 밖에서 실행하는 것도 가능함
  - 이때 부모-자식 관계가 없기 때문에 수명관리와 취소 처리를 직접 해야함
  - SwiftUI나 async main context에서 독립적인 작업을 수행할 때 사용됨

## Async Sequences

- 비동기 데이터 스트림을 처리하기 위해 Swift에서 제공하는 프로토콜
  - `for await` 구문을 통해 순차적으로 비동기 요소를 처리함
  - Combine의 Publisher와 유사한 개념

## 정리하자면, 

##### ☝🏻 Swift Concurrency에서 중요한 원칙 세 가지!

1. **Structured Concurrency** : 동시 작업의 생명주기를 명확하게 제어
2. **Actor isolation** : 공유 상태의 안전성 보장
3. **Async/await syntax** : 비동기 코드이 가독성과 명확성 향상

이 시스템을 통해 Swift는 **안전성, 예측 가능성, 가독성**을 모두 확보함

---

- [Apple Inc. (2023). The Swift Programming Language (Swift 5.9): Concurrency.](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency)