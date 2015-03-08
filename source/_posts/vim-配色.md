title: "vim 配色知识"
date: 2014-03-23 16:31:51
tags: vim
---

在vimcolorschemetest站点上，有数以千计的vim主题插件，然而每款主题或多或少都有我们不满意的地方，这就需要我们自己动手来进行扩展。

---
##基础知识

在vim中，主题也是以插件形式存在的。其中系统自带的主题，存放在$VIMRUNTIME/colors文件夹下，以*.vim命名。（注：查看$VIMRUNTIME请在vim中执行 :echo $VIMRUNTIME）
用户自定义的主题一般不放在系统目录下，而是放在`~/.vim/colors`目录下，这样不会干扰到其他用户，同时也方便备份自己的vim配置。
更换vim主题的命令为：
`:colorscheme 主题插件名称`
但是这样只能临时改变vim主题，退出后又会恢复原样，如果想永久改变，请在~/.vimrc中添加：
`colorscheme 主题插件名称`

由于我不使用Windows下的gvim，而是在linux下或远程ssh使用终端下的vim，因此首先需要在~/.vimrc中添加开启256颜色支持：
`set t_Co=256`
为了能在编辑程序时高亮显示关键字，还需要在~/.vimrc中开启语法高亮显示：
`syntax enable`
`syntax on`
做完上述的准备工作后，让我们正式开始定制主题之旅吧！
###主题色调
在配置其他属性前，首先要配置主题整体的色调，只有两个选择：dark和light（暗色调和亮色调）。对于经常阅读和编写代码的程序员来说，暗色调是更好的选择：

`set background=dark`
**<span style="color: #ff00ff;">接下来，需要重新设置一下语法高亮，否则设置不会生效：</span>**
```
if version > 580
hi clear
if exists("syntax_on")
syntax reset
endif
endif
```

###主题名称

主题名称是无参数调用 :colorscheme 时返回的信息，用于分辨不同主题，其设置如下：

`let g:colors_name="nslib_color256"`
####基础属性
由于vim可以在黑白终端、彩色终端、GUI界面下运行，所以需要对其分贝进行配置，下面给出一个简要的文档说明：

>term 黑白终端的属性
cterm  彩色终端的属性
ctermfg 彩色终端前景色
ctermbg 彩色终端背景色
gui GUI属性
guifg GUI前景色
guibg GUI背景色

对于黑白终端，我们没有配置的必要，因此主要的配置工作集中在彩色终端与GUI界面上，又由于彩色终端与GUI界面的配置只是关键字不同，因此这里只选取彩色终端进行说明。

<!--more-->
由于不是所有终端都支持256色，因此使用一些安全色会使我们的主题更有移植性，而GUI可以支持所有颜色，不在考虑范围之内，vim文档给出的安全色如下：

>"0 Black
"1 DarkBlue
"2 DarkGreen
"3 DarkCyan
"4 DarkRed
"5 DarkMagenta
"6 Brown, DarkYellow
"7 LightGray, LightGrey, Gray, Grey
"8 DarkGray, DarkGrey
"9 Blue, LightBlue
"10 Green, LightGreen
"11 Cyan, LightCyan
"12 Red, LightRed
"13 Magenta, LightMagenta
"14 Yellow, LightYellow
"15 White

<!--more-->
###配色语法
下面举例说明配色语法：
`hi Type ctermfg=LightYellow ctermbg=Black cterm=bold`
其中，hi是highlight命令的缩写，用于高亮配置；Type是要配色的元素名称；参数采用的是Key=Value的形式。

元素列表
配置颜色的语法非常简单，无需累赘，下面将分类介绍常用的元素标签：
####状态栏提示信息

>hi StatusLine 状态栏
hi StatusLineNC 非当前窗口的状态栏
ErrorMsg 错误信息
WarningMsg 警告信息
ModeMsg 当前模式
MoreMsg 其他文本
Question 询问用户
Error 错误

####文本搜索
>hi IncSearch 增量搜索时匹配的文本符串
hi Search 匹配的文本串

####弹出菜单

>Pmenu 弹出菜单
PmenuSel 菜单当前选择项

####窗体边框相关
>VertSplit 垂直分割窗口的边框
LineNr 行号
Cursor 光标所在字符
CursorLine 光标所在行
ColorColumn 光标所在列
ColorColumn 标尺
NonText 窗口尾部的~和@，以及文本里实际不显示的字符

####diff模式
>DiffAdd diff模式增加的行
DiffChange diff模式改变的行
DiffDelete diff模式删除的行
DiffText diff模式插入文本

