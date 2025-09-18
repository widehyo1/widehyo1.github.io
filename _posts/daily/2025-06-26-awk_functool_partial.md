---
layout: post
title: functools.partial 구현기 (using awk)
subtitle: implementation of functools.partial with awk
tags: [awk, functional-programming]
comments: true
author: widehyo
---

요새 awk에 대해 많은 관심을 가지고 있다. 특히 awk는 기본적으로 제공되는 feature가 가장 적은 언어중에 하나이기 때문에 다른 언어에서 편하게 사용했던 편의 기능을 직접 구현해서 사용해야 하는 경우가 많다. 하지만 awk 특유의 script스러움과 여러 편의기능 및 문법은 이에 익숙해진 사용자에게 빠져나가기 힘든 매력을 지니고 있기도 하다.

서론은 여기까지 하고 상당히 복잡한 구현이었던 functools.partial을 awk로 구현한 내용을 자세히 살펴보자.

먼저, awk에 대한 이해를 돕기 위해 언어가 가진 제약사항을 먼저 언급하고 가자.
1. 모든 자료형은 string, number, or array(associative array)이다.
2. 데이터를 표현하는 class나 struct를 제공하지 않는다.
3. nested funciton을 지원하지 않는다.
4. 따라서 closure를 지원하지 않는다.
5. array를 return할 수 없다.
6. multi return이 불가능하다.
7. function pointer를 사용할 수 없다 `(void *)`를 이용한 함수 객체 활용이 불가하다.
8. 변수는 기본적으로 global variable이다.
9. (gawk가 아닌 awk 한정) multi dimensional array를 지원하지 않는다.
10. 런타임에서 reflection이나 inspect를 할 수 있는 도구가 제공되지 않는다.

위의 제약사항 중 3, 4, 5, 6번은 C언어가 가진 제약사항을 고려하면 이해가 된다. 그러나 1번과 2번, 그리고 7번은 C언어 보다도 강력한 제약사항이라고 볼 수 있다. 그나마 7번은 gawk(GNU awk) 5.1 버전에서는 functionName = "myfunc"; @functionName을 이용하여 간접호출은 가능하다. 하지만 함수를 반환하거나, 변수에 함수를 할당하거나 파라미터에 함수를 넘기는 것은 불가능하다.
8번의 경우는 그나마 C언어의 함수 스코프가 지원되는 것을 이용하면 함수 내에서 지역변수 취급하고 싶은 변수를 parameter 자리에 넣음으로써 지역변수 취급이 가능하다.

그리고 이를 극복하기 위한 대응방안은 다음과 같다.

1번과 2번의 사용자정의 자료형 문제는 자료형을 표현할 수 있는 문자열을 설계(serialization과 같은 전략)하여 전달하다가 필요한 시점에 해당 문자열을 다시 원하는 형태로 복원하는 방법으로 극복할 수 있다. 이 전략은 posix awk가 multi dimensional array를 지원하지 않는 것에 대한 대응방안으로 `a[i][j]` 대신 `a[i, j]`로 사용한 점에서 착안했다. 편의를 위한 배열 인덱스 자리에 위치하는 `i, j`는 사실 "i\034j" 문자와 같다. 배열의 index에 위치하는 `,`는 키보드로 입력 불가능한 문자인 "\034"로 치환되며, awk에서의 강력한 사용성을 지원하기 위해 내장변수 SUBSEP으로 \034 문자를 사용할 수 있다.

3번과 4번의 경우는 함수형 프로그래밍 방식을 선호하는 필자에게는 많은 불편함을 가져다 주었고, 이번 포스팅의 주된 도전과제였다. 간략히 극복전략을 이야기하자면 global table(associative array)를 이용하여 storage에 원하는 내용을 넣었다가 사용하는 방식으로 극복하였다.

