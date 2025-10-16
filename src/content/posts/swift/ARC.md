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

- Swift에서 메모리를 자동으로 관리하기 위해 사용되는 메커니즘, **자동 참조 카운팅**
  - 참조 카운트를 자동으로 추적하고 관리하며, 더이상 사용되지 않는 인스턴스를 자동으로 메모리에서 해제함
  - 참조 카운트(Reference Count, RC) : 어떤 클래스 인스턴스를 **강한 참조**로 몇 개가 참조하고 있는지를 나타내는 수
  - RC가 0이 되는 순간, ARC가 그 인스턴스를 해제(deinit)함
  - RC가 증가하는 순간
    - 인스턴스를 상수/변수에 할당
    - 함수 파라미터로 전달
    - 컬렉션에 저장
    - 클로저에서 캡처
- 클래스의 인스턴스에서만 동작함
  - 구조체나 열거형은 값타입이기 때문에 참조를 통해 저장되거나 전달되지 않고 복사되어 전달되므로, ARC 관리 대상이 아님

> "참조한다"는 **어떤 클래스 인스턴스를 가리키고 있는 것**를 의미하며, 메모리 주소를 알고 있어 인스턴스에 접근 가능합니다.

## How ARC Work?

- 클래스 인스턴스가 생성되면, ARC는 인스턴스 관련 정보를 저장할 메모리 공간을 힙에 할당함
- 해당 인스턴스가 더이상 사용되지 않을 때, 즉 RC가 0일 때 ARC는 메모리에서 인스턴스를 해제함

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
// Person 인스턴스가 생성되며, reference1이 해당 인스턴스를 강하게 참조(Strong Reference)

reference2 = reference1
reference3 = reference1
// 동일한 Person 인스턴스를 reference1, reference2, reference3이 모두 참조
// 👉 Person 인스턴스의 RC = 3

reference1 = nil
reference2 = nil
// reference1, reference2에 nil을 할당하면서 두 개의 강한 참조가 해제됨
// 그러나 reference3이 여전히 인스턴스를 참조하고 있으므로 메모리에서 해제되지 않음
// 👉 Person 인스턴스의 RC = 1

reference3 = nil
// Prints "John Appleseed is being deinitialized"
// 마지막 하나 남은 강한 참조까지 해제되면서 RC가 0이 되었고, ARC에 의해 Person 인스턴스가 메모리에서 해제됨
// 👉 Person 인스턴스의 RC = 0
```

## 강한 참조 순환(Strong Reference Cycle)

- 두 클래스 인스턴스가 서로를 강하게 참조하고 있고, 이로 인해 각 인스턴스가 서로를 존재하게 하는 것을 **Strong Reference Cycle**이라고 함
- 강한 참조 순환으로 인해 메모리 누수 현상이 발생하는 문제가 있을 수 있음
  - 아래 예시를 보면, 강한 참조 순환이 존재하는 상태로 두 인스턴스에 nil을 할당했을 때 강한 참조 순환이 남아 서로가 서로를 강하게 참조함으로써 메모리에서 해제되지 않아 메모리 누수가 발생함

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
// 👉 Person 인스턴스 RC = 1
// 👉 Apartment 인스턴스 RC = 1

john?.apartment = unit4A
unit4A?.tenant = john
// 👉 Person 인스턴스 RC = 2
// 👉 Apartment 인스턴스 RC = 2
// 문서에는 강제 언래핑(!)으로 프로퍼티에 접근하지만, 일반적으로는 안전하게 접근하기 위해 옵셔널 체이닝으로 접근함 (샘플 코드라서 값이 존재함을 보장하기 때문에 강제 언래핑을 사용하지 않았을까 유추)

john = nil
unit4A = nil
// 두 객체 모두 deinit이 호출되지 않음!
// 아직 두 인스턴스가 서로를 참조함으로써 메모리에서 해제되지 않음을 알 수 있음
// 👉 Person 인스턴스 RC = 1
// 👉 Apartment 인스턴스 RC = 1
```

