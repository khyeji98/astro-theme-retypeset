---
title: Operating System Concepts - Processes
published: 2025-09-23
tags:
  - CS
  - Operating System Concepts
toc: false
lang: ko
abbrlink : operating-system-concepts-processes
---

> 이것은 시리즈물입니다🧶
>
> [2025.09.21 - Operating System Concepts - Introduction](https://kimhyejii.tistory.com/25)  
> [2025.09.22 - Operating System Concepts - O/S structures](https://kimhyejii.tistory.com/26)

---

# 프로세스(Process)

- 실행 중인 프로그램, 메모리에 올라온 프로그램
- OS에서 실행하는 작업의 단위
- 프로세스를 실행하기 위해 필요한 리소스 : CPU Time, Memory, Files, I/O 장치

# 프로세스의 메모리 구조(Process Memory Structure)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbseYOK%2FbtsQITwpzdM%2FAAAAAAAAAAAAAAAAAAAAADzvgkGrvy2HSvSavNvyY7t39WdX4Ad8NTwXowoFvA3N%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3Du9YHJMIZVFhhJy1OH%252FXNemn7xcg%253D)

- Text Section : 실행 가능한 코드 (= executable code)
- Data Section : 전역 변수
- Heap Section : 실행 중 동적으로 할당되는 메모리 영역 (malloc, new 등의 동적 메모리 할당 코드)
- Stack Section : 함수 호출시 임시로 쌓이는 영역 (매개변수, 반환 주소, 지역 변수 등)
- **동적 영역은 런타임 시 상황에 따라 stack과 heap이 차지할 수 있는 여유 공간**으로, heap과 stack 영역이 확장되다가 맞닿게 되면 더 이상 메모리를 확보할 수 없으며 stack overflow 혹은 heap overflow 발생함

# 프로세스 상태(Process State)

![프로세스 생명주기 다이어그램](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fu3EZV%2FbtsQK69dh5n%2FAAAAAAAAAAAAAAAAAAAAAMf-Zt6HUNVmNxG4tGcZLlsNlSLm_WhtPfFMb3yZJvz2%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3Dtl%252BIRju6BDazrgpclMCnBipNmdQ%253D)

- new : 프로세스 생성(= 탄생)
- running : CPU에서 프로세스 실행 중
- waiting : I/O completion을 기다리는 중
  - 여기에서 I/O는 마우스나 키보드 같은 인터페이스 이벤트만을 뜻하는 것이 아닌 네트워크 장치를 통한 네트워크 혹은 파일 시스템 접근 등의 모든 하드웨어(I/O)를 의미하며, I/O 요청에 대한 완료 신호를 기다리고 있음을 뜻함
- ready : 실행할 준비를 마쳤으나 아직 레디큐(Ready Queue)에서 CPU 할당을 기다리는 중
  - 준비 상태에서 다시 CPU를 점유함으로써 프로세스가 재개되는 것을 스케줄러가 CPU를 dispatch 해줬다고 하여 **scheduler dispatch**라고 함
- terminated : 실행 종료

# PCB(Process Control Block)

- 하나의 프로세스에 대한 정보들을 저장하는 구조체 블록(자료구조)으로, TCB(Task Control Block)라고도 함
- 프로세스마다 고유한 PCB가 존재하기 때문에 1:1 관계가 성립됨
- 모든 PCB들은 OS에서 관리함
- 멀티 프로세싱으로 인해 실행중인 프로세스가 바뀌는 경우 실행중인 프로세스 정보를 해당 프로세스의 PCB에 저장하고, 새로 실행될 프로세스의 PCB를 가져와 실행함

## PCB에 담기는 정보

- 프로세스 상태
- 프로그램 카운터(Program Counter, PC) : 다음 실행할 명령어가 저장된 메모리(IR) 주소
- CPU 레지스터들 : IR(Instruction Register), DR(Data Register), PC(Program Counter) 등
- CPU 스케줄링 정보 : 프로세스 실행 순서를 정하기 위한 정보
- 메모리 관리 정보
- 통계 정보 : CPU 사용량 등
- I/O 상태 정보 : 할당된 I/O 장치, 열린 파일 목록

# 스레드(Thread)

- 프로세스 실행의 기본 단위로, 프로세스처럼 독립적인 실행 흐름을 가지며 스케줄러에 의해 CPU를 할당받아 실행됨
  - **"작은 프로세스처럼 동작한다"**는 의미로 Lightweight Process라고도 함
- 멀티 프로세싱처럼 하나의 프로세스에 여러 개의 스레드가 존재하는 멀티 스레딩(Multithreading) 구조가 있는데, 멀티 프로세싱보다 더 효율적임
  - 프로세스를 전환할 땐 각 프로세스가 서로 다른 주소 공간(코드+데이터+힙 = 프로세스가 사용하는 가상 메모리 공간)을 갖고 있어 주소 공간도 교체가 필요한 반면, **스레드는 같은 프로세스 안에서 주소 공간을 공유하기 때문에 전환 비용 측면에서 가볍고 저렴함**
- 현대 운영체제에서 대부분의 병행성(concurrency)와 병렬성(parallelism)은 멀티스레드를 통해 구현됨

# 프로세스 스케줄링(Process Scheduling)

- 스케줄링이란 **누구에게 CPU 제어권을 줄지** 결정하는 메커니즘인데, 아래와 같은 목적을 위해 필요함
  - 멀티 프로그래밍의 목적 : 동시에 여러 프로세스를 실행시켜 CPU 효율성을 높이기 위함
  - 시분할 시스템(time sharing)의 목적 : CPU가 프로세스 간 전환을 자주 하여 사용자 입장에서 프로그램들이 동시에 실행되는 것처럼 보이게 하기 위함

## 스케줄링 큐(Scheduling Queue)

- **프로세스 스케줄링을 위한 자료구조**로, OS가 CPU와 I/O 자원을 효율적으로 분배하기 위해 프로세스들을 담아두는 큐들을 가리킴
- Ready Queue
  - CPU 할당을 기다리는 프로세스들이 모여있는 큐
  - 스케줄러가 여기에서 프로세스를 뽑아 CPU를 할당함
- Waiting Queue(= I/O Queue)
  - I/O 작업(하드웨어 접근 작업)이 마치기를 기다리는 프로세스들이 모여있는 큐
  - I/O 완료 이벤트(I/O completion)가 발생하면 다시 레디큐로 이동

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbNJJCj%2FbtsQKzKMo1J%2FAAAAAAAAAAAAAAAAAAAAAGR-Jxsujm6ugcrnn0FbG-wnpHtx6oAPr1rTZ8OuROEx%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DJFcL9WGTrFuSDNfSaFIxC58GNBA%253D)

## Queueing Diagram

스케줄링 큐들의 관계와 프로세스의 이동 흐름을 나타내는 다이어그램

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FdhkByh%2FbtsQH9flccF%2FAAAAAAAAAAAAAAAAAAAAANKwrk9z64vpfgMGye7MXuykpBcIpecOULHtaWSqjznY%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DPDdkQspKXGO9qAmv4u0%252FYWJg1e4%253D)

