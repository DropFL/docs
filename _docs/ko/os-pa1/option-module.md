---
title: 옵션 모듈
category: MFQ 스케줄링 시뮬레이터
order: 6
ref: os-pa1-option-module
---

*본 문서에서 설명하는 모듈은 [이 헤더](https://github.com/DropFL/OS_PA1/blob/master/option.h)와 [이 구현체](https://github.com/DropFL/OS_PA1/blob/master/option.c)에서 확인할 수 있습니다.*

프로그램 내에서 중요한 로직을 담당하지는 않지만, 실행에 있어 커맨드로 입력되는 모든 옵션을 분석 및 적용하는 모듈이기에 따로 기재합니다.

## 옵션 인자

이 모듈에서는 `extern`으로 선언되어 전역적으로 참조할 수 있는 인자들을 정의하며, 그 목록은 다음과 같습니다.

| 이름      | 타입          | 설명 |
|:---------:|:-------------:|------|
| `verbose` | `bool`[^1]    | Verbose 모드의 활성화 여부를 나타냅니다. `0`이면 off, `1`이면 on입니다. |
| `silent`  | `bool`[^1]    | Silent 모드의 활성화 여부를 나타냅니다. `0`이면 off, `1`이면 on입니다. |
| `input`   | `const char*` | 입력 파일의 경로입니다. |
| `output`  | `const char*` | 출력 파일의 경로입니다. |

[^1]: 값이 `0` 또는 `1`임을 의미합니다. C언어에는 `bool` 타입이 없으나 편의상 이와 같이 표현했습니다.

각각의 옵션에 따른 프로그램의 변화는 **개요** 문서를 참조하십시오.

이 모듈에서는 `main` 함수에 전달된 인자를 분석하고 그것을 실행 옵션으로 적용하는 `init` 함수를 제공합니다. 그리고 `extern`에 의해 전역적으로 참조할 수 있는 옵션 변수들을 선언합니다. 더불어 옵션에 의한 구문과 주요 구문을 구별할 수 있게 도와주는 매크로를 정의합니다.

전역 변수를 사용한 이유는 각 모듈이 동일한 옵션으로 제어될 수 있게 하기 위함입니다. 또한, 이를 별도의 헤더 및 구현체로 사용한 이유는 **모듈화**를 위한 것입니다.