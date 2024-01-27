# 和 AI 一起做动画 | 将人工智能融入动画工作流的案例和实践经验
嗨！我是海辛，最近在尝试用 AI 将写实视频风格化，上面做的这个效果在 DY 拿到了 17 万喜欢和 1.8 万评。很多朋友希望我能分享是怎么做的，刚好最近一直在探索将 AI 用于动画制作的生产，积累了许多 demo，决定趁机做一个梳理总结。  

目前用 AI 制作动画有许多不同的路径，大致包括：**A. 根据参考视频进行风格迁移、B. 文本生成动画、C. 根据静态图生成动画等。** 不同的方式也存在不同的技术路径。（在每一大类的末尾，我都会附上我推荐的教程，这些教程都是免费的，对我在探索的过程中起了很大的帮助。）

**一、根据参考视频进行风格迁移**

▶ 代表工具：Runway Gen-1, Stable Diffusion + EbSynth,  Rerender, Warpfusion

比起从文本生成视频，我更关注根据参考视频生成 AI 动画，因为视觉艺术业内的工作流程一直是从草稿【画面】细化到成品【画面】；而从文本生成画面的工具，由于可控性太低，会导致生产效率也很低。

放到 AI 时代的视角来看，**从电影分镜到成片，本质是一种 style transfer**.  而这就是 Runway Gen-1 在做的事。

**1\. Runway Gen-1**

Runway Gen-1 \[1\] 就是建立在风格迁移的基础之上的工具，提供原视频 + 参考图，就可以生成一个针对该参考图的新的视频。Karen X.Cheng 在她的 twitter 账户上分享了很多自己的测试 \[2\] 

效果还不错，但似乎目前的 Gen-1 更多只能停留在玩具的水准，**我在实际使用过程中，发现 Gen-1 没有办法完全按照我提供的参考图进行风格的迁移，导致基本不能用到真实的项目中。** 

比如我提供了铃芽之旅中的镜头，以及我希望转绘成的真人风格，最后 runway 生成的画面不是很符合我的预期…… ↓

![](_assets/640-19.png)

**2. Stable Diffusion + EbSynth**

如果想让 AI 稳定地按照我希望的风格进行迁移，我不能依赖于目前的 Gen-1. 我也不能用 Stable Diffusion 去跑画面的每一帧，如果重绘幅度高（即参考视频对 AI 的影响低，AI 发挥空间大），生成的画面抖动会非常厉害；如果重绘幅度低（即参考视频对 AI 的影响高，AI 发挥空间小），生成的画面虽然不抖了，但和原视频区别也不大，这种 AI 动画也没什么太大意义。

于是我想，**提高稳定性的思路不是让 AI 算每一帧，而是让 AI 算关键帧。关键帧以外的部分通过其他工具来根据 AI 算的关键帧来进行迁移。** 这样才能既保证 AI 的想象力，又能节省算力，又能维持画面稳定。

在这个过程中我想起了老牌开源 AI 风格迁移工具 EbSynth \[3\] 于是我打算用 Stable Diffusion 来绘制关键帧，再用 EbSynth 根据原视频+关键帧将其余帧补齐。

确定合适的关键帧是 key，通常我的经验是 **当画面引入全新的信息时就要新建一个关键帧**（eg：侧面镜头时抬低头不算引入新的信息，但侧面镜头中的人物忽然扭脖子转到正面，就是引入了全新的信息）  

在上面这个测试里，我每个镜头都只用了 1-2 个关键帧，因为是侧面镜头的原因，我需要设置的关键帧很少，基本只在角色睁眼和闭眼的时候。

![](_assets/640-17.png)

ps：这个测试也得到了 AK 和很多我喜欢的开发者在 twitter 上的转发。❤

这个工作流也是目前我最喜欢的方式。在这套玩法上面，还可以**结合传统的影视抠像工具进行叠加，从而指定 AI 仅对画面部分内容进行风格迁移。** 

Your browser does not support video tags

上面这个视频中，我先将人物进行动态抠像，然后只对人物进行 AI 渲染。目前动态抠像生成遮罩的工具中，我最常用的是 Runway 提供的遮罩生成功能，免费且效果好。

![](_assets/640-18.png)

这套流程在 Stable Diffusion 的 Reddit 版面也得到了很多关注。

**3. Rerender**

Stable Diffusion + EbSynth 这套工作流中，**最重要的是保持不同关键帧中的信息要连续和一致**，目前基于 diffusion 模型算不同的关键帧，算完后还是需要人手动来修。所以目前的技术方案还是会需要人来“肝”，其实“肝”的工作量就是调整关键帧之间信息的连续性。

最近看到一个不错的解决方案：Rerender A Video \[4\] - 这个解决方案也认为并不应该让 diffusion 模型来算每一帧，而是只算关键帧。**Rerender A Video 添加价值的地方是：TA 会自动判断需要哪些关键帧，并且保证每个关键帧之间是相似的。** 