5번과 6번은 생각보다 극복 난이도가 낮았는데, C언어의 call by reference 방식으로 우회하면 된다. C언어에서 array를 sort하기 위해 배열의 포인터를 함수에 넘기고 안에서 swap한 것과 근본적으로 같은 방식이다. 한편, multi return을 위해 새로운 배열을 만들어 return하고 싶을 때는 해당 배열을 parameter로 넘기고 함수 본문 시작시 delete 문을 이용해 우연히 같은 이름을 사용하는 global variable 문제를 방지하였다. 그리고 이 방법은 awk의 기본 함수인 split("text", "array", "separator")에서 자연스럽게 사용하는 방법이기 때문에 awk의 idiom과도 부합한다.

7번은 gawk의 힘을 빌려 @fn 을 이용한다. man awk를 읽다가 @fn을 보고 이거 잘하면 함수형 프로그래밍 할 수 있겠는데 라는 생각이 든 근본적인 모티베이션을 얻었다. 하지만 10번의 제약사항에 의하여 런타임에서 dynamic하게 함수/매개변수에 접근할 수 없었기 때문에 debugger가 참조하는 프레임 객체와 비슷한 것 중 functools.partial에서 필요한 정보만 담은 global variable을 직접 만들어 구현했다.

가장 먼저 진행한 부분은 함수 signature를 저장하는 방법을 찾는 것이었다. functools.partial과 같은 함수는 parameter의 위치 및 key를 이용하여 default parameter를 binding할 수 있었기 때문에 해당하는 정보를 저장해야 했다. 결론만 말하자면 다음과 같은 함수가 있을 때
```
function sample(a, b, c) { ... }
```
[(a, 1), (b, 2), (c, 3)] 과 같은 형태로 함수의 signature가 저장되기를 원했다.
하지만 array의 value는 array가 될 수 없기에, [(a, 1), (b, 2), (c, 3)]의 string representation을 다음과 같이 저장했다.
```
SSEP = "\035"
"1" SUBSEP "a" SSEP "2" SUBSEP "b" SSEP "3" SUBSEP "c"
```
구분자가 2개가 필요했는데 하나는 튜플 안에서 데이터를 구분하는 구분자로 사용된 SUBSEP, 그리고 리스트에서 각 튜플의 구분자로 사용된 SSEP이다. 이렇게 function signature를 string으로 표현하여 global variable인 FUNCSIG 테이블에 함수명을 키로, function signature의 string repr을 value로 저장한다.
그러면 대충 python으로 보면 이런 느낌이 된다.
```py
FUNCSIG["sample"] = "1\034a\0352\034b\0353\034c"
```
그리고 실제로 구조가 필요할 때는 SSEP과 SUBSEP으로 split하여 {"a":1,"b":2,"c":3} 과 같은 형태로 사용한다.

```awk
# save function signature to global variable FUNCSIG
# argument information is of form list of tuples (position, parameter name)
# list element delemter: SSEP, key-value delemeter: SUBSEP
function saveSignature(fnname, argInfo) {
  posRepr = bindPosition(argInfo)
  FUNCSIG[fnname] = posRepr
}

function bindPosition(array,    acc) {
  for (i = 1; i <= length(array); i++) {
    acc = acc i SUBSEP array[i] SSEP
  }
  return substr(acc, 1, length(acc) - 1)
}
```


두번째는 binding된 함수를 만드는 함수이다. 매개변수는 argument를 binding하는 정보를 담고 있는 array(bindings)와 원본 함수명(fnname) 그리고 binding한 함수명(key)으로 구성된다. 함수명을 통한 원본함수는 gawk에서 @fn 문법으로 가능하기 때문에 어떤 함수에 어떤 parameter가 바인딩 되었는지를 정보로 전달하고 binding된 새로운 함수를 key로 global variable BINDING이 가지고 있다가 call 함수에 의해 해당 정보를 찾아 reconstruction을 통해 호출하는 방법이다. 최대한의 유연성을 확보하기 위하여 bindings는 associative array로 설계했고, 위에서 저장한 function signature와 실제 실행 책임을 가진 함수 call에서 호출할 args와 함께 사용된다.

binding한 정보는 `[원본함수명] [(매개변수1, 값1), (매개변수2, 값2), ...]` 와 같은 형태로 저장된다. 물론, awk에는 string과 number만 primitive type으로 가지므로 string representation 형태로 저장한다.

