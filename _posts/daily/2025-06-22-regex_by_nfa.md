---
layout: post
title: NFA를 이용한 정규표현식 구현
subtitle: regular expression by non-deterministic finite automata
tags: [regex, nfa, string, algorithm, datastructure]
comments: true
author: widehyo
---

유한상태기계에 대한 자료를 조사하다 [좋은 자료](https://swtch.com/~rsc/regexp/regexp1.html)를 알게 되어 해당 내용을 구현해보기로 했다. 구체적인 구현은 다음 C언어 구현 예를 참고했다. [NFA Thompson Construction by Russ Cox · GitHub](https://gist.github.com/luchy0120/badcc9ce807e359067b0e0b17271963b). 이 알고리즘은 톰슨의 구성을 이용하여 정규표현식을 구현하는데, 구체적인 구현의 구성은 다음과 같다.

1. 정규표현식(regular expression)으로부터 후위표현식(postfix)을 만든다.
2. 후위표현식을 하나씩 읽으며 character와 특별 연산(`*`, `+`, `?`, `|`, `.`(여기서는 any character가 아니라 concat))로 state diagram을 만든다
3. 만든 state diagram에 search text를 한 글자씩 feeding해가며 처음으로 MATCH에 도달하면 True를 리턴하고 모든 search text가 소진되면 False를 리턴한다.

구현하면서 지난 번에 구현한 [Aho-corasick](https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm) 기반의 구현과 비슷한 느낌을 받았다. 사실 근본적인 생각은 같다. 원하는 pattern을 catch 할 수 있는 자료구조를 construct하고 대상 text를 한 글자씩 feeding해가며 hits를 그러모은다는 핵심은 동일하게 나타난다.

한편, Aho-corasick과는 달랐던 것은 pattern을 검증하는 자료구조가 tri(엄밀히 말하자면 fail function을 이용한 failure path가 추가된 형태의)에서 보다 일반적인 state diagram(엄밀히 말하면 SPLIT이라는 special node 여러개를 이용하여 successive multiple nodes를 표현한다는 점에서 완전히 일반적이지는 않지만)을 사용한다는 점이다. tree 기반 자료인 tri에서 보다 일반적인 directed graph인 state diagram에 더 가까워 졌다는 점에서 의미를 찾을 수 있을 것 같다.

가장 먼저 구현한 부분은 post2nfa이다. 코드는 참고한 C언어 코드를 최대한 참고했는데, 알고리즘을 이해하기 위해 postfix에서 나타나는 special character를 머리속에 그림으로 치환하는 점이 선행되어야 했다.

[image](/assets/img/nfa_fragments.dot.png)

```py
from dataclasses import dataclass
from collections import deque
from typing import Sequence, Tuple, Optional
from enum import Enum, auto
from pprint import pp
import sys


class Special(Enum):
    SPLIT = auto()
    MATCH = auto()


@dataclass
class State:
    c: str | Special
    out: Optional["State"] | None = None
    out1: Optional["State"] | None = None

    def to_tuple(self, attrname) -> Sequence[Tuple["State", str]]:
        return [(self, attrname)]

    def __hash__(self):
        return id(self)

    def ismatch(self):
        visited = set()
        stack = [self]

        while stack:
            state = stack.pop()
            if state in visited:
                continue
            visited.add(state)
            if state.c == Special.MATCH:
                return True
            elif state.c == Special.SPLIT:
                stack.append(state.out)
                stack.append(state.out1)
            elif state.out:
                stack.append(state.out)

        return False
    
    def extract_graph(self) -> dict:
        visited = set()
        node_inventory = {}
        stack = [(self, None, None)]
        nodes = []
        edges = []
        split_nodes = []
        match_nodes = []
        nodeid = 0
        while stack:
            state, parent, parentid = stack.pop()
            if state in visited:
                edges.append((parentid, node_inventory[state]))
                continue
            nodeid += 1
            visited.add(state)
            node_inventory[state] = nodeid
            edges.append((parentid, nodeid))

            if state.out:
                stack.append((state.out, state, nodeid))
            if state.out1:
                stack.append((state.out1, state, nodeid))

            if state.c == Special.MATCH:
                nodes.append((nodeid, ""))
                match_nodes.append(nodeid)
            elif state.c == Special.SPLIT:
                nodes.append((nodeid, ""))
                split_nodes.append(nodeid)
            else:
                nodes.append((nodeid, state.c))

        return {
            "nodes": nodes,
            "match_nodes": match_nodes,
            "split_nodes": split_nodes,
            "edges": edges,
        }

    def to_dot(self):
        graph_info = self.extract_graph()
        print("\n".join(generate_dotfile(graph_info)))

def generate_dotfile(graph_info: dict):
    nodes = graph_info["nodes"]
    matches = graph_info["match_nodes"]
    splits = graph_info["split_nodes"]
    edges = graph_info["edges"]
    yield 'digraph {'
    yield '  fontname="Helvetica,Arial,snas-serif"'
    yield '  node [fontname="Helvetica,Arial,snas-serif"]'
    yield '  edge [fontname="Helvetica,Arial,snas-serif"]'
    yield ''
    yield '  graph [center=1 rankdir=LR]'
    yield ''
    yield '  node [height=0.25 width=0.25 shape="circle" label=""]'
    yield '  node [shape="doublecircle"] ' + " ".join([f"n{id_:03}" for id_ in matches])
    yield '  node [shape="point"] ' + " ".join([f"n{id_:03}" for id_ in splits])
    yield '  node [shape="circle"]'
    yield ''
    yield from [f'  n{id_:03} [label="{c}"]' for id_, c in nodes]
    yield ''
    yield from [f'  n{from_:03} -> n{to:03}' for from_, to in edges if from_ is not None]
    yield '}'

@dataclass
class Frag:
    start: State
    outs: Sequence[Tuple[State, str]]


@dataclass
class Scope:
    atom_cnt: int = 0
    alt_cnt: int = 0


@dataclass
class StateWalker:
    state: State

    @classmethod
    def from_state(cls, state: State) -> "StateWalker":
        return StateWalker(state)

    def state_generator(self, maxlevel: int | None = None):
        queue = deque([(self.state, 0)])
        while queue:
            st, step_level = queue.popleft()

            if maxlevel is not None and step_level > maxlevel:
                continue

            yield st, step_level
            if st.c == Special.SPLIT:
                queue.append((st.out, step_level))
                queue.append((st.out1, step_level))
            elif st.out:
                queue.append((st.out, step_level + 1))

    def gen_filter(self, string: str):
        for idx, char in enumerate(string):
            for st, step_level in self.state_generator(idx):
                if idx == step_level and st.c == char:
                    yield (st, step_level)

    def ismatch(self, string) -> bool:
        target = len(string) - 1

        return any(
            st.ismatch()
            for st, step_level in self.gen_filter(string)
            if step_level == target
        )


def patch(outs: Sequence[Tuple[State, str]], target: State):
    for state, attr in outs:
        setattr(state, attr, target)


def append(
    s1: Sequence[Tuple[State, str]], s2: Sequence[Tuple[State, str]]
) -> Sequence[Tuple[State, str]]:
    return s1 + s2


def zero_or_more(stack):
    e = stack.pop()
    s = State(Special.SPLIT, e.start, None)
    patch(e.outs, s)
    stack.append(Frag(s, s.to_tuple("out1")))


def one_or_more(stack):
    e = stack.pop()
    s = State(Special.SPLIT, e.start, None)
    patch(e.outs, s)
    stack.append(Frag(e.start, s.to_tuple("out1")))


def zero_or_one(stack):
    e = stack.pop()
    s = State(Special.SPLIT, e.start, None)
    stack.append(Frag(s, append(e.outs, s.to_tuple("out1"))))


def concat(stack):
    e1 = stack.pop()
    e2 = stack.pop()
    patch(e2.outs, e1.start)
    stack.append(Frag(e2.start, e1.outs))


def alternate(stack):
    e1 = stack.pop()
    e2 = stack.pop()
    s = State(Special.SPLIT, e1.start, e2.start)
    stack.append(Frag(s, append(e1.outs, e2.outs)))


def character(stack, tok: str):
    s = State(tok)
    stack.append(Frag(s, s.to_tuple("out")))


def post2nfa(postfix) -> State | None:

    stack: Sequence[Frag] = []

    for tok in postfix:
        match tok:
            case "*": zero_or_more(stack)
            case "+": one_or_more(stack)
            case "?": zero_or_one(stack)
            case ".": concat(stack)
            case "|": alternate(stack)
            case _: character(stack, tok)

    assert len(stack) == 1, "invalid postfix"
    e = stack.pop()
    matchstate = State(Special.MATCH)
    patch(e.outs, matchstate)
    return e.start


def flush_atom(scope, postfix_buffer):
    if scope.atom_cnt > 0:
        postfix_buffer += ["."] * (scope.atom_cnt - 1)
    scope.atom_cnt = 0
    return postfix_buffer


def flush_scope(scope, postfix_buffer):
    postfix_buffer = flush_atom(scope, postfix_buffer)
    postfix_buffer += ["|"] * scope.alt_cnt
    scope.alt_cnt = 0
    return postfix_buffer


def re2post(regex: str) -> str:
    """
    a(bb)+a -> abb.+.a.
    """

    postfix_buffer = []
    stack = [Scope()]
    scope_level = 0
    scope = stack[scope_level]
    for re in regex:
        match re:
            case "*" | "+" | "?":
                postfix_buffer.append(re)
            case "(":
                if scope.atom_cnt > 1:
                    scope.atom_cnt -= 1
                    postfix_buffer.append(".")
                scope_level += 1
                stack.append(Scope())
                scope = stack[scope_level]
            case ")":
                assert scope_level != 0, "closing paren - can not close unopened paren"
                assert scope.atom_cnt != 0, "closing paren - atom is empty"
                postfix_buffer = flush_scope(scope, postfix_buffer)
                scope_level -= 1
                scope = stack[scope_level]
                scope.atom_cnt += 1
            case "|":
                assert scope.atom_cnt != 0, "alternate - atom is empty"
                postfix_buffer = flush_atom(scope, postfix_buffer)
                scope.alt_cnt += 1
            case _:
                if scope.atom_cnt > 1:
                    scope.atom_cnt -= 1
                    postfix_buffer.append(".")
                postfix_buffer.append(re)
                scope.atom_cnt += 1
    assert scope_level == 0, "invalid paren - unclosed paren"
    postfix_buffer = flush_scope(scope, postfix_buffer)

    return "".join(postfix_buffer)


def regex_by_nfa(regex, string):
    postfix = re2post(regex)
    nfa = post2nfa(postfix)
    sw = StateWalker.from_state(nfa)
    return sw.ismatch(string)

def print_dot(regex):
    postfix = re2post(regex)
    nfa = post2nfa(postfix)
    nfa.to_dot()


# main
if __name__ == "__main__":
    # regex = "a+"
    # for string in ["", "a", "aa", "aaa", "asve"]:
    #     print(regex_by_nfa(regex, string))
    # regex = "a(bb)+a|ab*ab"
    regexes = ["a+", "a?b+c*", "ab|cd", "((a|b)c)*", "a(b|c)*d", "a(b(cd)?)+", "a(bb)+a|ab*ab"]
    target = int(sys.argv[1])
    print_dot(regexes[target])
```