## 문맥 교환(Context Switch)

- Context : 프로세스가 실행을 이어나가기 위해 필요한 정보, 즉 CPU 입장에서 필요한 중단된 프로세스의 정보
  - 마지막으로 실행했던 명령어의 주소 정보(= PC, Program Counter)
  - CPU 레지스터 값
  - 프로세스 상태(Process State)
  - 스케줄링 정보
- 인터럽트(Interrupt)나 시스템 콜과 같은 특정 이벤트가 발생하면 **실행 중인 프로세스의 context를 PCB에 저장**하고, 이후 해당 프로세스가 I/O completion에 의해 레디큐로 갔다가 다시 CPU를 점유하면 **저장했던 context를 다시 복원(restore)**함
  - CPU 제어권을 다른 프로세스에 넘겨줌으로써 현재 프로세스의 상태를 저장하고 CPU 제어권을 할당받은 새로운 프로세스가 저장했던 프로세스 상태를 복원해서 가져오는 것을 CPU 입장에서 context가 전환(switch)되는 것으로 보기 때문에 Context Switch 라고 함
- 즉 **Context Switch**란, CPU 제어권이 한 프로세스에서 다른 프로세스로 넘어갈 때 현재 프로세스의 실행 상태(context)를 PCB에 저장하고 새로운 프로세스의 상태를 복원하여 실행을 이어가는 과정

