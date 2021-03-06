﻿^title 架構，性能和遊戲
^section Introduction

<span name="ammo"></span>在一頭紮入一堆設計模式之前，我想先講一些我對軟件架構和其應用到遊戲之中的理解，
這也許能幫你更好地理解這本書的其餘部分。
至少，在你被捲入一場關於設計模式和軟件架構有多麼糟糕（或多麼優秀）的辯論時，
這可以給你一些火力支援。

<aside name="ammo">

注意我沒有建議你在戰鬥中選哪一邊。就像任何軍火販子一樣，我願意向作戰雙方出售武器。

</aside>

## 什麼是軟件架構？

<span name="won't"></span>如果把本書從頭到尾讀一遍，
你不會學會3D圖形背後的線性代數或者遊戲物理背後的微積分。
這書不會告訴你如何用α-β修剪你的AI樹，也不會告訴你如何在音頻播放中模擬房間中的混響。

<aside name="won't">

Wow，這段給這本書打了個糟糕的廣告啊。

</aside>

相反，這本書告訴你在這些*之間*的事情。
與其說這本書是關於如何寫代碼，不如說是關於如何*架構*代碼的。
每個程序都有*一定*架構，哪怕這架構是“將所有東西都塞到`main()`中看看如何”，
所以我認為講講什麼造成了*好*架構是很有意思的。我們如何區分好架構和壞架構呢？

<span name="suffered"></span>我思考這個問題五年了。當然，像你一樣，我有對好設計有一種直覺。
我們都被糟糕的代碼折磨的不輕，你唯一能做的好事就是刪掉它們，結束它們的痛苦。

<aside name="suffered">

承認吧，我們中大多數人都該對一些糟糕代碼*負責*。

</aside>

少數幸運兒有相反的經驗，有機會在好好設計的代碼庫上工作。
那種代碼庫看上去是間豪華酒店，裡面的門房隨時準備滿足你心血來潮的需求。
這兩者之間的區別是什麼呢？

### 什麼是*好的*軟件架構？

對我而言，好的設計意味著當我作出改動，整個程序就好像正等著這種改動。
我可以僅使用幾個函數調用完成任務，而代碼庫本身無需改動。

這聽起來很棒，只是完全不可行。“把代碼寫到改動不會影響其平靜表面。”夠了。

讓我們通俗些。第一個關鍵點是*架構是有關於改動的*。
總有人改動代碼。如果沒人碰代碼——無論是因為代碼至善至美，
還是因為代碼糟糕透頂以至于沒人會為了修改它而玷污自己的文本編輯器——那麼它的架構設計就無關緊要。
評價架構設計的好壞就是評價它應對改動有多麼輕鬆。
沒有了改動，架構好似永遠不會離開起跑線的運動員。

### 你如何處理改動？

<span name="ocr"></span>在你改動代碼去添加新特性，去修復漏洞，或者隨便什麼需要使用文本編輯器的時候，
你需要理解代碼正在做什麼。當然，你不需要理解整個程序，
但你需要將所有相關的東西裝進你的大腦。

<aside name="ocr">

有點詭異，這字面上是一個OCR過程。

</aside>

我們通常無視了這步，但這往往是編程中最耗時的部分。
如果你認為將數據從磁碟上分頁到RAM上很慢，
那麼試試通過一對神經纖維將數據分頁到大腦中。

一旦把所有正確的上下文都記到了你的大腦裡，
想一會，你就能找到解決方案。
可能需要反復斟酌的時刻，但通常比較簡單。
一旦理解了問題和需要改動的代碼，實際的編碼工作有時是微不足道的。

<span name="tests"></span>你用手指在鍵盤上敲打一陣，直到屏幕上閃著正確的光芒，
搞定了，對吧？還沒呢！
在你為之寫測試並發送到代碼評審之前，你通常有些清理工作要做。

<aside name="tests">

我是不是說了“測試”？噢，是的，我說了。為有些遊戲代碼寫單元測試很難，但代碼庫的大部分是完全可以測試的。

我不會在這裡發表演說，但是我建議你，如果還沒有做自動測試，請考慮一下。
除了手動驗證以外你就沒別的事要做了嗎？

</aside>

你將一些代碼加入了遊戲，但不想下一個人被留下來的小問題絆倒。
除非改動很小，否則就還需要一些工作去微調新代碼，使之無縫對接到程序的其他部分。
如果你做對了，那麼下個編寫代碼的人無法察覺到哪些代碼是新加入的。

簡而言之，編程的流程圖看起來是這樣的：

