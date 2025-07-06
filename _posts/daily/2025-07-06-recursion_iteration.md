---
layout: post
title: 프로그래밍 언어의 이해: 재귀와 반복, 기본 collection
subtitle: programming language: recursion, iteration and basic collection
tags: [programming language, recursion, iteration, collection]
comments: true
author: widehyo
---

프로그래밍 언어의 최소 조건 중 하나는 조건문과 반복문의 존재이다. 이 중 반복문은 어셈블리 수준으로 내려가지 않는 한 크게 두 가지 중 하나의 메커니즘을 지원한다.
첫 번째는 for문을 이용한 이터레이션(iteration)이고 다른 하나는 재귀(recursion)이다. 일반적인 경험에 비추어 볼 때, 재귀만 제공하는 언어는 보편적이라고 하기는 어렵고, 함수형 프로그래밍에 특화된 언어에서 주로 볼 수 있다. 그 중에서는 아예 문법적으로 for문이 존재하지 않는(...) 경우도 존재한다. 후술할 내용에서 반복은 repeatation, 반복자체를 의미하고 반복을 구현하기 위한 방법으로써의 반복은 이터레이션(iteration)으로 표기하되, 두 용어가 명시적으로 구분될 필요가 있는 경우에는 영문표기를 병기한다.

논의를 더 전개하기 전에, 이터레이션과 재귀의 구현에 대해서 생각해보자. <summary>저수준으로 내려가면 결국 재귀도, 이터레이션도 state machine으로 구현된다. 단, 재귀는 jmp와 goto문으로 이터레이션은 내부 state값을 이용한다.<details>
이해를 돕기 위하여 어셈블리에서 반복(repeatation)을 구현한 방법을 state machine으로 이해해보자.
필자가 생각할 수 있는 가장 저수준 단계인 언어인 어셈블리에서는 반복을 구현하기 위해 두 가지 방법을 이용한다.
첫 번째는 goto와 label을 이용하는 방법이다. 프로그램 코드에 특정 위치에 이름을 붙이고(label) 주로 두 값을 비교(cmp 명령문)하여 컨트롤 레지스터(ZF: zero flag)에 값을 설정하고 해당 플래그에 기반하여 goto문으로 분기처리(jmp 계열 명령문)하는 방법이다.
두 번째는 loop {label}를 이용한 편의구문이다. 어셈블리는 특정 횟수만큼 반복하기 위해 ecx 레지스터에 반복할 카운트를 넣어놓고 아래에 label을 설정한 다음 loop {label} 형태의 instruction을 작성한다. 그러면 라벨부터 loop 까지의 블럭이 반복되며, 한 번 반복될 때마다 ecx에 담긴 값을 1씩 차감하고 해당 값이 0이되면 반복을 종료하는 식으로 동작한다.
반복(repeatation) 구현의 가장 초기라고 할 수 있는 위의 두 형태는 각각 state machine으로 볼 수 있다. 반복(repeatation)을 위해 jmp와 label을 이용하는 첫 번째의 경우 조건처리 후 빠져나오는 역할을 하는 instruction이 존재하기 마련이며(없다면 무한루프에 빠질테니까) 이때 비교하는 값을 state로 가지는 state machine으로 볼 수 있다. ecx의 값을 이용하는 두 번째 경우도 ecx의 값을 state로 가지는 state machine으로 취급할 수 있다.</details>
</summary>
이를 반복과 재귀의 관점에서 바라보면 jmp와 goto문을 이용한 방법은 재귀에 더 가깝고, 내부 state값을 이용한 방법은 c언어에서의 가장 기초적인 for문인 `for(int i = 0; i < n; i++) { /* block */ }`과 가깝다. 둘 모두 내부적으로 state machine을 이용한다면 왜 반복과 재귀가 그렇게 다른 모습으로 나타나는지 궁금할 수도 있다. 이유를 이해하기 위해서는 조금만 더 기다려보자. 약술하자면, 재귀는 보통 function call을 이용하기 때문에 사용되는 상황과 저수준에서의 동작이 많이 달라지기 때문이다.

반복의 구현으로써의 이터레이션과 재귀를 더 논의하기 전에 실제 프로그래밍 상황에서 반복이 필요한 경우를 잘 생각해보자. 구구단을 구현하거나 `*`을 이용해 삼각형을 출력하는 경우가 아니라면 일반적인 상황에서 반복이 필요한 경우는 collection을 기반한다. 특정 collection에 해당하는 변수를 순회하면서 반복하거나 DB에서 fetch한 rows를 기반으로 반복하거나, 아니면 파일시스템의 특정 path이하의 file 목록을 대상으로 반복하는 경우가 일반적이다. 이런 경우 실제 반복과정에서 반복시 변화하는 counter의 값이 중요한 경우보다 해당 counter값을 이용해 참조하는 collection의 element가 더 중요한 경우가 압도적으로 많다. 물론 예외도 있는데, 여러 배열을 한꺼번에 순회하고 싶은 경우나 순서가 중요한 정렬의 경우가 그러하다. 그러나 해당하는 경우는 고급 언어로 갈수록 원했던 바로 그 기능을 언어차원에서 제공하기 시작하고, 개발자는 그 방법만 정의하여 넘기는 형태의 방식이 점점 많아지게 된다. 그러므로 이후 내용에서 다루는 반복은 collection에 기반한, 해당 collection을 순회하는 반복을 주 대상으로 한다.

