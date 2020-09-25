flexmark-java Docx-Renderer扩展

### 概述

使用[docx4j]库将已解析的Markdown AST呈现为docx格式。

有关代码,请参见[DocxConverterCommonMark Sample],有关代码,请参见[Customizing Docx Rendering](ext/Customizing-Docx-Rendering.md)
概述和有关自定义样式的信息。

Pegdown版本可以在[DocxConverterPegdown Sample]中找到

:warning:Java7的Emoji扩展名将不会加载GitHub提供的图像。使用Java8 +还是不使用
将`EmojiExtension.USE_SHORTCUT_TYPE`设置为`EmojiShortcutType.GITHUB`或
`EmojiShortcutType.ANY_GITHUB_PREFERRED`导致使用GitHub提供的图像。

### 句法

渲染由flexmark-java解析器生成的AST。由此没有实现特殊语法
延期。

#### 有限属性节点处理

* 如果样式ID为,则段落元素上的`.className`会将docx styleId设置为`className`
  找到了。这允许使用特定的样式ID更改段落的格式,特别是
  `pagebreak`和`tab`类除外。
* 通过`{.pagebreak}`属性进行分页
* 通过`{.tab}`属性标签
* 使用`{style=""}`设置文本或块元素的属性。仅以下是
  处理:
  * `color`-文字颜色
  * `background-color`-阴影填充颜色,图案始终为实心。
  * `font-family`-**未实现**
  * `font-size`-以磅为单位,四舍五入至最接近的1/2磅。`pt`单位是可选的。
  * `font-weight`-设置/清除粗体(如果使用数字权重,则> = 550设置粗体,清除较少
    它)
  * `font-style`-设置/清除斜体
* 与`{align=}`的嵌入式图像对齐:
  * `left`-左对齐,将文本右对齐
  * `right`-右对齐,将文本左移
  * `center`-居中对齐,左右对齐文本
  * 否则图像不环绕,图像插入文本

### 解析详细信息

工件:`flexmark-docx-converter`

提供以下选项:

在`DocxRenderer`类中定义:


- `CODE_HIGHLIGHT_SHADING`默认值`""`,当非空将使用此颜色作为突出显示时,也将`NO_CHARACTER_STYLES`覆盖为true,请参见[NOTE on Highlight Colors]颜色。
- `CUSTOM_PROPERTIES`默认值`Collections.emptyMap()`,设置为`Map<String, String>`,其中包含要在文档中设置的自定义属性的属性名称到属性值的映射。参考。在某些情况下需要进行后期处理。
- `DEFAULT_LINK_RESOLVER`默认`true`,使用默认链接解析器,该解析器使用`DOC_RELATIVE_URL`和`DOC_ROOT_URL`选项
- `DEFAULT_TEMPLATE_RESOURCE`默认`"/empty.xml"`,默认模板资源路径
- `DOC_EMOJI_IMAGE_VERT_OFFSET`默认值`-0.10`,表情符号图片的垂直偏移是插入点处线条高度的因素。最终值将四舍五入到最接近的pt,因此可能会因该值的微小变化而跳1 pt。
- `DOC_EMOJI_IMAGE_VERT_SIZE`默认值` 1.05`,表情符号图片的大小作为插入点线高的一个因素。
- `DOC_RELATIVE_URL`默认值`""`,用于所有相对URL的前缀:不以协议或`/`开头
- `DOC_ROOT_URL`默认值`""`,用于所有绝对URL的前缀:以`/`开头的前缀
- `ERROR_SOURCE_FILE`默认值`""`,在错误日志中使用的源文件名
- `ERRORS_TO_STDERR`默认`false`,将错误记录到stdout
- `FORM_CONTROLS`默认值`""`,设置为表单控件引用的名称,以生成具有此键给定名称的表单控件`[name]{.type attributes}`
- `LINEBREAK_ON_INLINE_HTML_BR`默认`true`,将内嵌HTML `<br>`转换为docx中的换行符
- `LOCAL_HYPERLINK_MISSING_FORMAT`默认值`"Missing target id: #%s"`,当非空值在给定字符串上使用String.format()且缺少参考锚作为参数时,会为未解析的超链接生成工具提示
- `LOCAL_HYPERLINK_MISSING_HIGHLIGHT`默认值`"red"`,当非空时将以这种颜色突出显示文档本地未解析的超链接。参见[NOTE on Highlight Colors]颜色。
- `LOCAL_HYPERLINK_SUFFIX`默认值`""`,将此后缀附加到文档超链接锚参考中。在某些情况下需要进行后期处理。
- `LOG_IMAGE_PROCESSING`默认`false`,日志图像处理错误
- `MAX_IMAGE_WIDTH`默认`0`,最大图像宽度,0否最大
- `NO_CHARACTER_STYLES`默认值`false`,如果为true,则不会设置字符样式,而是通过样式显式设置运行值
- `NUMBERING_XML`默认值`getResourceString("/numbering.xml")`,如果字处理程序包中缺少默认编号部分
- `PREFIX_WWW_LINKS`默认值`true`,控制以`www.`开头的链接是否以`https://`为前缀
- `RENDER_BODY_ONLY`默认为`false`,渲染为字符串时仅输出文档部分的主体。用于测试。
- `STYLES_XML`默认值`getResourceString("/styles.xml")`,如果在文字处理包中缺少默认样式部分
- `TABLE_CAPTION_BEFORE_TABLE`默认`false`,在表格前插入标题
- `TABLE_CAPTION_TO_PARAGRAPH`默认`true`,将表标题转换为以`TableCaption`样式ID设置样式的段落
- `TABLE_LEFT_INDENT`默认值`120`,表格左缩进缇
- `TABLE_PREFERRED_WIDTH_PCT`默认`0`,首选表格宽度
- `TABLE_STYLE`默认`""`,表格字体样式
- `TOC_GENERATE`默认值`false`,即使文件中不存在TOC Markdown元素,是否生成TOC
- `TOC_INSTRUCTION`默认值`"TOC \\o \"1-3\" \\h \\z \\u "`,定义用于TOC元素的指令字符串

