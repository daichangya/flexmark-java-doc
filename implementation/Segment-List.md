
是用于构造字符序列的零件列表。每个部分代表一个范围
来自原始序列或`SegmentList`实现的原始序列之外的文本
具有属性:
* `range`:原始序列中所得序列的范围。`Range.NULL`列出
  不包含此类信息。:information_source:空范围的开始=
  `Integer.MAX_VALUE`和end = `Integer.MIN_VALUE`
* `startOffset`:`range`开始,结果序列在原始序列中的起始偏移量,
* `endOffset`:`range` end,结果序列在原始序列中的结束偏移量,
* `lastSegment`:列表中包含的最后一段,对于空列表为null。
* `segments`:段列表,描述结果序列及其在
  原始序列。
* `length`:结果序列的长度。
* `textLength`:结果序列中所有文本的长度,超出原始长度
  序列。

:warning:此文档未更新最新更改。不再有`STRING`类型,
`TEXT`不再具有任何基本偏移信息。之前/之后的`BASE`或`ANCHOR`段
`TEXT`提供基本序列上下文。如果`TEXT`段位于列表的首位,则
`SegmentBuilder` `startOffset`是其开始的基础上下文,如果它是最后一个,则`endOffset`
字符串生成器的基本上下文是`endOffset`

每个段部分都由它代表的原始序列中的范围和文本组成
不符合原始顺序。它具有`span`和`length`属性,其中`length`是长度
文字有助于构建序列。`span`是字符数
在原始顺序中,此片段表示或替换。`span`或`length`都可以
为0。如果两个都为零,则这是`NULL`段,对结果序列没有影响
并且不应属于细分受众群列表。

段列表可以包含以下段类型:
* `NULL`:空段,不应存在于规范化列表中。建立清单时被忽略
* `ANCHOR`:范围= 0,文本为空,`span`为范围,`length`和`span` = 0,
  position是此锚点在原始序列中的偏移量。该细分不存在于
  标准化列表。仅用于在构建时向`STRING`提供范围信息并列出
  一个列表。
* `STRING`:范围为null,文本不为空,`length`为文本长度,`span`为0
* `TEXT`:范围> = 0,文本不为空,`length`为文本长度,`span`为范围
  范围。如果Start是列表中的第一个元素,则它是列表的`startOffset`。否则是
  等于上一个`BASE`结尾。结束偏移等于列表中的`endOffset`时的最后一段
  一个列表。否则,它等于以下`BASE`的开始。
* `BASE`:范围> 0,文本为空,`span`为范围,`length`为`span`

只能将`ANCHOR`,`BASE`或`STRING`类型附加到细分列表中,并始终导致
规范化的细分列表。具体的`ANCHOR`和`BASE`是前者的`Range`实例
具有`Range.getSpan()` ==0。具体的`STRING`是`String`。

`ANCHOR`只能设置没有偏移信息的列表的`startOffset`或扩展其
如果小于`ANCHOR` end,则为end偏移量。它仅在构建列表时用于标记
段列表的开始/结束范围(按原始顺序)。它不添加任何内容,在以下情况下将被忽略
它不会影响结果序列的范围。

#### 段列表不变式

* `lastSegment`是列表中的最后一个段,对于空列表则为null
* `endOffset`,`range`末尾相等,如果`lastSegment`不为null,则其末尾等于
  `endOffset`
* `startOffset`,`range`开始等于,列表不为空,则第一个句段开始为
  等于`startOffset`
* `length`等于列表中`length`段的总和
* `textLength`等于列表中非`BASE`段的`length`之和。
* `STRING`段只能是具有空范围的列表中的唯一元素。
*如果第一个元素或上一个`BASE`的末尾,则`TEXT`的开始始终为`startOffset`。结束
  如果`TEXT`是列表中的最后一个片段,则offset将为`endOffset`,否则它将是下一个片段的开始
  `BASE`

### 附加规则

附加操作的结果仅由列表`range`和`lastSegment`确定
属性,即添加了细分。

仅`BASE`的附加可以是:`disjoint`,`adjacent`或`overlapped`,并且仅当列表
具有原始序列偏移信息。

* `disjoint`:附加的段没有范围信息,列表没有偏移信息,或者
  段开始> `endOffset`。
* `adjacent`:附加的段开始== `endOffset`
* `overlapped`:附加的段开始<`endOffset`

重叠需要解决,并且取决于序列的类型,并且具有以下结果
根据重叠分辨率的结果:
* null:抛出附加范围,列表保持不变,例如,如果最后一段包含
  附加范围。
* `BASE`:最后一个段的末端设置为附加范围的末端。重叠从附加范围中删除
  然后将相邻范围合并到最后一个细分中。
* `TEXT`:将重叠部分转换为原始序列的文本,起始位置设置为最后一段结束。的
  重叠包含附加范围,并转换为原始序列字符。
* `TEXT`,`BASE`:转换为原始序列字符和非重叠范围的文本。
  附加范围和最后一段的交集转换为原始序列
  字符和范围。以开始==最后一段的结尾。`TEXT`段的范围是
  最后一段的结尾和转换后的非重叠范围的开始。

#### 不相交或相邻的附加规则

|最后|个附加值|结果(=替换最后一个,+附加值)|说明|
| ---------- | ---------- | --------------------------- ----------- | -------------------------------------- ------------------------------------------------ |
| | `BASE` | + `BASE` | |
| | `STRING` | + `STRING` | |
| | `ANCHOR` | | `startOffset`/`endOffset`设置为锚点|
| `STRING` | `ANCHOR` | = `TEXT` | `startOffset`/`endOffset`和`TEXT`开始/结束=锚点位置|
| `STRING` | `STRING` | = `STRING` | `STRING`附加文本|
| `STRING` | `BASE` | = `TEXT` + `BASE` | `startOffset`和`TEXT`开始/结束设置为`BASE`开始,# #123 ##设置为范围结束|
| `TEXT` | `STRING` | = `TEXT` | `STRING`附加到`TEXT`文本|
| `TEXT` | `BASE` | = `TEXT` + `BASE` | `TEXT`结束,`endOffset` =范围开始|
| `BASE` | `BASE` | = `BASE` |附加范围开始== `endOffset`,`BASE` end =附加范围结束|
| `BASE` | `STRING` | + `TEXT` | `TEXT`开始=范围结束,文本= `STRING`文本|
| `BASE` | `BASE` | + `BASE` |附加范围开始>> `endOffset` |
| `BASE` | `BASE` |请参见重叠追加|附加范围开始<`endOffset`,需要重叠分辨率|