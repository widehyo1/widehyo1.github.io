---
layout: post
title: awk를 이용한 컬럼 substitution
subtitle: arbitrary column substitution using awk
tags: [commandline, awk]
comments: true
author: widehyo
---

```
" ~/.vim/ftplugin/awk.vim

iabbrev \begin; BEGIN {<C-u>}
iabbrev \end; END {<C-u>}
iabbrev \for; for (i = 1; i <= NF; i++) {}
iabbrev \forarr; for (idx in arr) {}
iabbrev \surr; function surround_str(str, start, end) {return start str end}
iabbrev \split; split(str, arr, sep)
iabbrev \strip; function strip(str) {gsub(regex, replace, str)return str}
iabbrev \join; function join(arr, sep) {acc = arr[1]for (i = 2; i <= length(arr); i++) {acc = acc sep arr[i]}return acc}
```

```awk
# column_replace.awk
BEGIN {
    FS = "\""
    OFS = "\""
    comment_str = "주관부서,예산,기대효과,홍보채널"
    split(comment_str, comment_arr, ",")
    name_str = "department_name,budget,expected_effect,information_channel"
    split(name_str, name_arr, ",")
}
{
    gsub(/location_name/, name_arr[NR], $1)
    print $1, comment_arr[NR], $3
}
```

```awk
# column_to_pylist.awk
function surround_str(str, start, end) {
    return start str end
}

function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

function join(arr, sep) {
    acc = arr[1]
    for (i = 2; i <= length(arr); i++) {
        acc = acc sep arr[i]
    }
    return acc
}

{
    split($0, arr, /: /)
    gsub(/idp.fv_/, "", arr[1])
    split(arr[2], column_arr, /,/)
    printf arr[1] "_field = "
    for (idx in column_arr) {
        column_arr[idx] = surround_str(strip(column_arr[idx]), "'", "'")
    }
    print surround_str(join(column_arr, ","), "[", "]")
}
```

```awk
# column_to_tuple.awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}
function join(arr, sep) {
    acc = arr[1]
    for (i = 2; i <= length(arr); i++) {
        acc = acc sep arr[i]
    }
    return acc
}
function surround_str(str, start, end) {
    return start str end
}
BEGIN {
    FS = "|"
}
{
    arr[NR] = strip($1)
}
END {
    print surround_str(join(arr, ","), "(", ")")
}

```
```awk
# csv_to_insert_sql.awk
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
```awk
# csv_to_json.awk
function transform(arr, out_arr) {
    count = 1
    for (key in arr) {
        out_arr[count] = key ":" arr[key]
        count++
    }
}
function join(arr, sep) {
    acc = arr[1]
    for (i = 2; i <= length(arr); i++) {
        acc = acc sep arr[i]
    }
    return acc
}
BEGIN {
    cmd = "wc -l < " ARGV[1]
    cmd | getline num_lines
    close(cmd)
    FS = ","
    print "["
}
NR == 1 {
    split($0, header_arr, FS)
    for (idx in header_arr) {
        header_arr[idx] = "\"" header_arr[idx] "\""
    }
}
NR > 1 {
    for (idx in header_arr) {
        res[NR][header_arr[idx]] = $idx
    }
    res[NR][header_arr[2]] = "\"" $2 "\""
    res[NR][header_arr[3]] = "\"" $3 "\""
    res[NR][header_arr[4]] = "\"" $4 "\""
}
END {
    for (idx in res) {
        transform(res[idx], out_res)
        cur = join(out_res, ",")
        if (idx != num_lines) {
            print "    {" cur "},"
        } else {
            print "    {" cur "}"
        }
    }
    print "]"
}
```
```awk
# investigate_column_with_type.awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}
BEGIN {
    FS = "|"
    table_count = 0
    split("", meta_arr, "")
    for (idx in meta_arr) {
        print "idx:",idx,"meta_arr[idx]:",meta_arr[idx]
    }
}
/Table/{
    col_count = 0
    split($0, arr, /\s+/)
    gsub(/"/, "", arr[3])
    meta_arr[++table_count] = arr[3]
    split("", table_arr, "")
}
NF == 9 && !/Column/{
    # print $1
    col_count++
    key = meta_arr[table_count]
    table_arr[col_count] = strip($1) ":" strip($2)
    for (idx in table_arr) {
        col_info_arr[key, col_count] = table_arr[col_count]
    }
}
END {
    for (index_ in col_info_arr) {
        split(index_, key_arr, SUBSEP)
        print "["key_arr[1]"("key_arr[2]")]"col_info_arr[index_]
    }
}
```
```awk
# investigate_columns.awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}
BEGIN {
    FS = "|"
    table_count = 0
    split("", meta_arr, "")
}
/Table/{
    split($0, arr, /\s+/)
    gsub(/"/, "", arr[3])
    meta_arr[++table_count] = arr[3]
    col_info_arr[meta_arr[table_count]] = ""
}
NF == 9 && !/Column/{
    col_info_arr[meta_arr[table_count]] = col_info_arr[meta_arr[table_count]] "," strip($1)
}
END {
    for (idx in col_info_arr) {
        print idx ": " col_info_arr[idx]
    }

}

```
```awk
# line_to_arr.awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

BEGIN {
    FS = "|"
}
{
    arr[NR] = strip($1)
}
END {
    for (idx in arr) {
        print "idx:",idx,"arr[idx]:",arr[idx]
    }
}

```
```awk
# tuple_reorder.awk
function surround_str(str, start, end) {
    return start str end
}

BEGIN {
    FS = ","
    OFS = ","
}
{
    gsub(/^\s+\(/, "", $0)
    gsub(/\),\s+$/, "", $0)
    target = $3 "," $2 "," $1
    print surround_str(target, "(", "),")
}


```