![강한 참조 순환만 남음](https://docs.swift.org/swift-book/images/org.swift.tspl/referenceCycle03@2x.png)

### Strong Reference Cycle을 해결해보자

- Weak Reference와 Unowned Reference를 통해 해결할 수 있음
    - 두 참조는 인스턴스를 참조할 때 강하게 참조하지 않아 RC를 증가시키지 않음
    - **Weak Reference** : 참조하고 있는 인스턴스의 수명이 더 짧은 경우 사용
    - **Unowned Reference** : 참조하고 있는 인스턴스와 수명이 비슷하거나 더 긴 경우 사용

#### Weak Reference(약한 참조)

- 참조하고 있는 인스턴스가 메모리에서 해제되었을 때, ARC가 인스턴스를 참조하고 있었던 변수에 nil을 할당함으로써 참조를 해제함
- 그렇기 때문에 **1) 변수로 선언해야 하며 2)옵셔널 타입이어야 함**

```swift
class Person {
    let name: String
    var apartment: Apartment?

    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    weak var tenant: Person? // 💡 변수 + 옵셔널

    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john
// 약한 참조로 선언했기 때문에 Person 인스턴스 RC가 증가하지 않음
// 👉 Person 인스턴스 RC = 1
// 👉 Apartment 인스턴스 RC = 2

john = nil
// Prints "John Appleseed is being deinitialized"
// john에 nil을 할당했을 때, 같은 인스턴스를 참조하는 unit4A.tenant가 약한 참조를 하고 있기 때문에 ARC가 unit4A.tenant에 nil을 할당함
// 👉 Person 인스턴스 RC = 0 (Person 인스턴스가 메모리에서 해제됨)

// Person 인스턴스가 해제되면서 Apartment 인스턴스를 참조하는건 unit4A 밖에 없음
// 👉 Apartment 인스턴스 RC = 1

unit4A = nil
// Prints "Apartment 4A is being deinitialized"
// nil을 할당함으로써 마지막 남은 강한 참조가 사라지고, Apartment 인스턴스도 문제없이 메모리에서 해제될 수 있음
// 👉 Apartment 인스턴스 RC = 0
```

- 정리하자면, 약한 참조로 선언했기 때문에 RC가 증가하지 않아 `john = nil`에서 Person 인스턴스가 메모리에서 해제되고 ARC에 의해 `unit4A.tenant` 프로퍼티에는 자동으로 nil이 할당됨

