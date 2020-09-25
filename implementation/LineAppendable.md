
`LineAppendableImpl`是旨在简化`LineFormattingAppendable`的替代品
创建基于行的格式化输出,包括通过以下方式跟踪源偏移信息
使用`SegmentedBuilder`累积输出到`SegmentedSequence`最终结果。

:warning:正在进行中

## Format

格式化功能包括在每行生成时删除/添加前缀,折叠
将空格跨度扩展为单个空格,删除每行的前导和/或尾随空格。

该实现将输出累积为行列表。对于每一行,可追加项保持不变
前缀结尾和行文本开始的每行的位置。

在附加EOL时,将提交累积的行并将其添加到累积的列表中
线。

正在累积的行以其自己的`SegmentBuilder`累积到
允许以最小的性能损失快速修改或重新生成该行的文本。

## Options

选项定义在输出构造过程中可附加组件的行为:

* `CONVERT_TABS`:在4的列倍数上扩展制表符
* `COLLAPSE_WHITESPACE`:将多个制表符和空格折叠为单个空格
* `SUPPRESS_TRAILING_WHITESPACE`:不输出尾随空格
* `PASS_THROUGH`:无需格式化即可将所有内容传递给可附加项
* `ALLOW_LEADING_WHITESPACE`:允许一行中的前导空格,否则删除
* `ALLOW_LEADING_EOL`:允许偏移量为0的EOL
* `PREFIX_PRE_FORMATTED`:在为行加上前缀时,在预格式化的行之前加上前缀

:information_source:`COLLAPSE_WHITESPACE`会覆盖`CONVERT_TABS`,因为转换后的标签会
转换为空间。

## Formatting Features

`LineAppendable`跟踪累加行的`length`和`column`以方便
通过呈现代码来格式化决策。

`LineAppendable`提供了允许基于以下条件应用条件格式逻辑的方法
给定节点的子元素生成的累积输出,特别是应用前缀
仅在附加EOL后更改,或在附加EOL后注册回调。

在构造线之后,还可以修改线的前缀。但是,那
核心渲染器执行其活动后,无需诉诸于修改线
承诺。

Markdown Navigator插件在累积文本时使用行前缀修改功能
来自JetBrains IDE的解析树,它不适合跟踪父级前缀。

为此,`LineAppendable`允许将每行的前缀结尾修改为父行
前缀将从文本中删除。