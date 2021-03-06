﻿^title 享元模式
^section Design Patterns Revisited

迷霧散盡，露出了古樸莊嚴的森林。古老的鐵杉，在頭頂編成綠色穹頂。
陽光在樹葉間破碎成金色頂棚。從樹幹間遠眺，遠處的森林漸漸隱去。

這是我們遊戲開發者夢想的超凡場景，這樣的場景通常由一個模式支撐著，它的名字低調至極：享元模式。

## 森林

用幾句話就能描述一片巨大的森林，但是在實時遊戲中做這件事就完全是另外一件事了。
當屏幕上需要顯示一整個森林時，圖形程序員看到的是每秒需要送到GPU六十次的百萬多邊形。

我們討論的是成千上萬的樹，每棵都由上千的多邊形組成。
就算有足夠的*內存*描述森林，渲染的過程中，CPU到GPU的部分也太過繁忙了。

每棵樹都有一系列與之相關的位：

* 定義樹幹，樹枝和樹葉形狀的多邊形網格。
* 樹皮和樹葉的紋理。
* 在森林中樹的位置和朝向。
* 大小和色彩之類的調節參數，讓每棵樹都看起來與眾不同。

如果用代碼表示，那麼會得到這樣的東西：

^code heavy-tree

這是一大堆數據，多邊形網格和紋理體積非常大。
描述整個森林的對象在一幀的時間就交給GPU是太過了。
幸運的是，有一種老辦法來處理它。

關鍵點在於，哪怕森林裡有千千萬萬的樹，它們大多數長得一模一樣。
它們使用了<span name="same">相同的</span>網格和紋理。
這意味著這些樹的實例的大部分欄位是*一樣的*。

<aside name="same">

你要麼是瘋了，要麼是億萬富翁，才能讓美術給整個森林的每棵樹建一個獨立模型。

</aside>

<span name="trees"></span>

<img src="images/flyweight-trees.png" alt="一行樹，每棵都有自己的網格、紋理、樹葉，調節參數和位置朝向。" />

<aside name="trees">

注意每一棵樹的小盒子中的東西都是一樣的。

</aside>

<span name="type"></span>
我們可以通過顯式將對象切為兩部分來更加明確地模擬。
第一，將樹共有的數據拿出來分離到另一個類中：

^code tree-model

遊戲只需要一個這種類，
因為沒有必要在內存中把相同的網格和紋理重複一千遍。
每個遊戲世界中樹的實例只需有一個對這個共享`TreeModel`的*引用*。
留在`Tree`中的是那些實例相關的數據：

^code split-tree

你可以將其想象成這樣：

<img src="images/flyweight-tree-model.png" alt="一行樹，每個都有自己的參數和位置朝向，指向另一個有網格、紋理、樹葉的樹模型。" />

<aside name="type">

這有點像<a href="type-object.html" class="pattern">類型對象</a>模式。
兩者都涉及將一個類中的狀態委託給另外的類，來達到在不同實例間分享狀態的目的。
但是，這兩種模式背後的意圖不同。

使用類型對象，目標是通過將類型引入對象模型，減少需要定義的類。
伴隨而來的內容分享是額外的好處。享元模式則是純粹的為了效率。

</aside>

把所有的東西都存在主存裡沒什麼問題，但是這對渲染也毫無幫助。
在森林到屏幕上之前，它得先到GPU。我們需要用顯卡可以識別的方式共享數據。

## 一千個實例

為了減少需要推送到GPU的數據量，我們想把共享的數據——`TreeModel`——只發送*一次*。
然後，我們分別發送每個樹獨特的數據——位置，顏色，大小。
最後，我們告訴GPU，“使用同一模型渲染每個實例”。