```awk
# save binding arguments to global variable BINDING
# key will be used as partial function(default parameter from binding info)
# the form value consists two parts, head and tails separated with first SSEP
# head contains `original function name`
# tail is of form string representation of the list of tuple, which argName SUBSEP value
# each elements is joined with SSEP
function bind(key, fnname, bindings,    argRepr) {
  argRepr = fnname SSEP

  if (length(bindings) == 0) {
    BINDING[key] = argRepr
    return
  }
  argRepr = argRepr serializeAssoc(bindings)
  BINDING[key] = argRepr
}
```

헬퍼 함수 serializeAssoc을 사용했다.


```awk
function serializeAssoc(assoc,    acc) {
  for (key in assoc) {
    acc = acc key SUBSEP assoc[key] SSEP
  }
  return substr(acc, 1, length(acc) - 1)
}
```

위에서 만든 바인딩된 함수를 호출하는 함수 call이다. 실제 함수호출을 담당하고 내부로직이 가장 복잡한 함수이다. function signature는 다음과 같다.
```awk
function call(key, args,    bindings)
```
위에서 정의한 key로 BINDING에서 argRepr을 가져온다.
argRepr의 첫 번째 위치에 있는 함수명을 추출한다
추출한 함수명으로 FUNCSIG에 저장된 posRepr를 가져온다.

argRepr의 두 번째 위치에 있는 binding 정보를 복원하고 원본함수의 function signature를 담고 있는 posRepr을 각각 복원하여, 실제로 호출할 args와 결합하여 호출한다.

전반부는 실제 파라미터를 생성하는 부분으로 구성된다.
```awk
function call(key, args,    bindings) {
  if (!BINDING[key]) {
    print "unregistered function call"
    return
  }
  buildParameter(key, args, params)
```

buildParameter 함수는 바인딩된 함수를 호출할 args를 우선적으로 적용하고 args에서 채워지지 않은 매개변수는 bindingRepr에서 표현된 매개변수 정보에서 채워넣는다. 만약 모든 parameter가 채워지지 않았다면 에러를 출력한다. multi return을 위하여 params 배열을 이용한다.

```awk
function buildParameter(key, args, params) {
  partition(BINDING[key], SSEP, fnBinding)
  fnname = fnBinding[1]
  inspectParams(fnname, paramDict)
  bindingRepr = fnBinding[2]
  bindingRepr2Dict(bindingRepr, bindingDict)

  for (idx in args) {
    params[idx] = args[idx]
  }
  for (idx in paramDict) {
    if (!params[paramDict[idx]]) {
      params[paramDict[idx]] = bindingDict[idx]
    }
  }
  if (length(paramDict) != length(params)) {
    print "invalid execution, mismatch between original function and binded function"
    delete params
  }
}
```

partition은 헬퍼함수이다. separator를 기준으로 전후를 잘라낸다. 역시 multi return이 필요하기 때문에 headtail 매개변수 배열을 이용한다.

```awk
function partition(str, sep, headtail) {
  headtail[1] = substr(str, 1, index(str, sep) - length(sep))
  headtail[2] = substr(str, index(str, sep) + length(sep))
}
```

bindingRepr2Dict는 string representation을 dictionary(associative array)로 복원하는 함수이다. 내용이 간단하므로 먼저 소개한다. BINDING global 변수와 관련있다.

```awk
function bindingRepr2Dict(repr, bindingDict,    tmp) {
  split(repr, tmp, SSEP)
  for (idx in tmp) {
    split(tmp[idx], keyVal, SUBSEP)
    bindingDict[keyVal[1]] = keyVal[2]
  }
}
```

inspectParams는 원본 함수의 function signature를 담고 있는 FUNCSIG global 변수와 관련있다.