Rerender 这个工具我还没有机会尝试，但是我很认可他的解决思路。目前这个工具在 Hugging Face 上可以测试 demo\[5\]，完整代码尚未开源，值得持续关注。

4\. AI + AE

最近我发现 Reddit Stable Diffusion 版面里 **AI 动画特效做得好的人，大多数都是影视/游戏/特效的背景。** 其实这是合理的，因为他们知道如何将 AI 生成的画面结合到其他管线的工具中，从而提升画面效果。

比如下面这个视频 \[6\] 是一名达拉斯的影视从业者根据 Marc Rebillet 的著名打碟视频 Your New Morning Alarm（你的早安闹铃）\[7\] 制作的 AI 渲染版，他结合使用了 Warp Fusion （一种基于 SD 的动画脚本，由 Sxela 开发 \[8\] ）+ AE 转场效果 + PR 剪辑。

忽然意识到可以用更多影视行业的特效和剪辑工具后，我更加受到启发，于是根据 DY @子康 老师的滑板视频，制作了一个 AI 动画版本。  

在这组镜头中，我测试了不同镜头里对 AI 抖动幅度不同的控制力会带来怎样的效果差异，我发现运动幅度越大的镜头，AI 闪烁反而会加强镜头的动态效果。  

当我希望**画面更稳定**的时候，比如镜2，我只会用 Stable Diffusion 渲染一个关键帧，然后使用 EbSynth 来补完剩余的画面；当我希望**画面动态感更强**的时候，比如镜1和镜3，我会用 SD 渲染多个关键帧，从而带来帧与帧之间更多的差异。

镜头4 换脸的部分，我通过使用 Photoshop Generative Fill 的功能，对画面进行局部修改（inpainting）后，使用 EbSynth 来生成多个镜头片段，再在 PR 里进行剪辑，从而完成比较连续丝滑的换脸。

镜1、2、4的画面过度是在 AE 里完成的，这个转场效果需要使用到镜头本身的深度视频，深度视频和遮罩我目前都使用 Runway 来帮我生成。

![](_assets/640-22.png)

这个作品在子康的 DY 上线后，24 小时内就拿到了 10 万点赞和 1 万以上的评论。

**推荐教程**

1.  Runway Academy：**Runway 的官方教程链接**，里面整合了各种 Gen-1, Gen-2 的使用方式：https://academy.runwayml.com/gen1  
    
2.  Stable Diffusion 动画推荐教程：都在 Reddit /stablediffusion 子论坛，**这个子论坛的特点是发作品需要带上自己的工作流。** 这个不成文的规定使得这个论坛充满了很有价值的一手经验：https://www.reddit.com/r/StableDiffusion/
    
3.  EbSynth 推荐教程：EbSynth 功能很简单，**跟着官方的教程一下就能学会**（https://www.youtube.com/watch?v=0RLtHuu5jV4&t=65s）如果想了解 EbSynth 的进阶用法，我推荐 **Academy of Edits** 的教程：https://www.youtube.com/watch?v=6FN7fKlJcPE&t=193s
    
4.  Warpfusion 推荐教程：讲得最清楚的是 **Prompt Muse**（https://www.youtube.com/watch?v=0AT8esyY0Fw&t=797s）你也可以在**开发者 Sxela** 的 DC 频道里直接问他更多的问题：https://discord.gg/vXBTtzmeJd  
    

**二、AI 生长类动画**

▶  代表工具：Disco Diffusion, Stable Diffusion Deforum

此类工具是用传统 AI 生成图像工具，进行画面的批量生成，通过指定不同关键帧上的画面内容，和关键帧上的镜头运动，从而使得 AI 能生成连续的动画。

![](_assets/640-16.png)

如上图 0= \["aaa"\], 100= \["bbb"\], 就是在第 0 帧是 aaa 的画面，在第 100 帧是 bbb 的画面。这种方式制作出来的动画是一镜到底，由于画面在生成的过程中不断解离和重新生成，成为了一种很有趣的艺术表达形式。因为实验性很强，所以很适合拿来做实验动画，或者 MV.  

我在去年有尝试用这种方式制作短片的故事动画，我的思路是，先将故事念一遍，根据录音计算每一句话需要多少秒，对应会需要生成多少帧，然后根据对应的帧来描述画面。（非常早期的生成案例了，请各位请拍。）

后来我发现，**控制 AI 生成的参数和影视镜头语言其实呈现一定的映射关系**， 比如 Translation\_X 对应的是画面左移或右移；Translation\_Y 对应的是画面上移或下移；Translation_Z 对应的则是镜头的推拉。Deforum 的官方操作指南里有很详细的各个参数和画面镜头运动的对应关系 \[9\].

这种类型做到登峰造极程度的是 Youtube 频道主 DoodleChaos 给 Resonate 的歌曲《Canvas》 做的动画音乐视频 \[10\].

![](_assets/640-20.png)

