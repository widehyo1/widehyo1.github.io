---
layout: post
title: 하스켈 기본동작 캐싱은 어떻게 이루어질까
subtitle: based on a paper, memo function implementation
tags: [haskell, garbage_collector, computer_science]
comments: true
author: widehyo
---

일단, 제목과는 다르게 하스켈 기본동작 캐싱이 어떻게 이루어졌는지 설명하는 글이 아님을 먼저 밝혀둔다.

https://www.cs.tufts.edu/comp/150FP/archive/simon-peyton-jones/stretching-storage.pdf

하스켈에서 메모리를 관리하는 방법에 대하여 제대로 설명된 자료를 찾아본 적이 없어 하스켈 학교라는 디스코드 채널에 관련 질문을 하니 위와 같은 논문 자료를 받게 되었다.

내용이 어려워 정확하지 않지만 이왕 한 번 논문을 다 읽은 김에 기억나는 것을 정리하는 차원에서 이 글을 남긴다. 아래 내용은 논문의 정확한 이해와는 거리가 멀고 나름의 이해를 위해 느낌 위주로 서술한 감상이기 때문에, 논문의 정확한 이해를 위해서는 위 링크를 통하여 실제로 읽는 것을 권한다.

하스켈의 기본 동작이 캐싱이라고 말하는 것은 어떤 함수가 있을 때, 그 함수가 인자로 받은 argument에 대하여 한 번 실제로 compute하여 result를 반환한 뒤에 다시 같은 argument를 이용하여 함수를 call할 때, 함수를 처음부터 다시 계산하는 것이 아니라, 일종의 (argument, result)로 구성된 hashmap을 사용하여 바로 result를 반환하도록 동작한다는 것을 의미한다.

그리고 논문은 바로 이 동작을 주어진 함수 f에 대하여 같은 동작을 하는(pointwise하게 같은 결과를 반환하는) 함수 memo f를 만드는 것으로 접근한다. 그리고 f가 주어질 때 memo f를 구성하는 방법에 대하여 서술하고 있으며, 이를 위해 4가지 개념을 소개한다. 4가지 개념은 각각 메모리, 포인터 역할을 도입하는 "unsafePerformIO", GC-stable한 객체 referencing type인 "stable names", memo f가 list of argument-result tuple :: [(argument, result)]로 구현이 야기할 수 있는 문제-memo f(실제로는 튜플의 리스트 혹은 hashmap)이 무한정으로 자라나 memory leak이 일어나는 문제-를 해결하기 위해 필요 없는 entry를 적절하게 지울 수 있도록 하는 "weak pointers", 마지막으로 memo f의 기본 개념 (argument, result) list를 변형하여 자신(list의 각 원소인 entry wise한 의미에서)의 result를 스스로 할당해제 할 수 있는 finalizer를 같이 매핑시키는 "Finalization"이다.

하스켈에도 메모리 개념은 존재하고, Eq class 차원에서 같은 객체라도(`이정도면 같다`가 명시적으로 재정의된 객체, 동등성) 실제 객체는 여러개 존재 할 수 있다.(data Node a = Int a, where a :: Int, (Node 1)-denoted n1, memory location is 0x0001, Node 1-denoted n2, memory location 0x0002 for example)

다만, GC(Garbage Collector)의 존재로 인하여 객체의 메모리 주소가 바뀔 수 있기 때문에-GC가 할당된 메모리의 주소를 절대로 바꾸지 않도록 특별히 커스터마이징 하여 작성되지 않았다면-런타임 환경에서 고정된 메모리 주소를 통하여 객체를 찾아가는(dereferencing) 방식으로는 원래 할당된 객체를 다시 얻지 못할 수도 있다. (위의 예에서 0x0001라는 메모리 주소가 런타임에서 세상에 어떤 일이 일어나도-`남산위에저소나무철갑을두른듯바람서리불변한`-반드시 n1객체를 가리키는 메모리 주소라고 보장할 수 없다는 이야기이다. 런타임이 아니라 조금 치사한 예시이기는 하지만, 메모리는 기본적으로 휘발성이기 때문에 컴퓨터를 껐다가 다시 켜면 0x0001은 n1을 다시 가리킬 리가 없고, 이와 같은 현상이 Garbage collector에 의하여 런타임에서도 일어날 수 있다는 말이다.)

그래서 (런타임에서 변경될 수도 있는 메모리주소를 사용하는 대신)GC의 동작에 대해서 불변하는 새로운 타입을 이용하여 객체를 나타내는데, 이 객체를 StableName이라고 하며, System.Mem.StableName 를 쓰면 레퍼런스 비교를 할 수 있다고 한다.

실제 구현은 Hash table과 또 다른 table을 이용하여 offset으로 동작하도록 하여 같은 StableName을 가지는 객체는 반드시 실제로 같은 객체가 되도록 했다고 설명하는데, 정확한 원리는 모르겠다.

그리고는 memo f가 사용할 데이터 구조를 hashmap :: [(argument, result)]에서 Weak라는 개념을 도입하여 [(argument, (Weak result)]로 확장하더니, finalizer에 대해 이런 저런 얘기를 한 다음에는 [(argument, (Weak result), (finalizer result)]를 통해 memo f를 구현한다는 것이 요지인 것 같다.

역시 말로 적으면서도 모르겠다.
