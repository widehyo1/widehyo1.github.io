---
layout: post
title: 명시적 파리미터와 암시적 파라미터
subtitle: explicit parameter(capture everything in parameter function to be pure), implicit parameter(global variable or closure)
tags: [thoughts, functional_programming, closure, pure, effect]
comments: true
author: widehyo
---

## effectful function to pure function
어떤 의미의 side-effect를 담고있는 함수를 pure한 함수로 만드는 가장 빠른 방법이 뭘까?

답은 그 side-effect를 파라미터에 담는 것이다.

## 상황 설정

다음의 굉장히 작위적인 상황을 생각하자. 당신은 개발자로, 어떠한 application을 제작하고 있으며 그 중 가장 핵심적인 역할을 하는 함수를 작성 중이다. 이 함수는 너무 핵심적이라 당신은 이 함수의 실행에 대한 (어떠한 종류의)메타 정보도 추적하고 싶다.

조금 더 구체적인 상황을 생각하자. 당신은 웹을 통하여 통계표를 조회하는 페이지를 제작중이고, 특정 통계표는 너무 많은 데이터를 담고 있기 때문에(> 1,000,000 rows) 우연히 사용자가 그 통계표를 조회하는데 500초가 걸린다고 하자.

그래서 해당 통계표를 처음 조회하는 사용자는 항상 운영팀에 민원을 넣는다. 이 상황을 견딜 수 없는 운영팀은 당신이 속한 개발 팀에 다음과 같이 요구한다. "이 페이지에 버그가 있는거 같은데 원인 파악하고 해결해주세요". 자, 이제 당신은 먼저 상황을 파악하기 위해 해당 페이지에 얼마나 많은 요청이 왔는지 확인하려고 한다.

이 상황에서의 일반적인 전략은 로그파일 중 해당 페이지를 조회하는 로그의 수를 세는 방법이지만, 그동안 로깅 전략이 잘못 되어 있어서 해당 방법을 실행하기 어려운 상황이다(일주일치의 로그만 보관하여 원하는 정보가 유실되었거나 모든 로그를 한 파일에 저장하여 파일을 열려는 시도는 시스템을 다운시킨다(로그파일의 사이즈가 20TiB이다)). 현재 상황에서 해당 페이지에 요청이 얼마나 왔는지 파악하기 불가능하다는 것을 파악한 당신은, 이제부터라도 해당 페이지에 대한 접근 하는 함수를 수정하여, 특정 페이지의 조회수만 따로 저장하도록 변경하였다. 이제 일주일만 지나면, 조회수 상승 추이를 팀장에게 보고할 수 있을 것 같다.

## 순수함수주의자

모든 문제를 해결한 후, 순수함수주의자인 당신은 생각한다. 변경된 함수는 이제 더이상 순수함수가 아니다. 기존의 함수의 역할 뿐만 아니라 side-effect인 특정 페이지에 대한 조회수를 증가시키는 동작을 하고 있기 때문이다. 자신의 코드에 effectful한 코드가 들어가는 것이 마음에 들지 않았던 당신은 이 함수를 pure한 함수로 만들고 싶고, 이내 한가지 방법을 떠올린다. 페이지의 조회수가 side-effect라면, 함수의 역할을 이것까지 확장시키면 되잖아(여러분은 그러면 안된다). 당신의 함수는 이제 기존의 parameter와 대상 페이지의 조회수를 추가적으로 parameter로 받는다.

자, 이제 작위적인 상황은 뒤로하고 조금 더 간단한 상황을 생각하자.
sum'이라는 함수가 존재하여 두 수(Int, Int)를 받아 합을 반환하되, 이 함수가 호출된 횟수를 누적하고 싶다.

```py
global count = 0
def sum'(a: int, b: int) -> int:
    count += 1
    return a + b
```

이 함수는 pure하지 않다. 무엇이 문제일까? parameter로 받은 a와 b만 가지고 함수가 하는 역할을 표현할 수 없다. 더 정확하게는 side-effect인 count의 증가라는 현상이 parameter로 받은 a와 b로는 표현할 수 없는 것이 문제이다.

이제 순수함수주의자의 해결방안을 보자
```py
def sum'(a: int, b: int, count: int) -> int:
    return { "return": a + b, "count": count + 1 }
```

어찌 되었건, 이제 이 함수는 순수함수가 되었다. 즉, 같은 parameter가 주어지면 같은 return을 반환한다.

## what is the function? what is effectful function?