##### 关于突出显示颜色的注意

Docx格式需要指定的颜色。提供的任何与命名颜色都不匹配的颜色将是
转换为最接近的命名颜色。

当`CODE_HIGHLIGHT_SHADING`设置为`"shade"`时,将使用最接近的命名颜色
从`SourceText`阴影填充颜色开始(如果有)。

##### 样式名称,用于呈现各种markdown元素

元素样式:


- `ASIDE_BLOCK_STYLE`默认`"AsideBlock"`,用于预留块的样式
- `BLOCK_QUOTE_STYLE`默认`"Quotations"`,用于块引用的样式
- `BOLD_STYLE`默认`"StrongEmphasis"`,用于markdown元素的样式
- `BULLET_LIST_STYLE`默认值`"BulletList"`,用于项目符号列表项段落的编号列表样式
- `DEFAULT_STYLE`默认`"Normal"`,用于markdown元素的样式
- `ENDNOTE_ANCHOR_STYLE`默认值`"EndnoteReference"`,用于markdown元素的样式
- `FOOTER`默认值`"Footer"`,用于markdown元素的样式
- `FOOTNOTE_ANCHOR_STYLE`默认`"FootnoteReference"`,用于markdown元素的样式
- `FOOTNOTE_STYLE`默认`"Footnote"`,用于脚注文本的样式
- `FOOTNOTE_TEXT`默认值`"FootnoteText"`,用于markdown元素的样式
- `HEADER`默认值`"Header"`,用于markdown元素的样式
- `HEADING_1`默认值`"Heading1"`,用于markdown元素的样式
- `HEADING_2`默认`"Heading2"`,用于markdown元素的样式
- `HEADING_3`默认`"Heading3"`,用于markdown元素的样式
- `HEADING_4`默认值`"Heading4"`,用于markdown元素的样式
- `HEADING_5`默认`"Heading5"`,用于markdown元素的样式
- `HEADING_6`默认值`"Heading6"`,用于markdown元素的样式
- `HORIZONTAL_LINE_STYLE`默认`"HorizontalLine"`,用于主题休息的样式
- `HYPERLINK_STYLE`默认值`"Hyperlink"`,用于markdown元素的样式
- `INLINE_CODE_STYLE`默认`"SourceText"`,用于markdown元素的样式
- `INS_STYLE`默认值`"Underlined"`,用于markdown元素的样式
- `ITALIC_STYLE`默认值`"Emphasis"`,用于markdown元素的样式
- `LOOSE_PARAGRAPH_STYLE`默认`"ParagraphTextBody"`,用于散列表类型项目的样式
- `NUMBERED_LIST_STYLE`默认值`"NumberedList"`,用于编号列表项段落的编号列表样式
- `PARAGRAPH_BULLET_LIST_STYLE`默认`"ListBullet"`,用于紧凑型列表项的样式
- `PARAGRAPH_NUMBERED_LIST_STYLE`默认值`"ListNumber"`,用于紧密列表类型项目的样式
- `PREFORMATTED_TEXT_STYLE`默认值`"PreformattedText"`,用于受防护代码和缩进代码的样式
- `STRIKE_THROUGH_STYLE`默认值`"Strikethrough"`,用于markdown元素的样式
- `SUBSCRIPT_STYLE`默认`"Subscript"`,用于markdown元素的样式
- `SUPERSCRIPT_STYLE`默认`"Superscript"`,用于markdown元素的样式
- `TABLE_CAPTION`默认`"TableCaption"`,用于表格标题的样式
- `TABLE_CONTENTS`默认`"TableContents"`,用于表体的样式
- `TABLE_GRID`默认值`"TableGrid"`,用于markdown元素的样式
- `TABLE_HEADING`默认`"TableHeading"`,用于表格标题的样式
- `TIGHT_PARAGRAPH_STYLE`默认`"BodyText"`,用于紧凑型列表项的样式

