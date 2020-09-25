flexmark-java扩展,用于flexmark-java规格测试示例处理

### 概述

将规范示例语法转换为flexmark-java AST节点。 

[Markdown Navigator]插件将这个扩展用于JetBrains IDE,以简化与
flexmark-java测试规范文件.

[Markdown Navigator]: http://vladsch.com/product/markdown-navigator

    ```````````````````````````````` example SpecExample: 1
    markdown content
    .
    html content
    .
    AST content
    ````````````````````````````````

测试规范示例:

    ```````````````````````````````` example GFM Emphasis: 7
    **a*b*c**
    
    .
    <p><strong>a<em>b</em>c</strong></p>
    .
    Document[0, 11]
      Paragraph[0, 10] isTrailingBlankLine
        StrongEmphasis[0, 9] textOpen:[0, 2, "**"] text:[2, 7, "a*b*c"] textClose:[7, 9, "**"]
          Text[2, 3] chars:[2, 3, "a"]
          Emphasis[3, 6] textOpen:[3, 4, "*"] text:[4, 5, "b"] textClose:[5, 6, "*"]
            Text[4, 5] chars:[4, 5, "b"]
          Text[6, 7] chars:[6, 7, "c"]
    ````````````````````````````````

### 句法

语法非常具体,并且元素以以下内容开头:

    ```````````````````````````````` example GFM Emphasis: 7

或带有选项:

    ```````````````````````````````` example(GFM Emphasis: 7) options(option1, option2, etc)

包含1到3个部分:markdown,html和AST。各节之间用包含
`.`:

    。

并以:

    ````````````````````````````````

### 解析详细信息 

使用工件`flexmark-ext-spec-example`中的类`SpecExampleExtension`。

提供以下选项:

在`SpecExampleExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------------------ |:---------- -------------------------------------------------- ------- |:----------------------------------------- -------------------------------------------------- ----------------------------------------- |
| `SPEC_EXAMPLE_RENDER_HTML` | `true` |除受防护的代码HTML |外,是否将HTML节呈现为原始HTML
| `SPEC_EXAMPLE_RENDERED_HTML_PREFIX` | `<div style="border:solid #cccccc 1px;padding:0 20px 10px 20px;">` |呈现HTML部分的前缀,默认灰色1像素框(围绕|)
呈现的HTML部分|的后缀| `SPEC_EXAMPLE_RENDERED_HTML_SUFFIX` | `</div>` |
| `SPEC_EXAMPLE_RENDER_AS` | `RenderAs.FENCED_CODE` |如何呈现示例:**防御代码**,简单或定义列表|
| `SPEC_EXAMPLE_BREAK` |规格示例折返勾号|更改识别的前缀。用于测试。由于规范示例中指定了测试,因此用于更改默认设置的测试。|
规格示例正文中的| `SPEC_TYPE_BREAK` | `.` |分节符。由于规范示例中指定了测试,因此用于更改默认设置的测试。|
| `SPEC_OPTION_NODES` | `true` |是否在AST |中包含`SpecExampleOptionsList`和`SpecExampleOption`节点