
在`flexmark-docx-converter`模块中找到`DocxRenderer`。

呈现过程涉及创建一个空的`WordprocessingMLPackage`并将其传递给
渲染器,以便可以将渲染的markdown内容添加到此包中。

渲染器使用包中定义的样式为markdown元素应用格式。
定制输出是更改传递给渲染器的样式的问题。

由于渲染将内容附加到包中,因此您可以将任何非空文档传递到
附加了markdown内容。这包括将多个markdown文档呈现为一个
包。

模块中包含XML格式的默认空文档,您可以简单地使用
`DocxRenderer.getDefaultTemplate()`静态方法来获取它。

还有其他方便的方法可以从流或资源中读取模板。

包装中的样式应具有ID,这些ID将与各种
markdown元素被渲染。默认值在`empty.xml`文档中使用
默认使用的模板。

渲染器将从包装中读取样式,并尽最大努力将样式传播到嵌套样式
markdown元素。

为了方便起见,还有一个`DocxContextImpl`类可用于创建docx内容
通过代码。渲染器使用它,`ComboDocxConverterSpecTest`测试文件也使用它
它将所有测试的状态,markdown来源以及由此产生的docx转换附加到
单一文件。如果需要,它是示例代码的良好来源。

[DocxConverter CommonMark Sample]和[DocxConverter Pegdown Sample]文件显示了如何使用此文件
模块。

## Customizing the Styles

渲染器依靠样式配置来生成文档。如果您修改样式
不一致的结果将反映出这一点。

样式以XML格式显示在用于渲染的默认模板文件`empty.xml`中,您
可以复制它并进行修改以供您使用。之后,传递字符串或输入
流到该内容的`DocxRenderer.getDefaultTemplate()`。通过流时,
数据不限于XML,可以是docx文档流。

除非您对操作Docx格式非常满意,并且不介意调试错误
样式分配,建议使用默认样式名称,并且仅使用样式
更改定义以反映您所需的样式。

使其工作的另一种方法是在支持以下功能的文字处理器中打开`empty.xml`程序包
(Libre Office,Word)并修改样式,然后将文档另存为XML或docx。

也可以使用MS Word修改样式,但为此您需要从docx开始
已经包含所有所需样式的文档。在分发的根目录中
jar文件,您将找到[flexmark-empty-template.docx]文件。该文件包含所有
您可以修改的转换中使用的markdown元素和样式。它还包含
有关成功实现此目标的最佳方法的说明。

该文件是通过在[empty.md]文件上运行docx转换而产生的,该文件也位于
jar,带有默认模板。

使用`empty.md`和`empty.xml`模板生成的样本
`flexmark-empty-template.docx`可以在
[DocxConverterEmpty Sample]

如果您确实创建了样式模板文档,则强烈建议您运行
`empty.md`通过转换,使用修改后的模板文档来确保所有样式
存在并且在此过程中没有任何混乱。

## Image Embedding

渲染器将`DocxLinkResolver`用于基本链接解析,用于文档相对和
网站相对网址。为此,您需要提供`DocxRenderer.DOC_RELATIVE_URL`
和`DocxRenderer.DOC_ROOT_URL`,以便您的链接可以正确映射到文件或http://
资源。

渲染器将在文档中嵌入通过图像或图像引用链接的所有图像。解决
链接可以是http:或file:协议。无论哪种情况,渲染器都将加载文件并嵌入
它在文档中。

您可以提供自己的链接解析器以自定义链接解析规则。链接解析器
`DocxRenderer`使用的代码与`HtmlRenderer`使用的代码相同,因此您可以重复使用
现有的HTML自定义链接解析器。

## Limitations

Docx格式不允许与浏览器中的HTML/CSS相同的灵活性。特别,
嵌套的边框不适用于文本段落。因此,如果markdown中的元素是
嵌套在多个渲染边界的父对象中,然后是最近的父对象的边界
将被渲染。这特别影响默认模板中的块引号。

此外,在docx中,距左边界的边界偏移量以pt(点,1/72
in。),并且最大限制为31 pt。其他测量值以缇(点的1/20)为单位,无
实际限制。这创造了孩子缩进与父母缩进的条件
缩进可以轻易超过31 pt的限制,从而无法保持孩子的边界
(实际上是父母的延伸到孩子)与父母的边界对齐。当这个情况发生时
遵守31 pt的限制,并且子边框会偏离父边框。

另一个警告是,左边距和悬空的缩进具有边框分辨率的20倍
偏移量。这意味着不可能使孩子的边界与
除非孩子的缩进和父母的缩进相差整数倍。

渲染器检测到不是这种情况时,这将导致子边框明显可见
错位,并调整孩子的左边界以消除这种错位。除非你的眼睛
非常敏感,您不会注意到文本左边缘的偏移小于1 pt,而
任何人都可以注意到直线中断了1/2磅。

## Acknowledgments

该模块的开发由以下机构赞助
[Johner Institut GmbH](https://www.johner-institut.de)。需要它以方便转换
他们的内部文档的价格降到了广大用户偏爱的docx格式。

该模块使用[docx4j]库来处理所有docx操作。

[docx4j]: https://www.docx4java.org/trac/docx4j
[DocxConverter CommonMark Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterCommonMark.java
[DocxConverter Pegdown Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterPegdown.java
[empty.md]: https://github.com/vsch/flexmark-java/blob/master/flexmark-docx-converter/src/main/resources/empty.md
[flexmark-empty-template.docx]: https://github.com/vsch/flexmark-java/blob/master/flexmark-docx-converter/src/main/resources/flexmark-empty-template.docx
[DocxConverterEmpty Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterEmpty.java