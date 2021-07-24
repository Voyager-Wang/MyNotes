## Vim

### 删除

删除多行

dd 删除本行

d3j 删除本行+下面的三行

d3k 删除本行+上面的三行

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





