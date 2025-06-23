---
layout: post
title: awk용 라이브러리 작성
subtitle: awk library record
tags: [awk]
comments: true
author: widehyo
---

gawk를 사용하면 다음과 같은 기능을 사용할 수 있다.
- multi dimensional array
- function reference by function name
- bit operation (and, or, xor)
- PROCINFO를 통한 연관배열 정렬방법 지정

awk는 @include를 통해 `:`으로 구분된 환경변수 AWKPATH 위치에 있는 awk 파일을 include할 수 있다. 예를 들어 AWKPATH=~ 이고 ~/library.awk가 있다면
```awk
@include "library"

{
  funcFromLib(param)
}
```
과 같이 사용할 수 있다.

이를 이용하면 언어 자체에서 제공하는 기능이 적은 awk도 충분히 강력한 기능을 사용할 수 있는데, 필자의 경우는 AWKPATH=~/.cli/awk/lib 로 설정하고 common.awk와 functool.awk를 만들어 사용하고 있다.

`~/.cli/awk/lib/common.awk`
```awk
# assume arr is 1 based array from 1 to length(arr)
function join(arr, sep, start, end) {
  if (length(arr) == 0) return start end

  acc = (start ? start : "") arr[1]
  for (i = 2; i <= length(arr); i++) {
    acc = acc sep arr[i]
  }
  if (end) acc = acc end

  return acc
}

function joinAssoc(arr, sep, start, end) {

  # for gawk version only
  # origin_sort = PROCINFO["sorted_in"]
  # PROCINFO["sorted_in"] = "@ind_str_asc"

  if (length(arr) == 0) return start end

  firstFlag = 1

  for (idx in arr) {
    if (firstFlag == 1) {
      acc = (start ? start : "") arr[idx]
      firstFlag = 0
      continue
    }
    acc = acc sep arr[idx]
  }
  if (end) acc = acc end

  # for gawk version only
  # PROCINFO["sorted_in"] = origin_sort
  return acc
}

function strip(str) {
  gsub(/^\s+|\s+$/, "", str)
  return str
}

function rindex(hay, needle,    arr, lastToken) {
  lastToken = arr[split(hay, arr, needle)]
  return length(hay) - length(needle) - length(lastToken) + 1
}

function format(fmt, target) {
  return sprintf(fmt, strip(target))
}
```

`~/.cli/awk/lib/functool.awk`
```awk
# out array set to global variable "mapres"
function map(arr, fn, res,     idx) {
  delete res
  for (idx in arr) {
    res[idx] = @fn(arr[idx])
  }
}

# inplace operation
function apply(arr, fn,     idx) {
  for (idx in arr) {
    arr[idx] = @fn(arr[idx])
  }
}

function filter(arr, pred, res,    count, i) {
  delete res
  for (i = 1; i <= length(arr); i++) {
    if (@pred(arr[i])) res[++count] = arr[i]
  }
}

function filterAssoc(arr, pred, res,     idx) {
  delete res
  for (idx in arr) {
    if (@pred(arr[idx])) res[idx] = arr[idx]
  }
}
```


그리고 이를 이용하여 vim에서 csv를 markdown table로 변환하는 awk script와 해당 키바인딩은 다음과 같다.

`~/.cli/awk/csv_to_mdtable.awk`
```awk
@include "common"
@include "functool"

function getname(name) {
  return format("%"maxlen"s", name)
}

function todash(str) {
  gsub(/./, "-", str)
  return str
}

function predicate(str) {
  return length(str) > 2
}

BEGIN {
  FS = ","
  maxlen = 0
}

NR == 1 {
  for (i = 1; i <= NF; i++) {
    if (length($i) > maxlen) maxlen = length($i)
    headers[i] = $i
  }
}
NR > 1 {
  for (i = 1; i <= NF; i++) {
    if (length($i) > maxlen) maxlen = length($i)
    contents[NR][i] = $i
  }
}
END {
  apply(headers, "getname")
  print join(headers, "|", "|", "|")
  map(headers, "todash", splitlines)
  print join(splitlines, "|", "|", "|")
  for (idx = 2; idx <= NR; idx++) {
    apply(contents[idx], "getname")
    print join(contents[idx], "|", "|", "|")
  }
}

```

`~/.vimrc`

`vnoremap <leader>c2m :! awk -f ~/.cli/awk/csv_to_mdtable.awk<CR>`
