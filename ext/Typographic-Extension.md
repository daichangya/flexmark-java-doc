flexmark-java扩展,用于印刷字符处理

### 概述

转换:

* `'`撇号`&rsquo;`&rsquo;
* `...`和`. . .`省略号`&hellip;`&hellip;
* `--`破折号`&ndash;`&ndash;
* `---` em破折号`&mdash;`&mdash;
*用单引号`'some text'`至`&lsquo;some text&rsquo;`"一些文字"
*双引号`"some text"`至`&ldquo;some text&rdquo;`"某些文字"
*双角引用`<<some text>>`至`&laquo;some text&raquo;`&laquo;一些文本&raquo;

### 句法

此扩展没有提供特定的语法。

### 解析详细信息 

在工件`flexmark-ext-typographic`中使用类`TypographicExtension`。

提供以下选项:

在`TypographicExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------- |:-------------- |:----- -------------------------------------------------- -------------------------- |
| `ENABLE_QUOTES` | `true` |处理引号`"`,`'`,`<<`,`>>` |
| `ENABLE_SMARTS` | `true` |流程智能:`'`,`...`,`. . .`,`--`,`---` |
| `ANGLE_QUOTE_CLOSE` | `"&raquo;"` |用于相应的印刷顺序|
| `ANGLE_QUOTE_OPEN` | `"&laquo;"` |用于对应的印刷顺序|
| `ANGLE_QUOTE_UNMATCHED` | `null` |当打开或关闭序列都不使用时使用什么,如果原样保留则为null。|
| `DOUBLE_QUOTE_CLOSE` | `"&rdquo;"` |用于对应的印刷顺序的内容|
| `DOUBLE_QUOTE_OPEN` | `"&ldquo;"` |用于对应的印刷顺序|
| `DOUBLE_QUOTE_UNMATCHED` | `null` |当打开或关闭序列都不使用时使用什么,如果原样保留则为null。|
| `ELLIPSIS` | `"&hellip;"` |用于对应的印刷顺序|
| `ELLIPSIS_SPACED` | `"&hellip;"` |用于对应的印刷顺序|
| `EM_DASH` | `"&mdash;"` |用于相应的印刷顺序|
| `EN_DASH` | `"&ndash;"` |用于对应的印刷顺序|
| `SINGLE_QUOTE_CLOSE` | `"&rsquo;"` |用于对应的印刷顺序的内容|
| `SINGLE_QUOTE_OPEN` | `"&lsquo;"` |用于相应的印刷顺序|
| `SINGLE_QUOTE_UNMATCHED` | `"&rsquo;"` |当打开或关闭序列都不使用时使用什么,如果原样保留则为null。|