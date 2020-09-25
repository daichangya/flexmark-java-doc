用于表处理的flexmark-java扩展

### 概述

像[GitHub Flavored Markdown][gfm-tables]中一样使用管道启用表。随着增加的选择
处理列跨度语法,具有多个标题行的能力,不同的列号
行之间等

将管道`|`分隔的文本转换为表格元素:

```markdown

|    Heading Centered    | Heading Left Aligned   |  Heading Centered  |   Heading Right Aligned |
|------------------------|:-----------------------|:------------------:|------------------------:|
| Cell text left aligned | Cell text left aligned | Cell text centered | Cell text right aligned |
| cell 21                | cell 22                |      cell 22       |                 cell 22 |

```

### 句法

如果未选择`GitHub table rendering`选项,则表可以具有多个标题行,
表主体中的列数不需要匹配标题行或分隔符行列,并且
可以通过在单元格末尾添加连续的`|`来指定列范围。每条管道
表示列跨度计数`||`跨两列,`|||`跨三列,依此类推。

启用列跨度语法时,空表单元格必须至少具有一个空格。除此以外
它们将被解释为上一个单元格的列跨度。

请注意,GitHub不支持此扩展语法。

:warning:表元素前面必须有一个空白行,此扩展名才能处理该元素。

### 解析详细信息

在工件`flexmark-ext-tables`中使用类`TablesExtension`。

提供以下选项:

在`TablesExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------------------------ |:---- ---------------------- |:-------------------------- -------------------------------------------------- -------------------------------------------------- ----------------------- |
| `MIN_HEADER_ROWS` | `0` |标题行的最小数量|
| `MAX_HEADER_ROWS` | `Integer.MAX_INTEGER` |标题行的最大数量|
| `HEADER_SEPARATOR_COLUMN_MATCH` | `false` |为true时,将仅识别其标题行包含与分隔符行相同列数的表|
| `APPEND_MISSING_COLUMNS` | `false` |表主体列应至少为数字列还是标题列|
| `DISCARD_EXTRA_COLUMNS` | `false` |是否放弃超出标题|中定义的正文列
| `COLUMN_SPANS` | `true` |将列末尾的连续管道视为定义扩展列。|
| `WITH_CAPTION` | `true` |为true时,将解析表格标题行,表格后一行的格式为`[` ... `]` |
| `CLASS_NAME` |``表|上要使用的类名
| `FORMAT_TABLE_LEAD_TRAIL_PIPES` | `true` |格式设置选项启用后在表行上添加打开和关闭管道,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_SPACE_AROUND_PIPES` | `true` |格式选项启用后会在单元格文本周围添加空格,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_ADJUST_COLUMN_WIDTH` | `true` |扩展列以跨行对齐列管道,格式设置请参阅:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_APPLY_COLUMN_ALIGNMENT` | `true` |当`true`和`FORMAT_TABLE_ADJUST_COLUMN_WIDTH`为`true`时,将列对齐方式应用于单元格文本,格式设置请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_FILL_MISSING_COLUMNS` | `false` |为true时,添加列以使所有行具有相同的列数,格式设置请参阅:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_CAPTION` | `false` |当删除真实的表标题时,启用格式化选项时将删除表标题,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_LEFT_ALIGN_MARKER` | `DiscretionaryText.AS_IS` |如何处理输出中的左对齐`:`,格式设置请参阅:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_TABLE_MIN_SEPARATOR_COLUMN_WIDTH` | `3` |格式化输出中的最小分隔符列|
| `FORMAT_TABLE_MIN_SEPARATOR_DASHES` | `1` |格式化输出中的最小分隔符破折号
| `FORMAT_TABLE_TRIM_CELL_WHITESPACE` | `true` |格式化输出中的修剪表单元格空格|
| `FORMAT_TABLE_CAPTION_SPACES` | `DiscretionaryText.AS_IS` |如何在格式化输出|中处理`[`之后和`]`之前的空格
| `FORMAT_TABLE_INDENT_PREFIX` | `""` |在格式化输出|中向表添加任意前缀
| `FORMAT_TABLE_MANIPULATOR` | `TableManipulator.NULL` |在将表附加到格式化输出之前调用的接口。允许表格操作|
| `FORMAT_CHAR_WIDTH_PROVIDER` | `CharWidthProvider.NULL` |接口用于提供实际字符宽度,因此表格式可以将这些字符考虑在内
| `TRIM_CELL_WHITESPACE` | `true` | false将在单元格文本|中留下周围的空格
| `MIN_SEPARATOR_DASHES` | `3` |表分隔符列中的`-`或`:`字符的最小数目

* `DiscretionaryText`
  * `AS_IS`:不变
  * `ADD`:添加到没有对齐的列
  * `REMOVE`:从左对齐的列中删除

与带有默认选项的`Formatter`渲染器一起使用时,将转换为:

```markdown
day|time|spent
:---|:---:|--:
nov. 2. tue|10:00|4h 40m
nov. 3. thu|11:00|4h
nov. 7. mon|10:20|4h 20m
total:|| **13h**
```

至

```markdown
| day         | time  |   spent |
|:------------|:-----:|--------:|
| nov. 2. tue | 10:00 |  4h 40m |
| nov. 3. thu | 11:00 |      4h |
| nov. 7. mon | 10:20 |  4h 20m |
| total:             || **13h** |

```

[gfm-tables]: https://help.github.com/articles/organizing-information-with-tables/