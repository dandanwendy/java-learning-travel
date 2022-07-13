[toc]

markdown写的文档，可以直接被浏览器解析，以和html相似的方式展示在网页上。但markdown的编辑方式可比html方便多了，用其写编程文档很适合。

# 块元素

## 段落

换行用以下两种方式

```
enter键
<br/>  //其实就是用html中的换行标签，markdown本身也是翻译成html嘛
```

## 标题

```
# 一级标题
###### 六级标题
```

## 列表

### 无序列表

以下三种符号都可创建无须列表

```
*
+
-
```

如

+ 刘备
+ 关羽
+ 张飞

### 有序列表

```
1. 有序列表
```

1. 红色
2. 黄色
3. 蓝色

## 代码块

markdown支持多种语言。输入```之后输入一个可选的语言标识符，然后按return键后输入代码，即可通过语法高亮显示它

```
```java
此处写java代码
···
```

## 数学公式

### 嵌入文本的公式

```
$latex公式$
```

你知道勾股定理$a^{2}+b^2=c^2$

### 单独公式

markdown可以直接渲染latex公式

```
$$
latex格式公式
$$
```


$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$

## 表格

表格式很常见的格式化展示数据的方式，typora中输入 `| First Header | Second Header |` 并按下 `return` 键将创建一个包含两列的表。

```
| First Header  | Second Header |
```

创建表后，焦点在该表上将弹出一个表格工具栏，您可以在其中调整表格，对齐或删除表格。您还可以使用上下文菜单来复制和添加/删除列/行。

您还可以在表格中包括内联 Markdown 语法，例如链接，粗体，斜体或删除线。

最后，通过在标题行中包含冒号：您可以将文本定义为左对齐，右对齐或居中对齐。

```
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```

## 水平线

输入 `***` 或 `---` 在空行上按 `return` 键将绘制一条水平线。

```
***
---
```

---

## 目录

输入 `[toc]` 然后按 `Return` 键将创建一个“目录”部分，自动从文档内容中提取所有标题，其内容会自动更新。本文开头的目录就是这中方式生成的。

# Span元素

在您输入后Span元素会被立即解析并呈现。在这些span元素上移动光标会将这些元素扩展为markdown源代码。以下将解释这些span元素的语法。

## 链接

链接文本都写在 [方括号] 内，

```
This is [an example](http://example.com/ "Title") inline link.
```



This is [an example](http://example.com/ "Title") inline link.

## 图片

图像与链接类似， 但在链接语法之前需要添加额外的 `!` 字符

```
![替代文字](/path/to/img.jpg)
![替代文字](/path/to/img.jpg "可选标题")
```

## URL网址

```
用 <括号括起来> 括起来的URL地址会以URL，如<i@typora.io> 成为 i@typora.io.
```

Typora也将自动链接标准URL。例如： www.google.com.

## 文本强调

### 斜体

```
*斜体*
_斜体_
```

*斜体*

想在文本中使用*就得用转义了

如：\*使用星号\*

### 粗体

```
**粗体**  双星
__粗体__  双__
```

**粗体**

## 行内代码

使用反引号 ` 包括的代码会以特殊格式显示

```
`printf()`
```

使用`printf()`函数。

## 下划线与删除线

### 下划线

```
<u>下划线</u> 
```

<u>下划线</u> 

### 删除线

标准的Markdown中缺少该格式，typora中删除线如下

```
~~错误的文字~~
```

~~错误的文字~~















