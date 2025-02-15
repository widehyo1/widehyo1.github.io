---
layout: post
title: awk를 이용한 컬럼 substitution
subtitle: arbitrary column substitution using awk
tags: [commandline, awk]
comments: true
author: widehyo
---

`script.awk`
```awk
function join(arr, sep) {
    acc = arr[1]
    for (i = 2; i <= length(arr); i++) {
        acc = acc sep arr[i]
    }
    return acc
}
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}
function surround_str(str, start, end) {
    return start str end
}
function apply_strip(format_arr) {
    for (idx in format_arr) {
        format_arr[idx] = strip($idx)
    }
}
function transform_header(header_str) {
    split(header_str, header_arr, ",")
    for (idx in header_arr) {
        header_arr[idx] = surround_str(header_arr[idx], "`", "`")
    }
    result = join(header_arr, ",")
    return result
}
function transform_content(content_str) {
    split(content_str, content_arr, ",")
    apply_strip(content_arr)
    content_arr[2] = surround_str(content_arr[2], "'", "'")
    content_arr[3] = surround_str(content_arr[3], "'", "'")
    content_arr[4] = surround_str(content_arr[4], "'", "'")
    result = join(content_arr, ",")
    return result
}
BEGIN {
    cmd = "wc -l < " ARGV[1]
    cmd | getline num_lines
    close(cmd)
    FS = ","
    OFS = ","
    header_format_str = "연번,행정동,관리단체,설치장소(도로명주소),위도,경도"
    content_format_str = "1,광평동,대한민국특수임무유공자회,구미시 구미대로 14길 7-5,36.107064,128.364115"
}
NR == 1 {
    header = ""
    insert_str = transform_header(header_format_str)
    print "INSERT INTO my_table(" insert_str ") VALUES"
}
NR > 1 && NR < num_lines {
    row_str = transform_content(content_format_str)
    print "(" row_str "),"
}
NR == num_lines {
    row_str = transform_content(content_format_str)
    print "(" row_str ");"
}
```

`sample_utf8_top10.csv`
```csv
연번,행정동,관리단체,설치장소(도로명주소),위도,경도
1,광평동,대한민국특수임무유공자회,구미시 구미대로 14길 7-5,36.107064,128.364115
2,광평동,대한민국특수임무유공자회,구미시 광평동 792-73,36.103988,128.364512
3,광평동,대한민국특수임무유공자회,구미시 광평동 529-7,36.104764,128.364196
4,광평동,대한민국특수임무유공자회,구미시 광평동 529-13,36.104905,128.364745
5,광평동,대한민국특수임무유공자회,구미시 화신로8길 13,36.114819,128.356295
6,광평동,대한민국특수임무유공자회,구미시 광평동 154-1,36.115549,128.356713
7,광평동,대한민국특수임무유공자회,구미시 광평동 159,36.116117,128.355903
8,광평동,대한민국특수임무유공자회,구미시 광평동 166-14,36.115742,128.355568
9,광평동,대한민국특수임무유공자회,구미시 광평동 165-4,36.116322,128.354838
10,광평동,대한민국특수임무유공자회,구미시 광평동 339-2 ,36.109199,128.354189
```

```console
2025/02/15 on  main [✘!?] ❯ awk -f script.awk sample_utf8_top10.csv
INSERT INTO my_table(`연번`,`행정동`,`관리단체`,`설치장소(도로명주소)`,`위도`,`경도`) VALUES
(1,'광평동','대한민국특수임무유공자회','구미시 구미대로 14길 7-5',36.107064,128.364115),
(2,'광평동','대한민국특수임무유공자회','구미시 광평동 792-73',36.103988,128.364512),
(3,'광평동','대한민국특수임무유공자회','구미시 광평동 529-7',36.104764,128.364196),
(4,'광평동','대한민국특수임무유공자회','구미시 광평동 529-13',36.104905,128.364745),
(5,'광평동','대한민국특수임무유공자회','구미시 화신로8길 13',36.114819,128.356295),
(6,'광평동','대한민국특수임무유공자회','구미시 광평동 154-1',36.115549,128.356713),
(7,'광평동','대한민국특수임무유공자회','구미시 광평동 159',36.116117,128.355903),
(8,'광평동','대한민국특수임무유공자회','구미시 광평동 166-14',36.115742,128.355568),
(9,'광평동','대한민국특수임무유공자회','구미시 광평동 165-4',36.116322,128.354838),
(10,'광평동','대한민국특수임무유공자회','구미시 광평동 339-2',36.109199,128.354189);
```

awk -f script.awk sample_utf8_top10.csv 에서 ARGV[1]는 sample_utf8_top10.csv이고 wc -l과 getline으로 num_lines 변수로 저장한 뒤 마지막 줄에만 ;을 붙여주기 위해 

```awk
NR == num_lines {
    row_str = transform_content(content_format_str)
    print "(" row_str ");"
}
```

데이터 출처: [공공데이터 포탈-경상북도 구미시_의류수거함 현황_20250207](https://www.data.go.kr/data/15127281/fileData.do)
