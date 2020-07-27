---
note:
    createdAt: 2020-06-21T06:59:30.083Z
    modifiedAt: 2020-06-21T08:41:15.078Z
    tags: []
output: pdf_document
---

@import "pie/pie.css"

# Markdown 简明使用

## 介绍
Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。

>Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。
Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。

Markdown 编写的文档后缀为 `.md, .markdown`

## 基本使用

### 标题

---

# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

---

### 字体

粗体： **粗体**           ->            两个`**`包裹
斜体：*斜体*            ->            一个`*`包裹

### 段落

Markdown中的段落由**一行或多行文本**组成，不同的段落之间使用**空行**来标记。

1. 如果行与行之间没有空行，则会被视为同一段落。
2. 如果行与行之间有空行，则会被视为不同的段落。
3. 空行是指行内什么都没有，或者只有空格和制表符。
4. 如果想在段内换行，则需要在上一行的结尾插入两个以上的空格然后按回车键

### 列表

在Markdown中支持使用**有序列表**和**无序列表**

>有序列表 = `数字序号` + `英文句号` + `空格` + `列表内容`
无序列表 = `* /+/-` + `空格` + `列表内容`

#### 无序列表

- test1
  - test2
    - test3
   - test4
 - test5

#### 有序列表

1. test1
2. test2
   3. test3
     4. test4

### 分割线

 ---

 三条 `-`即可组成一个分割线

---

### 图片与链接

![test1](https://www.runoob.com/wp-content/uploads/2019/03/iconfinder_markdown_298823.png)

[Mahiroi](https://www.mashiroi.top)

<https://www.mashiroi.top>

[asd]

### 引用与代码块

1. **\`\`** 表示行内引用  -----> `test`

2. **\>** 表示整行引用，最大只可到三级引用

> test1
> > test2
> > > test3

3. **\`\`\`name \`\`\`** 表示代码块，支持多种语言

```shell
# /bin/sh
$ cat /etc/passwd
$ ls /
```

```cpp
#include <iostream>
using namespace std;

int main() {
  cout << "Hello, Markdown!";
  return 0;
}
```

```python
def hello():
  print("Hello")

if __name__ == "__main__":
  main()
```

### 表格

![](http://reader.epubee.com/books/mobile/fd/fd2c265149367259b9f2e2054dddf75a/Image00107.jpg)

1. 单元格使用|来分隔，为了阅读更清晰，建议最前和最后都使用|。
2. 单元格和|之间的空格会被移除。
3. 表头与其他行使用-来分隔。
4. 表格对齐格式如下。

- 左对齐（默认）：: :
- 右对齐：-:
- 居中对齐：:-:

![](http://reader.epubee.com/books/mobile/fd/fd2c265149367259b9f2e2054dddf75a/Image00108.jpg)

---

| 序号 | 标题 | 网址 |
| ----|----|----|
|01|博客|www.mashiroi.top|
|02|谷歌|www.google.com
|03|大佬|www.gem-love.com|
|04|巨佬|www.leavesongs.com|

---

| 序号 | 标题 | 网址 |
| :----:|:----:|:----:|
|01|博客|www.mashiroi.top|
|02|谷歌|www.google.com
|03|大佬|www.gem-love.com|
|04|巨佬|www.leavesongs.com|

---

### 任务列表

1. 任务列表以-+空格开 [+空格/x+] 组成

2. x可以小写，也可以大写，有些编辑器可能不支持大写，所以为避免解析错误，推荐使用小写的x

3. 当方括号中的字符为空格时，复选框是未选中状态，为x时是选中状态

- [ ] 任务点一
- [x] 任务点二
  - [ ] 任务点三

### 锚点

锚点，也称为书签，用来标记文档的特定位置，使用锚点可以跳转到当前文档或其他文档中指定的标记位置。

![](http://reader.epubee.com/books/mobile/fd/fd2c265149367259b9f2e2054dddf75a/Image00116.jpg)

- [表格](#表格)
- [列表](#列表)
  - [无序列表](#无序列表)
  - [有序列表](#有序列表)

### markdown数学公式语法

**markdown支持完整LaTex数学公式语法**

基本使用方法为用`$$`包裹表达式，`$$ $$`包裹段

#### 1. 上标

- 下标使用符号`Y_1`表达,公式表达结果为: $Y_1$
- 上标使用`X^{2m}`或者`X^2`, 其中较为复杂的表达式用{}包含, 公式表达结果为: $X^2$

#### 2. 常见运算符

|符号|表达式|符号|表达式|符号|表达式|
|:----:|:----:|:----:|:----:|:----:|:----:|
|$\pm$|\pm|$\times$|\times|$\div$|\div|
|$\cdot$|\cdot|$\leq$|\leq|$\geq$|\geq|

$
\gamma
$

### HTML 元素
目前MarkDown支持的HTML元素：\<kbd\>  \<b\>  \<i\>  \<em\>  \<sup\>  \<sub\>  \<br\>等。

## 总结

markdown作为一个便捷的标记性语言，对于文档排版有很好的效果，剩下的就是多加使用并整理。


## 引用
- [了不起的makdown]()
- [Markdown 教程](https://www.runoob.com/markdown/md-tutorial.html)
- [markdown数学公式语法](https://www.jianshu.com/p/cf36c5af997a)
- [Markdown数学符号&公式](https://blog.csdn.net/katherine_hsr/article/details/79179622)
- [Github -- Markdown基本撰写和格式语法](https://docs.github.com/cn/github/writing-on-github/basic-writing-and-formatting-syntax)

## 推荐的编辑器

个人比较喜欢用`vscode`写md，加上这几个插件

```text
markdown preview enhanced     ->      预览加强，不知道比官方的高到哪去里了

markdownlint        ->      markdown 语法规范工具

crossnote       ->        实现typroa的功能，持有 md文件管理的作用

github theme      ->      强烈推荐这个主题
```