整个 AI MV 的制作逻辑是以 **音乐的节奏点作为 AI 镜头变化的关键帧**，以 **歌词的内容作为各段的文本描述**。在 AI 生成完画面后，DoodleChaos 使用了 FlowFrames 来给 AI 制作的视频进行升帧，使得 FPS 从 15 增加到了 30. 详细的项目介绍可以在 DoodleChaos 的 Patreon 里看到 \[11\]

按照 DoodleChaos 的方式，我重制了一版《Canvas》的 MV 视频。

**推荐教程**

1\. Deforum 的官方 Notebook 可以让你了解到各个参数是做什么的：https://deforum.github.io/animation.html

2\. 目前还有一些工具可以帮助你直接提取歌曲中的关键帧，比自己翻译可能来得会快很多：

https://www.chigozie.co.uk/audio-keyframe-generator/

**三、根据静态图生成动画**  

**1\. 让肖像画说话**

▶ 代表工具：D-ID, Movio, Artflow

让一张静态的肖像画动起来说话的功能，应该是 AI 动画里成熟得最早的，包括：D-ID, Movio, Artflow AI 这些平台都提供了类似的功能，底层的算法大多基于 First Order Motion \[12\] 进行开发。  

基于这类产品做的内容，效果最好之一的有汗青老师的 AI Talk 系列 \[13\] 和《如果哈利波特是巴黎世家拍的》\[14\] 

![](_assets/640-21.png)

![](_assets/640-21.png)

**2\. 让静态图（随机）动起来**

▶ 代表工具：Pika Labs, Gen-2

底层技术大概是 Animatediff \[15\]，支持根据单张图输入其随机动态效果，有很多人结合 Midjourney + Gen-2 来制作一些视频，可能可以用于一些前期概念的 previs 或者一些营销场景的动图制作。但由于不能精准控制画面内容，所以对于目前工业界的帮助还不是很显著。

第三部分的工具功能非常有潜力，最近有许多用 Gen-2 做的电影预告片受到业内外很多关注。不过，**做电影预告片是一个非常讨巧的方式，因为不需要前后镜头有很强的连贯性，所以在体裁上还是会比较局限。** 

**推荐教程**

1\. D-ID, Movio, Artflow 这些工具的使用方式都很直接，基本上手就能直接会，所以就没有推荐教程了。

2\. 针对 Gen-2 我目前看到最好的分享是 Nicolas Neubert @iamneubert 在 Twitter 上的分享：https://twitter.com/iamneubert/status/1684989102213476359 他本人也已经用 Midjourney + Gen-2 的工作流做过非常好的预告片案例了。

![](_assets/640-3.jpg)

AI 动画这个领域内目前我看来离工业界最近的几篇核心论文就是：

1\. Runway Gen1：Structure and Content-Guided Video Synthesis with Diffusion Models \[16\]   

2\. AnimateDiff: Animate Your Personalized Text-to-Image Diffusion Models without Specific Tuning \[15\] 

3\. First Order Motion Model for Image Animation \[12\] 

 论文链接我都附在了本文末尾。  

终于写完了，希望这篇文章可以对你有所帮助。**目前我也正在用 Gen-2 做一些影视及广告项目，如果顺利的话，应该可以在不久后和大家分享成果和经验。** 如果你想更及时地看到平时我的各种尝试的话，欢迎关注我的微博 @海辛Hyacinth 或者 即刻@海辛

谢谢你阅读到这里，提前预祝你立秋愉快。

* * *

**Reference**

\[1\] Runway：https://app.runwayml.com/

\[2\] Karen X. Cheng：https://twitter.com/karenxcheng

\[3\] EbSynth：https://ebsynth.com/  

\[4\] Rerender A Video：https://anonymous-31415926.github.io/

\[5\] Rerender - Hugging Face Demo：https://huggingface.co/spaces/Anonymous-sub/Rerender

\[6\] Marc Rebillet Diffused：https://www.reddit.com/r/StableDiffusion/comments/153juu9/marc\_rebillet\_diffused/

\[7\] Marc Rebillet：https://www.youtube.com/watch?v=enYdAxVcNZA

\[8\] WarpFusion：https://github.com/Sxela/WarpFusion

\[9\] Deforum Animation Parameter Examples: https://deforum.github.io/animation.html

\[10\] DoodleChaos：https://www.youtube.com/watch?v=0fDJXmqdN-A

\[11\] DoodleChaos Patreon：https://www.patreon.com/posts/i-used-ai-to-66518281

\[12\] First Order Motion：https://aliaksandrsiarohin.github.io/first-order-model-website/

\[13\] AI-Talk：https://space.bilibili.com/405083326

\[14\] Harry Potter by Balenciaga：https://www.youtube.com/watch?v=iE39q-IKOzA

\[15\] AnimateDiff：https://animatediff.github.io/

\[16\] Structure and Content-Guided Video Synthesis with Diffusion Models: https://arxiv.org/abs/2302.03011