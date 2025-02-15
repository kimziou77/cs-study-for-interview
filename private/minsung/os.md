[toc]

# 프로세스

“Program `in execution`"

하나 혹은 복수이 스레드에 의해 실행되는 프로그램

## 프로세스의 상태

- 프로세스는 상태가 변경되며 수행된다.

  1. ```
     Running
     ```

     1. CPU를 잡고 instruction을 수행중인 상태

  2. ```
     Ready
     ```

     1. CPU를 기다리는 상태(메모리 등 다른 조건을 모두 만족하고)

  3. ```
     Blocked
     ```

     (wait, sleep)

     1. CPU를 주어도 당장 instruction을 수행할 수 없는 상태
     2. Process 자신이 요청한 Event(I O 등)가 즉시 만족되지 않아 이를 기다리는 상태
     3. 해당 실행 코드 부분이 메모리가 아닌 디스크에 있는 상태.

  4. New

     1. 프로세스가 생성 중인 상태

  5. Terminated

     1. 수행이 끝난 상태
     2. 끝나고 프로세스를 정리하는 상황

## PCB

- 운영체제가 각 프로세스를 관리하기 위해 프로세스마다 유지하는 구조체

  - OS가 관리상 사용하는 정보

        - Process State, Process ID
        - scheduling information, priority(CPU 우선순위)
          - 꼭 먼저온 순서대로 cpu가 처리하는 것이 아님.
          - 우선순위가 높은 사람에게 cpu를 먼저 넘겨준다.

  - CPU 수행 관련 하드웨어 값(레지스터)

    - 다시 running 상태가 되었을 때 CPU 관련 정보들을 복원시키기 위함
    - 예시 : Program counter
      - 다음 실행해야 할 명령어가 있는 주소 저장
    - 예시 2 : Stack pointer
      - 스택의 최상단의 주소를 저장

  - Program counter, register
    - CPU에 어떤 값을 넣고 실행하고 있었는가

- 메모리 관련
  - code, data, stack의 주소
- 파일 관련 - 이 프로세스가 제어하고 있는 파일들

## Context Switching

### Context = 프로세스의 `문맥`(매우 중요)

- 운영체제가 여러 프로세스를 제어하기 때문에 매우 중요하다.
- 프로세스의 연속성 보장.

1. CPU 수행 상태를 나타내는 하드웨어 문맥
   - Program Counter
   - 각종 레지스터에 저장했던 정보들
     - 레지스터에 어떤 값을 넣어 놨는가
2. 프로세스의 주소 공간
   1. code
   2. data
   3. stack
3. 프로세스 관련 커널 자료 구조
   1. PCB (Process Control Block)
   2. Kernal Stack

- CPU를 한 프로세스에서 다른 프로세스로 넘겨주는 과정
- 운영체제는 CPU가 다른 프로세스로 넘어갈 때
  - CPU를 내어주는 프로세스의 상태를 그 프로세스 PCB에 저장
    - 프로세스가 사용하던 register의 값을 PCB에 복사
  - CPU를 새롭게 얻어오는 프로세스의 상태를 PCB에 읽어옴
    - 각종 CPU 관련 문맥들을 복원
- System Call 이나 Interrupt 발생 시 반드시 context switch가 일어나는 것은 아니다.
  - 운영체제에게 넘어가는 것은 Context Switch가 아니다.
- 프로세스 수준의 Context switching는 이전 프로세스가 사용하던 Cache 메모리를 지운다.

### CF

- 프로세스가 운영체제를 호출했을 때도 프로세스의 상태는 running이다.
  - mode가 kernel 모드일 뿐이다. (<-> user mode)

### 질문

- 운영체제에서 프로세스 제어 블록이 필요한 이유
- 프로세스의 상태 wait과 ready의 차이점
- 프로세스와 프로그램의 차이

## 스레드

프로세스 내에서 실행되는 흐름의 최소 단위

프로그램 실행 스케줄링의 최소 단위

### 독립적 구성 요소

