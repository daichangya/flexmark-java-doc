### 概述

创建块样式的辅助内容。基于[Admonition Extension, Material for MkDocs]
(*个人观点:MkDocs的材料令人眼花。乱。如果您没有看过它,则表示
缺少视觉享受。*)。

### 句法

#### 块样式的附带内容 

    !!! qualifier "Optional Title"
        block content 

#### 无标题内容

    !!! qualifier ""
        block content 

#### 可折叠块样式侧面内容: 

##### 默认折叠

    ??? qualifier "Optional Title"
        block content 

##### 默认打开

    ???+ qualifier "Optional Title"
         block content 

包含SVG图片和样式表作为默认限定符。用户可定义的限定词可以是
通过指定公认的样式限定符,它们的别名和图像映射来添加。

限定词用于选择块的图标和颜色。

<table>
<tr> <td>

* 抽象风格
  * `abstract`
  * `summary`
  * `tldr`
* 错误样式
  * `bug`
* 危险风格
  * `danger`
  * `error`
* 示例样式
  * `example`
  * `snippet`
* 失败的风格
  * `failure`
  * `fail`
  * `missing`
* 常见问题
  * `question`
  * `help`
  * `faq`
* 信息风格
  * `info`
  * `todo`
* 笔记样式
  * `note`
  * `seealso`
* 引用风格
  * `quote`
  * `cite`
* 成功风格
  * `success`
  * `check`
  * `done`
* 提示样式
  * `tip`
  * `hint`
  * `important`
* 警告样式
  * `warning`
  * `caution`
  * `attention`

</td> <td>

<img src ="https://github.com/vsch/flexmark-java/raw/master/flexmark-ext-admonition/src/main/javadoc/AdmonitionExample.png" alt ="AdmonitionExample.png" height = 1000/>

</td> </tr>
</table>

### 解析详细信息

使用工件`flexmark-ext-admonition`中的类`AdmonitionExtension`。

在`AdmonitionExtension`类中定义:

#### CSS和JavaScript必须包含在您的页面中 

默认的CSS和JavaScript作为资源包含在jar中:

* [admonition.css](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/resources/admonition.css)
* [admonition.js](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/resources/admonition.js)

也可以通过调用`AdmonitionExtension.getDefaultCSS()`和
`AdmonitionExtension.getDefaultScript()`静态方法。

该脚本应包含在文档正文的底部,并用于切换
可折叠警告元素的打开/关闭状态。

提供以下选项:

在`AdmonitionExtension`类中定义:

|静态字段|默认值|说明|
|:-------------------------------------- |:-------- ------ |:------------------------------------------ -------------------------------------------- |
| CONTENT_INDENT | `4` |子节点缩进|
| ALLOW_LEADING_SPACE | `true` |如果在打开标记|之前允许非缩进前导空间
| INTERRUPTS_PARAGRAPH | `true` |如果为false,则在|之前需要空白行
| INTERRUPTS_ITEM_PARAGRAPH | `true` |如果为false,则列表项|中之前需要空行
| WITH_SPACES_INTERRUPTS_ITEM_PARAGRAPH | `true` |在列表项|中是否具有前导非缩进空间是否需要空行
| ALLOW_LAZY_CONTINUATION | `true` |如果为true,则第一段可以是惰性的,否则要求内容缩进|
| UNRESOLVED_QUALIFIER | `"note"` |类型,当不知道限定符时使用
| QUALIFIER_TYPE_MAP |请参见下面的|限定符到|类型的映射
| QUALIFIER_TITLE_MAP |当未指定标题时,请参见下面的|限定词到默认标题的映射
| TYPE_SVG_MAP |请参见下面的|图标|的类型到svg的映射

* 默认限定符用于输入map:
  * abstract: `"abstract"`
  * summary: `"abstract"`
  * tldr: `"abstract"`
  * bug: `"bug"`
  * danger: `"danger"`
  * error: `"danger"`
  * example: `"example"`
  * snippet: `"example"`
  * fail: `"fail"`
  * failure: `"fail"`
  * missing: `"fail"`
  * faq: `"faq"`
  * question: `"faq"`
  * help: `"faq"`
  * info: `"info"`
  * todo: `"info"`
  * note: `"note"`
  * seealso: `"note"`
  * quote: `"quote"`
  * cite: `"quote"`
  * success: `"success"`
  * check: `"success"`
  * done: `"success"`
  * tip: `"tip"`
  * hint: `"tip"`
  * important: `"tip"`
  * warning: `"warning"`
  * caution: `"warning"`
  * attention: `"warning"`

* 标题地图的默认限定词:
  * abstract: `"Abstract"`
  * summary: `"Summary"`
  * tldr: `"tldr"`
  * bug: `"Bug"`
  * danger: `"Danger"`
  * error: `"Error"`
  * example: `"Example"`
  * snippet: `"Snippet"`
  * fail: `"Fail"`
  * failure: `"Failure"`
  * missing: `"Missing"`
  * faq: `"Faq"`
  * question: `"Question"`
  * help: `"Help"`
  * info: `"Info"`
  * todo: `"To Do"`
  * note: `"Note"`
  * seealso: `"See Also"`
  * quote: `"Quote"`
  * cite: `"Cite"`
  * success: `"Success"`
  * check: `"Check"`
  * done: `"Done"`
  * tip: `"Tip"`
  * hint: `"Hint"`
  * important: `"Important"`
  * warning: `"Warning"`
  * caution: `"Caution"`
  * attention: `"Attention"`

* SVG映射的默认类型提供了用于给定类型的SVG图标。提供的默认类型:
  
  * abstract:  <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-abstract.svg" alt="adm-abstract.svg" width="24" align="absmiddle" />
  * bug:       <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-bug.svg" alt="adm-bug.svg" width="24" align="absmiddle" />
  * danger:    <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-danger.svg" alt="adm-danger.svg" width="24" align="absmiddle" />
  * example:   <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-example.svg" alt="adm-example.svg" width="24" align="absmiddle" />
  * fail:      <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-fail.svg" alt="adm-fail.svg" width="24" align="absmiddle" />
  * faq:       <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-faq.svg" alt="adm-faq.svg" width="24" align="absmiddle" />
  * info:      <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-info.svg" alt="adm-info.svg" width="24" align="absmiddle" />
  * note:      <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-note.svg" alt="adm-note.svg" width="24" align="absmiddle">
  * success:   <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-success.svg" alt="adm-success.svg" width="24" align="absmiddle" />
  * tip:       <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-tip.svg" alt="adm-tip.svg" width="24" align="absmiddle" />
  * warning:   <img src="https://raw.githubusercontent.com/vsch/flexmark-java/master/flexmark-ext-admonition/src/main/resources/images/adm-warning.svg" alt="adm-warning.svg" width="24" align="absmiddle" />

  :warning: 添加自己的SVG图标时,请确保已为#
  svg,否则它将不遵循该类型的CSS颜色。


[Admonition Extension, Material for MkDocs]: https://squidfunk.github.io/mkdocs-material/extensions/admonition/