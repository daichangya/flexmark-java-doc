flexmark-java PDF Renderer扩展

### 概述

`flexmark-pdf-converter`模块使用以下命令将HTML呈现为PDF
[将HTML打开为PDF](https://github.com/danfickle/openhtmltopdf),因此需要完全转换
将Markdown呈现为HTML,然后将其传递给PDF转换器扩展。

### 用法

提供了示例应用程序
[PdfConverter.java](https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/PdfConverter.java)。

### 渲染非拉丁字符集

OpenHtmlToPDF库中的捆绑字体仅支持基本的拉丁Unicode字符。至
渲染其他字符集在转换的HTML中必须包含其他字体。

字体问题的一种解决方案是在样式或样式表中定义嵌入式TrueType字体
并设置`body`标签以使用此字体。OpenHtmlToPDF将使用字体中的字符
定义了它们。

例如,包括[Noto](https://www.google.com/get/noto/)Serif/Sans/Mono字体并添加
`noto-serif`,`noto-sans`和`noto-mono`系列CSS允许PDF将其用于
呈现文本。

但是,PDF转换器需要TrueType字体,而`Noto CJK`字体是OpenFonts,
不能使用。解决方案是下载支持CJK字符的TrueType Unicode字体
设置并将其添加到要用于PDF的自定义渲染配置文件中。

对于我的测试,我使用了https://www.wfonts.com/font/arial-unicode-ms的`arialuni.ttf`

如果字体的安装目录为`/usr/local/fonts/`,则
样式表或页面`<head>` `<style>...</style> </head>`应该添加:

:information_source:不要忘记更改操作系统和安装目录的路径。上
Windows路径应以`file:/X:/...`开头,其中`X:/...`是驱动器号,后跟
完整的安装路径。

```css
@font-face {
  font-family: 'noto-cjk';
  src: url('file:///usr/local/fonts/arialuni.ttf');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'noto-serif';
  src: url('file:///usr/local/fonts/NotoSerif-Regular.ttf');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'noto-serif';
  src: url('file:///usr/local/fonts/NotoSerif-Bold.ttf');
  font-weight: bold;
  font-style: normal;
}

@font-face {
  font-family: 'noto-serif';
  src: url('file:///usr/local/fonts/NotoSerif-BoldItalic.ttf');
  font-weight: bold;
  font-style: italic;
}

@font-face {
  font-family: 'noto-serif';
  src: url('file:///usr/local/fonts/NotoSerif-Italic.ttf');
  font-weight: normal;
  font-style: italic;
}

@font-face {
  font-family: 'noto-sans';
  src: url('file:///usr/local/fonts/NotoSans-Regular.ttf');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'noto-sans';
  src: url('file:///usr/local/fonts/NotoSans-Bold.ttf');
  font-weight: bold;
  font-style: normal;
}

@font-face {
  font-family: 'noto-sans';
  src: url('file:///usr/local/fonts/NotoSans-BoldItalic.ttf');
  font-weight: bold;
  font-style: italic;
}

@font-face {
  font-family: 'noto-sans';
  src: url('file:///usr/local/fonts/NotoSans-Italic.ttf');
  font-weight: normal;
  font-style: italic;
}


@font-face {
  font-family: 'noto-mono';
  src: url('file:///usr/local/fonts/NotoMono-Regular.ttf');
  font-weight: normal;
  font-style: normal;
}

body {
    font-family: 'noto-sans', 'noto-cjk', sans-serif;
    overflow: hidden;
    word-wrap: break-word;
    font-size: 14px;
}

var,
code,
kbd,
pre {
    font: 0.9em 'noto-mono', Consolas, "Liberation Mono", Menlo, Courier, monospace;
}

```

### 细节

使用工件`flexmark-pdf-converter`中的类`PdfConverterExtension`。

提供以下选项:

在`PdfConverterExtension`类中定义:

|静态字段|默认值|说明|
|:----------------------- |:----------------------- -------------------------- |:---------------------- -------------------------------------------------- ---- |
| DEFAULT_TEXT_DIRECTION | `(PdfRendererBuilder.TextDirection) null` |文档|的默认文本方向
| PROTECTION_POLICY | `(ProtectionPolicy) null` |文档保护策略[PDF Box Documentation],[Related OpenHtmlToPdf] |
| DEFAULT_CSS |默认的嵌入式CSS,用于改进TOC生成| [default.css] |

:information_source:仅在以下情况下添加默认CSS:
`PdfConverterExtension.exportToPdf(OutputStream, String, String, DataHolder)`用于提供
选项。对于所有其他调用,您需要先将默认的CSS嵌入HTML字符串中,
导出为PDF。您可以为此使用`PdfConverterExtension.embedCss(String html, String css)`
目的并使用获取默认CSS的内容
`PdfConverterExtension.DEFAULT_CSS.getFrom(null)`

:information_source:如果您使用的是`TocExtension`,则需要将`TocExtension.LIST_CLASS`设置为
`PdfConverterExtension.DEFAULT_TOC_LIST_CLASS`

[default.css]: https://github.com/vsch/flexmark-java/blob/master/flexmark-pdf-converter/src/main/resources/default.css
[PDF Box Documentation]: https://pdfbox.apache.org/docs/2.0.5/javadocs/org/apache/pdfbox/pdmodel/encryption/AccessPermission.html
[Related OpenHtmlToPdf]: https://github.com/danfickle/openhtmltopdf/issues/30