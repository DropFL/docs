---
title: 프로세스 모듈
category: MFQ 스케줄링 시뮬레이터
order: 2
ref: os-pa1-process-module
---

*본 문서에서 설명하는 모듈은 [이 헤더](https://github.com/DropFL/OS_PA1/blob/master/process.h)와 [이 구현체](https://github.com/DropFL/OS_PA1/blob/master/process.c)에서 확인할 수 있습니다.*

## `Process` 구조체

이 구조체는 프로세스가 갖춘 속성의 집합체로, 다음과 같이 구성됩니다.

{:ok: style="color:limegreen"}
{:no: style="color:red"}

| 이름            | 타입   | 입력 기반             | 상수                 | 설명 |
|:---------------:|:------:|:---------------------:|:--------------------:|------|
| `pid`           | `int`  | <span>✔</span>{: ok} | <span>✔</span>{: ok} | 프로세스의 식별자입니다. |
| `arrival`       | `int`  | <span>✔</span>{: ok} | <span>✔</span>{: ok} | 프로세스가 시작되는 시점입니다. |
| `term`          | `int`  | <span>✘</span>{: no} | <span>✘</span>{: no} | 프로세스가 종료되는 시점입니다. |
| `wait`          | `int`  | <span>✘</span>{: no} | <span>✘</span>{: no} | 프로세스가 종료될 때까지 CPU burst 상태에서 대기한 시간입니다. |
| `queue_idx`     | `int`  | <span>✔</span>{: ok} | <span>✘</span>{: no} | 프로세스가 현재 위치하거나(CPU burst) 앞으로 위치할(I/O burst) 큐의 인덱스입니다. |
| `num_cycles`    | `int`  | <span>✔</span>{: ok} | <span>✔</span>{: ok} | CPU burst, I/O burst의 총 개수입니다. 입력 파일에서의 사이클 개수 $$n$$에 대해, $$2n$$으로 계산됩니다. |
| `current_cycle` | `int`  | <span>✘</span>{: no} | <span>✘</span>{: no} | 현재 해당되는 burst의 인덱스 입니다. 홀수인 경우 CPU burst, 짝수인 경우 I/O burst에 해당됩니다.[^1] |
| `cycles`        | `int*` | <span>✔</span>{: ok} | <span>✘</span>{: no} | arrival time과 burst cycle로 이루어진 배열입니다. 일반적으로 이 속성은 후술할 매크로 함수를 통해 접근하게 됩니다. |

[^1]: 0인 경우는 예외적으로 arrival time입니다.

일반적으로 특정 `Process`에서 현재 실행 중인 사이클의 남은 시간은 다음과 같이 구합니다.

``` c
// Process* p = SOME_PROCESS;
int time = p->cycles[ p->current_cycle ]; // time of current cycle
```

이 방식은 굉장히 불편하고 실수하기 쉽기에, 이 모듈에서는 다음과 같은 매크로 함수들을 제공합니다. 인자는 모두 `Process*` 타입의 `p`이며, 대상은 `p`가 가리키는 `Process` 객체입니다.

| 이름            | 값         | 설명 |
|:---------------:|:----------:|------|
| `CUR_CYCLE(p)`  | `int`      | 현재 실행 중인 사이클의 시간을 가져오는 함수입니다. |
| `NEXT_CYCLE(p)` | `int`      | 다음으로 실행될 사이틀의 시간을 가져오는 함수입니다. |
| `PROC_END(p)`   | `bool`[^2] | 해당 `Process`가 종료되었는지를 조사합니다. |
| `IS_ACTIVE(p)`  | `bool`[^2] | 현재 `Process`가 CPU burst에 진입했는지 조사합니다. |

[^2]: 값이 `0` 또는 `1`임을 의미합니다. C언어에는 `bool` 타입이 없으나 편의상 이와 같이 표현했습니다.

보다 복잡한 작업의 경우는 함수로 제공되며, 그 목록은 다음과 같습니다.

| 정의                       | 반환값     | 설명 |
|:--------------------------:|:----------:|------|
| `read_process (FILE*)`     | `Process*` | 동적 할당을 통해 `Process`를 생성한 후, 주어진 `FILE`에서 `Process` 하나에 대한 정보를 읽고 초기화하여 메모리 주소를 반환합니다. 초기화에 실패한 경우 `NULL`을 반환하며, 메모리는 해제됩니다. |
| `free_process (Process*)`  | `void`     | 해당 `Process`에 대한 메모리를 해제합니다. |
| `print_process (Process*)` | `void`     | `Process`의 정보를 출력합니다. |

## `ProcCompare` 타입

`ProcCompare`는 두 개의 `Process`를 비교하기 위한 함수의 프로토타입으로, 그 형태는 다음과 같습니다.

``` c
int ProcCompare (Process* a, Process* b);
```

이 함수는 `a`와 `b`의 **특정한 멤버**[^3]를 비교한 결과를 반환합니다. 음수인 경우 `a < b`, 양수인 경우 `a > b`, 0인 경우 `a == b`의 관계를 만족합니다.

[^3]: 두 `Process`의 실행 우선순위가 아닌, 특정 속성의 대소를 비교합니다. 실제 우선순위는 다른 모듈에서 판별합니다.

실제로 이 타입의 함수들은 구현체 내에서 `static`으로 선언되어 직접 사용할 수 없고, 후술할 `get_compare`를 통해 함수에 대한 참조를 얻어와 간접적으로 호출할 수 있습니다.

## `Factor` 열거자

`Factor`는 `ProcCompare`로 비교할 인자의 목록입니다. 정의된 상수는 다음과 같습니다.

| 이름     | 비교 대상 |
|:--------:|-----------|
| `NONE`   | 비교를 실시하지 않습니다. 모든 프로세스를 동등하게 취급합니다. |
| `REMAIN` | 현재 사이클의 남은 시간 (`CUR_CYCLE`) |

`get_compare` 함수는 `Factor`를 인자로 받아 그에 맞는 `ProcCompare` 타입의 함수에 대한 참조를 반환합니다. 예를 들어, `get_compare(NONE)`의 반환값은 항상 0을 반환하는 함수에 대한 포인터입니다.

이 모듈 내부에 `static`으로 선언된 `ProcCompare`들과 `Factor`의 상수들은 1:1로 대응되고, `get_compare`는 이 관계를 기반으로 `Factor`에 대응되는 `ProcCompare`를 반환하여 이에 대한 접근 및 호출을 가능케 합니다.

{% include pagenator.html %}
