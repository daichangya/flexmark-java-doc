为了使其更加合理,我添加了ext-gfm-strikethrough并禁用了自动链接扩展
具有此选项的所有解析器,因为它会导致所有解析器的运行速度显着降低
解析器:

我还添加了一个flexmark-java,其中配置了除自动链接以外的所有扩展名
额外扩展如何影响性能。

最新,2017年1月28日flexmark-java 0.13.1,EAP 2017的intellij-markdown,commonmark-java
0.8.0:

|文件| commonmark-java | flexmark-java | intellij-markdown |挂钩|
|:----------------- | ----------------:| ------------ -:| ------------------:| ----------:|
|README-SLOW| 0.420ms | 0.812ms | 2.027ms | 15.483ms |
|版本| 0.743ms | 1.425ms | 4.057ms | 42.936ms |
| commonMarkSpec | 31.025ms | 44.465ms | 600.654ms | 575.131ms |
| markdown_example | 8.490ms | 10.502ms | 223.593ms | 983.640ms |
|规格| 4.719ms | 6.249ms | 35.883ms | 307.176ms |
|表| 0.229ms | 0.623ms | 0.800ms | 3.642ms |
|表格格式| 1.385ms | 2.881ms | 4.150ms | 23.592ms |
|wrap| 3.804ms | 4.589ms | 16.609ms | 86.383ms |

以上比率:

|文件| commonmark-java | flexmark-java | intellij-markdown |挂钩|
|:----------------- | ----------------:| ------------ -:| ------------------:| ----------:|
|README-SLOW| 1.00 | 1.93 | 4.83 | 36.88 |
|版本| 1.00 | 1.92 | 5.46 | 57.78 |
| commonMarkSpec | 1.00 | 1.43 | 19.36 | 18.54 |
| markdown_example | 1.00 | 1.24 | 26.34 | 115.86 |
|规格| 1.00 | 1.32 | 7.60 | 65.09 |
|表| 1.00 | 2.72 | 3.49 | 15.90 |
|表格格式| 1.00 | 2.08 | 3.00 | 17.03 |
|wrap| 1.00 | 1.21 | 4.37 | 22.71 |
| ----------- | --------- | --------- | --------- | ------- -|
|整体| 1.00 | 1.41 | 17.47 | 40.11 |

|文件| commonmark-java | flexmark-java | intellij-markdown |挂钩|
|:----------------- | ----------------:| ------------ -:| ------------------:| ----------:|
|README-SLOW| 0.52 | 1.00 | 2.50 | 19.07 |
|版本| 0.52 | 1.00 | 2.85 | 30.12 |
| commonMarkSpec | 0.70 | 1.00 | 13.51 | 12.93 |
| markdown_example | 0.81 | 1.00 | 21.29 | 93.66 |
|规格| 0.76 | 1.00 | 5.74 | 49.15 |
|表| 0.37 | 1.00 | 1.28 | 5.85 |
|表格式| 0.48 | 1.00 | 1.44 | 8.19 |
|wrap| 0.83 | 1.00 | 3.62 | 18.83 |
| ----------- | --------- | --------- | --------- | ------- -|
|整体| 0.71 | 1.00 | 12.41 | 28.48 |

* [VERSION.md]是我用于Markdown Navigator的版本日志文件
* [commonMarkSpec.md]是在[intellij-markdown]测试套件中使用的33k行文件,以提高性能
  评价。
* [commonmark-java]项目中的[spec.txt] commonmark规范markdown文件
* [hang-pegdown.md]是一个包含17个字符的单行文件`[[[[[[[[[[[[[[[[[`
  这会导致pegdown进入超指数解析时间。
* [hang-pegdown2.md]一个包含18个字符的一行的文件`[[[[[[[[[[[[[[[[[[`
  导致pegdown进入超指数解析时间。
* [wrap.md]是我用来测试打字性能的文件,只是发现它
  pegdown花费0.1秒来解析时,与键入代码的包装无关
  文件。在插件中,解析可能会发生多次:语法荧光笔传递,psi
  树建筑通行证,外部注释器。
* markdown_example.md包含10,000+行的文件,其中包含500kB +的文本。
* [table.md]是一个具有相当大表的文件,用于测试表解析性能

[commonmark-java]: https://github.com/atlassian/commonmark-java
[commonMarkSpec.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/commonMarkSpec.md
[hang-pegdown.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/hang-pegdown.md
[hang-pegdown2.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/hang-pegdown2.md
[intellij-markdown]: https://github.com/valich/intellij-markdown
[spec.txt]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/spec.md
[table.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/table.md
[VERSION.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/VERSION.md
[wrap.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/wrap.md