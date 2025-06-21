---
layout: post
title: roundrobin by generator
subtitle: 제네레이터를 활용하여 라운드로빈을 구현해보자
tags: [python, generator]
comments: true
author: widehyo
---

오랜만에 마음에 드는 코드를 작성했다.

```py
from operator import itemgetter
from itertools import (
    zip_longest as zl,
    groupby,
)

SENTINEL = object()

def gen_roundrobin(*iterables):
    return (item for tup in zl(*iterables, fillvalue=SENTINEL)
                 for item in tup if item is not SENTINEL)

def gen_transpose(*iterables):
    it_roundrobin_with_index = (
        (idx, item)
        for idx, tup in enumerate(zl(*iterables, fillvalue=SENTINEL))
        for item in tup if item is not SENTINEL
    )
    for _colidx, it_col in groupby(it_roundrobin_with_index, itemgetter(0)):
        yield (item for _, item in it_col)

a = [1, 2, 3]
b = [4, 5]
c = [6, 7, 8, 9]
print(list(gen_roundrobin(a, b, c)))
for it_col in gen_transpose(a, b, c):
    print(it_col)
    print(list(it_col))
```


결과
```console
[1, 4, 6, 2, 5, 7, 3, 8, 9]
<generator object gen_transpose.<locals>.<genexpr> at 0x7fb3686c4f90>
[1, 4, 6]
<generator object gen_transpose.<locals>.<genexpr> at 0x7fb3686c5000>
[2, 5, 7]
<generator object gen_transpose.<locals>.<genexpr> at 0x7fb3686c4f90>
[3, 8]
<generator object gen_transpose.<locals>.<genexpr> at 0x7fb3686c5000>
[9]
```