그럼 collection에 기반한 반복에 대해서 생각해보자. 반복에 있어서 기초가 되는 collection은 크게 두 가지 형태로 나타난다. 하나는 인접한 메모리에 collection의 각 원소를 차례로 저장하는 array형 기반 collection이다. 다른 하나는 서로 다른 메모리에 각 원소가 저장되며 next pointer를 함께 저장하는 linked list형 기반 collection이다.

일반적인 개발자가 친숙한 형태는 array 기반의 collection이며, 주로 C, C++, Java, python, javascript 등의 언어에서 collection 기반의 반복에서 사용된다. array 기반 collection의 장점은 데이터의 응집도에서 오는 성능상의 이득과 (같은 타입으로 저장되는 경우) n번째 원소에 O(1)으로 접근할 수 있다는 점이다. 전자는 결국 컴퓨터가 정보를 읽을 때 연속된 block단위(주로 4kb)로 reading이 일어나기 때문에 한 곳에 필요한 데이터가 모여있으면 각 원소를 읽을 때마다 새로 읽을 필요 없이 한번 읽을 때 가져온 block에서 그대로 가져올 수 있는 것이 근본적인 원인이고 후자는 가상메모리라는 subtle한 부분이 있긴 하지만 기본적으로는 collection의 메모리 시작 위치에서 각 원소의 unit size와 index를 곱해 한 번에 원하는 원소가 있는 memory에 접근할 수 있다는 사실에 기반한다. Java는 ArrayList와 LinkedList를 모두 제공하고, python과 javascript는 collection이 서로 다른 type을 가질수 있다는 점은 여기선 잠깐 넘어가자.

linked list기반의 collection은 주로 함수형 언어인 haskell, clojure, F# 등의 언어에서 사용되며, 메모리와 지연성측면에서 강점을 가지기 때문에 사용된다. 잠깐 자료구조를 배운 내용을 생각해보자. linked list의 기본 단위(atom)은 바깥쪽으로 향하는 간선이 최대 한 개인 노드(outdegree <= 1)이다. 기본적으로 각 노드는 객체취급받기 때문에 메모리와 사이즈를 이용해 한번에 접근하는 것이 불가능하다. 그래서 접근에 이론적으로 O(n)의 시간복잡도가 소요된다. 가장 많이 일어나는 연산인 접근(access)연산에 O(n)이 걸리는 것을 보고 array 기반의 O(1)을 놔두고 쓸 일이 없겠다고 생각할 수 도 있으나, linked list 기반 collection은 다른 강점을 가지고 있다. 대표적으로는 무한을 다룰 수 있다. array 기반 collection은 무한의 memory를 사용할 수 없기 때문에 무한 크기의 collection을 다룰 수 없다.(OS에게 무한의 memory를 요구해도 안돼, 돌아가 라는 소리를 들을 것이다) 반면, linked list는 순회시 한 번에 하나의 원소만 런타임에 존재하면 되기 때문에 필요할 때마다 해당 원소를 가져오거나 만드는 식으로 무한을 다룰 수 있다. 그리고 linked list 기반 collection은 *하나의 원소가 전체 list를 대표할 수 있다*는 특징이 있다. 하나의 원소가 어떻게 전체 list를 대표할 수 있는지 생각해 보면, 해당 원소로부터 다음 원소를 가리키는 메모리를 통해서 순회하는 방법이 있으니 하나의 원소를 해당 원소를 HEAD로 하는 linked list로 간주하지 못할 이유가 없다.

array 기반 collection보다는 linked list 기반 collection이 더 생소할 것이므로 linked list 기반 collection에 대해 조금 더 이야기하자. linked list의 atom이 되는 바깥쪽으로 최대 1개의 간선을 가지는 노드로 구성된 linked list의 모습을 떠올려보자. head node를 하나의 node로 보고 head node가 가리키는 node를 해당 노드를 head로 하는 linked list로 간주하면 linked list는 head 노드와 tail linked list로 분리될 수 있다. 그래서 함수형 프로그래밍에서 x:xs나 (cons 69 (cons 613 nil)) 와 같은 linked리스트를 concat하는 문법을 많이 볼 수 있는데, 근본적인 이유는 collection이 linked list형 기반이기 때문에 이런 방식으로 접근하는 것이 가장 자연스럽기 때문이다.

