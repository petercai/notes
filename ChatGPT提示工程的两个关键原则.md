# ChatGPT提示工程的两个关键原则 
不久前，吴恩达和OpenAI合作开发了一套针对ChatGPT提示工程的最佳实践课程，课程名为《ChatGPT Prompt Engineering for Developers》。

这门课程时长为1个小时，内容简单易懂，还提供了实践的环境。关键是，课程免费，而且讲师是吴恩达（Andrew Ng，DeepLearning.AI创始人）和伊莎·富尔福德（Isa Fulford，OpenAI的技术人员），含金量非常高。

 

本文将要介绍的是编写Prompt的两个关键原则，非常非常重要，一定要掌握。

毫不夸张的说，只要你掌握并灵活的去应用这两个原则，你就已经超过了大部分人编写Prompt的水平。

编写Prompt的两个关键原则
---------------

原则一：编写清晰、具体的说明

原则二：给予模型思考的时间

（什么，太简单了？没关系，下面会一步步分解并举例说明。示例内容有点长，建议打开电脑，耐心学习并花点时间去实践，值得！）

原则一：编写清晰、具体的说明
--------------

你应该通过提供尽可能清晰和具体的说明（instructions）来表达您希望模型做什么。这将引导模型朝着你期望的方向输出内容，并降低收到无关或错误回复的可能性。 请不要将编写清晰的提示（prompt）与编写简短的提示（prompt）混淆，因为在很多情况下，更长的提示（prompt）实际上会为模型提供更多的清晰度和上下文，从而可能获得更详细和相关的输出。

### 策略1：使用分隔符清晰地标识出输入的各个部分

分隔符可以是任何明确的标点符号，用来将特定文本片段与提示的其余部分分开。这些标点符号可以是三个反引号、引号、XML标签、章节标题等，只要能够让模型明确地知道这是一个独立的部分即可。

