flexmark-java扩展,用于枚举参考处理

### 概述

枚举引用基于Attributes elmenet指定的ID(请参见
[Attributes Extension](ext/Attributes-Extension))的形式为`category:elementId`。类别是任意类别
用于为类别中的每个元素生成数字序列的标识符。`elmentId`
part用于唯一地标识类别中的元素。

### 句法

要创建元素锚点或书签,请在之后使用`{#type:reference}`的id属性语法
要引用的元素。其中`type`是用于在其中保留单独编号的类别
类别和`reference`用于唯一标识类别中的元素。锚点
元素的ID将为`type:reference`,并且必须由枚举引用使用
标签或链接。

:information_source:`type`不能以数字开头。

要引用文档中的元素,请使用列举的引用元素:

1.参考链接语法`[@type:reference]`根据定义的类型模式转换为链接
   以枚举的参考格式定义。链接的目标将是带有
   分配给它的`{#type:reference}`属性。链接的文本将是
   由`type`的枚举参考定义定义。

2.参考文本语法`[#type:reference]`转换为由
   列举了`type`的参考定义。用于以下情况的识别文本:
   文档中的元素是必需的。

3.文档中必须包含每个`type`的参考定义。语法`[@type]:
   Label Text [#]`用于定义如何为给定`type`的所有元素创建标签。
   这通常包括元素在文档中的顺序位置。

   文本**以下** `[@type]: `用作标签,而`[@] or [#]`替换为
   文档中元素相对于同一`type`的其他元素的序号。
   第一个元素的序数为1,第二个为2,依此类推。

   如果类型没有枚举定义,则将使用`type [#]`,其中`type`是
   类别类型,并且`[#]`是元素在其类别中的序数。相当于
   在文档中指定`[@type]: type [#]`。

例如:

```
![Flexmark Icon Logo](https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-icon-logo%402x.png){#fig:test}
[#fig:test]

![Flexmark Icon Logo](https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-icon-logo%402x.png){#fig:test2}
[#fig:test2]

| heading | heading | heading |
|---------|---------|---------|
| data    | data    |         |
[[#tbl:test] caption]
{#tbl:test}

See [@fig:test2]

See [@fig:test]

See [@tbl:test]

[@tbl]: Table [#].

[@fig]: Figure [#].
```

等效于以下内容,而无需手动跟踪单个编号
元素:

```
![Flexmark Icon Logo](https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-icon-logo%402x.png){#fig:test}
Figure 1.

![Flexmark Icon Logo](https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-icon-logo%402x.png){#fig:test2}
Figure 2.

| table |
|-------|
| data  |
[Table 1. caption]
{#tbl:test}

See [Figure 2.](#fig:test2)

See [Figure 1.](#fig:test)

See [Table 1.](#tbl:test)

```

将呈现为:

```
<p><img src="http://example.com/test.png" alt="Fig" id="fig:test" /><br />
<span>Figure 1.</span></p>
<p><img src="http://example.com/test.png" alt="Fig" id="fig:test2" /><br />
<span>Figure 2.</span></p>
<table id="tbl:test">
  <thead>
    <tr><th>table</th></tr>
  </thead>
  <tbody>
    <tr><td>data</td></tr>
  </tbody>
  <caption><span>Table 1.</span> caption</caption>
</table>
<p></p>
<p>See <a href="#fig:test2"><span>Figure 2.</span></a></p>
<p>See <a href="#fig:test"><span>Figure 1.</span></a></p>
<p>See <a href="#tbl:test"><span>Table 1.</span></a></p>
```

## Enumerated Reference Text in Headings

由于标题包含自己的锚ID,因此仅使用`type`的枚举引用是
允许在标题中使用,并具有在标题文本中添加递增计数器的作用。

```markdown
# [#hd1] Heading 1

# [#hd1] Heading 2

# [#hd1] Heading 3

[@hd1]: [#].
```

将呈现为:

```html
<h1>1. Heading 1</h1>
<h1>2. Heading 2</h1>
<h1>3. Heading 3</h1>
```

## Compound Types

复合枚举引用类型是通过以下方式创建的:
`:`分隔每种类型。

复合引用的作用是将所有子引用计数器都重置为1以进行更改
在父类型的序数中允许使用枚举引用创建合法编号。

:information_source:当将枚举类型的序数字符串组合为枚举的化合物时
如果枚举格式定义的最后一个元素为空,则引用
参考文字`[#]`或空的枚举参考链接`[@]`,然后在#之后添加`.`
父级枚举序数文本。

:information_source:对于没有元素id的标题的复合类型,尾随`:`为
防止将最后一个`type`解释为元素ID。

```markdown
# [#hd1] Heading 1

## [#hd1:hd2:] Heading 1.1

### [#hd1:hd2:hd3:] Heading 1.1.1

### [#hd1:hd2:hd3:] Heading 1.1.2

## [#hd1:hd2:] Heading 1.2

### [#hd1:hd2:hd3:] Heading 1.2.1

### [#hd1:hd2:hd3:] Heading 1.2.2

# [#hd1] Heading 2

## [#hd1:hd2:] Heading 2.1

### [#hd1:hd2:hd3:] Heading 2.1.1

### [#hd1:hd2:hd3:] Heading 2.1.2

[@hd1]: [#].
[@hd2]: [#].
[@hd2]: [#].
```

将呈现为:

```html
<h1>1. Heading 1</h1>
<h2>1.1. Heading 1.1</h2>
<h3>1.1.1. Heading 1.1.1</h3>
<h3>1.1.2. Heading 1.1.2</h3>
<h2>1.2. Heading 1.2</h2>
<h3>1.2.1. Heading 1.2.1</h3>
<h3>1.2.2. Heading 1.2.2</h3>
<h1>2. Heading 2</h1>
<h2>2.1. Heading 2.1</h2>
<h3>2.1.1. Heading 2.1.1</h3>
<h3>2.1.2. Heading 2.1.2</h3>
```

同样,可以使用其他枚举引用:

```markdown
# [#hd1] Heading 1

![image-base64.png](https://github.com/vsch/MarkdownTest/raw/master/image-base64.png) {#hd1:fig:img1} 
[#hd1:fig:img1]

# [#hd1] Heading 2

![image-base64.png](https://github.com/vsch/MarkdownTest/raw/master/image-base64.png) {#hd1:fig:img2} 
[#hd1:fig:img2]

[@hd1]: [#].
[@fig]: Figure [#].
```

将呈现为:

```html
<h1>1. Heading 1</h1>
<p><img src="https://github.com/vsch/MarkdownTest/raw/master/image-base64.png" alt="image-base64.png" /><span id="hd1:fig:img1"> </span>
Figure 1.1.</p>
<h1>2. Heading 2</h1>
<p><img src="https://github.com/vsch/MarkdownTest/raw/master/image-base64.png" alt="image-base64.png" /><span id="hd1:fig:img2"> </span>
Figure 2.1.</p>
```

### 解析详细信息 

使用工件`flexmark-ext-enumerated-reference`中的类`EnumeratedReferenceExtension`。

:exclamation:注意[Attributes](Extensions#attributes)扩展名是为了
要正确解析以进行渲染的引用。