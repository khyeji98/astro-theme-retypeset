---
title: Operating System Concepts - O/S
published: 2025-09-22
tags:
  - CS
  - Operating System Concepts
toc: false
lang: ko
abbrlink : operating-system-concepts-os
---

> 이것은 시리즈물입니다🧶  
>   
> [2025.09.21 - Operating System Concepts - Introduction](https://kimhyejii.tistory.com/25)

---

# OS가 제공하는 서비스

- UI(User Interface)  
- 프로그램 실행  
- 입출력 연산  
- 파일 시스템 조작  
- 에러 탐색  
- 자원 할당  
- 로깅  
- 보호 및 보안  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F9OHAU%2FbtsQKBO4qGP%2FAAAAAAAAAAAAAAAAAAAAANTd94clxm0Ma7VwTroR79d8tRYy-YwgQXHCG7VOyG3y%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DW%252BtEUhLcHfS5r2ZLQBiMF8yzGmc%253D)

# OS가 제공하는 인터페이스

- **CLI(Command Line Interface)** : 명령어 기반 인터페이스  
  - 예: sh(쉘), bash, csh, zsh
- **GUI(Graphical User Interface)** : 그래픽 기반 유저 인터페이스  
  - 예: 아이콘, 버튼, 윈도우
- **Touch-Screen Interface** : 터치 기반 인터페이스  
  - 예: 스마트폰, 태블릿, ATM, 키오스크  

:::note[쉘(Shell)?]
사용자가 입력한 명령을 해석해서 OS에 전달하고 실행 결과를 다시 사용자에게 출력하는 인터페이스 프로그램으로, 사용자와 OS 사이 다리 역할을 합니다.  
그리고 우리가 쉘에서 흔히 사용하는 명령어(`ls`, `cd` 등)을 **쉘 스크립트 언어**라고 합니다.  
  
기존 쉘을 좀 더 개선해서 편리하게 사용하도록 도와주는 확장쉘들 중 대표적으로 **Bash, zsh** 등이 있습니다.
:::

# 시스템 콜(System Call)

- 응용 프로그램은 시스템 콜을 통해 OS에서 제공하는 서비스를 호출할 수 있음  
  - 응용 프로그램이 직접 하드웨어 자원에 접근하거나 제어할 수 없기 때문에 OS에 요청해야 하는데, 이때 요청을 전달하는 방법이 **시스템 콜(System Call)**  
- 프로그래머가 시스템 라이브러리를 통해 API를 호출하면 시스템 콜을 통해 OS에 작업을 요청하는 흐름  
  - **API (Application Programming Interface)** : 프로그래머가 직접 호출하는 함수 집합  

!["API ↔ System Call ↔ Kernel" 흐름도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbZ0BIo%2FbtsQHEsXe5U%2FAAAAAAAAAAAAAAAAAAAAAIQ77tZv5QLs8Dn3Y7EzF3mp8X_3FuEXd5O2o9YKFP7e%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DpPdAWDd6rlg0n8S5TXbQQUkZEZo%253D)