```
`三个双引号："""``三个反引号: ``` ``三个破折号: ---``尖括号: <>``XML标签: <tag></tag>`
```

输入Prompt提示，让ChatGPT将用三个反引号分隔的文本总结成一个句子:

```
`Summarize the text delimited by triple backticks into a single sentence.`` ```You should express what you want a model to do by providing instructions that are as clear and specific as you can possibly make them. This will guide the model towards the desired output, and reduce the chances of receiving irrelevant or incorrect responses. Don't confuse writing a clear prompt with writing a short prompt. In many cases, longer prompts provide more clarity and context for the model, which can lead to more detailed and relevant outputs.``` `
```

输出结果：

```
To guide a model towards the desired output and reduce irrelevant or incorrect responses, it is important to provide clear and specific instructions, which may require longer prompts for more clarity and context.
```

### 策略2：提示模型以结构化格式输出结果

通过明确提示模型，以JSON、HTML等结构化格式输出结果。

输入Prompt提示，让ChatGPT生成一个由三个虚构的书名及其作者和类型组成的列表，并以JSON格式和给定字段输出结果:

```
`Generate a list of three made-up book titles along with their authors and genres.``Provide them in JSON format with the following keys:``book_id, title, author, genre.`
```

输出结果：

```
`[` `{` `"book_id": 1,` `"title": "Whispers of the Forgotten Realm",` `"author": "Aria Ravenswood",` `"genre": "Fantasy"` `},` `{` `"book_id": 2,` `"title": "The Quantum Paradox",` `"author": "Dr. Maxwell Sterling",` `"genre": "Science Fiction"` `},` `{` `"book_id": 3,` `"title": "A Symphony of Shadows",` `"author": "Lilith Blackwood",` `"genre": "Gothic Romance"` `}``]`
```

### 策略3：让模型检查是否满足条件

输入Prompt提示，让ChatGPT分析文本内容，如果文本包含一个指令序列，则以指定格式重写这些指令。否则，输出“没有提供步骤”。

```
`You will be provided with text delimited by triple quotes.` `If it contains a sequence of instructions, re-write those instructions in the following format:``Step 1 - ...``Step 2 - ...``...``Step N - ...``If the text does not contain a sequence of instructions, then simply write "No steps provided."``"""``Making a cup of tea is easy! First, you need to get some water boiling. While that's happening, grab a cup and put a tea bag in it. Once the water is hot enough, just pour it over the tea bag. Let it sit for a bit so the tea can steep. After a few minutes, take out the tea bag. If you like, you can add some sugar or milk to taste. And that's it! You've got yourself a delicious cup of tea to enjoy.``"""`
```

输出结果

```
`Step 1 - Get some water boiling.``Step 2 - Grab a cup and put a tea bag in it.``Step 3 - Once the water is hot enough, pour it over the tea bag.``Step 4 - Let it sit for a bit so the tea can steep.``Step 5 - After a few minutes, take out the tea bag.``Step 6 - Add some sugar or milk to taste.``Step 7 - Enjoy your delicious cup of tea.`
```

输入Prompt提示，分析另一段文本内容：

```
`You will be provided with text delimited by triple quotes.` `If it contains a sequence of instructions, re-write those instructions in the following format:``Step 1 - ...``Step 2 - ...``...``Step N - ...``If the text does not contain a sequence of instructions, then simply write "No steps provided."``"""``The sun is shining brightly today, and the birds are singing. It's a beautiful day to go for a walk in the park. The flowers are blooming, and the trees are swaying gently in the breeze. People are out and about, enjoying the lovely weather. Some are having picnics, while others are playing games or simply relaxing on the grass. It's a perfect day to spend time outdoors and appreciate the beauty of nature.``"""`
```

输出结果

```
No steps provided.
```

### 策略4：少样本提示

少样本提示（Few-shot prompting），是指提供1个或多个示例，让ChatGPT学习以达到我们期望的效果。

输入Prompt提示，让ChatGPT以与示例一致的风格来回答。示例内容是，孩子问什么是耐心，然后祖父母用比喻的方式来回答：

```
`Your task is to answer in a consistent style.``<child>: Teach me about patience.``<grandparent>: The river that carves the deepest valley flows from a modest spring; the grandest symphony originates from a single note; the most intricate tapestry begins with a solitary thread.``<child>: Teach me about resilience.`
```

输出结果：

```
<grandparent>: Resilience is like a tree that bends with the wind but does not break. It is the ability to recover from difficult situations and keep moving forward. Just like a tree that endures storms and still stands tall, resilience comes from within and helps us face life's challenges with strength and perseverance.
```

回答效果非常好。

原则二：给予模型思考的时间
-------------

### 策略1：明确说明完成任务所需的步骤

输入Prompt提示，让ChatGPT按照指定的步骤来回答。先总结文本，然后翻译成法语，最后总结指定内容并以JSON格式输出：

```
`Perform the following actions:` `1 - Summarize the following text delimited by triple backticks with 1 sentence.``2 - Translate the summary into French.``3 - List each name in the French summary.``4 - Output a json object that contains the following keys: french_summary, num_names.``Separate your answers with line breaks.``Text:`` ```In a charming village, siblings Jack and Jill set out on a quest to fetch water from a hilltop well. As they climbed, singing joyfully, misfortune struck—Jack tripped on a stone and tumbled down the hill, with Jill following suit. Though slightly battered, the pair returned home to comforting embraces. Despite the mishap, their adventurous spirits remained undimmed, and they continued exploring with delight.``` `
```

输出结果：

```
`Two siblings, Jack and Jill, go on an adventure to fetch water from a well on a hill, but misfortune strikes and they both tumble down the hill, causing them to return home slightly battered but with their adventurous spirits undimmed.``Deux frères et sœurs, Jack et Jill, vont chercher de l'eau d'un puits sur une colline, mais ils chutent tous les deux et retournent chez eux légèrement meurtris mais avec leurs esprits d'aventure intacts.` `Jack, Jill.` `{` `"french_summary": "Deux frères et sœurs, Jack et Jill, vont chercher de l'eau d'un puits sur une colline, mais ils chutent tous les deux et retournent chez eux légèrement meurtris mais avec leurs esprits d'aventure intacts.",` `"num_names": 2``}`
```

要求以另一种指定的格式输出：

```
`Your task is to perform the following actions:` `1 - Summarize the following text delimited by <> with 1 sentence.``2 - Translate the summary into French.``3 - List each name in the French summary.``4 - Output a json object that contains the following keys: french_summary, num_names.``Use the following format:``Text: <text to summarize>``Summary: <summary>``Translation: <summary translation>``Names: <list of names in Italian summary>``Output JSON: <json with summary and num_names>``Text: <In a charming village, siblings Jack and Jill set out on a quest to fetch water from a hilltop well. As they climbed, singing joyfully, misfortune struck—Jack tripped on a stone and tumbled down the hill, with Jill following suit. Though slightly battered, the pair returned home to comforting embraces. Despite the mishap, their adventurous spirits remained undimmed, and they continued exploring with delight.>`
```

输出结果：

```
`Summary: Jack and Jill go on a quest to fetch water, but while climbing a hill, Jack falls and both siblings tumble down, returning home slightly injured but with undimmed adventurous spirits.``Translation: Jack et Jill partent chercher de l'eau, mais Jack tombe et ils descendent tous les deux la colline, rentrant chez eux légèrement blessés mais avec des esprits aventureux indomptables.``Names: Jack, Jill``Output JSON: {"french_summary": "Jack et Jill partent chercher de l'eau, mais Jack tombe et ils descendent tous les deux la colline, rentrant chez eux légèrement blessés mais avec des esprits aventureux indomptables.", "num_names": 2}`
```

### 策略2：提示模型不要匆忙得出结论，而是要找出自己的解决方案

输入Prompt提示，让ChatGPT判断学生解答财务问题的答案是否正确：

```
`Determine if the student's solution is correct or not.``Question:``I'm building a solar power installation and I need help working out the financials.` `- Land costs $100 / square foot``- I can buy solar panels for $250 / square foot``- I negotiated a contract for maintenance that will cost me a flat $100k per year, and an additional $10 / square foot``What is the total cost for the first year of operations as a function of the number of square feet.``Student's Solution:``Let x be the size of the installation in square feet.``Costs:``1. Land cost: 100x``2. Solar panel cost: 250x``3. Maintenance cost: 100,000 + 100x``Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000`
```

输出结果：

```
The student's solution is correct.
```

实际上，以上答案是错误的。

我们可以通过提示模型首先找出自己的解决方案，然后将ChatGPT的解决方案与学生的解决方案进行比较，并评估学生的解决方案是否正确来解决这个问题：

```
`Your task is to determine if the student's solution is correct or not.``To solve the problem do the following:``- First, work out your own solution to the problem.` `- Then compare your solution to the student's solution and evaluate if the student's solution is correct or not.` `Don't decide if the student's solution is correct until you have done the problem yourself.``Use the following format:``Question:`` ```question here``` ``Student's solution:`` ```student's solution here``` ``Actual solution:`` ```steps to work out the solution and your solution here``` ``Is the student's solution the same as actual solution just calculated:`` ```yes or no``` ``Student grade:`` ```correct or incorrect``` ``Question:`` ```I'm building a solar power installation and I need help working out the financials.` `- Land costs $100 / square foot``- I can buy solar panels for $250 / square foot``- I negotiated a contract for maintenance that will cost me a flat $100k per year, and an additional $10 / square foot``What is the total cost for the first year of operations as a function of the number of square feet.``` ` `Student's solution:`` ```Let x be the size of the installation in square feet.``Costs:``1. Land cost: 100x``2. Solar panel cost: 250x``3. Maintenance cost: 100,000 + 100x``Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000``` ``Actual solution:`
```

输出结果：

```
`Let x be the size of the installation in square feet.``Costs:``1. Land cost: 100x``2. Solar panel cost: 250x``3. Maintenance cost: 100,000 + 10x``Total cost: 100x + 250x + 100,000 + 10x = 360x + 100,000``Is the student's solution the same as actual solution just calculated:``No``Student grade:``Incorrect`
```

这次得出了正确的结论，学生的答案是错误的。


参考
--

https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/

https://learn.deeplearning.ai/chatgpt-prompt-eng/lesson/2/guidelines