





## 编辑模式
### 编辑命令

-  `v` 进入多选模式


### 设置环境

- `set number` 显示行号
- `set paste` 设置为粘贴模式，让 `vim` 对有特殊内容的文本保持原有格式
- `set nopaste` 取消粘贴模式。


### 匹配然后删除
#### 基础匹配删除操作
1. 删除匹配的字符串：`%s/<pattern>//g`
    
    本质上是**替换**（substitute），用空字符替换了 `<pattern>`。
    
2. 删除匹配行：`:%g/<pattern>/d`
    
    例如 `:%g/cloud/d`，表示删除当前文件内包含 `cloud` 的行。

#### 删除空行
1. `g/^\s*$/d`：以空格类字符（`^\s`）开头，连续匹配直到行结束（`*$`），删除（`d`）。
    
    https://stackoverflow.com/questions/706076/vim-delete-blank-lines
#### 删除匹配前后n/m行
来自 [Deleting a range of n lines before and after a matched line?](https://vi.stackexchange.com/questions/3411/deleting-a-range-of-n-lines-before-and-after-a-matched-line)：

1. `:%g/<pattern>/-10,+28d`：找到包含 `<pattern>` 的行，然后从该行前面 10 行一直删除到该行后面 28 行。
2. `:%g/<pattern>/-10,.d`：删除从匹配行 `-10` 行直到当前行，小数点表示当前行。

#### 删除不匹配的行 `:g!/<pattern>/d`
#### 删除文档中所有空行
```bash
:g/^\s*$/d     # 删除空行，其中可能包括若干个空格或 tab
:g/^$/d        # 删除纯空行（其中不包含空格或 tab）

:%s/\s\+$//g   # 删除 trailing 空格或 tab

:g!/pattern/d  # 删除不匹配的行
```

### 替换
#### 将多个连续空行合并为一行
1. 通用方式，将多个连续空行合并为一行：`:%s/\n\{3,}/\r\r/e`
2. Linux 上，用 cat 命令合并空行: `:%!cat -s`。
    `-s, --squeeze-blank`: suppress repeated empty output lines。

#### 追加相同内容到所有行
1. `%s/$/<your content>/gc`
2. 用替换 \n 实现，例如，在每行后面追加一个分号，注意后面的 pattern 里换行符得用 \r：`%s/\n/;\r/gc`

#### 将windows上的换行 `^M` 替换为linux换行
`:%s/<Ctrl-V><Ctrl-M>/\r/g`

`<C-V><C-M>` 才能按出 ^M 符号。

#### 将指定的patern替换为换行符
搜索文本中的换行符需要使用 `\n` 作为 pattern， 替换需要使用 **==`\r`==** 作为换行符，`\n`不行：

`:%s/<pattern>/\r/gc`





## 命令行

### 删除一个字符
- Ctrl + H == Backspace
- Ctrl + U 删除一行
- Ctrl + W 删除当前一个单词

### 打开文件
以二进制模式打开文件
```bash
vim -b <file>
```

打开文件后跳转至第N行
```bash
vim +5 test.txt
```

打开文件后跳转到{pattern}第一次出现的地方
```bash
$ vim +/{pattern} <file>

# 跳转到 'vim' 字符串第一次出现的位置
$ vim +/vim test.txt
```

```bash
$ vim "+{command}" <file>
$ vim -c {command} <file>

# example: turn off showing number on opening index.html
$ vim "+set nonu" index.html
```

以diff模式打开多个文件进行对比
```bash
vim -d <file1> <file2> ... <fileN>
```
效果和 `vimdiff` 一样。

```bash
$ vimdiff file1 file2

#]c 跳到下一个差异点
#[c 跳到前一个
#:diffupdate 手动重新加载文件内容（正常会自动加载）
```

一次在多个窗口打开
```bash
# 窗口上下排列， N 可省略
$ vim -o[N] <file1> <file2> ... <fileN>

# 窗口左右排列， N 可省略
$ vim -O[N] <file1> <file2> ... <fileN>
```


