---
title: 스케줄러 모듈
category: MFQ 스케줄링 시뮬레이터
order: 3
ref: os-pa1-scheduler-module
---

*본 문서에서 설명하는 모듈은 [이 헤더](https://github.com/DropFL/OS_PA1/blob/master/scheduler.h)와 [이 구현체](https://github.com/DropFL/OS_PA1/blob/master/scheduler.c)에서 확인할 수 있습니다.*

이 모듈은 프로세스 모듈에 의존하여, **특정 `Process`에 할당되어야 할 시간을 계산하는 로직**을 결정합니다.

## `Scheduler` 구조체

이 구조체는 상술한 로직을 다음의 멤버들을 통해 저장합니다.

| 이름       | 타입                        | 설명 |
|:----------:|:---------------------------:|------|
| `schedule` | `(Process*, void*) -> int`  | 전달받은 `Process` 객체에 할당할 시간을 반환하는[^1] 함수입니다. |
| `arg`      | `void*`                     | `schedule`에 두번째 인자로 전달되는 값 또는 객체입니다. |

[^1]: `schedule`은 계산만을 수행하고, 전달받은 `Process` 객체의 값을 직접 수정하지는 않습니다. 단, 경우에 따라 `arg`는 수정할 수 있습니다.

2개의 멤버가 밀접하게 연결되어 실행되기 때문에, 직접 멤버를 수정하는 작업은 **높은 확률로** 프로그램의 동작을 해치는 행위가 될 수 있습니다. 따라서 `Scheduler` 구조체에 대한 할당 및 초기화 작업은 후술할 `Policy`와 `get_scheduler` 함수를 통해서만 이루어지고, 그 외에는 오로지 참조를 이용해 `schedule` 함수를 호출하게 됩니다.

## `Policy` 열거자

`Policy`는 스케줄링 전략에 대응되는 열거자로, 다음과 같은 상수가 정의되어 있습니다.

| 이름   | 설명 | 요구 인자 (`arg`) |
|:------:|------|:-----------------:|
| `RR`   | 일정한 시간 간격을 두고 프로세스를 교체하며 실행하는 전략입니다. | `time_slice`: 시간 간격을 의미하는 정수입니다. |
| `SRTN` | 현재 사이클의 남은 시간이 가장 짧은 프로세스를 우선적으로 실행하는 전략입니다. | `ready_queue`: 현재 I/O burst 상태인 `Process`의 목록입니다. Preemption에 사용됩니다. |
| `FCFS` | 가장 먼저 요청이 들어온 프로세스를 우선적으로 실행하는 전략입니다. | - |

아래와 같이 각 `Policy` 상수들의 선언부에도 요구 인자가 서술되어 있습니다.

``` c
typedef enum {
    /* `RR` is an abbreviation of **R**ound **R**obin.
       This policy needs `time_slice` parameter to use. */
    RR,

    /* `SRTN` is an abbreviation of **S**hortest **R**emaining **T**ime **N**ext.
       This policy needs `ready_queue` parameter since it is preemptive. */
    SRTN,
    
    /* `FCFS` is an abbreviation of **F**irst **C**ome **F**irst **S**erved.
       This policy does not require any parameter. */
    FCFS
} Policy;
```

`get_scheduler`는 `Policy`와 `arg`를 인자로 받아 `Scheduler`를 할당 및 초기화하여 반환합니다. `Scheduler.schedule`에 해당되는 함수들은 `Policy`의 각 원소와 1:1로 대응되며, 구현체에 `static`으로 선언되어 있습니다. 이 관계를 기반으로 `get_scheduler`는 `Policy`에 대응되는 함수에 대한 참조와 전달받은 인자(`arg`)를 `Scheduler`에 저장합니다. 또한, `Scheduler` 객체의 초기화에 쓰인 `Policy`를 조사하기 위해서는 `get_policy(Scheduler*)`를 사용합니다.[^2]

[^2]: SRTN의 Preemption을 구현하는 데에 쓰입니다.

<!-- 개인적으로 get_policy 함수는 실패적인 디자인의 결과라고 생각합니다. -->

{% include pagenator.html %}
