---
layout: post
title: 알고리즘문제풀이
subtitle: algorithms
tags: [algorithm]
comments: true
author: widehyo
---

## 7/24
- 답안 버전1
```py
from typing import NamedTuple

class Walker(NamedTuple):
    idx: int
    zero_to_fill: int
    possible_reprs: tuple

def count_of_one(bit_repr):
    return sum(1 for bit in str(bin(bit_repr))[2:] if bit == '1')

def fill_zero(idx, possible_reprs):
    """
    for each possible_repr in possible_reprs:
    see possible_repr's binary representation.
    if idx'th digit is 1, remove it
        it means by filling zero at idx'th digit, word can not made
    """
    return tuple(
        possible_repr for possible_repr in possible_reprs
        if possible_repr & (1 << idx) == 0
    )

def sol(n, k, words):
    # 1. construct distinct char set
    # 2. make bit_repr for each word
    char_set = set()
    char_to_idx_dict = {}
    char_cnt = 0
    bit_reprs = []
    for word in words:
        bit_repr = 0b0
        for char in word:
            if char in char_set:
                bit_repr |= 1 << char_to_idx_dict[char]
                continue
            char_set.add(char)
            char_to_idx_dict[char] = char_cnt
            char_cnt += 1
            bit_repr |= 1 << char_to_idx_dict[char]
        bit_reprs.append(bit_repr)

    # print(f"{n=}, {k=}, {char_cnt=}")
    # 3. check base case1: all words are possible
    if char_cnt <= k:
        # print("all words are possible")
        return n

    num_of_ones = [count_of_one(bit_repr) for bit_repr in bit_reprs]
    # print(f"{num_of_ones=}")

    # 4. check base case2: any word can not be made
    if k < min(num_of_ones):
        # print("any word can not be made")
        return 0

    # 5. store associative array for debugging
#     bin_arr = [bin(bit_repr) for bit_repr in bit_reprs]
#     assoc = dict(zip(words, zip(bin_arr, num_of_ones)))
# 
#     print(char_to_idx_dict)
#     print(assoc)
#     for word, value in sorted(assoc.items(), key=lambda item: item[1][1]):
#         bit_repr, num_of_one = value
#         print(f"{word:20}, {num_of_one:<2}, {str(bit_repr)[2:]:>26}")


    # 6. consider each bit_repr as a bit string of length char_cnt
    #    character is 1:
    possible_word_cnt = 0

    idx = 0
    zero_to_fill = char_cnt - k
    possible_reprs = tuple(bit_reprs)

    walker = Walker(idx, zero_to_fill, possible_reprs)

    def backtrack(walker: Walker):
        nonlocal char_cnt, possible_word_cnt

        idx, zero_to_fill, possible_reprs = walker
        # print(f"{idx=}, {zero_to_fill=}, {len(possible_reprs)=}")
        # exit condition
        if zero_to_fill == 0:
            # print("exit condition")
            possible_word_cnt = max(possible_word_cnt, len(possible_reprs))
            # print(f"{possible_word_cnt=}")
            return possible_word_cnt
        # base case1: no more room to fill zero
        if idx == char_cnt:
            # print("base case1")
            return None
        # base case2: can not update result
        if len(possible_reprs) <= possible_word_cnt:
            # print("base case2")
            # print(f"{len(possible_reprs)=}, {possible_word_cnt=}")
            return None
        # base case3: can not use remaining zeroes
        if zero_to_fill + idx >= char_cnt:
            return None


        # biz logic
        # try to fill zero
        backtrack(Walker(idx + 1, zero_to_fill - 1, fill_zero(idx, possible_reprs)))
        # skip this index
        backtrack(Walker(idx + 1, zero_to_fill, possible_reprs))

    backtrack(walker)
    return possible_word_cnt

n, k = map(int, input().split())
words = [input() for _ in range(n)]
result = sol(n, k, words)
print(result)
```

- 이진탐색
```py
def condition_factory(n):
    def condition(x):
        return x * (x + 1) / 2 <= n
    return condition

def binary_search(start, end, n):
    """
    find the last number k that satisfies condition such that

    condition(start) is True
    condition(start + 1) is True
    ...
    condition(k) is True
    condition(k + 1) is False
    ...
    condition(end) is False

    invariant:
    1) condition(left) is True
    2) condition(right) is False
    note) left < right, by 1) and 2)
    """
    # make condition
    condition = condition_factory(n)

    # base condition
    if not condition(start):
        return -1
    if condition(end):
        return end

    # thus, condition(start) is True and condition(end) is False

    # init phase
    left, right = start, end

    # biz logic
    while True:
        mid = (left + right) // 2
        if mid == left:
            # this happens only if right = left + 1
            # thus, left is the last number that satisfies the condition
            return left
        # because mid != left, following logic strictly shirinks the search range
        # left -> mid: left increase to right
        # right -> mid: right decrease to left
        # because left and right are finite, the loop must end
        if condition(mid):
            left = mid
        else:
            right = mid

if __name__ == '__main__':
    for i in range(1, 21):
        print(f"sol({i}) is: {binary_search(1, i, i)}")

```

## 7/14

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