<span name="hardware"></span>
幸運的是，今日的圖形介面和顯卡正好支持這一點。
這些細節繁瑣且超出了這部書的範圍，但是Direct3D和OpenGL都可以做[*實例渲染*](http://en.wikipedia.org/wiki/Geometry_instancing)。

在這些API中，你需要提供兩部分數據流。
第一部分是一塊需要渲染多次的共同數據——在例子中是樹的網格和紋理。
第二部分是實例的列表以及繪製第一部分時需要使用的參數。
然後調用一次渲染，繪製整個森林。

<aside name="hardware">

這個API是由顯卡直接實現，意味著享元模式也許是唯一的有硬件支持的GoF設計模式。

</aside>

## 享元模式

好了，我們已經看了一個具體的例子，下面我介紹模式的通用部分。
享元，就像它名字暗示的那樣，
當你需要共享類時使用，通常是因為你有太多這種類了。

實例渲染時，每棵樹通過匯流排送到GPU消耗的更多的是*時間*而非內存，但是基本要點是一樣的。

這個模式通過將對象的數據分為兩種來解決這個問題。
第一種數據沒有特定指明是哪個對象的*實例*，因此可以在它們間分享。
Gof稱之為*固有*狀態，但是我更喜歡將其視為“上下文無關”部分。
在這裡的例子中，是樹的網格和紋理。

數據的剩餘部分是*變化*狀態，那些每個實例獨一無二的東西。
在這個例子中，是每棵樹的位置，拉伸和顏色。
就像這裡的示例代碼塊一樣，這種模式通過在每個對象出現時共享一份固有狀態，來節約內存。

就目前而言，這看上去像是基礎的資源共享，很難被稱為一種模式。
部分原因是在這個例子中，我們可以為共享狀態划出一個清晰的*身份*：`TreeModel`。

我發現，當共享對象沒有有效定義的實體時，使用這種模式就不那麼明顯（使用它也就越發顯得精明）。
在那些情況下，這看上去是一個對象同時被魔術般的分配到了多個地方。
讓我展示給你另外一個例子。

## 紮根之所

這些樹長出來的地方也需要在遊戲中表示。
這裡可能有草，泥土，丘陵，湖泊，河流，以及其它任何你可以想到的地形。
我們*基于區塊*建立地表：世界的表面被劃分為由微小區塊組成的巨大網格。
每個區塊都由一種地形覆蓋。

每種地形類型都有一系列特性會影響遊戲玩法：

* 決定了玩家能夠多快的穿過它的移動開銷。
* 表明能否用船穿過的水域標識。
* 用來渲染它的紋理。

<span name="learned"></span>
因為我們遊戲程序員偏執于效率，我們不會在每個區塊中保存這些狀態。
相反，一個通用的方式是為每種地形使用一個枚舉。

<aside name="learned">

再怎麼樣，我們已經從樹的例子吸取教訓了。

</aside>

^code terrain-enum

然後，世界管理巨大的網格：

<span name="grid"></span>

^code enum-world

<aside name="grid">

這裡我使用嵌套數組存儲2D網格。
在C/C++中這樣很有效率的，因為它會將所有元素打包在一起。
在Java或者其他內存管理語言中，那樣做會實際給你一個數組，其中每個元素都是對數組的列的*引用*，那就不像你想要的那樣內存友好了。

反正，隱藏2D網格資料結構背後的實現細節，能使代碼能更好的工作。
我這裡這樣做只是為了讓其保持簡單。

</aside>

為了獲得區塊的實際有用的數據，我們做了一些這樣的事情：

^code enum-data

你知道我的意思了。這可行，但是我覺得很醜。
移動開銷和水域標識是區塊的*數據*，但這裡它們散佈在代碼中。
更糟的是，簡單地形的數據被眾多方法拆開了。
如果能夠將這些包裹起來就好了。畢竟，那是我們設計對象的目的。

如果我們有實際的地形*類*就好了，像這樣：

<span name="const"></span>

^code terrain-class

<aside name="const">

你會注意這裡所有的方法都是`const`。這不是巧合。
由於同一對象在多處引用，如果你修改了它，
改變會同時在多個地方出現。

這也許不是你想要的。
通過分享對象來節約內存的這種優化，不應該影響到應用的顯性行為。
因此，享元對象几乎總是不可變的。

</aside>

但是我們不想為每個區塊都保存一個實例。
如果你看看這個類裡面，你會發現裡面實際上*什麼也沒有*，
唯一特別的是區塊在*哪裡*。
用享元的術語講，區塊的*所有*狀態都是“固有的”或者說“上下文無關的”。

鑒於此，我們沒有必要保存多個同種地形類型。
地面上的草區塊兩兩無異。
我們不用地形區塊對象枚舉構成世界網格，而是用`Terrain`對象*指針*組成網格：

^code world-terrain-pointers

每個相同地形的區塊會指向相同的地形實例。

<img src="images/flyweight-tiles.png" alt="一行區塊，每個區塊指向共享的草、河、山丘對象。" />

由於地形實例在很多地方使用，如果你想要動態分配，它們的生命周期會有點複雜。
因此，我們直接在遊戲世界中存儲它們。

^code world-terrain

然後我們可以像這樣來描繪地面：

<span name="generate"></span>

^code generate

<aside name="generate">

我承認這不是世界上最好的地形生成算法。

</aside>

現在不需要`World`中的方法來接觸地形屬性，我們可以直接暴露出`Terrain`對象。

^code get-tile

用這種方式，`World`不再與各種地形的細節耦合。
如果你想要某一區塊的屬性，可直接從那個對象獲得：

^code use-get-tile

我們回到了操作實體對象的API，几乎沒有額外開銷——指針通常不比枚舉大。

## 性能如何？

<span name="cache"></span>
我在這裡說几乎，是因為性能偏執狂肯定會想要知道它和枚舉比起來如何。
通過解引用指針獲取地形需要一次間接跳轉。
為了獲得移動開銷這樣的地形數據，你首先需要跟著網格中的指針找到地形對象，
然後再找到移動開銷。跟蹤這樣的指針會導致緩存不命中，降低運行速度。

<aside name="cache">

對於更多指針追逐和緩存不命中，看看<a href ="data-locality.html" class="pattern">數據局部性</a>這章。

</aside>

就像往常一樣，優化的金科玉律是*需求優先*。
現代計算機硬件過于複雜，性能只是遊戲的一個考慮方面。
在我這章做的測試中，享元較枚舉沒有什麼性能的優勢。
享元實際上明顯更快。但是這完全取決於內存中的事物是如何排列的。

我*可以*自信使用享元對象而不會搞到不可收拾。
它給了你面向對象的優勢，而且沒有產生一堆對象。
如果你創建了一個枚舉，又在它上面做了很多分支跳轉，考慮一下這個模式吧。
如果你擔心性能，在把代碼編程為難以維護的風格之前，至少先做些性能分析。

## 參見

*  在區塊的例子中，我們只是為每種地形創建一個實例然後存儲在`World`中。
這也許能更好找到和重用這些實例。
但是在多數情況下，你不會在一開始就創建*所有*享元。

如果你不能預料哪些是實際上需要的，最好在需要時才創建。
為了保持共享的優勢，當你需要一個時，首先看看是否已經創建了一個相同的實例。
如果確實如此，那麼只需返回那個實例。

這通常意味需要將建構子封裝在查詢對象是否存在的介面之後。
像這樣隱藏構造指令是<a href="http://en.wikipedia.org/wiki/Factory_method_pattern" class="gof-pattern">工廠方法</a>的一個例子。

*  為了返回一個早先創建的享元，需要追蹤那些已經實例化的對象池。
正如其名，這意味著<a href="object-pool.html" class="pattern">對象池</a>是存儲它們的好地方。

* 當使用<a class="pattern" href="state.html">狀態</a>模式時，
經常會出現一些沒有任何特定欄位的“狀態對象”。
這個狀態的標識和方法都很有用。
在這種情況下，你可以使用這個模式，然後在不同的狀態機上使用相同的對象實例。
