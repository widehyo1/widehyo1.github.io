---
layout: post
title: 알고리즘 풀이 모음
subtitle: algorithm solutions
tags: [algorithm]
comments: true
author: widehyo
---


## 7/15
`kakaoid.py`
```py
from itertools import groupby

def solution(new_id):
    vaild_special = {"-", "_", "."}
    new_id = new_id.lower()
    new_id = ''.join(char for char in new_id if char.isalnum() or char in vaild_special)
    new_id = ''.join(gen_group_dot(new_id))
    if new_id == ".":
        new_id = ""
    if len(new_id):
        if new_id[0] == ".":
            new_id = new_id[1:]
        if new_id[-1] == ".":
            new_id = new_id[:-1]
    if not new_id:
        new_id = "a"

    if len(new_id) > 15:
        new_id = new_id[:15]
        if new_id[-1] == ".":
            new_id = new_id[:-1]
    if len(new_id) == 2:
        new_id = new_id + new_id[-1]
    if len(new_id) == 1:
        new_id = new_id * 3
    return new_id


def gen_group_dot(new_id):
    for groupper, iterator in groupby(new_id):
        if groupper == ".":
            yield "."
            continue
        yield from iterator


# new_id = "...!@BaT#*..y.abcdefghijklm"
# new_id = "z-+.^."
# new_id = "=.="
# new_id = "123_.def"
new_id = "abcdefghijklmn.p"
print(solution(new_id))
```

---

`trailingzeros.py`
```py
def trailing_zeros(n):
    if n == 0:
        return 0
    sieve = list(range(1, n + 1))
    res = 0
    while sieve:
        sieve = [item // 5 for item in sieve if item % 5 == 0]
        res += len(sieve)
    return res

# print(trailing_zeros(0))
# print(trailing_zeros(1))
print(trailing_zeros(5))
```

---

`tribonacchi.py`
```py
def tribonacchi(n):
    if n == 0:
        return 
    if n == 1:
        return 1
    if n == 2:
        return 1
    sol = [0, 1, 1] + [-1]*34
    for i in range(3, n + 1):
        sol[i] = sol[i-1] + sol[i-2] + sol[i-3]
    return sol[n]

for i in range(10):
    print(tribonacchi(i))

print(tribonacchi(25))
```


---

`wordbreak.py`
```py
def sol(s, wordDict):
    idx = 0
    backlen = 0

    word_set = set(wordDict)
    tries = set()

    def backtrack(str_, idx, backlen):
        print(f"{str_=}")
        print(f"{idx=}")
        print(f"{backlen=}")
        # base condition
        if str_ in word_set:
            print(f"flag!!!")
            return True
        if (str_, idx, backlen) in tries:
            return False
        for item in wordDict:
            if str_.startswith(item):
                print(f"{item=}")
                n = len(item)
                if backtrack(str_[n:], idx + n, n):
                    return True
        else:
            tries.add((str_, idx, backlen))
            return False

    return backtrack(s, idx, backlen)

if __name__ == '__main__':
    # s = "catsandog"
    # wordDict = ["cats","dog","sand","and","cat"]
    # s = "leetcode"
    # wordDict = ["leet","code"]
    # s = "cars"
    # wordDict = ["car","ca","rs"]
    s = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab"
    wordDict = ["a","aa","aaa","aaaa","aaaaa","aaaaaa","aaaaaaa","aaaaaaaa","aaaaaaaaa","aaaaaaaaaa"]
    res = sol(s, wordDict)
    print(res)
```