<img src="images/architecture-cycle.png" alt="獲得問題 &rarr; 研究代碼 &rarr; 編寫解決方案 &rarr; 清理代碼 &rarr; 回到開始。" />

<span name="life-cycle"></span>

<aside name="life-cycle">

令人震驚的死循環，我看到了。

</aside>

### 解耦幫了什麼忙？

雖然並不明顯，但我認為很多軟件架構都是關於研究代碼的階段。
將代碼載入到神經元太過緩慢，找些策略減少載入的總量是件很值得做的事。
這本書有整整一章是關於[*解耦*模式](decoupling-patterns.html)，
還有很多*設計模式*是關於同樣的主題。

可以用多種方式定義“解耦”，但我認為如果有兩塊代碼是耦合的，
那就意味著無法只理解其中一個。
如果*解*耦了它倆，就可以獨自地理解某一塊。
這當然很好，因為只有一塊與問題相關，
只需將*這一塊*加載到你的大腦腦中而不需要加載另外一塊。

對我來說，這是軟件架構的關鍵目標：
**最小化在編寫代碼前需要瞭解的信息**。

當然，也可以從後期階段來看。
另一種解耦的定義是：當一塊代碼有*改動*時，沒必要修改另一塊的代碼。
肯定需要修改*一些東西*，但耦合程度越小，改動會波及的範圍就越小。

## 代價呢？

聽起來很棒，對吧？解耦任何東西，然後像風一樣編碼。
每個改動都只需修改一兩個特定方法，在代碼庫上行雲流水地編寫代碼。

這就是人們對抽象，模組化，設計模式和軟件架構興奮的原因。
在架構優良的程序上工作是極佳的體驗，每個人都希望能更有效率地工作。
好架構能造成生產力上*巨大的*不同。它影響大得無以復加。

<span name="maintain"></span>但是，天下沒有免費的午餐。好的設計需要汗水和紀律。
每次做出改動或是實現特性，你都需要將它優雅的整合到程序的其他部分。
需要花費大量的努力去管理代碼，
在開發過程中面對數千次變化仍然*保持*它的結構。

<aside name="maintain">

第二部分——管理代碼——需要特別關注。
我看到無數程序有優雅的開始，然後死於程序員一遍又一遍添加的“微小黑魔法”。

就像園藝，僅僅種植是不夠的，還需要除草和修剪。

</aside>

你得考慮程序的哪部分需要解耦，然後再引入抽象。
同樣，你需要決定哪部分能支持擴展來應對未來的改動。

人們對這點變得狂熱。
他們設想，未來的開發者（或者他們自己）進入代碼庫，
發現它極為開放，功能強大，只需擴展。
他們想要有“至尊代碼應眾求”。（譯著：這裡是“至尊魔戒禦眾戒”的梗，很遺憾翻譯不出來）

但是，事情從這裡開始變得棘手。
每當你添加了抽象或者擴展支持，你是*賭*以後這裡需要靈活性。
你向遊戲中添加了代碼和複雜性，這需要時間來開發，調試和維護。

<span name="yagni"></span>如果你賭對了，後來使用了這些代碼，那麼功夫不負有心人。
但預測未來*很難*，模組化如果最終無益，那就有害。
畢竟，你得處理更多的代碼。

<aside name="yagni">

