---
layout: post
title: 펑터를 이해하려고 적는 기록
subtitle: understanding functor
tags: [functional_programming]
comments: true
author: widehyo
---

## Functor를 보면서 느끼는 감상

### Functor가 하고자 하는 것
결국 내가 가장 잘 이해하는 것을 이용하여 다른 구조도 이해하고 싶다가 핵심인 것 같다.

```Functor: 나는 일반 값과 일반 함수가 있는데 이걸 다른 구조에도 적용하기를 원한다.```

내가 일반적인 세계의 일반 값과 일반 함수를 이해하고 이것이 다른 구조에도 그대로 적용될 것이라고
충분한 확신을 가지게 되면 내가 이해하고 있는 일반적인 세계를 이용하여 다른 구조(문맥)에도 같은 방식으로
동작할 것이라고 예상을 할 수 있게 된다. 이것은 내가 속해 있는 일반적인 세계에서의 행위(일반 함수)가
다른 세계(문맥)에서도 내가 납득할 수 있을 정도로 충분히 비슷하게, 구조를 보존하며 행동하리라 예상한다.
이러한 상황에서 내가 속해 있는 일반적인 세계에서의 행위를 통해 다른 세계를 내가 원하는 방식대로 조작할 수
있게 되고 일반 세계에서 (내가 충분히 이해하고 있는) 조작은 다른 세계를 조작하는 하나의 도구가 된다.

>도구(툴)가  손에 익어  도구가 자신의 손만큼 자유롭고 신체일부처럼 느껴지면  연장이 된다.

(... 도구(툴)가  손에 익어  도구가 자신의 손만큼 자유롭고 신체일부처럼 느껴지면  연장이 된다. 라는 논지의 말이 매우 익숙한데, 원 출처가 어디인지 찾으려다가 못 찾았다. 누군가 알고 있다면 알려주길 바란다.)


맞는 비유일지는 모르겠지만 물리 기기로서의 마우스의 움직임과 모니터 상의 마우스 포인터의 움직임을 생각해보자.
현실의 물리세계를 살아가는 우리들은 운동과 힘 등 각종 물리 법칙을 체감하고 있다. (공식 등으로 나타나는 학문적인 이해가 아니라 실제로 하더니 되더라 라는 좀더 경험적인 의미에서 이러한 물리 법칙을 알고 있다.)
내가 마우스를 오른쪽으로 움직이면 모니터 상의 마우스 포인터도 오른쪽으로 움직인다. 그리고 우리는 이 현상에 보통 크게 신경쓰지 않는다. 이것이 바로 위에서 인용한 도구가 손에 익어 자유롭고 신체 일부처럼 느껴지는 상태이고 이 상태가 되면 우리는 이러한 현상(물리적 마우스의 움직임을 통해 모니터 상의 마우스 포인터를 움직이는 것)을 명백히 다른 행위임에도 불구하고 동일시하게 된다.

우리가 이해하고 있는 것은 물질 세계이며, 물질 세계(카테고리 C)에서의 마우스(대상 a)를 오른쪽으로 움직(f :: a -> b)이는 행위를 통하여 모니터 안의 세계(카테고리 D)에서의 마우스 포인터(대상 a_D)를 오른쪽으로 움직(f_F :: a_F -> b_F)이고 있는 것이다.

현실의 물리세계와 모니터 안의 세계를 잠시 걷어내고 구조에만 주목하자.
우리는 잘 알고 있는 대상과 그 대상에 대한 매핑이 있다. 이미 잘 알고 있는 구조를 통하여 다른 구조를 파악하는 것은 인류가 아주 잘 하는 종류의 생각이고 우리는 이러한 생각에 매우 익숙하다(ex. [하스스톤의 전설랭크는] 롤로 치면 챌린저 - 잘 알고 있(다고 가정한)롤의 랭크 시스템을 이용하여 잘 모르는 하스스톤의 랭크 시스템을 이해). 우리는 이것을 통해 다른 컨텍스트에서의 구조를 보존 하고 싶어 한다. 그리고 만약 다른 구조(category D, target context)에서의 동작(a_D, f_D :: a_D -> b_D)이 내가 알고 있는 구조(cateogry C, well known context)에서의 동작(a, f :: a -> b)과 어떤의미에서 동일하게 동작하면(there exists a homomorphism) 우리는 그 구조에 대하여 우리가 알고 있는 매핑과 property를 모두 알 수 있다. 이것이 수학의 강력함이고 어떤 구조가 보존된다는 것만 보장되면 우리는 그 구조에 대한 모든 property를 다른 context에서도 그대로 사용할 수 있다.

