## Vim

### 翻页

ctrl+y 向上翻一行（常用）

ctrl+e 向下翻一行（常用）

ctrl+f 向下翻页（常用）

ctrl+b 向上翻页（常用）

ctrl+d 向下翻半页

ctrl+u 向上翻半页

### 光标移动

hjkl 方向键【head jump kick last】（常用）

[space] 向后移动一个字符，自动跳转到下一行（常用）

[backspace] 向前移动一个字符，自动跳转到上一行（常用）

[enter] 向后移动一行（到行首）（常用）

0 移动到行首（常用）

$ 移动到行尾（常用）

G 移动到文档最后一行（常用）

gg 移动到文档第一行（常用）或1G，等效

n[hjkl] 移动n个位置

n[enter] 向后移动n行

n[space] 向后移动n个字符

### 删除字符

x 删除1个字符

20x 向后删除20个字符

### 剪切

dd 剪切本行（常用）

d0 剪切到行首的字符

d$ 剪切到队尾的字符

20dd 向下剪切20行

dgg 剪切第一行到本行

di" 删除引号内的内容，同理di(删除括号内内容，di{,di},di)等

dG 剪切本行到最后一行

cc 剪切本行并进入编辑模式

### 复制

yy 复制本行（常用）

y0 复制到行首的字符

y$ 复制到队尾的字符

20yy 向下复制20行

ygg 复制第一行到本行

yG 复制本行到最后一行

### 粘贴

p 向下粘贴（可以多次粘贴）

P 向上粘贴（可以多次粘贴）

### 重复

. 重做上一个指令

### 撤销

u 撤销

ctrl+r 重做

### 合并

J 合并本行和下一行（补充空格）

### 替换

:1,$s/word1/word2/gc 全文查找和替换word1为word2

### 显示行号

:set nu

:set nonu

### 列选

ctrl+v 

### 保存退出

ZZ 保存退出

:n1,n2 w [filename] 将n1到n2保存为filename

:r [filename] 读入文件内容

### 多文件编辑

vim file1 file2

:n 下一个文件

:N 上一个文件

:files 查看所有文件

### 多窗口编辑

:sp 分割窗口

:q 关闭窗口

ctrl+w & j 跳转到下一个窗口

ctrl+w & k 跳转到上一个窗口

## Idea

Ctrl+Option+O 自动优化import

Command+shift+8 列选/取消列选

## Typora

### 格式

Command+1～6  一～六级标题

Command+b  字体加粗

Command+i  字体倾斜

Command+u  下划线

Ctrl+shift+` 删除线

### 移动

Fn+左方向   返回Typora顶部

Fn+右方向   返回Typora底部

### 选择

Command+d 选中单词

Command+e 选中相同格式的文字

### 操作

Command+z 取消

### 表格

Option+Command+t

## iTerm2

左右分屏 Cmd+d

上下分屏 Cmd+shift+d

删除一个单词 Ctrl+w

删除光标之前的字符 Ctrl+u

删除光标之后的字符 Ctrl+k

移动到头部 Ctrl+a

移动到尾部 Ctrl+e

清屏 Ctrl+l

## less

同时打开多个文件：less file1 file2

:f 查看当前文件信息（文件名等）

:n 查看下一个文件

:p 查看前一个文件

-N [enter] 显示/隐藏 行号



## Finder

前往文件夹 Cmd+shift+G



## awk

求和：awk '{s+=$1} END {printf "%.0f\n", s}' number.file

```
$ cat number.file
1
2
3
4
$ awk '{s+=$1} END {printf "%.0f\n", s}' number.file
10
```

提取列：awk -F '.' '{print $2}' split.file > split.file2



```
$ cat split.file
aaa.e1.e2
bbb.e3.e4
ccc.e5.e6
$ awk -F '.' '{print $2}' split.file > split.file2
$ cat split.file2
e1
e3
e5
```

## git

关联本地和远程分支

git remote add origin ssh://git@g.hz.netease.com:22222/wangtao3/beeline.git

git branch --set-upstream-to=origin/develop develop

git branch --set-upstream-to=origin/master master



## hive

-- 查询数据

select * from db_ndc.test_wangtao3_06022;

-- 查看指定分区的数据

select * from db_ndc.test_wangtao3_06022 where day ='curr_day_full';

select * from db_ndc.test_wangtao3_06022 where day ='2021-06-01';



-- 查看表结构

show create table db_ndc.test_wangtao3_06022;

-- 查看表详细信息（包含路径）

describe formatted db_ndc.test_wangtao3_06022;



-- 查看partition

show partitions db_ndc.test_wangtao3_06022;

-- 删除partition

alter table db_ndc.test_wangtao3_0602  DROP IF EXISTS partition(day="2021-06-01");



-- 查看目录中的文件

-- hive目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/db_ndc.db/test_wangtao3_0602;

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/db_ndc.db/test_wangtao3_0602/day=2021-06-01

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/db_ndc.db/test_wangtao3_0602/day=curr_day_full;



-- schema目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_schema/jobId-1714/ddb/test105/



-- realtime目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_realtime/jobId-1714/ddb/test105/

-- temp目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_temp/jobId-1714/ddb/test105/

-- stage目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_stage/jobId-1714/ddb/test105/



-- 全量目录

dfs -ls hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_full/jobId-1710/ddb/test105/



-- 查看文件内容

dfs -cat hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_temp/jobId-1714/ddb/test105/srcDb-d3RfdGVzdA==_secondaryName-s1_ddlId-1_threadId-2761_time-2021060211_mv-1622603382466.csv

dfs -cat hdfs://bdms-test/user/test_bdms/hive_db/ndc/ndc_temp/jobId-1710/ddb/test105/srcDb-d3RfdGVzdA==_secondaryName-s1_ddlId-0_threadId-249_time-2021060119_mv-1622547096139.csv



-- 删除

dfs -rm -r hdfs://bdms-test/user/test_bdms/hive_db/db_ndc.db/test_wangtao3_0602/day=2021-05-31

[home](#翻页)