```awk
function inspectParams(fnname, paramDict) {
  posRepr = FUNCSIG[fnname]
  posRepr2Dict(posRepr, paramDict)
}

function posRepr2Dict(posRepr, paramDict,    tmp, posNameTuple) {
  split(posRepr, tmp, SSEP)
  for (idx in tmp) {
    split(tmp[idx], posName, SUBSEP)
    paramDict[posName[2]] = posName[1]
  }
}
```


아래 부분이 비즈니스 로직으로, 명시적으로 호출하는 매개변수를 이용해 원본 함수의 funciton signature를 최대한 채워 넣은 후 비어있는 매개변수에 대하여 bindingDict에서 정의되어 있다면 나머지를 채워 넣는 식으로 작동한다.
```awk
  for (idx in args) {
    params[idx] = args[idx]
  }
  for (idx in paramDict) {
    if (!params[paramDict[idx]]) {
      params[paramDict[idx]] = bindingDict[idx]
    }
  }
```

물론, 명시적으로 주어진 parameter와 binding된 매개변수 모두 사용해도 함수가 요구하는 매개변수를 모두 채우지 못한다면 에러메시지를 출력한다.

```awk
  if (length(paramDict) != length(params)) {
    print "invalid execution, mismatch between original function and binded function"
    delete params
  }
```

사실 위의 코드에서는 exit로 나가도 좋다

이렇게 parameter를 구성하면 남은 작업은 간단하다. 매개변수의 개수에 따라 해당하는 함수를 동적으로 호출하는 부분만 남았다. call 함수의 후반부 코드를 보자.

```awk

  switch (length(params)) {
    case 1:
      executeFunc1(fnname, params)
      break
    case 2:
      executeFunc2(fnname, params)
      break
    case 3:
      executeFunc3(fnname, params)
      break
    case 4:
      executeFunc4(fnname, params)
      break
    default:
      print "#(function params) must one of 1, 2, 3, or 4"
      return
  }
```

여러 helper 함수를 사용하는데, 이것은 awk가 동적 프로그래밍이 불가능하기 때문이다. 쉽게 말하면 eval이 없어서 각종 템플릿에 맞는 함수를 하나씩 사전에 정의해 두어야 한다.

```awk

function executeFunc1(fnname, params) {
  @fnname(params[1])
}

function executeFunc2(fnname, params) {
  @fnname(params[1], params[2])
}

function executeFunc3(fnname, params) {
  @fnname(params[1], params[2], params[3])
}

function executeFunc4(fnname, params) {
  @fnname(params[1], params[2], params[3], params[4])
}
```

이상을 모두 적용한 전체 코드는 다음과 같다.

