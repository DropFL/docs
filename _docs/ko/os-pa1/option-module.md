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

{% assign select = site.docs | where: "ref", "os-pa1-overview" | where: "lang", page.lang %}
{% assign overview = select[0] %}

각각의 옵션에 따른 프로그램의 변화는 [개요]({{ overview.url | absolute_url }}) 문서를 참조하십시오.

`input`과 `output`은 문자열이며 자주 사용되지 않기 때문에 직접 접근하는 것으로 충분하지만, `verbose`와 `silent`의 경우 타 모듈 내에서 매우 자주 사용되며, 특히 둘 사이의 우선순위가 존재하기 때문에 이에 대한 `if`문을 일일히 적는 것은 매우 비효율적입니다. 따라서, 다음과 같은 매크로를 지원합니다.

| 이름      | 내용                      | 설명 |
|:---------:|:-------------------------:|------|
| `VERBOSE` | `if (verbose && !silent)` | Verbose 모드가 활성화되었을 때 실행할 `statement`혹은 `block`을 지정합니다. |
| `NSILENT` | `if (!silent)`            | Silent 모드가 활성화되지 **않았을** 때 실행할 `statement`혹은 `block`을 지정합니다. |

이들을 사용하는 예시는 다음과 같습니다.

```c
VERBOSE printf("verbose mode is activated.\n");
NSILENT
{
    printf("silent mode is off.\n");
    printf("value of \"silent\" is %d.", silent);
}
printf("this message is always printed.");
```

또한, 사용의 편의를 위해 다음과 같은 함수를 제공합니다.

| 함수명         | 인자 목록                    | 설명 | 반환값 |
|:--------------:|:----------------------------:|------|--------|
| `init`         | `int argc`<br>`char *argv[]` | `main`에 전달된 커맨드 인자를 바탕으로 옵션을 초기화합니다. | 성공한 경우에는 `0`, 실패한 경우는 에러 코드[^2]를 반환합니다. |
| `printOptions` | -                            | 현재 옵션 상태에 관한 메세지[^3]를 출력합니다. | - |

[^2]: 에러 코드는 구현체에서 임의로 지정됩니다. 이 프로젝트의 경우 에러가 발생한 인자의 종류에 따라서만 값을 달리 했습니다.
[^3]: 메세지의 포맷 등은 구현체에 따라 달라질 수 있습니다.

전역 변수를 사용한 이유는 매크로를 통해 코드를 간결하게 하기 위함입니다. 물론 `is_verbose` 등의 함수를 구현하여 `if (is_verbose()) ...` 형태로 쓸 수 있겠지만, 이 경우 코드가 불필요하게 길어집니다.[^4]

[^4]: `if (is_verbose())`를 매크로로 등록하는 방법이 뒤늦게 떠올랐습니다. 향후 이 모듈을 재사용할 때 수정하도록 하겠습니다.

이 기능을 별도의 모듈로서 제작한 이유는 재사용성을 위한 것입니다. 지금은 [GetOpt](http://www.gnu.org/software/libc/manual/html_node/Getopt.html)나 [Argp](http://www.gnu.org/software/libc/manual/html_node/Argp.html) 등을 사용하지 않고 naïve하게 구현되었으나, 이 모듈은 외부 라이브러리를 사용해 인자를 해석하더라도 옵션을 관리하는 모듈이므로 다른 프로그램에서도 사용할 가치가 있습니다.

{% include pagenator.html %}
