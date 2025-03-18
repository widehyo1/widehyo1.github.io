---
layout: post
title: awk script storage
subtitle: awk script storage
tags: [commandline, awk]
comments: true
author: widehyo
---

```
" ~/.vim/ftplugin/awk.vim

iabbrev <buffer> \begin; BEGIN {<CR><C-u>}
iabbrev <buffer> \end; END {<CR><C-u>}
iabbrev <buffer> \for; for (i = 1; i <= NF; i++) {}
iabbrev <buffer> \forarr; for (idx in arr) {}
iabbrev <buffer> \printarr; for (idx in arr) {<CR>print "idx: " idx " arr[idx]: " arr[idx]<CR>
iabbrev <buffer> \striparr; for (idx in arr) {<CR>arr[idx] = strip(arr[idx])<CR>
iabbrev <buffer> \surr; function surround_str(str, start, end) {<CR>return start str end<CR>
iabbrev <buffer> \surrd; "\""str"\""
iabbrev <buffer> \surrq; "'"str"'"
iabbrev <buffer> \surrp; "("str")"
iabbrev <buffer> \surrs; "["str"]"
iabbrev <buffer> \surrb; "<"str">"
iabbrev <buffer> \surrc; "{"str"}"
iabbrev <buffer> \split; split(str, arr, sep)
iabbrev <buffer> \strip; function strip(str) {<CR>gsub(/^\s+\|\s+$/, "", str)<CR>return str<CR>}
iabbrev <buffer> \join; function join(arr, sep) {<CR>acc = arr[1]<CR>for (i = 2; i <= length(arr); i++) {<CR>acc = acc sep arr[i]<CR>return acc<CR>}
iabbrev <buffer> \gsub; gsub(regex, replace, str)

setlocal tabstop=2
setlocal shiftwidth=2
vnoremap <buffer> gcc :s/^/# /<CR>
```

`column_replace.awk`
```awk
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

`column_to_pylist.awk`
```awk
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

`column_to_tuple.awk`
```awk
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

`csv_to_insert_sql.awk`
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

`csv_to_json.awk`
```awk
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

`git_changed.awk`
```awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

/Changes not staged for commit:/,/Untracked files:/{
    if ($0 ~ /modified/) {
        print substr($0,14)
    }
}
/Untracked files:/{
    uflag = 1
}
uflag {
    lnum++
    if (lnum >= 3) {
        print strip($0)
    }
    if ($0 ~ /^$/) {
        exit
    }
}
```

`investigate_column_with_type.awk`
```awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}
BEGIN {
    FS = "|"
    OFS = "|"
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
        print "["key_arr[1]"("key_arr[2]")]",col_info_arr[index_]
    }
}
```

`investigate_columns.awk`
```awk
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
    print length(meta_arr)

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


`line_to_arr.awk`
```awk
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

`make_code.awk`
```awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

BEGIN {
    RS = ""
}
NR == 1{
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
        declare[idx] = lines[idx]
    }
}
NR == 2{
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
    }
    for (idx in lines) {
        split(lines[idx], info_arr, ",")
        info_arr[5] = strip(info_arr[5])
        comment[idx] = substr(info_arr[5], 4)
    }
}
NR == 3{

    for (idx in declare) {
        print declare[idx]
        print "    '''"
        print "    " comment[idx], "생성 프로세서"
        print "    '''"
        print $0
        print "\n\n"
    }
}
```

`pycode_work1.awk`
```awk
BEGIN {
    FS = ","
}
{
    for (i = 1; i <= NF; i++) {
        print $i "_seq_max = max([data[0] for data in " $i "_data]) + 1"
    }
}
```

`pycode_work2.awk`
```awk
BEGIN {
    FS = ","
}
{
    $4 = "'" $4 "'"
    $5 = "'" $5 "'"
    $7 = "'" $7 "'"
    $8 = "True"
    printf ",("
    for (i = 1; i <= NF; i++) {
        printf $i ","
    }
    printf ")\n"
}
```

