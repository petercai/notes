# MCP：打通AI與外部世界的新標準，開啟Cursor使用MCP的正確姿勢
![](https://media.greatfire.org/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_jpg/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDgrOLKw12FoxF35ianYtIep3eT4mdmnBibBtPwO8DeMrakgkBIpoxtaVQ/0?wx_fmt=jpeg)
 Original 明日知潮者 AI前哨 ，文章內容不知真假，也不代表本網立場和觀點。

##### ✈️ “每天5分鐘時間，了解一個AI概念”

模型上下文協議（MCP）作為[人工智慧](https://www.bannedbook.org/bnews/zh-tw/tag/%e4%ba%ba%e5%b7%a5%e6%99%ba%e8%83%bd/ "標籤 人工智慧 下的日誌")領域的一項革命性技術，正在悄然改變AI與外部數據源交互的方式。這個由Anthropic公司在2024年11月推出的開源標準，為AI模型提供了一個統一的介面，使它們能夠更加高效地與各種外部數據源和工具進行交互。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDDGeCjrCsYqcMkgLIa2nspDv4VrE40MYbqIkygrJZk8mtsccA1HtIibA/640?wx_fmt=png&from=appmsg)

模型上下文協議的設計精巧而實用，它包含了幾個關鍵組件，共同構成了一個完整的生態系統。

從整體架構看，這個協議由主機、客戶端、伺服器和傳輸層組成。主機是啟動連接的應用程序，比如Claude桌面版或Cursor編輯器；客戶端與伺服器保持一對一的連接，位於主機應用程序內部；而伺服器則向客戶端提供上下文、工具和提示。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDiapCsUpCicA0VmT8NIdAICgV5RkQJF0c8Fd0ucCTGXR80dJlg2bBnceA/640?wx_fmt=png&from=appmsg)

伺服器實現方式與工具使用API非常相似。以Brave搜索伺服器為例，它內部定義了可用的工具，並將其可用性傳達給客戶端。每個MCP伺服器都有一個URI，作為單獨的進程運行，並通過JSON-RPC與客戶端通信。

客戶端需要知道它可以連接到哪個伺服器，然後建立連接。連接過程中，客戶端發送帶有協議版本和功能的初始化請求，伺服器響應其協議版本和功能，客戶端發送初始化通知作為確認，然後開始正常的消息交換。

MCP支持多種傳輸機制，包括標準輸入/輸出傳輸（適用於本地進程）和基於HTTP的伺服器發送事件(SSE)傳輸（用於伺服器到客戶端的消息和客戶端到伺服器的HTTP POST消息）。所有傳輸都使用JSON-RPC 2.0交換消息。

模型上下文協議的一大亮點是其豐富的開源組件生態系統，這些組件使開發者能夠更輕鬆地構建和使用MCP伺服器。

在框架方面，有Python和TypeScript版本的FastMCP，它們是構建MCP伺服器的高級框架。SDK方面則提供了TypeScript和Python兩種選擇，滿足不同開發者的需求。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDwtnx4VM2sLAow0hlOIBWy4x1RUnsVPe4k7P8XAfMvQO3V95bBZzyhA/640?wx_fmt=png&from=appmsg)

生態系統中還包括Inspector（用於測試和調試MCP伺服器的開發者工具）和Specification（MCP協議的規範）。

伺服器組件更是豐富多樣，包括文件系統（具有可配置訪問控制的安全文件操作）、[GitHub](https://www.bannedbook.org/bnews/zh-tw/tag/github/ "標籤 GitHub 下的日誌")（倉庫管理、文件操作和GitHub API集成）、[Google](https://www.bannedbook.org/bnews/zh-tw/tag/google/ "標籤 Google 下的日誌") Drive（Google Drive的文件訪問和搜索功能）、PostgreSQL（具有架構檢查的只讀資料庫訪問）、Slack（頻道管理和消息功能）等多種服務。

[社區服務](https://www.bannedbook.org/bnews/zh-tw/tag/%E7%A4%BE%E5%8C%BA%E6%9C%8D%E5%8A%A1/ "標籤 社區服務 下的日誌")器也在不斷湧現，如MCP YouTube（使用yt-dlp下載YouTube字幕）、mcp-security-scanner（用於分析JavaScript代碼的安全漏洞掃描器）、Cloudflare MCP伺服器（用於與Cloudflare服務交互的統一介面）等。

儘管MCP還處於發展初期，但已經有一些主流開發工具率先採用了這一協議，為開發者提供了更加強大的功能。

### Cursor中怎麼使用MCP？

Cursor編輯器提供了簡便的方式來添加和管理MCP伺服器。用戶可以通過Cursor設置 > 功能 > MCP，點擊”+ 添加新MCP伺服器”按鈕來添加伺服器。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDcjWb9ib3pCScy9fVf3238mqMSA5VI0jic3bw85xJ1S84OdUQ4Z0IIicwQ/640?wx_fmt=png&from=appmsg)

添加伺服器時，用戶需要選擇傳輸類型，並填寫伺服器的昵稱以及根據傳輸類型填寫運行命令或伺服器URL。例如，配置MCP快速啟動天氣伺服器時，假設它已經構建並放置在指定位置，整個命令字元串類似於”node ~/mcp-quickstart/weather-server-typescript/build/index.js”。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDeXXM2QCQ1pybr3IXFoxdr5qlWOKb14CkAX5QmeGMERC8eC1Y081tpg/640?wx_fmt=png&from=appmsg)