列出元素样式

无序列表使用编号为`BulletList`的编号列表样式,而有序列表使用
`NumberedList`。如果不存在,则使用默认编号样式(id = 2)
无序列表和默认编号样式(id = 3)用于有序列表。

以下等效于具有相同名称的`Renderer`属性。包括在
`DocxRenderer`为方便起见。

有关`TOC_INSTRUCTION`字符串,请参见
[Docx4j入门](https://www.docx4java.org/docx4j/Docx4j_GettingStarted.pdf)
标题`TOC Content Control`

注意:Word不能很好地处理插入的HTML。任何未被抑制的HTML都将被转义:即。
它将作为文本呈现到文档中。`<br>`标签为例外,如果启用
将呈现为换行符。


为了方便起见,在`DocxRenderer`中提供了HTML渲染选项:

- `ESCAPE_HTML_BLOCKS`的默认值`ESCAPE_HTML`,可转义文档中找到的html块
- `ESCAPE_HTML_COMMENT_BLOCKS`的默认值`ESCAPE_HTML_BLOCKS`,可转义文档中找到的html注释块。
- `ESCAPE_HTML`默认值`false`,转义文档中找到的所有html
- `ESCAPE_INLINE_HTML_COMMENTS`的默认值`ESCAPE_HTML_BLOCKS`,转义文档中找到的内联html
- `ESCAPE_INLINE_HTML`的默认值`ESCAPE_HTML`,可在文档中找到转义内联html
- `PERCENT_ENCODE_URLS`默认`false`,百分比编码网址
- `RECHECK_UNDEFINED_REFERENCES`默认为`false`,重新检查`Parser.REFERENCES`中是否存在标记为未定义的链接和图像引用的引用。解析后添加新引用时使用
- `SUPPRESS_HTML_BLOCKS`的默认值`SUPPRESS_HTML`,取消html块的html输出
- `SUPPRESS_HTML_COMMENT_BLOCKS`的默认值`SUPPRESS_HTML_BLOCKS`,取消html注释块的html输出
- `SUPPRESS_HTML`默认`false`,禁止所有html的html输出
- `SUPPRESS_INLINE_HTML_COMMENTS`的默认值`SUPPRESS_INLINE_HTML`,禁止将HTML输出用于内嵌html注释
- `SUPPRESS_INLINE_HTML`的默认值`SUPPRESS_HTML`,取消内联html的html输出
- `HEADER_ID_GENERATOR_NO_DUPED_DASHES`默认值`false`,如果ID中的`true`重复`- `被单个`- `替换
- `HEADER_ID_GENERATOR_RESOLVE_DUPES`默认值`true`,当`true`将添加一个递增整数以重复ID以使其唯一
- `HEADER_ID_GENERATOR_TO_DASH_CHARS`默认值`"_"`,用于生成ID的文本中要转换为`- `的字符集,未设置的非字母数字字符将被删除
- `HEADER_ID_GENERATOR_NON_ASCII_TO_LOWERCASE`,默认为`true`。设置为`false`时,更改默认的标头ID生成器,以不将非ascii字母字符转换为小写。需要`GitHub` id兼容性。
- `HEADER_ID_REF_TEXT_TRIM_LEADING_SPACES`,默认为`true`。当设置为`false`时,标题中链接参考文本中的前导空格不会为用于生成ID的文本修剪。
- `HEADER_ID_REF_TEXT_TRIM_TRAILING_SPACES`,默认为`true`。当设置为`false`时,标题中链接参考文本中的尾部空格不会为生成ID的文本修剪。
- `HEADER_ID_ADD_EMOJI_SHORTCUT`,默认为`false`。当设置为`true`时,表情符号快捷方式节点会将快捷方式添加到用于生成标题ID的收集文本中。
- `HEADER_ID_GENERATOR_TO_DASH_CHARS`默认值`"_"`,用于生成ID的文本中要转换为`- `的字符集,未设置的非字母数字字符将被删除
- `RENDER_HEADER_ID`默认值`false`,使用已配置的`HtmlIdGenerator`为标头呈现标头ID属性

[docx4j]: https://www.docx4java.org/trac/docx4j
[DocxConverterCommonMark Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterCommonMark.java
[DocxConverterPegdown Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterPegdown.java
[NOTE on Highlight Colors]: #note-on-highlight-colors