有些人喜歡使用術語“YAGNI”——[You aren't gonna need it（你不需要那個）](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it)——來對抗這種預測將來需求的強烈衝動。

</aside>

當你過分關注這點時，代碼庫就失控了。
介面和抽象無處不在。插件系統，抽象基類，虛方法，還有各種各樣的擴展點，它們遍地都是。

你要消耗無盡的時間回溯所有的腳手架，去找真正做事的代碼。
當需要作出改動，當然，有可能某個介面能幫上忙，但能不能找到就只能聽天由命了。
理論上，解耦意味著在修改代碼之前需要瞭解更少的代碼，
但抽象層本身也會填滿大腦。

像這樣的代碼庫讓人*反對*軟件架構，特別是設計模式。
人們很容易沉浸在代碼中，忽略目標是要發佈*遊戲*。
無數的開發者聽著加強可擴展性的警報，花費多年時間製作“引擎”，
卻沒有搞清楚做引擎是*為了什麼*。

## 性能和速度

<span name="templates"></span>軟件架構和抽象有時因損傷性能而被批評，而遊戲開發尤甚。
讓代碼更靈活的許多模式依靠虛擬調度、 介面、 指針、 消息和其他機制，
它們都會加大運行時開銷。

<aside name="templates">

一個有趣的反面例子是C++中的模板。模板編程有時可以給你抽象介面而無需運行時開銷。

這是靈活性的兩極。
當寫代碼調用類中的具體方法時，你在*寫*的時候指定類——硬編碼了調用的是哪個類。
通過虛方法或介面，直到*運行*時才知道調用的類。這更加靈活但增加了運行時開銷。

模板編程是在兩極之間。在*編譯時*初始化模板，決定調用哪些類。

</aside>

還有一個原因。很多軟件架構的目的是使程序更加靈活，作出改動需要更少的付出，編碼時對程序有更少的假設。
你可以使用介面，讓代碼可與*任何*實現了介面的類交互，而不僅僅是*現在*寫的類。
今天，你可以使用<a href="observer.html" class="gof-pattern">觀察者</a>和<a href="event-queue.html" class="pattern">消息</a>讓遊戲的兩部分交流，
以後可以很容易地擴展為三個或四個部分相互交流。

但性能與假設相關。實現優化需要基于確定的限制。
敵人永遠不會超過256個？好，可以將敵人ID編碼為一個位元組。
只在這種類型上調用方法嗎？好，可以做靜態調度或內聯。
所有實體都是同一類？太好了，可以使用 <a href="data-locality.html" class="pattern">連續數組</a>存儲它們。

但這並不意味著靈活性不好！它可以讓我們快速改進遊戲，
*開發* 速度是獲取有趣開發經驗的絕對重要因素。
沒有人，哪怕是Will Wright，能在紙面上構建一個平衡的遊戲。這需要迭代和實驗。

越快嘗試想法看看效果，就能嘗試越多東西，越可能找到有價值的東西。
就算找到正確的機制，你也需要足夠的時間調整。
一個微小的不平衡就有可能破壞整個遊戲的樂趣。

這裡沒有普適的答案。
讓你的程序更加靈活，在損失一點點性能的前提下更快地做出原型。
或者，優化代碼，損失一些靈活性。

就我個人經驗而言，讓有趣的遊戲變得高效比讓高效的遊戲變有趣簡單得多。
一種折中的辦法是保持代碼靈活直到確定設計，再去除抽象層來提高性能。

## 糟糕代碼的優勢

下一觀點：不同的代碼風格各有千秋。
這本書的大部分是關於保持乾淨可控的代碼，所以我堅持應該用*正確*方式寫代碼，但糟糕的代碼也有一定的優勢。

編寫良好架構的代碼需要仔細地思考，這會消耗時間。
在項目的整個周期中*保持*良好的架構需要花費大量的努力。
你需要像露營者處理營地一樣小心處理代碼庫：總是保持其優於你剛剛接觸它的時候。

當你要在項目上花費很久時間的話，這很好。
但，就像早先提到的，遊戲設計需要很多實驗和探索。
特別是在早期，寫一些你*知道*要扔掉的代碼是很普遍的事情。

如果只想試試遊戲的某些點子是否可行的，
良好的架構意味著在屏幕上看到和獲取反饋之前要消耗很長時間。
如果最後證明這點子不對，那麼刪除代碼時，那些讓代碼更優雅的時間就付之東流了。

原型——一坨勉強拼湊在一起，只能完成某個點子的簡單代碼——是個完全合理的編程實踐。
雖然當你寫一次性代碼時，*必須* 保證將來可以扔掉它。
我見過很多次糟糕的經理人在玩這種把戲：

> 老闆：“嗨，我有些想試試的點子。只要原型，不需要做的很好。你能多快搞定？”

> 開發者：“額，如果刪掉這些部分，不測試，不寫文檔，允許很多的漏洞，那麼幾天能給你臨時的代碼檔案。”

> 老闆：“太好了。”

*幾天後*

> 老闆：“嘿，原型很棒，你能花上幾個小時清理一下然後變為成品嗎？”

<span name="throwaway"></span>你得讓人們清楚，可拋棄的代碼即使看上去能工作，也不能被*維護*，*必須* 重寫。
如果*有可能*要維護這段代碼，就得防禦性好好編寫它。

<aside name="throwaway">

一個小技巧能保證原型代碼不會變成真正用的代碼：使用和遊戲實現不同的編程語言。
這樣，在將其實際應用於遊戲中之前必須重寫。

</aside>

## 保持平衡


有些因素在相互角力：

<span name="speed"></span>
1. 為了在項目的整個生命周期保持其可讀性，需要好架構。
2. 需要更好的運行時性能。
3. 需要讓現在的特性更快的實現。

<aside name="speed">

有趣的是，這些都是速度：長期開發的速度，遊戲運行的速度，和短期開發的速度。

</aside>

這些目標至少是部分對立的。
好架構長期來看提高了生產力，
也意味著每個改動都需要消耗更多努力保持代碼整潔。

草就的代碼很少是*運行時*最快的。
相反，提升性能需要很多的編程時間。
一旦完成，它就會污染代碼庫：高度優化的代碼不靈活，很難改動。

總有今日事今日畢的壓力。但是如果儘可能快地實現特性，
代碼庫就會充滿黑魔法，漏洞和混亂，阻礙未來的產出。

沒有簡單的答案，只有權衡。
從我收到的郵件看，這傷了很多人的心，特別是那些只是想做個遊戲的人。
這似乎是在恐嚇，“沒有正確的答案，只有不同的錯誤。”

<span name="ditch"></span>但，對我而言，這讓人興奮！看看任何人們從事的領域，
你總能發現某些相互牴觸的限制。無論如何，如果有簡單的答案，每個人都會那麼做。
一周就能掌握的領域是很無聊的。你從來沒有聽說過有人討論挖坑。

<aside name="ditch">

也許你會討論挖坑；我沒有深究這個類比。
可能有挖坑熱愛者，挖坑規範，以及一整套亞文化。
我算什麼人，能在此大放厥詞？

</aside>

對我來說，這和遊戲有很多相似之處。
國際象棋之類的遊戲永遠不能被掌握，因為每個棋子都很完美的與其他棋子相平衡。
這意味你可以花費一生探索廣闊的可選策略。糟糕的遊戲就像井字棋，玩上幾遍就會厭倦就退出。

## 簡單

最近，我感覺如果有什麼能簡化這些限制，那就是*簡單*。
在我現在的代碼中，我努力去寫最簡單，最直接的解決方案。
你讀過這種代碼後，完全理解了它在做什麼，想不到其他完成的方法。

我的目標是正確獲得資料結構和算法（大致是這樣的先後），然後在從那裡開始。
我發現如果能讓事物變得簡單，就有更少的代碼，
就意味著改動時有更少的代碼載入腦海。

它通常跑的很快，因為沒什麼開銷，也沒什麼代碼需要執行。
（雖然大部分時候事實並非如此。你可以在一小段代碼里加入大量的循環和遞歸。）

<span name="simple"></span>但是，注意我並沒有說簡單的代碼需要更少的時間*編寫*。
你會這麼覺得是因為最終得到了更少的代碼，但是好的解決方案不是往代碼中注水，而是*蒸乾*代碼。

<aside name="simple">

Blaise Pascal有句著名的信件結尾，“我沒時間寫的更短。”

另一句名言來自Antoine de Saint-Exupery：“臻于完美之時，不是加無可加，而是減無可減。”

言歸正傳，我發現每次重寫本書，它就更短。有些章節比剛完成時短了20%。

</aside>

我們很少遇到優雅表達的問題，一般反而是一堆用況。
你想要X在Z情況下做Y，在A情況下做W，諸如此類。換言之，一長列不同行為。

最節約心血的方法是為每段用況編寫一段代碼。
看看新手程序員，他們經常這麼幹：為每個手頭的問題編寫條件邏輯。

但這一點也不優雅，那種風格的代碼遇到一點點沒想到的輸入就會崩潰。
當我們想象優雅的代碼時，想的是*通用*的那一個：
只需要很少的邏輯就可以覆蓋整個用況。

找到這樣的方法有點像模式識別或者解決謎題。
需要努力去識別散亂的用例下隱藏的規律。
完成時你會感覺好得不能再好。

## 就快完了

几乎每個人都會跳過介紹章節，所以祝賀你看到這裡。
我沒有太多東西回報你的耐心，但還有些建議給你，希望對你有用：

* 抽象和解耦讓擴展代碼更快更容易，但除非確信需要靈活性，否則不要在這上面浪費時間。

* <span name="think"></span>在整個開發周期中考慮併為性能設計，但是儘可能推遲那些底層的，基于假設的優化，那會鎖死代碼。

<aside name="think">

相信我，發佈前兩個月*不是*開始思考“遊戲運行只有1FPS”問題的時候。

</aside>

* 快速地探索遊戲的設計空間，但不要跑的太快，在身後留下爛攤子。畢竟，你總得回來打掃。

* 如果打算拋棄這段代碼，就不要嘗試將其寫完美。搖滾明星將旅店房間弄得一團糟，因為他們知道明天他們就走人了。

* 但最重要的是，**如果你想要做出讓人享受的東西，那就享受做它的過程。**
