---
title: 프로세스 큐 모듈
category: MFQ 스케줄링 시뮬레이터
order: 4
ref: os-pa1-queue-module
---

*본 문서에서 설명하는 모듈은 [이 헤더](https://github.com/DropFL/OS_PA1/blob/master/queue.h)와 [이 구현체](https://github.com/DropFL/OS_PA1/blob/master/queue.c)에서 확인할 수 있습니다.*

이 모듈은 프로세스 모듈과 스케줄러 모듈에 의존하여, **독립적인 스케줄링 전략을 갖는 프로세스 큐**를 구현합니다.

## `Element` 구조체

이 구조체는 이중 연결리스트에 속한 하나의 노드이며, 다음과 같이 구성되어 있습니다.

| 이름   | 타입       | 설명 |
|:------:|:----------:|------|
| `next` | `Element*` | 바로 다음 노드에 대한 참조입니다. |
| `prev` | `Element*` | 직전 노드에 대한 참조입니다. |
| `proc` | `Process*` | 이 노드에 연결된 데이터에 해당하는 `Process` 객체입니다. |

## `ProcQueue` 구조체

이 구조체는 프로세스 큐에 대한 정보를 다음과 같이 저장합니다.

| 이름        | 타입           | 설명 |
|:-----------:|:--------------:|------|
| `head`      | `Element*`     | 연결리스트에서 가장 **앞**에 오는 `Element`입니다. |
| `tail`      | `Element*`     | 연결리스트에서 가장 **뒤**에 오는 `Element`입니다. |
| `compare`   | `ProcCompare*` | `Process`를 배치할 때 우선순위를 판별하기 위한 비교 함수입니다. |
| `scheduler` | `Scheduler*`   | 이 프로세스 큐가 갖는 스케줄링 전략입니다. |

이 모듈에 의해 정의되는 프로세스 큐는 우선순위 큐입니다. 즉, 스케줄링에 있어 `head`에 있는 `Process` 객체가 가장 우선하는 대상이며, `tail`은 우선도가 가장 낮음을 의미합니다. 이러한 우선순위는 다음과 같이 결정됩니다.

1. `ProcQueue.compare` 함수에 의한 비교에서 동등하지 않을 경우, 이를 통해 우선순위를 결정합니다.
2. 동등하다고 판단된 경우, `ProcQueue`에 먼저 등록된 `Process`가 높은 우선순위를 갖습니다.

## `IterFunc` 타입

이 타입은 `ProcQueue`를 순회하며 그에 속한 `Process`들을 대상으로 호출되는 함수로, 다음의 2가지 인자를 받습니다.

| 타입       | 설명 |
|:----------:|------|
| `Process*` | 실행 대상 `Process` 객체입니다. 포인터로 주어지므로 객체를 수정하는 작업도 가능합니다. |
| `void*`    | `Scheduler.arg`와 마찬가지로 함수에 주어지는 추가적인 인자입니다. |

여타 함수형 타입과 달리, 이 타입을 선언하는 주체는 **외부**입니다. 즉, 이 모듈을 사용하는 쪽에서 `IterFunc` 타입의 함수를 선언하여 인자로 전달해야 합니다. 이는 함수적 프로그래밍을 통해 다양한 작업을 보다 추상화하기 위함입니다.

## 모듈 내 함수

이 모듈에서는 상술한 구조체 및 타입에 관련된 다양한 작업을 함수로 제공하며, 다음은 그 목록입니다.

| 함수명      | 인자 목록                                         | 설명 | 반환값 |
|:-----------:|:-------------------------------------------------:|------|--------|
| `enqueue`   | `ProcQueue* q`<br>`Process* p`                    | `q`에 `p`를 등록합니다. | - |
| `dequeue`   | `ProcQueue* q`                                    | `q`에서 가장 우선순위가 높은 `Process`를 제거합니다. | 제거된 `Process` 객체에 대한 포인터입니다. |
| `is_empty`  | `ProcQueue* q`                                    | `q`가 비어있는지 조사합니다. | 비었으면 `1`, 아니면 `0` |
| `get_queue` | `Policy p`<br>`void* arg`                         | 새로운 `ProcQueue`를 할당하고 `p`와 `arg`를 이용해 초기화를 수행합니다. `compare`를 지정하는 데에는 `p`가, `scheduler`를 초기화하는 데에는 `p`와 `arg` 모두 사용됩니다.  | 초기화가 완료된 `ProcQueue` 객체에 대한 포인터입니다. |
| `peek`      | `ProcQueue* q`                                    | `q`에서 가장 우선순위가 높은 `Process`를 조사합니다. | 해당 `Process` 객체에 대한 포인터입니다. |
| `schedule`  | `ProcQueue* q`                                    | `q`를 스케줄링했을 때 소요될 시간을 계산합니다. 내부적으로는 `q->scheduler`에게 위임됩니다. | 소요되는 시간을 나타내는 정수입니다. |
| `iterate`   | `ProcQueue* q`<br>`IterFunc* func`<br>`void* arg` | `q`에 속한 `Process`들에 대해 `func`를 `arg`와 함께 호출합니다. `q->head->proc`부터 `q->tail->proc`까지 순서대로 호출함이 보장됩니다. 호출 당 최대 1회의 `dequeue`가 가능합니다. | - |
| `move_head` | `ProcQueue* from`<br>`ProcQueue* to`              | `enqueue(to, dequeue(from))`과 동일한 작업을 수행합니다. `enqueue`/`dequeue`는 `Element`에 대한 메모리를 할당/해제하는 작업을 수행하는데, 이를 번갈아 하는 경우 메모리의 경제성이 떨어지므로 이 함수를 사용하는 것이 좋습니다. | - |

{% include pagenator.html %}
