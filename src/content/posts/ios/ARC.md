---
title: ARC, 근데 이제 Closure를 곁들인...
published: 2025-10-16
tags:
  - Swift
toc: false
lang: ko
abbrlink : arc-with-closure
---

# ARC(Automatic Reference Counting)

- Swift에서 메모리를 관리하기 위해 사용되는 메커니즘
- 참조 카운트(RC)를 자동으로 추적하고 관리하며, **더이상 사용되지 않는 인스턴스를 메모리에서 해제함**

:::note[RC란?]
**Reference Count**의 약자로, 직역하자면 **참조 수!**  
특정 클래스 인스턴스를 **강하게 참조하고 있는 수**를 뜻합니다.
  
*참조한다는 것은 인스턴스를 가리키고 있다는 의미로,  
인스턴스의 메모리 주소를 저장함으로써 접근이 가능한 상태를 말합니다.*
  
##### RC가 증가하는 경우

- 인스턴스를 상수/변수에 할당
- 함수 파라미터로 전달
- 컬렉션에 저장
- 클로저에서 캡처

##### RC가 0이 되면?

참조하는 곳이 없다는 의미로, ARC가 해당 인스턴스를 메모리에서 해제합니다.
:::

## ARC는 어떻게 동작할까?

- 클래스의 인스턴스에 대한 참조만 취급하며, 인스턴스가 생성되면 인스턴스 관련 정보를 저장할 메모리 공간(Heap)에 저장함
- RC가 0이 되면 인스턴스를 해제함

> 값타입은 참조를 통해 저장/전달되지 않으므로 참조 대상이 아닙니다!

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "John Appleseed")
// "John Appleseed is being initialized"
// Person 인스턴스 생성과 동시에, reference1이 강하게 참조

reference2 = reference1
reference3 = reference1
// Person 인스턴스의 RC = 3
// 같은 Person 인스턴스를 reference1, reference2, reference3이 모두 참조

reference1 = nil
reference2 = nil
// Person 인스턴스의 RC = 1
// 두 개의 강한 참조가 해제되었으나, 아직 reference3이 인스턴스를 참조하고 있으므로 메모리에서 해제되지 않음

reference3 = nil
// "John Appleseed is being deinitialized"
// Person 인스턴스의 RC = 0
// 마지막으로 남은 강한 참조까지 해제됨으로써 RC가 0이 되었고, ARC에 의해 Person 인스턴스가 메모리에서 해제됨
```

## Strong Reference Cycle(강한 참조 순환)

- 두 개 이상의 클래스가 **서로를 강하게 참조**하여, 참조 카운트가 0이 되지 않는 상태  
  (= 서로의 인스턴스를 프로퍼티에 저장함)
- 강한 참조 순환은 메모리 누수 현상의 원인이 될 수 있음
  - 강한 참조 순환으로 인해 **참조 카운트가 0에 도달하지 못해** ARC가 메모리를 해제하지 못하므로 메모리 누수가 발생함

```swift
class Person {
    let name: String
    var apartment: Apartment?

    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    var tenant: Person?

    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")
// Person 인스턴스 RC = 1
// Apartment 인스턴스 RC = 1

john?.apartment = unit4A
unit4A?.tenant = john
// Person 인스턴스 RC = 2
// Apartment 인스턴스 RC = 2
// 💥 서로의 인스턴스를 각 인스턴스 프로퍼티에서 참조함으로써, 강한 참조 순환 발생

john = nil
unit4A = nil
// Person 인스턴스 RC = 1
// Apartment 인스턴스 RC = 1
// 두 객체 모두 deinit이 호출되지 않음!
// 아직 두 인스턴스가 서로를 참조함으로써 메모리에서 해제되지 않음
// 💥 남은 참조를 해제할 방법이 없으므로, 메모리 누수 발생
```

### 해결 방법

*RC를 증가시키지 않는 참조 방법을 통해 해결!*
  
1. **Weak Reference** : 참조하고 있는 인스턴스의 수명이 더 짧은 경우 사용
2. **Unowned Reference** : 참조하고 있는 인스턴스와 수명이 비슷하거나 더 긴 경우 사용
  
💡*두 참조는 인스턴스를 참조할 때 강하게 참조하지 않아 RC를 증가시키지 않음*

#### Weak Reference(약한 참조)
  
- 참조 카운트를 증가시키지 않으며, 참조 중인 인스턴스가 해제되면 **ARC가 자동으로 nil을 할당함**
  - ARC가 nil을 할당할 수 있도록 **변수(var)이자 옵셔널 타입**이어야 함

```swift
class Person {
    let name: String
    var apartment: Apartment?

    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    weak var tenant: Person? // 변수이자, 옵셔널 타입

    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")
john?.apartment = unit4A
unit4A?.tenant = john
// 약한 참조로 선언했기 때문에 Person 인스턴스 RC가 증가하지 않음
// Person 인스턴스 RC = 1
// Apartment 인스턴스 RC = 2

john = nil
// "John Appleseed is being deinitialized"
// Person 인스턴스 RC = 0
// john에서의 강한 참조가 1개 해제되고 같은 인스턴스를 참조하는 unit4A.tenant는 약한 참조를 하고 있기 때문에 ARC가 unit4A.tenant에 nil을 할당함으로써, Person 인스턴스는 해제됨
// 그리고 Person 인스턴스가 해제되어, Apartment 인스턴스를 참조하는건 unit4A 밖에 없음

unit4A = nil
// "Apartment 4A is being deinitialized"
// Apartment 인스턴스 RC = 0
// 마지막 강한 참조가 해제됨으로써, Apartment 인스턴스도 해제됨
```

##### 정리하자면...

1. 약한 참조로 선언했기 때문에 RC가 증가하지 않았고,
2. `john = nil`에서 Person 인스턴스가 메모리에서 해제되고
3. ARC에 의해 `unit4A.tenant` 프로퍼티에는 자동으로 nil이 할당됨

#### Unowned Reference(미소유 참조)

- **항상 값이 존재함을 보장**하기 때문에, 인스턴스가 해제되어도 ARC가 nil을 할당하지 않으며 **참조하던 인스턴스의 메모리 주소를 계속 갖고 있음**
  - 해제되지 않은 인스턴스를 참조함을 보장해야 하므로, 먼저 해제될 일이 없는 인스턴스를 참조해야 함
  - 약한 참조와 달리 ARC가 nil을 할당하지 않으므로, 옵셔널 타입이 아님

```swift
class Customer {
    let name: String
    var card: CreditCard?