![](https://docs.swift.org/swift-book/images/org.swift.tspl/weakReference01@2x.png)

#### Unowned Reference(미소유 참조)

- 약한 참조와 달리 **항상 값이 존재하는 것으로 가정**하기 때문에, 인스턴스가 해제되어도 ARC가 nil을 할당하지 않으며 **참조하던 인스턴스의 메모리 주소를 계속 갖고 있음**
  - 그렇기 때문에 먼저 해제될 일이 없는 인스턴스를 참조할 때 사용해야 함
  - 참조하고 있는 인스턴스가 먼저 해제됐을 때 해당 인스턴스에 접근할 시, 해제된 메모리 주소를 가리키기 때문에 **런타임 에러가 발생**함
  - ARC가 자동으로 nil을 할당하지 않으므로 옵셔널 타입으로 선언하지 않아도 됨

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
// 👉 Customer 인스턴스 RC = 1
// 👉 CreditCard 인스턴스 RC = 1

eddy = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
// 👉 Customer 인스턴스 RC = 0
// 👉 CreditCard 인스턴스 RC = 0

// eddy에 nil을 할당함으로써 Customer 인스턴스의 유일한 강한 참조가 해제되고 RC가 0이 됨으로써 Customer 인스턴스가 해제됨
// Credit Card 인스턴스는 eddy.card 외에 다른 곳에서 참조하고 있지 않기 때문에 Customer 인스턴스가 해제됨으로써 함께 해제됨
```

- 정리하자면, 미소유 참조로 선언했기 때문에 RC(참조 카운트)가 증가하지 않아 `eddy = nil` 에서 이미 RC가 0이기 때문에 Customer가 메모리에서 해제될 수 있음

![](https://docs.swift.org/swift-book/images/org.swift.tspl/unownedReference01@2x.png)

#### 💥 Runtime Error with Unowned Reference

```swift
var eddy: Customer? = Customer(name: "John")
var card: CreditCard? = CreditCard(number: 1234_5678, customer: eddy!)
// Credit Card 인스턴스를 변수에 별도로 저장
// 👉 Customer 인스턴스 RC = 1
// 👉 CreditCard 인스턴스 RC = 1

eddy = nil
// Prints "John is being deinitialized"
// 👉 Customer 인스턴스 RC = 0
// 👉 CreditCard 인스턴스 RC = 1 (Credit Card는 card에서 아직 참조하고 있기 때문에 RC가 감소하지 않음)

print(card!.customer.name)  // 💥 런타임 에러 발생!
// unowned로 참조한 customer는 이미 해제됐으나, 해제된 메모리 주소에 접근함
```

#### Unowned Optional Reference

- unowned reference는 "참조 대상은 절대 nil이 될 수 없다"를 보장할 수 있는 경우에만 사용해야하지만, nil이 될 수도 있어야 하는 경우 unowned optional reference를 사용함
  - 사실 "이럴거면 weak를 쓰지?" 싶지만, 인스턴스 해제로 인해 ARC가 자동으로 nil을 할당하는 것을 원하지 않는 경우 때때로 사용할 수 있음
  - ARC는 클래스 인스턴스만 추적하기 때문에 Optional로 감싸고 있는 클래스 인스턴스는 참조 카운팅하지 않음
- 정리하자면, 인스턴스를 참조할거면 먼저 해제될 일이 없어야 하지만, 참조할 인스턴스가 없을 수도 있음
- 대신 unowned로 선언했기 때문에 nil이 할당되지 않은 상태에서 이미 해제된 인스턴스에 접근하려고 하면 크래시가 발생하기 때문에, 사용자(개발자)가 직접 nil을 할당함으로써 관리할 책임이 있음

# Closure

- 주변 context에서 정의된 값(상수, 변수)를 캡처해서 저장할 수 있는 코드블럭
- Swift에서는 함수가 일급 객체이기 때문에 클로저를 변수에 할당하거나, 함수의 파라미터로 전달하거나, 반환값으로 전달할 수 있음
  - 함수 = 이름 있는 클로저
  - 클로저 = 이름 없는 함수

## Capturing Values

- 클로저는 주변 context에서 정의된 값을 캡처하고 저장할 수 있으며, 주변 context가 종료되어도 클로저는 캡처한 값을 계속 사용할 수 있음

```Swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0

    func incrementer() -> Int {
        // runningTotal, amount 모두 캡처!
        runningTotal += amount
        return runningTotal
    }

    // 클로저를 반환하기 때문에 makeIncrementer(forIncrement:)가 종료되어도 캡처한 값은 살아있고 값 수정도 가능함
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen() // returns a value of 10
incrementByTen() // returns a value of 20
incrementByTen() // returns a value of 30

// 반환된 클로저(incrementer)를 incrementByTen에 저장하고 계속 호출할 수 있음
// 클로저에서 캡처한 값을 수정함

let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven() // returns a value of 7
// 반환된 클로저를 incrementBySeven에 새로 저장한 것이기 때문에, 캡처한 값이 incrementByTen와 다른 저장 공간에 위치함
```

## Capture List

- 클로저 내부에서 한 개 이상의 참조 타입을 캡처할 때 어떻게 캡처할지 명시적으로 선언하는 리스트
- 캡처리스트를 통해 참조 방식(weak/owned)을 명시할 수 있음
  - 참조 방식을 명시하지 않으면 기본적으로 강한 참조

```swift
// 클로저에 파라미터가 없는 경우
lazy var someClosure = { [unowned self, weak delegate = self.delegate] in
    // closure body goes here
}

// 클로저에 파라미터가 있는 경우
lazy var someClosure = { [unowned self, weak delegate = self.delegate] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```

## 클로저는 참조 타입

```Swift
let alsoIncrementByTen = incrementByTen 
alsoIncrementByTen() // returns a value of 50
incrementByTen() // returns a value of 60
// alsoIncrementByTen()를 호출했을 때 incrementByTen()에서 캡처한 값을 수정함
// = 참조 타입이기 때문에 클로저를 다른 상수에 할당해도 같은 클로저를 가리킴
```

- 클로저에서 캡처하는 값이 참조 타입인 경우, 둘 다 참조 타입이기 때문에 캡처한 값에 대해 **강한 참조**가 생성됨 (클로저 -> 캡처한 값)
  - 만약 클로저와 클로저가 참조한 객체 인스턴스가 서로를 참조하게 되면 강한 참조 순환이 발생함
  - 특히 Escaping Closure의 경우 클로저가 함수를 탈출해서 동작할 수 있기 때문에 Retain cycle이 생길 위험이 있음

## Escaping Closure

- 함수의 파라미터로 전달된 클로저를 저장하거나, 반환하거나, 비동기로 실행함으로써 함수 스코프를 벗어나 나중에 호출될 수 있는 클로저
  - 함수의 파라미터로 전달돼도 함수 내에서 즉시 실행되면 Non-Escaping Closure

```Swift
var completionHandlers: [() -> Void] = []

func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) { 
    completionHandlers.append(completionHandler) // 클로저를 따로 저장함으로써 나중에 호출될 수 있음
}
```

### Retain Cycle in Escaping Closure

- Escaping Closure 내부에서 `self` 를 참조하고 self가 클로저를 참조하는 경우, 클로저가 self를 강하게 참조하기 때문에 강한 순환 참조가 발생할 수 있음
  - 이 위험을 명확하게 인지시키기 위해 Swift 컴파일러가 `self.` 를 명시하거나 `[weak self]` 와 같은 캡처리스트를 작성하도록 강제함

```Swift
class NetworkManager {
    var onFinishRequest: ((String) -> Void)?
    
    func makeRequest(completionHandler: @escaping (String) -> Void) {
        onFinishRequest = completionHandler // Escaping closure 저장 = 강한 순환 참조 발생!
        
        // ...작업 수행...
        
        self.finishRequest()
    }
    
    func finishRequest() {
        onFinishRequest?("some data")
    }
}

class ViewController {
    let networkManager = NetworkManager()
    var data: String = ""
    
    func fetchData() {
        networkManager.makeRequest { result in
            self.data = result // 컴파일러가 self 명시 요구
        }
    }
}

// ViewController -> networkManager -> onFinishRequest -> self(ViewController)
```

- 만약 이전 화면으로 돌아가거나 ViewController가 할당됐던 프로퍼티에 nil을 할당하는 등의 인스턴스 할당 값을 해제하는 동작을 거치게 되면, NetworkManager의 onFinishRequest에 아직 클로저가 저장되어 있기 때문에 강한 순환 참조가 발생하고 메모리 누수가 발생할 수 있음
  - 순환 참조를 해결하려면 직접 weak나 owned을 통해 self를 약하게 참조한다고 명시해야 함

```Swift
networkManager.makeRequest { [weak self] result in
    guard let self else { return }
    self.data = result
}
```

## Non-Escaping Closure

- Non-Escaping Closure에서도 강한 참조 순환은 발생함
- 그러나 함수 내에서 즉시 실행되고 종료되기 때문에, 정의된 함수가 종료될 때 자동으로 해제되어 영구적인 참조 순환을 만들 수 없음

## 값타입을 참조한다면?

- 값타입을 사용하는 것은 참조타입이 아니기 때문에, 애초에 참조를 하지 않음
- 대신 값타입의 복사본을 가져와서 사용함
- 그러나 값타입 내부에서 escaping closure가 self를 변경하기 위한 참조는 허용되지 않음 (= 컴파일 에러 발생)

```Swift
struct SomeStruct { 
    var x = 10
           
    mutating func doSomething() { 
        someFunctionWithNonescapingClosure { x = 200 } // Ok ✅
        someFunctionWithEscapingClosure { x = 100 } // Error 💥
    }
 }
```

## Strong Reference Cycle for Closure

- 클래스 인스턴스 프로퍼티가 클로저인 경우, 클로저 바디에서 인스턴스를 캡처하면 강한 참조 사이클이 발생함
  - 클로저 내부에서 `self.someProperty`나 `self.someMethod()`와 같이 인스턴스에 접근하기 위해 캡처가 발생함
  - 캡처(capture) : 클로저가 자신이 정의된 주변 컨텍스트에 선언된 변수나 상수를 **기억하고 저장**하는 것
  - 클로저는 정의된 범위 밖에서도 실행될 수 있기 때문에 내부에서 호출하고 사용하는 외부 값들을 내부에 "캡처"해서 보관함
  - 이때 강한 참조 사이클이라는 것은 서로가 강하게 참조하는 것을 의미하는데, 클래스 내부 프로퍼티가 클로저이기 때문에 `클래스가 클로저를 강하게 참조`하고, 클로저 내부에서 클래스 인스턴스에 접근하면서 `클로저가 클래스 인스턴스(self)를 강하게 참조(= 캡처)`함으로써 *강한 참조 사이클이 발생*

```swift
class HTMLElement {
    let name: String
    let text: String?

    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }
}