####C/C++语法
>Comment 注释
PreProc 预处理
Type 数据类型
Constant 常量
Statement 控制语句
Special 字符串中的中的特殊字符
String 字符串
cCppString Cpp字符串
Number 数字
Todo TODO、HACK、FIXME等标签


###我的主题

```
set background=dark

if version &gt; 580
hi clear
if exists("syntax_on")
syntax reset
endif
endif

let g:colors_name="nslib_color256"

hi Normal ctermfg=Grey ctermbg=Black
hi ColorColumn ctermfg=White ctermbg=Grey
·
hi ErrorMsg term=standout
hi ErrorMsg ctermfg=LightBlue ctermbg=DarkBlue
hi WarningMsg term=standout
hi WarningMsg ctermfg=LightBlue ctermbg=DarkBlue
hi ModeMsg term=bold cterm=bold
hi ModeMsg ctermfg=LightBlue ctermbg=Black
hi MoreMsg term=bold ctermfg=LightGreen
hi MoreMsg ctermfg=LightBlue ctermbg=Black
hi Question term=standout gui=bold
hi Question ctermfg=LightBlue ctermbg=Black
hi Error term=bold cterm=bold
hi Error ctermfg=LightBlue ctermbg=Black
·
hi LineNr ctermfg=LightBlue ctermbg=Black
hi CursorColumn ctermfg=White ctermbg=Grey
hi CursorLine ctermfg=LightBlue ctermbg=Black
hi ColorColumn ctermfg=White ctermbg=Grey
·
hi IncSearch ctermfg=Black ctermbg=DarkGrey
hi Search ctermfg=Black ctermbg=DarkGrey
hi StatusLine term=bold cterm=bold
hi StatusLine ctermfg=Black ctermbg=Grey
hi StatusLineNC term=bold cterm=bold
hi StatusLineNC ctermfg=Black ctermbg=Grey
·
hi VertSplit ctermfg=Grey ctermbg=Grey
hi Visual term=bold cterm=bold
hi Visual ctermfg=Black ctermbg=Grey
·
highlight Pmenu ctermfg=Black ctermbg=Grey
highlight PmenuSel ctermfg=LightBlue ctermbg=DarkBlue
·
hi Comment ctermfg=DarkCyan ctermbg=Black
hi PreProc ctermfg=Blue ctermbg=Black
hi Type ctermfg=LightYellow ctermbg=Black cterm=bold
hi Constant ctermfg=Blue ctermbg=Black cterm=bold
hi Statement ctermfg=LightYellow ctermbg=Black cterm=bold
hi Special ctermfg=Red ctermbg=Black cterm=bold
hi SpecialKey ctermfg=Red ctermbg=Black cterm=bold
hi Number ctermfg=Blue ctermbg=Black
hi cCppString ctermfg=Red ctermbg=Black
hi String ctermfg=Red ctermbg=Black
hi Identifier ctermfg=Red ctermbg=Black cterm=bold
hi Todo ctermfg=Black ctermbg=Gray cterm=bold
hi NonText ctermfg=LightBlue ctermbg=Black
hi Directory ctermfg=Blue ctermbg=Black
hi Folded ctermfg=DarkBlue ctermbg=Black cterm=bold
hi FoldColumn ctermfg=LightBlue ctermbg=Black
hi Underlined ctermfg=LightBlue ctermbg=Black cterm=underline
hi Title ctermfg=LightBlue ctermbg=Black
hi Ignore ctermfg=LightBlue ctermbg=Black

hi Directory ctermfg=LightBlue ctermbg=Black
hi browseSynopsis ctermfg=LightBlue ctermbg=Black
hi browseCurDir ctermfg=LightBlue ctermbg=Black
hi favoriteDirectory ctermfg=LightBlue ctermbg=Black
hi browseDirectory ctermfg=LightBlue ctermbg=Black
hi browseSuffixInfo ctermfg=LightBlue ctermbg=Black
hi browseSortBy ctermfg=LightBlue ctermbg=Black
hi browseFilter ctermfg=LightBlue ctermbg=Black
hi browseFiletime ctermfg=LightBlue ctermbg=Black
hi browseSuffixes ctermfg=LightBlue ctermbg=Black

hi TagListComment ctermfg=LightBlue ctermbg=Black
hi TagListFileName ctermfg=LightBlue ctermbg=Black
hi TagListTitle ctermfg=LightBlue ctermbg=Black
hi TagListTagScope ctermfg=LightBlue ctermbg=Black
hi TagListTagName ctermfg=LightBlue ctermbg=Black
hi Tag ctermfg=LightBlue ctermbg=Black
```