내가 알고 있는 Functor중 그나마 이해도가 있는 것은 Maybe와 List이므로 Maybe와 List에서 이것을 확인해보자.
Int에서의 +와 *에 대해서는 우리는 충분히 알고 있다. 그리고 이 구조를 Maybe에서도 적용하고 싶다.

그러면 Int(물리적 도구로서의 마우스)와 Maybe Int(모니터 안의 마우스포인터)를 어떤 의미에서 동일하다고 생각하고 그것이 말이 되는지 검증하자.

### Maybe Functor, List Functor

먼저 Int를 어떻게 Maybe Int로 볼 것인지를 결정해야 하고, 이것은 매우 직관적이다.
1 -> Just 1, 2 -> Just 2, ...
단, Nothing에 대해서는 일대일 대응이 되는 Int가 존재하지 않는다. 그러니 이 사실을 염두해 두고 다음 작업을 진행하자.
Int의 (+3) (f :: Int -> Int)에 대하여 생각해 보자.
(+3) 1 = 4이고 우리는 이것이 Maybe context에서도 그대로 동작하기를 바란다
그렇다면 가장 직관적으로는 Just와 Just의 +를 일반 값들 간의 +로 동작하면 좋을 것 같다.
(+3) Just 1 = Just 4
조금 더 살펴보면 우리가 실제로 한 것은 다음과 같다.
(+3) Just 1 = Just 4
            = Just (+3) 1

fmap : (a -> b) -> a -> f a -> f b
단, f는 Functor

그러나 Maybe라는 context는 Nothing이라는 요소도 있으니 따로 고려해 주어야 한다.
이것을 어떻게 처리하면 좋을까? 입력이 Nothing이면 어떤 일반함수 f를 가져오더라도 Nothing을 반환하도록 처리하자
_ Nothing = Nothing
그러면 잘 동작(there exists an identity 0 for the operator +, and it holds associateive law)한다.

아래의 두 논지는 수학에서의 엄밀한 증명이 아니다. 이해를 돕기 위한 식이고 논리간에 대충 같은 것으로 치고 다음을 전개하는 방식으로 쓰여졌다.

```hs
data Maybe a = Nothing | Just a
```

```hs
instance Functor Maybe where
    fmap :: (a -> b) -> a -> Maybe a -> Maybe b
    fmap _ Nothing = Nothing
    fmap f Just x = Just (f x)
    ```

잘 동작: 아래의 Claim1과 Claim2가 성립한다.
주어진 일반함수 f에 대한 id를 maybe에서의 id인 id_maybe와 일반 값에서의 id로 구분하여 표기하자.
```hs
id_maybe :: Maybe a -> Maybe a
id_maybe Nothing = Nothing
id_maybe Just x = Just x
```

```hs
id :: a -> a
id x = x
```

Claim1: fmap id = id_maybe
fmap id가 pointwise하게 id_maybe와 같음을 보이면 된다. 즉, 다시 말하면
fmap id와 id_maybe가 Maybe a 타입의 같은 입력일 때 같은 결과를 반환한다는 것을 보이면 된다.

1) Nothing인 경우
```hs
fmap id Nothing
    = Nothing -- by the definition of fmap, fmap _ Nothing = Nothing
    = id_maybe Nothing -- by the definition of id_maybe
    ```

2) Just인 경우
```hs
fmap id Just x
    = Just (id x) -- by the definition of fmap
    = Just x -- by the definition of id
    = id_maybe Just x -- by the definition of id_maybe
    ```