var element: HTMLElement? = HTMLElement(name: "p", text: "hello")
// element ──강한참조──> HTMLElement 인스턴스
// HTMLElement 인스턴스's RC : 1

print(element!.asHTML()) // "<p>hello</p>" 출력
// asHTML 클로저 실행(self 캡처) ──강한참조──> HTMLElement 인스턴스
// HTMLElement 인스턴스's RC : 2
// asHTML 클로저's RC : 1 (lazy로 선언되어 있어 호출시 생성)

/* < 현재 상황 >

    [변수 element] ──강한참조──> [HTMLElement 인스턴스] <──강한참조── [asHTML 클로저]
                              RC: 2                             (self 캡처)
                              │        *강한 참조 사이클 생성*
                              │
                              └──강한참조──> [asHTML 클로저]
                                           RC: 1
*/

element = nil
// element에서의 HTMLElement 인스턴스 참조가 끊김
// HTMLElement 인스턴스의 RC가 0이 아니므로 메모리 해제 안됨
// HTMLElement 인스턴스와 클로저 서로가 참조하고 있기 때문에 메모리 누수 발생!
```

![](https://docs.swift.org/swift-book/images/org.swift.tspl/closureReferenceCycle01@2x.png)

### How to Resolve Strong Reference Cycle for Closure

- 클로저 정의시 캡처리스트를 함께 정의함으로써 클로저와 클래스 인스턴스 간의 강한 참조 사이클을 해결할 수 있음
  - weak나 unowned 중 캡처 방식을 선택하여 정의함
  - Swift는 클로저 내부에서 self 인스턴스의 내부 멤버를 참조할 때마다 캡처했음을 명시적으로 나타내기 위해 `self` 를 작성할 것을 권장함

#### Weak and Unowned Reference in Closure

- `weak` 는 어디에서는 안전하게 사용할 수 있지만, `unowned` 는 특정 케이스에서만 사용해야 함
- `unowned` 는 강한 참조가 아니긴 하지만, 이미 해제된 미소유 참조 인스턴스에 접근하는 경우 크래시가 발생함
  - 클래스가 다른 객체에 전달되거나 저장되는 경우에는 클래스를 정의한 인스턴스가 이미 해제된 상태에서 호출될 가능성이 높기 때문에 사용하면 안됨

#### Unowned Reference 크래시 예시코드 - 1

```swift
class HTMLElement {
    lazy var asHTML: () -> String = { [unowned self] in
        return "<\(self.name) />"
    }
}

