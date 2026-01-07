---
title: Operating System Concepts - 1. Introduction
published: 2025-09-21
tags:
  - CS
  - Operating System Concepts
toc: false
lang: ko
abbrlink : operating-system-concepts-introduction
---
  
그 유명한 공룡책을 정독하기에는 너무나 양이 많아, 강의로 학습하고 내용을 기록용으로 정리해봤습니다!
개인적으로 궁금한 부분을 추가 조사했기 때문에 혹여나 잘못된 정보가 기재되어 있다면 정정 요청 부탁드립니다🙇‍♀️

[🔗 운영체제 공룡책 전공강의](https://www.inflearn.com/course/운영체제-공룡책-전공강의/dashboard)

---

# 운영체제(Operating System)

- 하드웨어와 소프트웨어 사이에 위치하며, 사용자, 응용 프로그램, 하드웨어 사이에서 중재자 역할을 함
- 사용자가 입력 장치를 통해 이벤트를 발생시킴 -> 응용 프로그램에서 이벤트 처리를 위해 OS에게 요청 -> OS는 요청에 따라 하드웨어 제어 및 작업 수행 -> OS가 하드웨어로부터 결과값을 받으면 응용 프로그램에 전달 -> 응용 프로그램은 인터페이스를 통해 사용자에게 출력

![OS 흐름도](https://blog.kakaocdn.net/dna/3501C/btsQI1UsLFX/AAAAAAAAAAAAAAAAAAAAAEX4y7kPjvuEJlUR3n5SxqAl6WSK5OLI0eoaLUwWT62-/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=qLNbZ6swMPa2miKXGy5faxvkOeA%3D)

# 컴퓨터 구성 요소

= 1개 이상의 CPU + 여러 개의 디바이스 컨트롤러 + 이들을 이어주는 공통 버스(bus)

![](https://blog.kakaocdn.net/dna/bJ527G/btsQGXMF4Wo/AAAAAAAAAAAAAAAAAAAAAMHFv5PU1BESLuA67Nm104FVdpy_DoORrUXAG7F7aKyD/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=n41UX6g1rshOUQKOHrgIEXTfeMA%3D)

# 부트스트랩 프로그램(bootstrap program)

:::note[왜 부트스트랩일까?]
가죽신발(boots)을 신기 위해 뒷쪽에 달린 스트랩을 "bootstrap"이라고 하며,
가죽신발을 신을 때 끌어올리기(boot up) 위한 요소의 의미와 같습니다.
:::

- 컴퓨터 전원을 켰을 때, 가장 먼저 실행되는 프로그램
- 모든 하드웨어를 켜고 메모리 등 장치들을 초기화함
- ROM(Read Only Memory)에 저장되어 있는 OS를 메모리(RAM)에 로드함

# 인터럽트(Interrupt) 시스템

- 하드웨어나 소프트웨어에서 이벤트가 발생했을 때 CPU에 알림을 보내는 메커니즘
- 예를 들어, I/O 디바이스 입력으로 인해 이벤트가 발생했을 때 CPU가 이를 처리하기 위해 진행중이던 프로세스를 일시 중단하고 이벤트를 처리한 뒤 중단한 프로세스를 재개함

![](https://blog.kakaocdn.net/dna/MTC8r/btsQIN3czcq/AAAAAAAAAAAAAAAAAAAAAM3Ff8mSIbA6rfgZfIIaz_q7PbASD7JC9UIcQThM0qxr/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=CUU1%2Fn6IFkmvAGxJmQY8HlNjec8%3D)

# 폰 노이만 아키텍처(명령-실행 주기)

1. 프로그램이 실행되면 프로그램이 메모리에 올라가고, 메모리에서 명령어를 하나씩 가져와 IR에 저장함
   - IR(Istruction Register, 명령어 레지스터) : 명령어를 저장하는 레지스터
2. CPU가 명령어를 하나씩 fetch 및 execute함
   - IR이 필요한 값을 가져오기 위해 레지스터들에 접근하여 값을 가져와 명령어 동작을 수행함

# 저장소 계층 구조

![](https://blog.kakaocdn.net/dna/1l7QE/btsQHV1BbTK/AAAAAAAAAAAAAAAAAAAAANBPfcfM-FHLnzHPK04C90r5ga978pkoQpfH4lXVUmDf/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=WFnio4de%2BEisOl4dADgnpeYgXck%3D)

1. **레지스터(Register)**
   - CPU 안에 회로로 묶여 존재하는 초고속 저장 장치
   - 용량은 매우 작음
2. **캐시(Cache)**
   - 램보다 훨씬 빠르지만 용량이 작고 비쌈
   - 자주 쓰이는 데이터를 임시저장함으로써 CPU 접근 속도 향상
3. **RAM(= Random Access Memory)**
   - 현재 실행중인 프로그램과 데이터를 저장하는 영역
   - 전원이 꺼지면 데이터가 날아가는 휘발성 메모리 (= Dynamic RAM 기반)
   - CPU가 직접 접근 가능한 실행 공간 이라서 메인 메모리(주 저장장치)라고 함
     - 예: 프로그램 실행 -> 디스크에서 프로그램을 RAM에 로드 -> CPU가 RAM에서 명령어와 데이터를 fetch
4. **SSD(Solid-State Disk)**
   - 플래시 메모리 기반의 보조 저장 장치 (=비휘발성)
   - HDD보다 훨씬 빠르고 내구성이 강하지만 더 비쌈
5. **HDD(Hard Disk Drive)**
   - 마그네틱 카드 자기장을 이용한 저장 장치
   - SSD보다 느리지만 대용량 저장시 사용
6. **Optical Disk(광디스크)**
   - HDD에 저장하기에 용량이 큰 대용량 데이터나 장기 보관용 데이터를 저장함
   - CD, DVD, Blu-ray 등
7. **Magnetic Tapes(자기 테이프)**
   - 순차 접근 방식이라 느리지만 대규모 백업 및 아카이브 용으로 활용됨
   - 수십년 단위 저장 가능하며 저렴함

> [!IMPORTANT]
> RAM의 “Random Access”는 임의 접근이라는 뜻으로, 주소만 알면 위치에 상관없이 동일한 속도로 접근 가능하다는 의미입니다.

# I/O 구조

![](https://blog.kakaocdn.net/dna/bAuypb/btsQI8lKYl8/AAAAAAAAAAAAAAAAAAAAAFf4DvqVfLKMjIx1_rEPmgZmPEEelcpliTJp8V2_Cb_t/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=VhgDxhyePeP7%2BrQwgMD3aCvILVA%3D)

- **DMA(Direct Memory Access)**
  - 간단하게 메모리 저장 혹은 값 출력만을 요하는 경우, 입출력을 위해 CPU가 계속 잡혀있지 않고 디바이스와 메모리가 직접 데이터를 전달하는 방식
  - 이 방식을 담당하는 장치를 DMA 컨트롤러라고 함
  - 이를 통해 CPU는 다른 작업을 동시에 수행할 수 있음

# 컴퓨터 시스템 부품들

- **CPU** : 명령어를 실행하는 하드웨어
- **Processor**: 1개 이상의 CPU를 포함하는 물리적 칩
  - CPU라는 개념적 용어의 물리적인 칩을 일컫기 때문에 일상에서는 둘을 같은 존재로 봐도 무방함
- **Core** : CPU 내부에서 실제로 연산을 수행하는 연산 단위
  - 하나의 코어는 독립적으로 명령어를 처리하여 명령-실행 주기를 가짐
  - 예를 들어, 듀얼코어 CPU는 2개의 코어가 있으므로 동시에 2개의 명령어 실행 가능

# SMP(Symmetric Multiprocessing, 멀티프로세서)

- 가장 흔한 멀티 프로세서 시스템 (= 하나의 시스템에에 CPU가 여러개라는 뜻)
- 모든 CPU가 동등한 권한으로 메모리와 I/O 장치를 공유하며, 각 CPU가 각각의 레지스터와 캐시를 가짐
- 여러 CPU가 동시에 일을 나눠서 병렬적으로 수행하기 때문에 응답 속도 향상, 높은 CPU 활용률, 독립적이기 때문에 안정성 향상

![](https://blog.kakaocdn.net/dna/bO3kkl/btsQKDesEPX/AAAAAAAAAAAAAAAAAAAAAO4FKGD1REwL7MzJeyK0x9bIR3vastfmRjZB7VU3pqOq/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=IFKv9e9pSq5JkalJSgz8zTqaO3Q%3D)

# Multi-core

- 한 개의 CPU(프로세서) 칩 안에 2개 이상의 코어가 있음

![](https://blog.kakaocdn.net/dna/7OCE4/btsQIZbx5DY/AAAAAAAAAAAAAAAAAAAAAMJ4dob1oWK2FCAKO8w02taIZRg4fwlQ6UJ8RTpTVKp2/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=0lJdEV4uJapiz5aCfBIBCBhY71w%3D)

## 멀티프로세서(SMP)도 병렬 작업인데, 무슨 차이지?

- 둘 다 동시에 여러 작업을 수행하는 점에서는 비슷하지만, 하드웨어 구조와 자원 공유 방식에 차이가 있다.
- **멀티 프로세서(SMP)**
  - CPU 간 통신시 데이터 동기화 비용이 커짐 (ex. 캐시 일관성 문제)
  - CPU 칩이 여러개이기 때문에 하드웨어 가격이 비쌈
- **멀티 코어**
  - 같은 CPU 칩 안에서 코어가 여러개이기 때문에 CPU 간 통신보다 속도가 빠름

# Multiprogramming(다중 프로그래밍)

- 여러 개의 프로그램을 동시에 메모리에 로드하고 실행시키는 방식으로, **실제로 CPU는 한번에 하나의 프로그램만 실행하지만 하나씩 번갈아 실행시킴**
- **시스템 입장**에서 CPU 활용을 극대화시키기 위함이 목적
- A 프로그램이 결과를 실행 결과를 기다리는 동안 B 프로그램을 실행하여, 하나의 프로그램 동작을 오랫동안 기다리느라 다른 프로그램을 실행하지 못할 일이 없음

![](https://blog.kakaocdn.net/dna/cenO1Z/btsQISDrilJ/AAAAAAAAAAAAAAAAAAAAAGVUJq-ryoY0KUsooOd8VUf09n5y-abmRLINGADN-prI/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=dxKBvUT2pB6QIeC0K2E9k9OInjQ%3D)

# Multitasking(다중 작업)

- 멀티프로그래밍의 논리적 확장으로, 여러 프로그램이 동시에 실행되는 것처럼 보이게 **CPU 시간을 나눠(= 시분할)** 빠르게 교체하며 실행시키는 방식
- **사용자 입장**에서 동시에 여러 프로그램이 실행되는 것처럼 보이게 함이 목적 (= 사용자 체감 환경 발전)
- CPU 스케줄링
  - CPU가 여러 프로그램을 빠르게 교체하여 실행하기 위해 어떤 프로그램을 먼저 수행할지 고르는 작업
  - **스케줄링의 주체는 OS**, CPU는 연산기일 뿐 어떤 프로세스가 CPU를 점유할지는 OS가 스케줄러 모듈을 통해 결정함

# 운영체제 연산 모드들

- CPU는 보안과 안정성을 위해 두가지 모드로 나눠 동작함
- **유저 모드**
  - 하드웨어 자원에 직접 접근 제한
  - 접근 요청은 system call 통해 OS에게 위임
- **커널 모드**
  - 모든 자원에 직접 접근 가능 (= 최고 권한)
  - 오해하지 말아야 할 것, 유저에게 접근 권한 주어지는 것이 아닌 OS가 접근 권한을 부여받는 것

![유저모드 <-> 커널모드 전환 흐름도](https://blog.kakaocdn.net/dna/lsB2k/btsQHyzsqgG/AAAAAAAAAAAAAAAAAAAAAHH5vQX-rq8jXebXS5pztMMoXgapuOts485Fm1fJOBcU/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=615fr9V2wPfCdu3gsFE4o94%2F1ZI%3D)

- 사용자가 응용 프로그램을 사용하다가 하드웨어 자원에 직접적인 접근이 필요한 이벤트를 발생시키면 **system call을 통해 CPU가 커널모드로 전환**된다. 이때 운영체제가 하드웨어 자원에 접근해 이벤트 처리를 위한 동작을 수행하고 결과값을 받아 **유저 프로세스로 진입하기 전에 CPU를 다시 유저모드로 전환**하고 인터페이스를 통해 사용자에게 결과값을 출력시키는 흐름입니다.

# 가상화(Virtualization)

![](https://blog.kakaocdn.net/dna/cEIf3P/btsQKCzRUGT/AAAAAAAAAAAAAAAAAAAAAP-QBdevPbSkJqaRVnPTKt60OZmglAoTRNQMwzTWAEha/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=4h5FYchKmMLJ%2FMjebFeYZ0Naovo%3D)

- 하나의 하드웨어 위에 여러 OS를 동시에 실행시킬 수 있도록 하는 기술
- **VMM(Virtual Machine Manager, 하이퍼바이저)**
  - 하드웨어와 OS 사이에 존재
  - VMware, XEN, WSL 등이 대표적

# 컴퓨팅 환경(Computing Environments)

운영체제가 동작할 수 있는 여러 환경들이 있습니다. 이는 컴퓨터가 사용되는 방식, 맥락, 목적에 따라 구분됩니다.

- **Traditional Computing** : 일반 컴퓨터
- **Mobile Computing**
  - 스마트폰, 태블릿과 같은 모바일 환경
  - 배터리/무선 네트워크/터치 UI 지원 필요
- **Client-Server Computing**
  - 클라이언트가 요청을 보내고 서버가 응답하는 구조
  - 예: 웹 서비스, 데이터베이스 서버

![Client-Server Computing](https://blog.kakaocdn.net/dna/CAxqP/btsQJ5ipTe1/AAAAAAAAAAAAAAAAAAAAAM8oAFyqWCUA_tMy_yQftFSl3bkBlUCp_v8x28mTfg8h/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=jRfpSZH7Fb6aph75Z2h1j7NGvJI%3D)

- **Peer to Peer(P2P)**
  - 중앙 서버 없이 사용자(노드) 간 직접 통신을 통해 자원 공유 (=N:N 컴퓨팅)
  - 예: BitTorrent, 블록체인

![P2P Computing](https://blog.kakaocdn.net/dna/ca4TmX/btsQH0WLgNl/AAAAAAAAAAAAAAAAAAAAANdrT8VFkWWWDbK1YCVVgV-S3Jwi-PP63aF1ahhizy7X/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=6ydW5DXTRNUuvVFnEOUNh5VBcu0%3D)

- **Cloud Computing**
  - 하드웨어를 가상화하여 가상머신이나 컨테이너와 같은 단위로 컴퓨팅 자원(서버/스토리지/네트워크)을 인터넷을 통해 서비스로 제공 (= 가상화 기반)
  - 예: AWS, Azure, GCP, 그 밖의 네이버클라우드 등

![Cloud Computing](https://blog.kakaocdn.net/dna/cIZS2u/btsQK68WAFS/AAAAAAAAAAAAAAAAAAAAAOJnV0GXmKDEOTTbQd5QvRG62rJS-vxxTNm_NeCXTk35/img.png?allow_ip=&allow_referer=&credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&signature=yzfp6h%2Fwac3%2B3mL3ydPE2OiMXmY%3D)

- **Real-Time Embedded Systems**
  - 자동차, 의료기기, 가전제품, IoT 기기 등에 들어가는 실시간 OS
  - 비용/전력/공간 제약으로 인해 최소 자원으로 구성되어 하드웨어 제약이 있고, 정해진 시간 안에 무조건 응답해야 하는 실시간성이 보장되어야 함