1)과 2)의 첫 번째 식에서 fmap id 부분만 따로 떼어 놓고 보면 마지막 식의 id_maybe와 pointwise하게 같은 함수임을 알 수 있다.

Claim2: fmap (g . f) = fmap g . fmap f

Functor의 context가 단순히 일반 값에 context를 준 것이 아니라 일반세계에 대한 구조를 보존한다는 것을 보이는 부분이다.
이것이 성립하면 우리는 일반세계에 대한 구조가 Functor가 준 context(일종의 boxing)에서도 만족한다는 것을 확신할 수 있으면 일반세계를 통해 context가 씌워진 세계를 이해할 수 있다. (마우스 포인터의 행동을 이해하는 데에는 물리적 마우스의 움직임만 예측하고 이해하면 되었던 것 처럼)

단, f :: a -> a, g :: a -> a, g . f :: a -> a

역시 이 상황에서도 두 함수 fmap (g . f) :: Maybe a -> Maybe a, fmap f . fmap b :: Maybe a -> Maybe a 가 pointwise하게 같은 함수임을 보이면 된다.

1) Nothing인 경우
```hs
fmap (g . f) Nothing
    = Nothing -- by the definition of fmap, fmap _ Nothing = Nothing
    = fmap g Nothing -- by the definition of fmap, fmap _ Nothing = Nothing
    = fmap g (fmap f Nothing) -- Nothing = fmap f Nothing
    = fmap g . fmap f Nothing
    = (fmap g . fmap f) Nothing
    ```

2) Just인 경우
```hs
fmap (g . f) Just x
    = Just ((g . f) x) -- by the definition of fmap, fmap f Just x = Just (f x)
    = Just (g (f x)) -- by the definition of composition, g . f x = g (f x)
    = fmap g Just (f x) -- by Just (g a) = fmap g Just a, definition of fmap, a = f x
    = fmap g (fmap f Just x) -- by Just (f x) = fmap f Just x
    = fmap g . fram f Just x -- by the definition of composition, g (f x) = g . f x
    = (fmap g . fram f) Just x
    ```

1), 2)에 의하여 fmap (g . f)는 fmap g . fmap f 와 pointwise하게 같은 함수이다.

한번 더 연습해보자.
```hs
data List a = Nil | Cons a (List a)
```

```hs
instance Functor List where
    fmap :: (a -> b) -> a -> List a -> List b
    fmap _ Nil = Nil
    fmap f (Cons head tail) = (Cons f head) fmap f tail
    ```

여기서 tail은 List a 타입이다.

역시 일반 함수 f :: a -> a에 대한 identity 함수 id와 List에서의 identity 함수 id_List를 생각하자.
정의에 의하여 다음이 성립한다.
```hs
id :: a -> a
id a = a
```

```hs
id_List :: List a -> List a
id_List Nil = Nil
id_list Cons head tail = Cons head tail
```

위처럼 정의한 fmap이 List라는 context에 대하여 잘 동작함을 보이면 된다.

Claim1) fmap id = id_List

위에서 보인 것 처럼 fmap id :: List a -> List a가 id_List와 pointwise하게 같은 함수임을 보이면 된다.

1) Nil인 경우
```hs
fmap id Nil
    = Nil -- by the definition of fmap, fmap _ Nil = Nil
    = id_List Nil -- by the definition of id_List
    ```

