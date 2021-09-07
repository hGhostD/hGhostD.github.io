---
layout: 日记
title: Vim使用技巧笔记
date: 2017-11-07 14:44:14
tags:
- 日记
- Vim
categories:
- 日记

---
边学习 Vim 技巧 边做笔记 学到一点就记录一点
<!--- more --->
从今天开始练习使用Vim，从最开始开始。
- 查询： 可以在 Vi 模式下直接使用 `:h ctrl-w` 查询 `<C-w>` 的用法。
- 输出： :echo 指令。

        Vim中的printf。 :echo "Hello,world!" 
- 持久化消息， 使用 `:echom "Hello again,World!"` 指令，然后使用 `:messages` 。能够再次看到 echom打印 "Hello again,World!" 的指令

 - 基本映射 `:map \ dd` 使用 '\' 指令代表 `dd` (删除行)  
- 缩写 可以快捷输入一些类内容

        :iabbrev hu Copyright 2011 Steve Losh
---
- 插入模式映射设置  `:inoremap jk <esc>`  在插入模式下按jk退出插入模式
- `J` (大写) 讲两行拼接成一行
- 多行注释 `Ctrl+v, 选中行，I（大写I）, #, ESC`

---
- `f` `F` 移动光标到当前行的字母 例如 `fc` 移动到光标后面第一个 c 字母上。`Fc` 移动到光标前第一个 c 字母。
	使用 `;` 查找本行下一个符合的字符 例如 `;` 跳转到下一个 c 字母上。使用 `,` 向前查找。
- `t` `T` 类似 `f` `F` 。差别是移动到字母前一个位置

---
- `\` 或者 `?` 查找 例如 `\error` `?error` 查找全文的error
按 `n` 向后查找匹配项 按 `N` 向前查找匹配项
- `CTRL-O` 光标跳转到上一个查找对象处 `CTRL-I` 光标跳转到下一个查找处
- 通过 `:set ic` 来忽略大小写 通过 `:set noic` 来分辨大小写 如果仅在第一次搜索时忽略大小写 直接在 搜索内容后添加 `\c` 例如: `\error\c`
- 设置搜索高亮 `:set hls is`  注: 取消搜索高亮 `:nohlsearch`

---
今天终于在xcode 9 上安装了 [XVim](https://github.com/XVimProject/XVim2) 了！！！
- `sp` 或 `Ctrl+w s` 水平分割窗口
- `vs` 或 `Ctrl+w v` 垂直分割窗口
- `Ctrl+w q` 关闭当前窗口
- `Ctrl+w o` 关闭除当前串口外的所有窗口
- `Ctrl+w w` 遍历切换窗口

---
- `#` 匹配单词 寻找下一个相同的单词。
- `di"` 删除 " 之间的内容。

---
- `>G` 使当前行到文档末尾处增加一个缩进，可以使用 `.` 继续重复命令。
- `s` 替换当前单词 == `cl`。
- `S` == `C` 删除整行 并重新输入。

---
- `:%s/\.o/\.c/g` 将 .o 替换为 .c。 `%` 表示所有行，`g` 表示全部替换
  `:s/aaa/bbb` 只替换第一行的第一个。`1,12s/aaa/bbb/g` 替换1到12行所有的。
- `daw` `diw` 可以更快向前删除一个单词 `aw` 比 `iw` 多删除空格。 
- `dab` `dib` 删除小括号内中的内容. `ab`  比 `ib` 多删除括号。等于 `da(` `di(`,类似还有 `daB` `diB` 等于 `da{` `di{` 删除大括号内容。
- `dit` `dat` 删除匹配标签栏中的内容。如 <a color=#00ff00>title</a>,输入 `dit` 会删除 title，删除掉标签中的内容。
- `<C-a>` `<C-x>` 把当前数字加一或减一。Tips: 如果是对 007 使用 `<C-a>` 将会得到结果为 010，是因为 vim 认为 007 是8进制，在 .vimrc 中设置 `set nrformats=` 会转为十进制。

---
- <font color=#0089ff>蓝色代表插入模式下指令</font>。
- <font color=#0089ff>Ctrl+w</font> 删除一个单词。
- <font color=#0089ff>Ctrl+u</font> 删除至行首。
- `yt,` 复制光标到 "," 之间所有的内容。<font color=#0089ff>Ctrl+r0</font> 在光标位置粘贴复制内容。yt 的意思是复制内容到专用寄存器中，所以 Ctrl+r0 表示取出第0个内容。
- <font color=#0089ff>Ctrl+r=</font> 使用运算功能。例如 计算 6 \* 35:在插入模式输入 `<C+r>=6*35`。直接获得结果210。

---
- <font color=#008900>绿色代表可视模式下指令</font>。
- `gv` 重选上次高亮选区。
- <font color=#008900>**o**</font> 切换高亮选区活动端。
- <font color=#008900>**u U**</font> 对选中单词进行小写/大写切换。

---
`:[range] delete [x]` 删除指定范围内的行[到寄存器 x 中]。
`"[x]p` 粘贴寄存器 x 中的内容，例如粘贴寄存器 2 中的内容：`"2p`。
`:reg` 查看寄存器内容。
`@:` 重复上一条命令。
`:!` 执行 Shell 中的程序，例如：`:!ls`。

---
`<C+o>` 类似网页后退，后退到上一个打开的文本。对应的还有 `<C+i>` 前进。
`:jumps` 打开历史文件列表。

---
`m{a-zA-z}` 把位置标记 {a-zA-Z} 设在当前光标位置 (不移动光标，这不是动作命令)。
`'{mark}` 跳转到标记行首。 \`\` 跳转到上次光标位置。
``{mark}` 跳转到标记处。 

---
`qa` (需要记录的指令) `q` 使用 `q` + 寄存器位置来记录宏操作，最后以 `q` 指令结尾。
`@a` 执行寄存器(a)中指令。 `:reg` 查看寄存器中所有的内容。
`~` 大小写转换，并使光标跳转到下一个字符。

---
`q/` 查看历史指令。

--- 
`<C+]>` 跳转光标到关键字的定义处。 
`<C+n>`  和 `<C+p>` 可以激活自动补全功能。
`<C+x>` `<C+l>` 组合使用可以补全本行内容。
`:set spell` 检查拼写错误。 通过 `[s` 和 `]s` 命令在拼写错误间进行跳转。还可以通过 `z=` 命令获取建议列表。

--- 
`ZZ` 快速退出并保存。
`dfi` 从光标处删除至i(包括i), `dti` 从光标处删除至i(不包括i)。

---
有的时候需要在单词左右添加引号括号之类的，需要在 .vimrc 中添加定义

```
function! s:surround()
    let word = expand("<cword>")
    let wrap= input("wrap with: ")
    let command = "s/".word."/".wrap.word.wrap."/"
    execute command
endfunction
nmap cx :call <SID>surround()<CR>
```

这样在 nomal 模式下使用 `cx` + `"` 就能在单词左右添加引号。