```awk

function sample(a, b, c) {
  printf "a: %s, b: %s, c: %s\n", a, b, c
}

# save binding arguments to global variable BINDING
# key will be used as partial function(default parameter from binding info)
# the form value consists two parts, head and tails separated with first SSEP
# head contains `original function name`
# tail is of form string representation of the list of tuple, which argName SUBSEP value
# each elements is joined with SSEP
function bind(key, fnname, bindings,    argRepr) {
  argRepr = fnname SSEP

  if (length(bindings) == 0) {
    BINDING[key] = argRepr
    return
  }
  argRepr = argRepr serializeAssoc(bindings)
  BINDING[key] = argRepr
}

function serializeAssoc(assoc,    acc) {
  for (key in assoc) {
    acc = acc key SUBSEP assoc[key] SSEP
  }
  return substr(acc, 1, length(acc) - 1)
}

function bindPosition(array,    acc) {
  for (i = 1; i <= length(array); i++) {
    acc = acc i SUBSEP array[i] SSEP
  }
  return substr(acc, 1, length(acc) - 1)
}

function flip(assoc) {
  for (idx in assoc) {
    assoc[assoc[idx]] = idx
  }
}

function printArray(arr, arrName) {
  for (idx in arr) {
    printf "%s[%s]: %s\n", (arrName ? arrName : "arr"), idx, arr[idx]
  }
}

function apply(arr, fn) {
  for (idx in arr) {
    arr[idx] = @fn(arr[idx])
  }
}

function strReprArr2dict(strReprArr, dict,    strRepr) {
  delete dict
  for (i = 1; i <= length(strReprArr); i++) {
    strRepr = strReprArr[i]
    partition(strRepr, SUBSEP, headtail)
    dict[headtail[1]] = headtail[2]
  }
}

function partition(str, sep, headtail) {
  headtail[1] = substr(str, 1, index(str, sep) - length(sep))
  headtail[2] = substr(str, index(str, sep) + length(sep))
}

function call(key, args,    bindings) {
  if (!BINDING[key]) {
    print "unregistered function call"
    return
  }
  buildParameter(key, args, params)

  switch (length(params)) {
    case 1:
      executeFunc1(fnname, params)
      break
    case 2:
      executeFunc2(fnname, params)
      break
    case 3:
      executeFunc3(fnname, params)
      break
    case 4:
      executeFunc4(fnname, params)
      break
    default:
      print "#(function params) must one of 1, 2, 3, or 4"
      return
  }

}

function buildParameter(key, args, params) {
  partition(BINDING[key], SSEP, fnBinding)
  fnname = fnBinding[1]
  inspectParams(fnname, paramDict)
  bindingRepr = fnBinding[2]
  bindingRepr2Dict(bindingRepr, bindingDict)

  for (idx in args) {
    params[idx] = args[idx]
  }
  for (idx in paramDict) {
    if (!params[paramDict[idx]]) {
      params[paramDict[idx]] = bindingDict[idx]
    }
  }
  if (length(paramDict) != length(params)) {
    print "invalid execution, mismatch between original function and binded function"
    delete params
  }
}

function bindingRepr2Dict(repr, bindingDict,    tmp) {
  split(repr, tmp, SSEP)
  for (idx in tmp) {
    split(tmp[idx], keyVal, SUBSEP)
    bindingDict[keyVal[1]] = keyVal[2]
  }
}

function inspectParams(fnname, paramDict) {
  posRepr = FUNCSIG[fnname]
  posRepr2Dict(posRepr, paramDict)
}

function posRepr2Dict(posRepr, paramDict,    tmp, posNameTuple) {
  split(posRepr, tmp, SSEP)
  for (idx in tmp) {
    split(tmp[idx], posName, SUBSEP)
    paramDict[posName[2]] = posName[1]
  }
}

function executeFunc1(fnname, params) {
  @fnname(params[1])
}

function executeFunc2(fnname, params) {
  @fnname(params[1], params[2])
}

function executeFunc3(fnname, params) {
  @fnname(params[1], params[2], params[3])
}

function executeFunc4(fnname, params) {
  @fnname(params[1], params[2], params[3], params[4])
}

# save function signature to global variable FUNCSIG
# argument information is of form list of tuples (position, parameter name)
# list element delemter: SSEP, key-value delemeter: SUBSEP
function saveSignature(fnname, argInfo) {
  posRepr = bindPosition(argInfo)
  FUNCSIG[fnname] = posRepr
}

function zip(arr1, arr2, dict) {
  minlen = (length(arr1) < length(arr2) ? length(arr1) : length(arr2))
  for (i = 1; i <= minlen; i++) {
    dict[arr1[i]] = arr2[i]
  }
}

BEGIN {
  SSEP = "\035"
  split("a b c", arginfo)
  saveSignature("sample", arginfo)
  split("b c", keys)
  split("5678 asdf", vals)
  zip(keys, vals, bindings)
  bind("partial", "sample", bindings)
  split("1", args)
  call("partial", args)
}
```


적절한 라이브러리에 분리하고 import를 사용하면 다음과 같다.

```awk

@include "common"
@include "functool"


function sample(a, b, c) {
  printf "a: %s, b: %s, c: %s\n", a, b, c
}

BEGIN {
  SSEP = "\035"
  split("a b c", arginfo)
  saveSignature("sample", arginfo)
  split("b c", keys)
  split("5678 asdf", vals)
  zip(keys, vals, bindings)
  bind("partial", "sample", bindings)
  split("1", args)
  call("partial", args)
}


```

출력 결과는 아래와 같다.

```
a: 1, b: 5678, c: asdf
```