2) x = Cons head tail인 경우
Cons head tail은 다음과 같이 쓸 수 있다.
Cons head (Cons head_1 (Cons head_2 (Cons head_3 ...
편의를 위하여 Cons head tail을 다음과 같이 나타내자
```hs
x = Cons head tail
  = Cons x_0 tail
  = Cons x_0 (Cons x_1 (Cons x_2 ...
  ```

```hs
fmap id Cons x_0 tail
    = (Cons id x_0) fmap id tail -- by the definition of fmap, fmap f (Cons head tail) = (Cons f head) (fmap f tail)
    = Cons x_0 fmap id tail -- by the definition of id, id a = a
    = Cons x_0 fmap id (Cons x_1 tail_1) -- where tail_1 = Cons x_2 (Cons x_3 ...), 약속한 표기에 의하여
    = Cons x_0 Cons (id x_1) fmap f tail_1 -- by the definition of fmap
    = Cons x_0 Cons x_1 fmap tail_1 
    = ...
    = Cons x_0 (Cons x_1 (Cons x_2 ...
    = x -- 약속한 표기에 의하여
    = id_List x -- by the definition of id_List
    ```

1), 2)에 의하여 fmap id :: List a -> List a는 id_List :: List a -> List a와 pointwise하게 같다.

Claim2) fmap g . f = fmap g . fmap f

역시 fmap g . f :: List a -> List a와 fmap g . fmap f :: List a -> List a가 pointwise하게 같은 함수임을 보이면 된다.

1) Nil인 경우
```hs
fmap g . f Nil
    = Nil
    = fmap f Nil -- by the definition of fmap, fmap _ Nil = Nil
    = fmap g Nil -- by the definition of fmap, fmap _ Nil = Nil
    = fmap g (fmap f Nil) -- by Nil = fmap f Nil, above
    = fmap g . fmap f Nil -- by the definition of composition
```

2) x = Cons x_0 (Cons x_1 (Cons x_2 ... 인 경우

```hs
fmap g . f Cons x_0 tail
    = Cons (g . f) x_0 fmap (g . f) tail -- by the definition of fmap
    = Cons (g . f) x_0 fmap (g . f) Cons x_1 tail_1 -- where tail_1 = Cons x_2 (Cons x_3 ...
    = Cons (g . f) x_0 Cons (g . f) x_1 fmap (g . f) Cons x_2 tail_2 -- where tail_2 = Cons x_3 (Cons x_4 ...
    = Cons (g . f) x_0 (Cons (g . f) x_1 (Cons (g . f) x_2 ...
    = Cons g (f x_0) (Cons g (f x_1) (Cons g (f x_2) ...
    = fmap g y -- where y = Cons y_0 (Cons y_1 (Cons y_2 ..., where y_0 = f x_0, y_1 = f x_1, ...
    = fmap g (fmap f x)
    = fmap g . fmap f x
```

1), 2)에 의하여 fmap g . f :: List a -> List a와 fmap g . fmap f :: List a -> List a는 pointwise하게 같은 함수이다.


### Functor: 타입을 받아 타입을 반환하는 함수, boxing, adding context

Maybe에 대하여 생각해보자. Maybe 자체는 concrete한 type이 될 수 없으며, 타입변수 a를 받아 concrete한
type인 Maybe a를 만든다.

이렇게 보면 Maybe는 타입변수 a를 받아 다른 타입 Maybe a를 반환하는 함수이고 함수 시그니처는 다음과 같다
Maybe :: a -> Maybe a

타입을 타입으로 반환하는 함수가 Functor이며 Functor law라는 것은 이렇게 정의한 context(혹은 boxing)이 원본 대상에 대한 동작을 보존한다고 여기기 위한 조건이다. 따라서 Functor에서 Functor law를 그렇게 중요하게 여기는 이유는 Functor law가 원본 카테고리에 적용된 매핑이 다른 카테고리(context)에서도 내가 원하는(예상하는) 방향으로 동작하는 조건이고 이래야 우리는 우리가 다루는 새로운 context를 백지 상태에서 처음부터 이해하는 노력을 기울이는 대신 기존에 잘 알고 있는 원본에 대한 이해를 활용할 수 있기 때문이다.


출처:
[[번역] 프로그래머를 위한 카테고리 이론 - 7. 펑터](https://evan-moon.github.io/2024/03/15/category-theory-for-programmers-7-functors/)
가장 쉬운 하스켈 책: 느긋하지만, 우아하고 세련된 함수형 언어(원제: Learn You a Haskell for Great Gool!: A Befinner's guide)