- register information
  - program counter
  - stack pointer
  - # etc
- Program counter
- register set
- stack space

### 동료 스레드 공유 구성 요소

- code
- data
  - 동적 할당 변수
  - non thread-local 변수
- 운영체제 자원

### 장점

- 하나의 서버 스레드가 blocked 상태에서도 동일한 태스크 내의 다른 스레드가 실행 되어 빠른 처리를 할 수 있다.

  ex) 웹 브라우저의 네트워크 요청 - io작업

  - 네트워크를 읽는 동안에 웹브라우저가 단일 스레드일 경우 프로그램 전체가 blocked가 된다.
  - 하지만 네트워크를 읽는 동안에도 여러 작업을 할 수 있다.(그동안 응답을 기다린다.)
  - 뛰어난 사용자 반응성

- 동일한 일을 수행하는 다중 스레드가 협력하여 높은 처리율과 성능 향상을 얻을 수 있다.

- 병렬성(다중 CPU를 사용할 경우에만 적용)

  - 1000 \* 1000 행렬
    - 서로 다른 cpu가 행렬을 나누고 연산 후에 합쳐서 결과 출력

- Process를 만드는 비용 > Thread 만드는 비용

  - Context Switch의 비용 > Thread Switch의 비용

### 구현 방식

- 커널 지원(Kernal Thread)
  - 운영체제 커널이 스레드의 존재를 인식
  - 운영체제가 직접 cpu를 스레드에게 넘겨줌
- 라이브러리 형태(User Thread)
  - 운영체제가 프로세스 안에 스레드 존재를 인식하지 못한다.
  - 사용자 프로그램이 스스로 스레드를 관리 - 운영체제가 스케줄링 하듯이 프로세스가 CPU 제어
  - 해당 프로세스는 가상 머신처럼 구동
  - 가상 머신에 의해 구현된다.

### 멀티 프로세스 vs 멀티 스레드

- 가장 큰 차이점은 자원 공유 유무

  - 멀티 프로세스는 자원을 공유하지 않는다.(process isolation)

    - 자원에 대한 race condition이 일어날 가능성이 없음.
    - 자원을 공유하지 않기 때문에 멀티 스레드 방식에 비해 메모리를 많이 차지한다.

  - 멀티 스레드는 자원을 공유한다.

  - race condition이 일어날 수 있다.

    - 자원의 변경으로 해당 자원을 공유하고 있는 다른 스레드에 예상치 못한 변경을 초래할 수 있다.

- 비용

  - 생성, 제거, context switching비용이 멀티 프로세스 방식이 더 크다.
    - 특히 프로세스 단위의 context switching은 cache flushing 과 같은 작업을 동반하기 때문.
  - thread 단위의 context switching은 kernel과 상호작용을 하지 않기 때문에 훨씬 저렴하다.
    - scheduling 등이 운영 체제가 아닌 스케줄링 내부에서 발생하기 때문

- 안정적인 실행 유무

  - 한 스레드가 system call을 호출해서 프로세스를 block 할 경우, 해당 스레드 뿐만 아니라 같은 프로세스에 있는 다른 스레드들도 실행되지 않는다.
    _ 실제 가상 머신에서는 non-blocking I/O를 통해 해결한다고 함.
    _ 호출한 thread만 프로세스 내부적으로 block시키고 non-blocking I/O를 호출한 다음, 다른 스레드들을 실행
    \_ non-blocking I/O만 호출하는 방식으로 코드를 짜서 해결하는 경우도 있음
  - 반면 한 프로세스의 상태가 block 된다고 해서, 다른 프로세스의 상태에 영향을 주지 않는다.

### 질문

- 멀티 스레드와 멀티 프로세스를 비교해보세요
- 스레드가 프로세스 내의 다른 스레드들과 공유하지 않는 자원들을 이야기해 보세요.
- 멀티 스레드 방식의 문제점 하나와 그것을 위한 해결책 예시 하나를 들어주세요!