`pycode_work3.md`
```md
`p1.awk`

function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

BEGIN {
    RS = ""
}
NR == 1{
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
    }
    for (idx in lines) {
        if (idx == 1) continue
        split(lines[idx], arr, ",")
        print arr[3]
    }
}


`p2.awk`

function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

BEGIN {
    RS = ""
}
NR == 2{
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
    }
    for (idx in lines) {
        if (idx == 1) continue
        split(lines[idx], arr, ",")
        print arr[3]
    }
}



33950  2025-03-17_08:46:02 awk -f p1.awk ~/temp.txt
33951  2025-03-17_08:46:11 awk -f p1.awk ~/temp.txt  | sort | uniq
33952  2025-03-17_08:46:27 awk -f p1.awk ~/temp.txt  | sort | uniq > uniq_amddong.txt
33954  2025-03-17_08:47:27 awk -f p2.awk ~/temp.txt
33955  2025-03-17_08:47:37 awk -f p2.awk ~/temp.txt  | sort | uniq
33956  2025-03-17_08:47:54 awk -f p2.awk ~/temp.txt  | sort | uniq > fv_address_unique.txt

```

`python_error_to_gF_format.awk`
```awk
BEGIN {
    FS = "\""
}
{
    split($3, arr, /\s+/)
    print $2 ":" arr[3]
}
```

`skeleton_to_iabbr.awk`
```awk
NR == 1 {
  to = $0
}
NR > 1 {
  to = to "<CR>" $0
}
END {
  print "inoremap \\abbr; " to
}
```

`swap_camel_snake.awk`
```awk
function snake_to_camel(text, result) {
  result = ""
  split(text, arr, "_")
  for (idx in arr) {
    result = result toupper(substr(arr[idx], 0, 1)) substr(arr[idx], 2)
  }
  result = tolower(substr(result, 0, 1)) substr(result, 2)
  return result
}
function camel_to_snake(text, result) {
  result = text
  while (match(result, /[A-Z]/) > 1) {
    result = substr(result, 0, RSTART - 1) "_" tolower(substr(result, RSTART, 1)) substr(result, RSTART + 1)
  }
  return result
}
{
  split("|",arr,$0)
  gsub(/^_+/, "", arr[0])
  if (match(arr[0], "_")) {
    print snake_to_camel(arr[0])
  } else if (match(arr[0], /[A-Z]/) > 1) {
    print camel_to_snake(arr[0])
  } else {
    print arr[0]
  }
}
```

`target_column_uniq.awk`
```awk
BEGIN {
    FS = ","
}
{
    if (!uniq[$1,$2,$3]++) print $1,$2,$3
}
```

`tuple_reorder.awk`
```awk
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

`word_info.awk`
```awk
function strip(str) {
    gsub(/^\s+|\s+$/, "", str)
    return str
}

BEGIN {
    RS = ""
    FS = ":"
}
NR == 1{
    print NR
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
    }
    for (idx in lines) {
        split(lines[idx], arr, ":")
        left_key = strip(arr[1])
        left_word[left_key] = -1
    }
    for (idx in left_word) {
        print idx, left_word[idx]
    }
}
NR == 2{
    print NR
    split($0, lines, "\n")
    for (idx in lines) {
        lines[idx] = strip(lines[idx])
    }
    for (idx in lines) {
        split(lines[idx], arr, ":")
        right_key = strip(arr[1])
        right_word[right_key] = 1
        is_contain_type = match(arr[2], /\[([^]]+)\]/, type_arr)
        if (is_contain_type) {
            field_type = type_arr[1]
            right_type[right_key] = field_type
        }
    }
    for (idx in right_word) {
        print idx, right_word[idx]
    }
}
END {
    print "\nword info:\n"
    for (idx in left_word) {
        word_info[idx] += left_word[idx]
    }
    for (idx in right_word) {
        word_info[idx] += right_word[idx]
    }
    for (idx in word_info) {
        if (word_info[idx] == 1) {
            # print idx, word_info[idx]
            print idx " : Optional[" right_type[idx] "]"
        }
    }
}

```
