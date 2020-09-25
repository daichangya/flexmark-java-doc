flexmark-java扩展,用于在markdown中定义和扩展简单的宏

### 概述

宏定义是块元素,可以包含任何markdown元素,但可以是
在块或内联上下文中扩展,允许在仅内联的地方使用块元素
语法允许使用元素。

### 句法

宏定义以`>>>macroName`开头,其中宏名称由az,AZ,
0-9,-或_,并以`<<<`结束。这些行之间的所有文本都解析为
markdown块。宏定义块只能在文档级别定义,而无需任何定义
缩进,并且不会在输出中呈现。

宏引用是形式为`<<<macroName>>>`的内联元素,并由宏替换
定义呈现的内容,包括任何块元素。这意味着有可能
包括通常使用纯markdown不可能使用的块元素。

当宏定义包含单个段落节点时,它将被呈现为文本
没有`<p></p>`包装器的段落中包含的内容,以允许将纯文本呈现为
内联文本。

如果宏引用包含未定义的`macroName`,则该节点将呈现为纯文本
由`<<<macroName>>>`组成的文本

通过在第一个宏引用上停止宏扩展来解析递归宏
指当前正在扩展的宏定义。第一个递归宏
参考将不会导致扩展。

## Examples

```markdown
Paragraph with a <<<simpleMacro>>>.

>>>simpleMacro
**bold** text
<<<
```

等效于:

```markdown
Paragraph with a **bold** text.
```

```markdown
Paragraph with a <<<blockMacro>>> inserted.

>>>blockMacro
1. item 1
1. item 2
<<<
```

呈现为:

```html
<p>Paragraph with a 
<ol>
  <li>item 1</li>
  <li>item 2</li>
</ol>
inserted.</p>
```

给予:

<p>带有
<ol>
  <li>项目1 </li>
  <li>项目2 </li>
</ol>
插入。</p>


使用宏可以向表中添加复杂的内容。

```markdown
|   Complex   |     Data     |
|-------------|--------------|
| <<<macro>>> | <<<macro2>>> |

>>>macro
1. Item 1
2. Item 2
3. Item 3

| Column 1 | Column 2 |
|----------|----------|
| a        | b        |
| c        | d        |

> Block Quote and more

<<<

>>>macro2
- Item 1
- Item 2
- Item 3
<<<

```

呈现为:

```html
<table>
  <thead>
    <tr><th>   Complex   </th><th>     Data     </th></tr>
  </thead>
  <tbody>
    <tr><td> 
    <ol>
      <li>Item 1</li>
      <li>Item 2</li>
      <li>Item 3</li>
    </ol>
    <table>
      <thead>
        <tr><th> Column 1 </th><th> Column 2 </th></tr>
      </thead>
      <tbody>
        <tr><td> a        </td><td> b        </td></tr>
        <tr><td> c        </td><td> d        </td></tr>
      </tbody>
    </table>
    <blockquote>
      <p>Block Quote and more</p>
    </blockquote>
    </td><td> 
    <ul>
      <li>Item 1</li>
      <li>Item 2</li>
      <li>Item 3</li>
    </ul>
    </td></tr>
  </tbody>
</table>
```

以导致:

<table>
  <thead>
    <tr> <th>复杂</th> <th>数据</th> </tr>
  </thead>
  <body>
    <tr> <td>
    <ol>
      <li>项目1 </li>
      <li>第2项</li>
      <li>第3项</li>
    </ol>
    <table>
      <thead>
        <tr> <th>第1列</th> <th>第2列</th> </tr>
      </thead>
      <body>
        <tr> <td> a </td> <td> b </td> </tr>
        <tr> <td> c </td> <td> d </td> </tr>
      </tbody>
    </table>
    <blockquote>
      <p> Block Quote等</p>
    </blockquote>
    </td> <td>
    <ul>
      <li>项目1 </li>
      <li>第2项</li>
      <li>第3项</li>
    </ul>
    </td> </tr>
  </tbody>
</table>

### 解析详细信息

在工件`flexmark-ext-macros`中使用类`MacrosExtension`。

提供以下选项:

在`MacrosExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------------- |:--------------- ---------- |:-------------------------------------- -------------------------------------------------- --------------- |
| `MACRO_DEFINITIONS` |新的存储库|用于文档的宏定义的存储库|
| `MACRO_DEFINITIONS_KEEP` | `KeepType.FIRST` |可重复保存。|
| `MACRO_DEFINITIONS_PLACEMENT` | `ElementPlacement.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `MACRO_DEFINITIONS_SORT` | `ElementPlacementSort.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `SOURCE_WRAP_MACRO_REFERENCES` | `false` |为true时,会将宏参考生成的代码包装在带有源位置信息|的`div`或`span`中