![위와 같은 일련의 과정이 Context Switch](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F0wH1n%2FbtsQK8MGmHv%2FAAAAAAAAAAAAAAAAAAAAAE4V_-RgOQCbZxSEdTn6QZQJE-mKA1bLwq4Iao6vCECk%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DPONjdfxKaptPP7tcblP0DBgOZts%253D)

:::note[idle?]
원래 idle은 실행중이 아니라서 놀고 있는 상태를 의미하는데, 주체가 무엇이냐에 따라 다르게 해석될 수 있다.
위 Context Switch 다이어그램에서는 프로세스 입장에서의 idle이므로, 해당 프로세스가 실행되지 않고 쉬고 있는 상태를 의미한다.
:::

# Parent Process & Child Process

프로세스(Parent Process, 부모 프로세스)가 새로운 프로세스(Child Process, 자식 프로세스)를 만들 수 있음

![프로세스 트리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fc8uxpc%2FbtsQNdNKMyZ%2FAAAAAAAAAAAAAAAAAAAAAHo5dCK55RF2LsIxpL1i_3TaggnY3KJou7wDu9frbuUd%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3D2iPaZb6Lt%252Bzu6Qg0D81iQ3Kaqpw%253D)

## 프로세스 실행 관점에서 두가지 가능성

1. 부모 프로세스와 자식 프로세스가 동시에 실행 (= concurrent)
2. 부모 프로세스가 자식 프로세스가 종료될 때까지 기다림

## 프로세스 주소 공간 관점에서 두가지 가능성

1. 자식 프로세스가 부모 프로세스의 복제본을 가져, 같은 프로그램과 데이터를 가짐 (= 주소 공간 복제)
2. 자식 프로세스의 주소 공간에 새로운 프로그램 로드 (= 부모 프로세스와 다른 프로그램) 

## Unix 계열 운영체제에서의 `fork()`

- `fork()`는 UNIX 계열 운영체제에서 제공하는 시스템 콜
- fork()를 통해 생성된 child process는 parent process의 주소 공간 복제본으로 구성되어 exec 계열 함수로 메모리 공간에 새로운 프로그램을 덮어쓰지 않으면 복제된 프로그램을 그대로 실행함
  - 두 프로세스는 fork() 실행 이후의 명령문(instruction)을 계속해서 실행하는데, 각각 독립된 프로세스이기 때문에 만약 어디에서도 wait() 실행을 하지 않는다면 둘 다 ready queue로 이동하여 CPU 제어권을 할당받을 준비를 함
  - 이때 CPU 제어권은 스케줄러의 결정에 의해 할당되는데, 두 프로세스 중 어떤 프로세스가 먼저 할당 받을지는 알 수 없음
  - 스케줄러 결정 기준은 우선순위, 시스템 정책 등에 따라 다름

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbaSqvw%2FbtsQNcBjWMo%2FAAAAAAAAAAAAAAAAAAAAAMU5-RUOB_zEuVV6qx80igo7IyG0FpMxtYmAS-Epdlan%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DWjp0NOlJkGCTv8CC7rQKn0Ic8CI%253D)

