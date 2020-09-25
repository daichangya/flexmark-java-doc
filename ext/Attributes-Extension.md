flexmark-java扩展,用于属性列表处理

### 概述

将属性`{...}`语法转换为属性AST节点,并将属性提供程序添加到
设置HTML呈现期间紧邻的前一个兄弟元素的属性。

### 句法

该属性是以下各项之一的以空格分隔的属性语法列表:

* `name=value`
* `name='value'`
* `name="value"`
* `#id`
* `.class`

以`#`开头的属性(例如`#id-string`)等效于`id="id-string"`和
`.class-name`等效于`class="class-name"`

**注意**:属性的多值分配处理取决于其名称:

* `class`值作为空格(` `)分隔列表累积。
* `style`值以分号(`;`)分隔列表的形式累积。
*所有其他值将覆盖具有相同名称的所有先前值。

### 解析详细信息

* 与`ASSIGN_TEXT_ATTRIBUTES`选项向后兼容的是`false`

  应用于前一个元素,如果该元素是纯文本,则将属性应用于
  文本的父元素。

  如果attributes元素是段落的第一个元素,则attribute将是
  应用于段落的父元素。有效地将属性元素放在
  段落允许您设置列表项,块引号或其他段落的属性
  容器。

* 带`ASSIGN_TEXT_ATTRIBUTES`选项的默认扩展行为是`true`

  如果属性元素与先前的纯文本元素`some text
  {attributes}`隔开,则属性用于文本的父项。如果它们附在文本上
  元素`some text{attributes}`然后将它们分配给紧接在前的纯文本
  跨度。内联元素中应用相同的规则:`**bold text {attr}**`用于强
  重点跨度,而`**bold text{attr}**`会将文字放在重点之间
  具有给定属性的范围内的标签。

  如果纯文本范围被HTML注释打断,则将其视为两个单独的
  纯文本范围。即。`some <!---->text{attr}`将导致`some <!----><span
  attr>text</span>`渲染。

由工件`flexmark-ext-attributes`在`AttributeExtension`中定义

* `ASSIGN_TEXT_ATTRIBUTES`,默认为`true`。当`false`对节点的属性分配规则是
  更改为不允许文本元素获取属性。

* `FENCED_CODE_INFO_ATTRIBUTES`,默认为`true`。当`false`属性分配在末尾时
  受防护的代码信息字符串将被忽略。

    :information_source:只有在信息字符串行末尾的attribute元素才是
    解析。

    例如:

    ````
    ```info {#not-id} not {title="Title" caption="Cap"} {caption="Caption"}

    ```
    ````

    `{#not-id}`不会解析为属性,因为它会跟随非属性文本和
    将产生以下HTML:

    ```html
    <pre title="Title" caption="Caption"><code class="language-info">
    </code></pre>
    ```

    和AST:

    ```
    Document[0, 81]
      FencedCodeBlock[0, 76] open:[0, 3, "```"] info:[3, 22, "info {#not-id} not "] attributes:[22, 71, "{title=\"Title\" caption=\"Cap\"} {caption=\"Caption\"}"] content:[72, 73] lines[1] close:[73, 76, "```"]
        AttributesNode[22, 51] textOpen:[22, 23, "{"] text:[23, 50, "title=\"Title\" caption=\"Cap\""] textClose:[50, 51, "}"]
          AttributeNode[23, 36] name:[23, 28, "title"] sep:[28, 29, "="] valueOpen:[29, 30, "\""] value:[30, 35, "Title"] valueClose:[35, 36, "\""]
          AttributeNode[37, 50] name:[37, 44, "caption"] sep:[44, 45, "="] valueOpen:[45, 46, "\""] value:[46, 49, "Cap"] valueClose:[49, 50, "\""]
        AttributesNode[52, 71] textOpen:[52, 53, "{"] text:[53, 70, "caption=\"Caption\""] textClose:[70, 71, "}"]
          AttributeNode[53, 70] name:[53, 60, "caption"] sep:[60, 61, "="] valueOpen:[61, 62, "\""] value:[62, 69, "Caption"] valueClose:[69, 70, "\""]
        Text[72, 73] chars:[72, 73, "\n"]
    ```

* `USE_EMPTY_IMPLICIT_AS_SPAN_DELIMITER`默认为`false`。`true`时,将处理`{.}`或`{#}`
  作为最接近匹配属性节点开始的标记,以更好地控制位置
  属性以文本形式附加。

使用工件`flexmark-ext-attributes`中的类`AttributesExtension`。

完整规格:
[ext_attributes_ast_spec](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-attributes/src/test/resources/ext_attributes_ast_spec.md)