
commonmark spec.txt文件是一种出色的格式,可提供叙述性描述,
源和生成的HTML。它是运行解析器一致性测试所依据的格式。

修改了此文件的格式,以添加AST输出,以便测试生成的AST
这对于使用此解析器突出显示语法至关重要。所有测试util类均为
修改以处理原始格式和扩展格式。

为了允许测试解析器选项和扩展,扩展了格式以指定
选项应在运行测试中使用。测试子类必须提供从
测试特定选项集的选项字符串。这允许使用单个测试规范文件来
测试多个解析器配置。

在所有情况下,都支持原始格式,并且原始spec.txt文件用于验证
解析器合规性。

`ComboSpecTestCase`类,结合了`SpecTestCase`和
`FullSpecTestCase`。如果所有测试都可以通过以下方式完成,则每个扩展只需要一个测试类
一个spec.txt文件。

`FullSpecTestCase`重新生成规范文本,并用预期的HTML和AST替换为
解析器生成结果,然后断言这等于原始结果,此外
运行各个测试。这样一来,您就可以将符合性与完整规范进行比较
在JetBrains IDEA中运行测试的情况下,可以轻松地将生成的结果复制到
预期输入,使创建和更新预期结果更加容易.....

如果规格文件没有AST部分,则不会生成预期的AST或
已验证,也不会出现在生成的完整文件结果中。

规范中的markdown标头用于标记部分。每个测试用例都有以下内容
格式:

-最后一个markdown标头用作`SpecExample`实例的`Section`值

-忽略示例开始行中`example`和行尾之间的所有文本

-如果起始示例行以`options(....)`结尾,则将`()`之间的文本用作
  选项集标识符,标识符的前导和尾随空白将被忽略。如果
  结果标识符为空,则将使用默认解析器配置。

  选项中的字符串作为逗号分隔的列表,空格被剪掉。如果超过
  存在一个选项,然后将使用DataHolder内容的组合来运行
  测试用例。

  如果在两个选项中设置了相同的键,则分配给该键的值将出现在列表的后面
  将会被使用。

  :warning:选项`IGNORE`由基类处理,如果存在,将导致
  抛出AssumptionViolatedException,导致忽略测试用例。

  选项`FAIL`由基类处理,如果存在,将导致期望
  该示例无法进行比较。

  选项`NO_FILE_EOL`是默认选项,如果有,它将从示例源中删除尾随EOL
  前面没有空行。`FILE_EOL`可用于禁用此行为。

-`FullSpecTestCase`将添加`Section`,示例编号和原始文件中提供的任何选项
  规格文件。

带有和不带有`options()`子句的样本规范示例

    ## Reference Repository Keep First tests
    
    Test repository KEEP_FIRST behavior, meaning the first reference def is used
    
    ```````````````````````````````` example Reference Repository Keep First tests: 1
    [ref]
    
    [ref]: /url1
    [ref]: /url2
    [ref]: /url3
    .
    <p><a href="/url1">ref</a></p>
    .
    Document[0, 46]
      Paragraph[0, 6]
        LinkRef[0, 5] textOpen:[0, 0] text:[0, 0] textClose:[0, 0] referenceOpen:[0, 1, "["] reference:[1, 4, "ref"] referenceClose:[4, 5, "]"]
          Text[1, 4] chars:[1, 4, "ref"]
      Reference[7, 19] refOpen:[7, 8, "["] ref:[8, 11, "ref"] refClose:[11, 13, "]:"] urlOpen:[0, 0] url:[14, 19, "/url1"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
      Reference[20, 32] refOpen:[20, 21, "["] ref:[21, 24, "ref"] refClose:[24, 26, "]:"] urlOpen:[0, 0] url:[27, 32, "/url2"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
      Reference[33, 45] refOpen:[33, 34, "["] ref:[34, 37, "ref"] refClose:[37, 39, "]:"] urlOpen:[0, 0] url:[40, 45, "/url3"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
    ````````````````````````````````
    
    ## Reference Repository Keep Last tests
    
    Test repository KEEP_LAST behavior, meaning the last reference def is used
    
    ```````````````````````````````` example(Reference Repository Keep Last tests: 1) options(keep-last)
    [ref]
    
    [ref]: /url1
    [ref]: /url2
    [ref]: /url3
    .
    <p><a href="/url3">ref</a></p>
    .
    Document[0, 46]
      Paragraph[0, 6]
        LinkRef[0, 5] textOpen:[0, 0] text:[0, 0] textClose:[0, 0] referenceOpen:[0, 1, "["] reference:[1, 4, "ref"] referenceClose:[4, 5, "]"]
          Text[1, 4] chars:[1, 4, "ref"]
      Reference[7, 19] refOpen:[7, 8, "["] ref:[8, 11, "ref"] refClose:[11, 13, "]:"] urlOpen:[0, 0] url:[14, 19, "/url1"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
      Reference[20, 32] refOpen:[20, 21, "["] ref:[21, 24, "ref"] refClose:[24, 26, "]:"] urlOpen:[0, 0] url:[27, 32, "/url2"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
      Reference[33, 45] refOpen:[33, 34, "["] ref:[34, 37, "ref"] refClose:[37, 39, "]:"] urlOpen:[0, 0] url:[40, 45, "/url3"] urlClose:[0, 0] titleOpen:[0, 0] title:[0, 0] titleClose:[0, 0]
    ````````````````````````````````

第一部分是markdown源,预期的HTML在行上用单个`.`分隔。
预期的AST作为第三部分添加到原始spec.txt中,也由单个`.`分隔
在线上。

为扩展创建测试的最佳方法是从现有副本的副本开始,然后
修改扩展名的markdown源,删除所需的HTML和AST文本,但
离开`.`分隔线。运行`RendererSpecTest`派生测试将创建一个
完整的规格文件,所有部分均填写了实际结果。这些应该被验证然后
复制到规格文件。

```java
import com.vladsch.flexmark.ext.typographic.TypographicExtension;
import com.vladsch.flexmark.parser.Parser;
import com.vladsch.flexmark.util.data.DataHolder;
import com.vladsch.flexmark.util.data.MutableDataSet;

import java.util.Arrays;
import java.util.Collections;

public class ComboCustomSpecTest extends RendererSpecTest {
    final private static String SPEC_RESOURCE = "/ext_typographic_ast_spec.md";
    final public static @NotNull ResourceLocation RESOURCE_LOCATION = ResourceLocation.of(COMBO_CUSTOM_SPEC_TEST.class, SPEC_RESOURCE);

    final private static DataHolder OPTIONS = new MutableDataSet()
            .set(Parser.EXTENSIONS, Arrays.asList(TypographicExtension.create()))
            .toImmutable();

    final private static Map<String, DataHolder> optionsMap = new HashMap<String, DataHolder>();
    static {
        optionsMap.put("option", new MutableDataSet()
                .set(CustomExtension.USE_CUSTOM_OPTION, true)
        );
    }
    public ComboCustomSpecTest() {
        super(example, optionsMap, OPTIONS);
    }

    @Parameterized.Parameters(name = "{0}")
    public static List<Object[]> data() {
        return getTestData(RESOURCE_LOCATION);
    }

    @Override
    public @NotNull
    ResourceLocation getSpecResourceLocation() {
        return RESOURCE_LOCATION;
    }
}
```