var element: HTMLElement? = HTMLElement(name: "p")
let closure = element!.asHTML  // 클로저를 변수에 저장

// 클로저가 독립적으로 존재함

element = nil  // 인스턴스만 해제, 클로저는 살아있음!

closure()  // 💥 unowned self로 캡처했지만 self는 이미 해제됨, 크래시 발생!
```

- 아까 코드와 달리, 클로저를 저장함으로써 HTMLElement 인스턴스가 해제되었음에도 캡처 리스트(HTMLElement 인스턴스 해제 전 주소)에 접근할 수 있으므로 크래시 발생

![](https://docs.swift.org/swift-book/images/org.swift.tspl/closureReferenceCycle02@2x.png)

#### Unowned Reference 크래시 예시코드 - 2

```swift
class MyClass {
    func setupTimer() {
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [unowned self] _ in
            self.update()  // 💥 인스턴스 해제 후 호출 가능성 높음, 즉 크래시 발생 가능성 높음!
        }
    }
}
```

- 외부 객체에게 메서드 파라미터로 클로저를 전달하고, 만약 인스턴스가 해제된 이후 클로저가 호출되는 경우 `unowned self` 로 인해 해제된 인스턴스에 접근하게 되어 크래시 발생

:::fold[Autoclosure]
### Autoclosure란
- 클로저를 파라미터로 전달받는 함수를 호출했을 때, 클로저의 반환 타입과 일치하는 값을 그대로 넘겨도 알아서 클로저로 감싸서 전달하도록 만드는 속성

```Swift
func log(_ message: @autoclosure () -> String) {
    print("LOG:", message())
}

log("Something happened") // ✅ 그냥 값처럼 넘겨도 자동으로 클로저로 감싸짐
// log { "Something happened" } 처럼 클로저로 값을 넘기지 않아도 됨
```

- 사용자 문법이 간결해지는 장점이 있지만, 코드의 의도가 드러나지 않아 이해 비용이 발생할 수 있음
:::