다시 돌아와서 반복(repeatation)의 구현으로써의 이터레이션과 재귀에 대해 생각해보자. 그 중에서 첫 번째 주제는 이터레이션이다. 앞서 linked list는 지연평가를 이용하여 무한을 다룰 수 있다고 했다. 그러나 무한을 다룰 수 있는 이유는 linked list 기반 collection을 사용하기 때문은 아니다. 정말로 중요한 것은 런타임에 모든 데이터를 메모리에 로드하지 않는다는 것이다. 이것이 지연 평가(lazy evaluation)의 힘이다. 그리고 iteration은 이를 가능하게 한다. iteration에서 중요한 것은 현재의 상태값을 기반으로 다음값을 가져오는 방법이다. 그러한 방법만 있다면 전체 collection을 메모리에 올리지 않고도 collection을 순회하면서 원하는 작업을 할 수 있다. 그 구체적인 방법은 각 언어마다 다르게 구현되는데, python에서는 __next__메서드를 통해, C++은 ++연산자의 연산자 오버로딩을 통해 다음 원소를 가져오는 방법을 지정하면 원하는 형태로 동작하는 iterator를 만들 수 있다. 그리고 이렇게 iterator를 만드는 방법은 언어가 지원을 하더라도 번거롭기 때문에, java, javascript, python 등의 언어에서는 generator, yield 키워드를 제공하기도 한다.

두 번째 주제는 재귀이다. 재귀는 자신을 정의할 때 자기 자신을 재참조하는 방법을 뜻하며, 이를 프로그래밍에 적용한 재귀 호출(recursive call)의 형태로 많이 사용된다. 앞서 재귀는 function call을 이용하기 때문에 저수준에서의 동작이 많이 달라진다고 말했는데 이에 대해 이야기해보자. 대부분의 main 함수로부터 시작해서 각 scope에 해당하는 call stack을 쌓거나 빼내면서 실행되게 된다. 각 function call은 새로운 frame을 stack에 넣는데, 저수준에서는 해당 함수의 context를 frame의 형태로 구성하고 현재 stack에 링크드 리스트 형태로 연결하는 식으로 동작하게 된다. 많은 언어에서는 이런 call stack에 frame을 쌓는 작업이 보통 비싼 작업이기 때문에 (간단히 생각해도 function scope에서 사용하는 변수를 메모리에 할당하는 작업, call stack에서 파라미터를 넘기는 과정(어셈블리에서의 push와 pop), 그리고 call stack 자체는 메모리를 요구하며 메모리를 할당할 수 있는 권한은 OS 시스템콜으로 비싼 작업이다) 재귀호출은 maximum recursion depth가 정해져있는 경우가 많다. 그래서 재귀를 적극적으로 활용하는 함수형 프로그래밍 언어들은 런타임의 콜스택이 너무 커지지 않도록 컴파일러 단에서 특별한 기능을 제공하기도 하는데, 그 중의 하나가 꼬리재귀 최적화(Tail Call Optimization)이다. 꼬리재귀 최적화는 쉽게 말하면, 재귀호출이 함수의 마지막 부분에만 존재하는 경우(재귀호출 이후의 어떤 코드도 없는 경우) 콜스택을 쌓지 않고, 파라미터만 바꾼 함수로 갈아끼우는 동작으로 replace함으로써 tail call이 무한정 자라지 않도록 하는 방법이다. python은 해당 기능을 지원하지 않고, Java에서는 Function 타입을 이용하여, Kotlin은 tailrec 키워드를 이용해서, javascript는 ES6의 도입으로 지원한다.

한편, 결국 반복(repeatation)이 state machine임을 이해하면, 직접 state machine을 코드내에 구현함으로써 반복을 직접 구현할 수도 있다. 이론적으로 재귀와 이터레이션은 서로 동등하며, 이터레이션으로 구현될 수 있는 코드는 재귀로도 구현 가능하며 반대도 성립한다. 그리고 이에 착안하면 tail call recursion을 지원하지 않는 python에서도 recursion과 동등한 이터레이션 코드를 state machine을 이용하여 직접 구현하면 언어의 한계를 우회할 수 있다. 이때 state machine이 저장할 변수가 하나인지 아니면 동적으로 변경되는 collection인지에 따라 하나의 변수를 검사하는 while문으로 구현하거나 collection이 필요한 경우 while stack의 형태로 구현하게 된다.

반복(repeatation)의 구현으로써의 이터레이션과 재귀 그리고 기반 collection에 대한 이해는 프로그래밍 언어와 저수준에서의 동작 이해도를 높이는 데 도움이 될 수 있을 것이다.

