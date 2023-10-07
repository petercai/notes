# OpenAI官方ChatGPT提示工程 - 最佳实践11篇
今年，在OpenAI开放了ChatGPT不久后，网络上出现了许多关于如何编写Prompt的教程。这些课程和相关的Prompt模版，在一定程度上弥补了这块新鲜领域的空白，使得大家对于如何编写Prompt有了一定的认知。

在这不久之后，OpenAI和吴恩达合作开发了一套针对ChatGPT提示工程的最佳实践课程，课程名为《ChatGPT Prompt Engineering for Developers》。

随后，OpenAI官方又正式的发布了一份"GPT 最佳实践"指南，可以说是期盼已久。官方出品，必是精品。想快速了解，可以先看看这篇文章：[GPT最佳实践 - 提升Prompt效果的六个策略](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484727&idx=1&sn=02a1f71559e9625024359dddfc55561c&scene=21#wechat_redirect)。

**本文汇总了这些官方权威教程的所有文章，内容包括：** 

*   OpenAI官方GPT最佳实践的六个策略，共6篇文章
    
*   OpenAI与吴恩达合作开发的ChatGPT提示工程课程，共5篇文章
    
*   扩展内容
    

*   OpenAI关于AGI通用人工智能及未来技术的规划
    
*   比尔·盖茨关于“人工智能的风险是真实存在的”的思考
    

一、OpenAI官方GPT 最佳实践指南
--------------------

本指南分享了提高GPT的效果的策略和方法，这些方法有时可以结合使用以获得更好的效果。同时鼓励多尝试试验，找到最适合自己的方法。

以下是提高Prompt效果的六大关键策略：

### 1.编写清晰的提示

如果GPT输出的内容过长，可以要求模型进行简短的回复；如果输出过于简单，可以要求模型使用专业的写作水准输出内容。如果你对输出的格式不满意，可以提供自己想要的格式。越是明确表达自己的需求，越有可能得到满意的答案。

[提升GPT Prompt效果最佳实践 - 编写清晰的提示](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484740&idx=1&sn=4e4c53e73790d07b3f1454c9a731e5b2&scene=21#wechat_redirect)

### 2.提供参考文本

GPT模型可以自信地编造虚假答案，尤其是在涉及深奥主题或引用和URL时。就像学生在考试时可以查看笔记来帮助自己更好地回答问题一样，向GPT模型提供参考文本可以帮助其减少编造虚假答案的情况。

[提升GPT Prompt效果最佳实践 - 提供参考文本](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484775&idx=1&sn=7d4126852b8d2ace925ab3cfb844d91c&scene=21#wechat_redirect)

### 3.将复杂的任务拆分为更简单的子任务

就像在软件工程中将复杂系统分解为一组模块化组件一样，在提交给GPT模型的任务中也是如此。复杂任务往往比简单任务出错率更高。此外，复杂任务通常可以重新定义为一系列简单任务的工作流程，其中前置任务的输出作为后续任务的输入。

[提升GPT Prompt效果最佳实践 - 拆解复杂任务](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484835&idx=1&sn=70407187141776a1f1571509b4c575b0&scene=21#wechat_redirect)

### 4.给 GPT 时间思考

如果让你计算17乘以28，你可能不会立即知道答案，但是却可以花时间计算出来。类似地，当GPT试图立即回答问题时，它会犯更多的推理错误，而不是花时间计算出答案。在回答问题之前，要求模型给出一系列的推理过程可以帮助GPT更可靠地推理正确的答案。

[提升GPT Prompt效果最佳实践 - 给 GPT 时间思考](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484997&idx=1&sn=8e6fe2a656a78bae91ea14b1b723572d&scene=21#wechat_redirect)

### 5.使用外部工具

通过使用其他工具的输出来弥补GPT的不足。例如，使用文本检索系统来告诉GPT相关文档的信息，或者使用代码执行引擎来帮助GPT进行数学计算和代码运行。如果有其他工具可以更可靠或更有效地完成某个任务，就应该使用这些工具，以获得最佳效果。

[提升GPT Prompt效果最佳实践 - 使用外部工具](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247485155&idx=1&sn=f302965f114aea364a63efb463422283&scene=21#wechat_redirect)

### 6.系统地测试变更

如果能够进行测量，那么提高效果就会更容易。在某些情况下，对提示的修改会在几个孤立的示例上实现更好的效果，但会导致在一组更具代表性的示例上整体表现变差。因此，为了确保更改对效果能够产生积极的影响，可能有必要定义一个全面的测试套件（也称为“评估(eval)”）。

[提升GPT Prompt效果最佳实践 - 系统的测试变更](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247485183&idx=1&sn=1e06b8aab97d34d905608db60fa0c6e9&scene=21#wechat_redirect)

二、OpenAI与吴恩达合作的ChatGPT提示工程课程
----------------------------

这门课程时长为1个小时，内容简单易懂，还提供了实践的环境。讲师是吴恩达（Andrew Ng，DeepLearning.AI创始人）和伊莎·富尔福德（Isa Fulford，OpenAI的技术人员），含金量非常高。

以下是该课程的5篇文章：

### 1.编写Prompt的两个关键原则

*   原则一：编写清晰、具体的说明
    
*   原则二：给予模型思考的时间
    

[ChatGPT提示工程的两个关键原则](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484623&idx=1&sn=34d87e870df6c6a088d09d132e4591f5&scene=21#wechat_redirect)

### 2.文本总结

这篇主要介绍了如何对内容进行总结，可以让模型来总结/提取重点内容，限制结果的长度。

[ChatGPT提示工程 - 总结](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484692&idx=1&sn=56462d4d746f72705747e09fdfc98fa2&scene=21#wechat_redirect)

### 3.文本推理

可以让模型来识别一段内容的情绪，或者提取指定的内容。

也可以一次性执行多个任务，从而节省多次请求的整体耗时和成本。

[ChatGPT提示工程 - 推理](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484706&idx=1&sn=b751c72c4443314cdb499508bae4cd8c&scene=21#wechat_redirect)

### 4.文本转换

在这篇文章中，介绍了如何使用大型语言模型来进行文本转换工作，如语言翻译、语调调整和格式转换。

[ChatGPT提示工程 - 转换](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484717&idx=1&sn=23130951359582d0fa3b275ea06d0e70&scene=21#wechat_redirect)

### 5.邮件回复与营销文案

在这篇文章中，介绍了如何使用大型语言模型来进行邮件自动回复、生成营销文案。

[ChatGPT提示工程 - 邮件回复、营销文案](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484722&idx=1&sn=40853b26d8b2c46c8da90c650f1f88fc&scene=21#wechat_redirect)

三、规划与风险
-------

[OpenAI关于AGI通用人工智能及未来技术的规划（全文译文）](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484631&idx=1&sn=26a34e7bc79962da50a425396ec93a7a&scene=21#wechat_redirect)

[人工智能的风险是真实存在的 \- 比尔·盖茨](https://mp.weixin.qq.com/s?__biz=MzU1MDg5NTExNg==&mid=2247484821&idx=1&sn=c199741d6f3dac45e51bda2fdddb0b02&scene=21#wechat_redirect)



参考
--

https://platform.openai.com/docs/guides/gpt-best-practices

https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/