---
title: MFQ 모듈
category: MFQ 스케줄링 시뮬레이터
order: 5
ref: os-pa1-mfq-module
---

*본 문서에서 설명하는 모듈은 [이 헤더](https://github.com/DropFL/OS_PA1/blob/master/MFQ.h)와 [이 구현체](https://github.com/DropFL/OS_PA1/blob/master/MFQ.c)에서 확인할 수 있습니다.*

이 모듈은 프로세스 큐 모듈에 의존하여, 최종적으로 **다단계 피드백 큐로서의 작업을 수행합니다.**

## `MFQ` 구조체

이 구조체는 여러 개의 프로세스 큐를 우선순위에 따라 단계적으로 배치하고, I/O 대기 프로세스 큐를 더해 여기에 속한 모든 프로세스들을 관리합니다. 내부는 다음과 같이 구성되어 있습니다.

| 이름          | 타입          | 설명 |
|:-------------:|:-------------:|------|
| `num_queue`   | `int`         | 이 MFQ에 속한 프로세스 큐의 개수입니다. |
| `queues`      | `ProcQueue**` | 프로세스 큐의 목록입니다. `queues[0]`이 가리키는 `ProcQueue`의 순위가 가장 높습니다. |
| `ready_queue` | `ProcQueue*`  | I/O burst에 진입한 프로세스들이 위치하는 `ProcQueue`의 참조입니다. |

`mfq.num_queue`를 $$n$$이라 하면, `mfq.ready_queue`의 존재로 인해 실제로 `mfq`가 관리하는 `ProcQueue`는 총 $$n+1$$개입니다.

MFQ 모듈이 제공하는 함수는 다음과 같습니다.

| 함수명      | 인자 목록                                            | 설명 | 반환값 |
|:-----------:|:----------------------------------------------------:|------|--------|
| `get_mfq`   | `int num_q`                                         | `num_q`개의 계층을 갖는 `MFQ`를 동적으로 할당합니다. 이후 `num_queue`를 `num_q`로, `queues`를 크기가 `num_q`인 `ProcQueue*`형 배열로[^1], `ready_queue`를 새로운 `ProcQueue`로 초기화합니다. | 생성된 `MFQ`에 대한 참조입니다. |
| `set_queue` | `MFQ* mfq`<br>`int idx`<br>`Policy p`<br>`void* arg` | `mfq`의 `idx`번째 계층의 프로세스 큐를 `p`와 `arg`를 이용해 초기화합니다. 이 작업은 스케줄러 모듈의 `get_queue` 함수에 위임됩니다. | - |
| `proceed`   | `MFQ* mfq`<br>`Process** p`<br>`int* t`              | `mfq`의 스케줄링을 실시합니다. 성공한 경우, `*t`의 값은 스케줄링에 의해 진행된 시간입니다. `*p`의 값이 `NULL`이면, 그 동안은 IDLE 상태였음을 의미합니다. 그렇지 않은 경우, 이번 스케줄링의 대상이 된 프로세스 객체가 `**p`임을 의미합니다. | 스케줄링에 실패한 경우에만 `0`이 반환됩니다. |

[^1]: 단, 배열의 원소가 초기화되지는 않습니다.

정리하자면, `MFQ`를 사용하기 위해서는 다음과 같은 절차를 거쳐야 합니다.

1. `get_mfq`로 `MFQ` 객체를 할당한다.
2. `set_queue`를 각 계층마다 한 번씩 호출하여 모두 초기화한다.
3. `Process`를 원하는 `ProcQueue`에 `enqueue`한다.[^2]
4. 스케줄링이 끝날 때까지 `proceed`를 반복 호출하여 결과를 받아낸다.

[^2]: [이 라인](https://github.com/DropFL/OS_PA1/blob/master/main.c#L50)과 [이 라인](https://github.com/DropFL/OS_PA1/blob/master/main.c#L54)처럼, `MFQ`의 멤버에 직접 접근하여 `enqueue`를 호출해야 합니다.

`MFQ`는 사용된 모듈 중 가장 규모가 작지만 상대적으로 복잡도가 가장 높은데, 이는 `proceed` 함수가 가장 복잡하기 때문입니다. 이 함수는 스케줄링 대상 `Process`와 소요 시간의 결정, 대상 `MFQ` 내 모든 `Process`의 관리 및 수정 작업이 모두 발생합니다.[^3] 물론 이 역할을 쪼갤 수도 있지만, 그다지 큰 의미가 없거나 오히려 다른 모듈이 원래대로라면 불필요한 인자를 추가로 받게 됩니다.[^4]

[^3]: 정확히는 모듈 내 `static`으로 선언된 콜백 함수를 통해 발생합니다. 그러나 그 콜백을 사용하는 함수는 `proceed`입니다.
[^4]: 함수가 많은 역할을 수행하면서 내부의 작업을 일일히 관찰하기 어렵다는 단점이 있는데, Verbose 옵션을 사용하면 출력을 통해 관찰이 가능합니다.
