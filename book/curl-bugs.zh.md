
## 报告bug

开发团队做了大量的测试.我们有一个完整的测试套件,它每天都在许多平台上频繁运行,以便执行所有代码,并确保一切正常工作.

尽管,有些时候,事情并没有像他们应该的那样运作.然后我们感谢这些问题的提交.

### bug是个问题

任何问题都可以看作是一个bug.手册中一个奇怪的措辞,阻止你理解某些东西是一个bug. 组合多个选项的一个令人惊讶的副作用可能是一个bug,或者它应该被更好地格式? 也许这个选项根本不符合你的期望. 这是个问题,我们应该解决!

### 必须知道问题的解决.

这听起来很简单, 但在我们和其他项目中有个基本的事实.不能仅仅因为它是一个老项目,并且拥有数千用户,就意味着开发团队知道您刚刚遇到的问题.也许用户没有像你一样足够的关注细节,或者也许它从来没有为其他人触发过.

我们依赖于,用户遇到问题并报告它们.我们需要学习存在问题的原因,以便我们能够解决它们.

### 解决问题

软件工程在很大程度上是关于解决问题的.要修复一个问题,开发人员需要理解如何重复它,并且需要告诉调试人员是什么环境触发了问题.

### 一个很好的bug报告

一个好的报告解释了发生了什么和你认为会发生什么. 确切地告诉我们您使用的不同组件的版本,并一步一步地指导我们如何处理这个问题.

在提交bug报告之后,可以预期会有后续问题,或者您尝试各种事情和任务的请求,以便开发人员能够缩小嫌疑人的范围,并确保您的问题被适当地困住.

如果开发人员不能理解bug报告、不能再现bug报告,或在处理bug报告时面临其他问题,而提交的人放弃了bug提交,将面临关闭的风险. 请不要放弃你的报告!

报告curl bug 在[Github上的 issues跟踪器](https://github.com/curl/curl/issues)!

## 测试

全面且正确地测试软件是一项很重要的工作. 测试运行在操作系统和数十个CPU架构上的软件,以及服务器实现,以及它们自己的一组bug和规范的解释,就更多工作了.

curl项目有一个测试套件,它遍历所有现有的测试用例,运行测试并验证结果的正确性, 并且没有发生其他问题,比如内存泄漏或协议层中的可疑问题.

测试套件应该可以在您自己构建好curl之后运行,并且有相当多的志愿者也通过每天几次自动运行测试套件来帮忙, 以确保最新的提交得到运行. 希望这样,在问题出现后,我们可以最快发现最坏的缺陷点.

我们不会测试所有东西,甚至当我们试图测试东西时,总会通过微妙的细节,事实上, 有时,发生多年之后,我们才发现是bug.

由于互联网上不同系统和有趣的用例的性质, 最终当用户运行代码以执行他们自己的用例时,最好的测试却是由一些用户完成的.

测试套件的另一个限制因素是测试设置本身不像curl和libcurl那样具有可移植性, 所以实际上有一些平台, 明明curl运行良好,但是测试套件就根本不能执行.