    init(name: String) {
        self.name = name
    }

    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer

    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }

    deinit { print("Card #\(number) is being deinitialized") }
}

var eddy: Customer? = Customer(name: "John Appleseed")
eddy!.card = CreditCard(number: 1234_5678_9012_3456, customer: eddy!)
// Credit Card에서 Customer를 미소유 참조하기 때문에 RC가 증가하지 않음
// Customer 인스턴스 RC = 1
// CreditCard 인스턴스 RC = 1

eddy = nil
// "John Appleseed is being deinitialized"
// "Card #1234567890123456 is being deinitialized"
// Customer 인스턴스 RC = 0
// CreditCard 인스턴스 RC = 0

// Customer 인스턴스의 강한 참조가 해제되고 RC가 0이 됨으로써, Customer 인스턴스가 해제됨
// Credit Card 인스턴스는 eddy.card에서만 참조했기 때문에 RC가 0이 됨으로써, 함께 해제됨
```

#### 💥 참조하고 있는 인스턴스가 이미 해제됐음에도 호출하면?

**이미 해제된 메모리 주소를 가리키기 때문에 "런타임 에러" 발생함!**
  
```swift
var eddy: Customer? = Customer(name: "John")
var card: CreditCard? = CreditCard(number: 1234_5678, customer: eddy!)
// CreditCard 인스턴스 RC = 1
// Customer 인스턴스 RC = 1

eddy = nil
// "John is being deinitialized"
// CreditCard 인스턴스 RC = 1 (card 변수가 참조)
// Customer 인스턴스 RC = 0
// ⭐️ Customer 인스턴스가 메모리에서 해제됨

print(card!.customer.name)  // 💥 런타임 에러 발생!
```

#### Unowned Optional Reference

- **nil이 될 가능성이 있으면서 unowned의 특성이 필요한 경우** 사용함
  - 미소유 참조이므로 해제된 인스턴스를 참조하는 상태에서 접근하면 마찬가지로 컴파일 에러 발생
- 클래스를 Optional로 감싼 형태(예: `Optional<Customer>`)이기 때문에 RC를 증가시키지 않음
  - ARC 대상도 아님

:::fold[weak를 사용하면 되지 않아요?]
인스턴스가 해제될 때 ARC가 자동으로 nil을 할당하는 것을 원하지 않고, 개발자가 직접 nil 할당 시점을 제어하고 싶거나 명시적으로 생명주기를 관리하고 싶을 때 사용합니다.  
  
그런데 말입니다... 사실 저는 실무에서 사용한 적이 없었어요!  
그래서 그냥 "이런 개념도 있구나~" 정도로 생각하셔도 됩니다😅
:::

---

:::note[왜 Closure를 같이 설명해요?]
실무에서 강한 참조 순환이 가장 자주 발생하는 곳은 바로 **Closure**이기 때문에, ARC와 함께 다뤄 이해도를 높이고자 합니다!
:::

# Closure

- 주변 context에서 정의된 값(상수, 변수)를 **캡처하여 저장할 수 있는** 코드 블록
- Swift에서는 함수가 일급 객체이므로, **클로저를 값처럼 전달하거나 변수에 저장할 수 있음**
  - 함수 = 이름 있는 클로저
  - 클로저 = 이름 없는 함수

## Capturing Values

- 클로저가 **주변 context에 정의된 값을 독립된 저장 공간에 캡처**해서 사용하는 것
  - 값타입을 캡처하는 경우 : 복사본을 저장
  - 참조타입을 캡처하는 경우 : 참조를 저장

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0

    func incrementer() -> Int {
        // runningTotal와 amount 캡처
        runningTotal += amount
        return runningTotal
    }

    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)
incrementByTen() // 10
incrementByTen() // 20
incrementByTen() // 30
// 클로저를 계속 호출할 수 있고 클로저가 캡처한 값도 유지됨

let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven() // 7
// incrementByTen과 값이 공유되지 않음
// ⭐️ 각 클로저는 독립된 캡처 값을 가짐
```

## Capture List

- 클로저에서 한 개 이상의 값을 캡처할 때 명시적으로 선언하는 리스트
- 참조 타입을 캡처할 때, **참조 방식(weak/unowned)을 선언**할 수 있음
  - 참조 타입을 캡처하면 기본적으로 강하게 참조함

```swift
// self와 self.delegate 캡처
{ [unowned self, weak delegate = self.delegate] in
    // ...
}
```

## 클로저는 참조 타입

- 클래스처럼 메모리 주소를 전달함으로써 참조함
  - 즉, 같은 메모리 주소를 가리킴!

```swift
let alsoIncrementByTen = incrementByTen 
alsoIncrementByTen()    // 50
incrementByTen()        // 60
// 같은 클로저를 참조함 = 같은 캡처값을 사용함
```

## Escaping Closure

- 함수의 인자로 전달된 클로저가 함수가 종료된 후에도 실행될 수 있는 클로저
  - 함수 내에서 즉시 실행되면 *Non-Escaping Closure*

```swift
var completionHandlers: [() -> Void] = []

func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) { 
    completionHandlers.append(completionHandler)
    // 클로저를 프로퍼티에 저장함으로써 나중에 호출될 수 있음
}
```

### Retain Cycle in Escaping Closure

- 클래스 인스턴스가 escaping closure를 소유하고, 그 클로저가 `self`를 캡처하여 발생하는 강한 참조 순환
  - Swift 컴파일러는 이 위험을 명확하게 인지시키기 위해 escaping closure 내에서 `self.` 를 명시적으로 작성하도록 강제함

```swift
class NetworkManager {
    var onFinishRequest: ((String) -> Void)?
    
    func makeRequest(completionHandler: @escaping (String) -> Void) {
        onFinishRequest = completionHandler // Retain Cycle 발생!
        // ...
        onFinishRequest?("some data")
    }
}

class ViewController {
    let networkManager = NetworkManager()
    var data: String = ""
    
    func fetchData() {
        networkManager.makeRequest { result in
            self.data = result
        }
    }
}

// Retain Cycle 흐름
// ViewController -> NetworkManager -> Escaping Closure(onFinishRequest) -> self(ViewController)
```

#### Retain Cycle 해결 방법

- Escaping Closure에서 Retain Cycle을 방지하려면, `[weak self]` 또는 `[unowned self]`와 같은 **캡처 리스트**를 사용해야 함
- **weak** : 어디에서든 안전하게 사용 가능 (대부분 사용)
- **unowned** : 이미 해제된 인스턴스에 접근하는 경우 크래시 발생

### 값타입 캡처 in Escaping Closure

- 값타입은 복사본을 캡처함

- 그러나 값타입 내부에서 escaping closure가 self를 변경하기 위한 참조는 허용되지 않음 (= 컴파일 에러 발생)

```swift
struct SomeStruct { 
    var x = 10
           
    mutating func doSomething() { 
        someFunctionWithNonescapingClosure { x = 200 } // Ok ✅
        someFunctionWithEscapingClosure { x = 100 } // Error 💥
    }
 }
```