Cursor還支持項目特定的MCP配置，用戶可以通過.cursor/mcp.json文件來配置項目特定的MCP伺服器。該文件遵循特定的格式，可以配置命令行界面(CLI)或伺服器發送事件(SSE)類型的伺服器。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibD4UD6ia0mS9okL3azSg8j2HJo2gb1ffmSz1Z2DiahIV4bksuzVJGSzRLw/640?wx_fmt=png&from=appmsg)

Cursor的Composer Agent會自動使用MCP設置頁面上”可用工具”下列出的任何MCP工具，如果它認為這些工具相關的話。用戶也可以通過名稱或描述明確告訴代理使用特定工具。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDuOFSTPm84MtXThaADJxYXiav8dXcemq6hjZgwIHzbquA1YG1GicvBiawA/640?wx_fmt=png&from=appmsg)

默認情況下，當Agent想要使用MCP工具時，它會顯示一條消息請求用戶批准。用戶可以展開消息查看工具調用參數。Cursor還提供了”Yolo模式”，允許Agent自動運行MCP工具而無需批准，類似於執行終端命令的方式。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDvWxTuOYbCibuKzdHNwajClbAPwxNgQicNnOtLGX3PByibSCKGH0kTsSPA/640?wx_fmt=png&from=appmsg)

### Sourcegraph Cody和Zed編輯器

Sourcegraph Cody通過MCP增強了編碼任務的上下文檢索能力。在Cody的博客文章中展示了一個例子，Cody連接到PostgreSQL資料庫，在查看資料庫架構后編寫Prisma查詢。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibD8WDmhzqeBfMhNfU2m0MnlpTwRhaWibr4ZFk2nZlGjHJ6cyiaXtB81pAg/640?wx_fmt=png&from=appmsg)

Zed編輯器也通過MCP擴展了開發者的能力。Zed編輯器的博客文章展示了使用PostgreSQL擴展立即向模型提供整個資料庫架構的信息，這樣當要求它編寫查詢時，它確切地知道存在哪些表和列，以及它們的類型。

![](https://media.greatfire.org/cdn-cgi/image/width=711/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDH7OUltaicjKAjIKaBFK3iaGezgEJrMc3B6O3eHia8JmCR4V516fYrczmg/640?wx_fmt=png&from=appmsg)

Claude桌面應用是測試MCP最簡單的方式之一。用戶需要在指定位置創建或編輯配置文件，添加相應的配置信息，告訴Claude桌面版有一個名為brave\_search的MCP伺服器，並使用特定命令啟動它。

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDv0nRzZmHPXaI9xToER78ZNeiaFsusBNpfBBQy3m5iazkXKYmLBPKqc8w/640?wx_fmt=png&from=appmsg)

保存文件並重啟Claude桌面版后，用戶就可以要求Claude”搜索網路”了。例如，嘗試讓Claude桌面版”搜索網路獲取glama.ai”的信息。系統會首先提示允許Claude桌面版使用MCP，點擊”允許此次聊天”后，Claude桌面版就會使用Brave搜索MCP伺服器搜索網路並獲取結果。

目前MCP還存在一些局限性。現實世界中使用MCP的應用程序相對較少，但這種情況會隨著時間的推移而改變。