함수 외부(parameter에 capture되지 않는)의 global variable를 함수 안에서 변경한다면 이것은 side-effect이다. 이건 말이 되는 것 같다. 조금 더 subtle한 상황을 생각하자.

```py
def create_message_template(msg: str):
    def greeting(name: str):
        return f"hello, {name}! " + msg
    return greeting

nice_message = create_message_template("nice to meet you.")
nice_message("peter") # hello, peter! nice to meet you.
```

자, nice_message라는 함수는 순수한가?

답하기 어려운가?

nice_message("james")의 return은 무엇인가?

당연히 "hello, james! nice to meet you."지만, nice_message = create_message_template("go home.")인 경우에도 그러한가?

nice_message라는 fixed parameter가 주어졌을 때, output이 determistic한가? input이 같을 때 output이 같다고 말할 수 있는가?

당신이 관심을 가지는 함수는 간단하게 파일 명을 인자로 받아 해당 파일의 내용을 출력하는 함수이다.

이 함수는 pure한가? 아니라면, side-effect는 어디서 왔는가.

자, database를 조회하는 함수를 생각해보자. 당신의 함수는 pk를 받아 "something"이라는 table의 해당하는 row를 조회하는 함수이다. input이 주어졌을 때, output이 determistic한가?

조금 더 subtle한 상황을 생각해보자.
```py
from dataclasses import dataclass
from typing import Optional

@dataclass
class Wrapper:
    field1: Optional[str]

    def __eq__(self, other):
        if not isinstance(other, Wrapper):
            return False
        return self.field1 == other.field1

def my_map(a: Wrapper) -> Wrapper:
    a.field1 = f"{a.field1}{a.field1}"
    return a

a = Wrapper("field1")
aa = my_map(a)
a.field1 = "field2"
aaa = my_map(a)

```

자, my_map은 Wrapper를 받아 Wrapper를 반환하는 함수이다. input이 같으면 output이 같은가?

이 함수는 같은 input에 대하여 output이 determistic한가?

여기서 input이 같다는 것은 무엇인가? 객체가 같다는 것인가? 객체가 같다는 것은 무엇인가? 동등성(a == b, where a : Wrapper, b : Wrapper)인가 동일성(a is b, a and b has same memory address)인가. 같은 물음은 output에도 적용된다.

State 모나드를 받아 State 모나드를 반환하는 함수가 내부 state를 변환한다면 이 함수는 pure한가?

## implicit parameter 개념 제안

이제부터는 함수형 개념을 익히고 있는 입장에서 정확하지 않은 이해를 바탕으로 가지고 있는 입장을 정리한 것이다.


side-effect가 발생하는 경우 혹은 side-effect가 발생한다고 볼 수 있는 경우는 다음을 포함한다.
- global variable
- closure에서의 free variable(up value), 내부 함수의 scope에서 참조하고 있는 상위 scope 변수
- IO action: 상호작용하는 파일의 내용. 여기서 파일은 (리눅스에서의) 일반화된 파일으로, 일반 파일 뿐 아니라 standard in, standard out인 file descriptor, socket 파일(각종 네트워크), process를 포함한다.
- database: 넓은 의미에서 보면 파일 IO와 같다. database는 데이터를 파일단위로 저장하여 사용하기 때문이다.
- State 모나드의 내부 상태(간단하게, global variable을 사용하는 대신 global variable을 State 모나드에 담고 이 변수를 변화시키는 경우)

그리고 이 side-effect를 해소하는 방법은 다음과 같다.
- global variable
  - 함수의 parameter에 global parameter를 더 넣어주면 된다.
- closure
  - free variable(up value)이 function signature와 실제 함수동작을 바꾸는 원인이므로, free variable을 parameter로 더 넣어주면 된다.
- IO action, database
  - 파일의 내용(넓은 의미에서는 콘솔 입력도 포함)이 실제 함수동작을 바꾸는 원인이므로, 파일의 내용을 parameter로 더 넣어주면 된다.
- State 모나드
  - State 모나드 내부의 상태가 실제 함수동작을 바꾸는 원인이므로, State 모나드 내부의 상태롤 추가적으로 parameter에 넣어주면 된다.

이러한 관점에서 effectful한 함수를 판단하기 위한 조건으로 implicit parameter라는 개념을 제안한다.
여기서 제안하는 implicit parameter는 위의 함수를 pure하게 만드는 과정에서 추가된 parameter이다.