:::note[fork() 시스템 콜이 왜 fork라는 네이밍을 갖게 되었는지 아십니까?]
fork()를 하면 parent process에서 child process로 분기가 되는데, 대충 예시로 3개의 child process로 분기되면 마치 포크(fork) 같죠. 그래서 fork라고 합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FczHUwD%2FbtsQL9SwOZl%2FAAAAAAAAAAAAAAAAAAAAAHeVByxCnH3bTFKO2NTmwF0V17cgmfhh9HdaiIltS3B1%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3D24IiZyEB5W5C5vInlP%252B%252FWDtiVbo%253D)
:::

## 예제 소스 코드 & 실행 결과

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    pid_t pid;
    pid = fork(); // child process 생성

    if (pid < 0) { // fork 실패
        fprintf(stderr, "Fork Failed");
        return 1;
    }
    else if (pid == 0) { // child process
        execlp("/bin/ls", "ls", "-l", NULL); // child process 프로그램을 ls 명령어로 덮어씀
    }
    else { // parent process
        wait(NULL); // 자식 프로세스가 종료될 때까지 대기
        printf("Child Complete\n");
    }

    return 0;
}

// 👇 실행 결과
// ...(ls 명령어 실행으로 인해 디렉토리 파일들 출력)...
// Child Complete
```

1. parent process에서 fork() 시스템 콜 반환값이 생성한 child process의 pid(양수값)이기 때문에 마지막 else 문을 타게 되고, wait(NULL)로 인해 wait queue로 이동하여 child process가 종료될 때까지 대기함
2. child process에서는 fork() 시스템 콜의 반환 값이 0이기 때문에 pid가 0이고, `pid == 0`조건문을 통해 execlp를 실행함
   - execlp는 UNIX 계열 OS에서 exec 계열 함수 중 하나로, 현재 프로세스를 다른 실행 파일로 교체하는 함수이며 여기에서는 child process의 프로그램이 교체됨
   - 따라서 child process에서는 execlp() 이후 코드는 실행되지 않지만, 실패하면 프로그램 변경이 되지 않았기 때문에 parent process 프로그램이 유지되어 이후 코드가 실행됨
3. fork() 시스템 콜이 실패하면 음수값을 반환해서 `pid < 0` 조건문을 통해 실패 여부 확인 가능

> [!TIP]
> 👩🏻‍💻 저는 실습을 위해 VSCode를 통해 c 파일을 생성하고, 터미널을 통해 컴파일 및 바이너리 파일(실행 파일)을 실행시켰습니다.
> ```sh
> $ gcc {파일 이름, 예: main.c}
> $ ./{바이너리 파일 이름, 예: a.out}
> ```
> - gcc : 컴파일 명령어
>   - 컴파일을 통해 생성되는 바이너리 파일 이름을 지정하고 싶다면, `gcc main.c -o {바이너리 파일 이름}`과 같이 실행하면 됩니다.
> - `./{바이너리 파일 이름}`은 현재 디렉토리에 있는 특정 바이너리 파일을 실행하라는 뜻입니다.
> - 만약 터미널을 통해 파일을 생성하고 싶다면, `nano {파일 이름, 예: main.c}` 명령어를 통해 생성할 수 있습니다.

## 프로세스 종료

- 프로세스는 main 함수가 종료됨으로써 끝남
  - 반환(return), exit() 호출, 비정상 종료(예: 시그널) 등이 있는데 사실 내부적으로는 exit()가 호출됨
- 프로세스가 종료되면 OS는 프로그램을 메모리에서 해제시키고 프로세스로 인해 열린 자원들(메모리, 파일 시스템, I/O 등)을 해제함
  - 만약 child process가 종료된다면 parent process에게 종료 상태(exit code)를 전달

## 좀비 프로세스(Zombie Process)

- 자식 프로세스가 종료되었음에도 부모 프로세스에서 처리하지 않아 프로세스 테이블(PCB)이 남아있음

## 고아 프로세스(Orphan Process)

- 부모 프로세스가 자식 프로세스보다 먼저 종료해버림
  - 자식 프로세스가 종료하면 init/systemd이 새로운 부모 프로세스로서 대신 처리해줌