對於Claude桌面版，MCP伺服器的設置過程目前是手動的；每次重新打開應用程序時，都必須給予Claude桌面版使用MCP的許可權；MCP目前只支持本地使用（伺服器必須在用戶自己的機器上運行）；大多數說明都是針對macOS的，不過也有人在[Windows](https://www.bannedbook.org/bnews/zh-tw/tag/windows/ "標籤 Windows 下的日誌")上成功使用了MCP。

在未來，我們可以期待更多的開發工具和AI應用集成MCP，形成一個豐富的生態系統。通用的LLM客戶端，如Glama、Claude桌面版或Cursor，可能會擁有MCP伺服器的市場，用戶將能夠輕鬆地將這些服務插入到他們的工作流程中。

雖然MCP是Anthropic公司的一項受歡迎的倡議，但它是否會獲得廣泛採用還需要時間來驗證。同時，[計算機](https://www.bannedbook.org/bnews/zh-tw/tag/%E8%AE%A1%E7%AE%97%E6%9C%BA/ "標籤 計算機 下的日誌")使用功能可能會成為AI驅動自動化的標準，包括MCP試圖解決的所有問題。

想象一下，你正坐在電腦前，你的AI助手不再局限於簡單的對話，而是能夠為你搜索網路、查詢資料庫、管理文件、甚至控制其他應用程序。這不是[科幻電影](https://www.bannedbook.org/bnews/zh-tw/tag/%E7%A7%91%E5%B9%BB%E7%94%B5%E5%BD%B1/ "標籤 科幻電影 下的日誌")中的場景，而是模型上下文協議（MCP）正在為我們帶來的現實。

作為開發者，你是否曾經希望你的AI工具能夠無縫地與你的開發環境交互？MCP就像是給AI裝上了一雙靈巧的手，讓它能夠觸及數字世界的每一個角落。無論是Cursor編輯器中智能代碼補全，還是Claude桌面版中的[網路搜索](https://www.bannedbook.org/bnews/zh-tw/tag/%E7%BD%91%E7%BB%9C%E6%90%9C%E7%B4%A2/ "標籤 網路搜索 下的日誌")功能，MCP都在幕後默默工作，連接AI與外部世界。

現在，就是你加入這場技術革命的最佳時機。不需要複雜的設置，只需幾行配置代碼，你就能在自己的項目中體驗MCP的強大功能。想象一下，當你告訴你的AI助手”幫我查詢資料庫中的用戶數據”時，它不僅理解你的指令，還能直接執行並返回結果的場景。

別讓這個機會從指尖溜走！今天就訪問GitHub上的MCP項目，下載示例伺服器，配置你的Cursor編輯器或Claude桌面版。加入Discord社區，與其他開發者分享你的經驗和創意。每一個探索者都是這項技術未來的塑造者，而你，完全可以成為其中的一員。

模型上下文協議不僅僅是一項技術，它是AI與人類協作的新篇章。當你親手搭建起AI與外部世界的橋樑時，你會發現，科技的魅力不僅在於它的複雜性，更在於它如何簡化我們的生活和工作。

所以，準備好你的開發環境，捲起袖子，讓我們一起踏上這段激動人心的旅程吧！未來的AI應用，正等待著你的創造和定義。

歡迎大家一起加入我的社群從0開始學AI，前100名免費贈送價值299元AI學習材料！

![](https://media.greatfire.org/cdn-cgi/image/width=880/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWzEISa6Zb4QmeGiaGXfZeKibDjReFib7JFf2XPxXo1fGLILrRyHA5ibXUTicDicQSmh3nKE1IIdm18uad8A/640?wx_fmt=png&from=appmsg)

![](https://media.greatfire.org/cdn-cgi/image/width=677/proxy/?url=https://mmbiz.qpic.cn/sz_mmbiz_png/hibqU8FasEWy2QiawxqibTRibzokW3RicawupK4Mk2HA3nDwiaynQd4eAaicEgCxBneiajJrOdSKRanIm7bVjAy654PLSA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
▲ 未來的我們

✨ 在[數字時代](https://www.bannedbook.org/bnews/zh-tw/tag/%E6%95%B0%E5%AD%97%E6%97%B6%E4%BB%A3/ "標籤 數字時代 下的日誌")的浪潮中，我們既是探索者，也是創造者。正如演算法在數據的海洋中尋找規律，人生的意義也在於于無盡的未知中，用智慧與勇氣編織出屬於自己的獨特軌跡。

*   [面試完別傻等！前**Google**高層揭「第一件事」必做 原因全說了](https://www.bannedbook.org/bnews/zh-tw/cnnews/20250307/2163788.html)
*   [**Google** 在 Colab 中推出數據科學智能體](https://www.bannedbook.org/bnews/zh-tw/itnews/20250305/2162562.html)
*   [**Google** 確認 Gmail 將放棄簡訊驗證碼身份驗證](https://www.bannedbook.org/bnews/zh-tw/itnews/20250224/2158590.html)
*   [AI也會霸凌？**Google**機器人竟辱罵學生「去死！」](https://www.bannedbook.org/bnews/zh-tw/cnnews/20250220/2156995.html)
*   [別被騙了 **Google**地圖上的「黑洞」其實是…](https://www.bannedbook.org/bnews/zh-tw/cnnews/20250219/2156686.html)

![](https://www.bannedbook.org/images/youtube.png)
站長動態，轉發分享↓Follow Us 責任編輯：趙凌雲