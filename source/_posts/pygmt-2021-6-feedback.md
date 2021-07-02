---
title: 'PyGMT, 在 Python 製圖領域推廣 GMT 的利器? 我 ok, 你先用?'
tags:
  - genericmappingtools
  - geophysics
  - earthscience
  - mapping
  - python
  - freesoftware
  - opensource
date: 2021-06-21 07:37:03
---


Generic Mapping Tools，通用製圖工具，官方以及常見的簡稱為 GMT，是在地球科學領域廣泛使用的地理製圖工具之一。它可以在各種平臺結合 Shell Script (bash、csh)，Batch file(Windows) 的特性以及第三方工具(如: awk)，撰寫腳本以進行空間資訊的數據處理及高度客製化的地圖、圖表繪製。PyGMT 於 2020 年 5 月釋出，作為 GMT 在 Python 的 API。究竟這樣的專案對於 GMT 的使用族群而言有什麼可利用之處呢？以下分享個人的使用心得。

<!--more-->

## GMT 與 PyGMT 介紹

Generic Mapping Tools，通用製圖工具，官方以及常見的簡稱為 GMT。於 1988 年釋出第一個 1.0 版本，並且受[美國國家科學基金會 (NSF)](http://www.nsf.gov/)的贊助持續開發至今，在 2021 年 6 月 5 日已經釋出 6.2.0 版本，是在地球科學領域廣泛使用的地理製圖工具之一。它可以在各種平臺結合 Shell Script (bash、csh)，Batch file(Windows) 的特性以及第三方工具(如: awk)，撰寫腳本以進行空間資訊的數據處理及高度客製化的地圖、圖表繪製。

GMT 團隊在 5.x 版釋出時，就已經積極投入應用程式介面(API)的開發，並陸續開發了 Fortran，MATLAB/Octave，Julia 等語言的 API。GMT 6.x 於 2019 年釋出後，於隔年 5 月便有了 GMT 在 Python 的 API，被命名為 PyGMT 專案。讓使用者能夠利用 Python 語法呼叫 PyGMT 提供的函式透過 GMT 的官方 API (而非透過 Shell 呼叫)進行製圖腳本的撰寫。究竟這樣的專案對於 GMT 的使用族群而言有什麼可利用之處呢？以下分享個人的使用心得。


## PyGMT 潛在優勢

以下先分享使用 PyGMT 的好處：
1. 可以更輕鬆地結合在 Python 更為強大的套件(`pandas`，`numpy`，`obspy`)進行科學數據的運算與結果製圖
2. 避開對系統 Shell 的直接呼叫，以提升以 GMT 為基礎建構應用程式的安全性。(相對於很多人偷懶直接使用 `system()` 直接呼叫 `gmt` 指令)
3. 在 PyGMT 函數內的參數名稱，會設計成一般人較容易理解的長參數名稱，相較於 GMT 在 shell 指令的選項以短參數為主，可讀性大幅提升

關於參數名稱的部分，這邊引用並微幅修改 [GMT 教學手冊](https://gmt-tutorials.org/making_first_map.html) 的範例作為對照。

預計出來的原圖如下：
![](https://gmt-tutorials.org/_images/making_first_map_gmt6_fig4.png)

若使用 GMT 5 的語法在 Shell 進行製圖：
```bash
gmt pscoast \
    -R19.42/22.95/59.71/60.56 \
    -JM6i \
    -W0.1p,black \
    -Gdarkseagreen2 \
    -Scornflowerblue \
    -Df \
    -P \
    -Ba2f0.5g1 \
    -BWSne+t"Archipelago Sea" \
    -A0.1 \
    -U > archi_sea.ps
```

GMT 6 則是：
```bash
gmt coast \
    -R19.42/22.95/59.71/60.56 \
    -JM6i \
    -W0.1p,black \
    -Gdarkseagreen2 \
    -Scornflowerblue \
    -Ba2f0.5g1 \
    -BWSne+t"Archipelago Sea" \
    -A0.1 \
    -U \
    -png archi_sea
```

而 PyGMT 則是：
```python3
import pygmt
fig = pygmt.Figure()
fig.coast(
        region=[19.42, 22.95, 59.71, 60.56],
        projection="M6i",
        shorelines=['0.1p', 'black'],
        land='darkseagreen2',
        water='cornflowerblue',
        frame=['a2f0.5g1', 'WSne+t"Archipelago Sea"'],
        area_thresh=0.1,
        timestamp=True,
)
fig.show()
fig.savefig('archi_sea_pygmt.png')
```

在查看文件之前，哪個腳本較能夠讓人直覺去猜到各參數的意義，相信答案已經很明顯了吧。


## PyGMT 的疑慮與待改進之處

然而，相對於 GMT 的開發歷史，PyGMT 仍然是一個非常非常年輕的套件。縱使有了 NSF 的贊助，相對於其他類似規模的開源專案資源開發能量仍然十分有限。也因此我們可以輕易地找出許多理由，去建議剛接觸指令製圖的新手「先不要」輕易嘗試 PyGMT 這個套件。理由如下：
1. PyGMT 支援的 GMT 模組數量仍然有限
2. 即使某些 GMT 模組能被 PyGMT 支援，但能使用的長選項參數、能夠讀取的資料格式仍相對受限
3. 開發/教學文件與範本數量仍極度缺乏，但此部分至少可以對照 GMT 指令版本的文件

第 1,2 點的部分，以目前釋出的 0.3.1 版為例，該版本不支援、支援有限但常被使用的 GMT 指令模組包含了 [`project`](https://docs.generic-mapping-tools.org/6.2/project.html) (對平面直線附近包含深度資訊的座標點進行剖面繪製)，[`pssac`](https://docs.generic-mapping-tools.org/6.2/supplements/seis/pssac.html) (讀取 SAC 檔並繪製地震波形)， [`histogram`](https://docs.generic-mapping-tools.org/6.2/histogram.html) (繪製長條圖，目前已於 [git](https://zh.wikipedia.org/zh-tw/Git) `master` 開發中版本開始支援)，[`meca`](https://docs.generic-mapping-tools.org/6.2/supplements/seis/meca.html) (舊稱 `psmeca`，已實作，但不支援震源機制球置放在震央以外的位置)...等。

即使 `pssac` 和 `histogram` 功能可分別用 `obspy` 和 `matplotlib` 替代，但使用介面上的差異，進而造成腳本轉換所要花費的時間，卻遠大於從 GMT 5,6 轉換到其他 PyGMT 支援的模組所要付出的。

另外在傳統 GMT 繪圖的過程中，我們需要 `-V` 選項來讓程式輸出更多明顯的提示訊息以利除錯校正，在 PyGMT 中開發者嘗試使用 `verbose=True` 使其參數更容易被理解，但大量使用此選項後便會發現許多函式仍然只能使用 `V=True` 來應付其 alias 無法實作的問題。

上面所遇到的問題會徒然花費使用者時間去想辦法應付這些問題，進而大幅降低使用者嘗試的意願。包含並不限於使用替代套件、透過 `pygmt.clib.Session.call_module()` 呼叫 API、回去使用 `subprocess`...等，其中必須用到 `subprocess()` 及其 pipe 功能來應付模組 `project` 不被 PyGMT 支援的問題時，已經開始思考當初使用這個套件的理由了。


## PyGMT 還有搞頭嗎?

身為開源軟體的使用者、推廣者而言，仍然希望這個專案的開發能夠繼續下去。畢竟上面實際使用該套件後遇到的問題，多偏向開發、使用者社群尚未完整地建立起來的因素，再加上一般使用者多偏向從現有/舊有腳本去修改來繪製研究成果以節省時間，這也成為了推廣新工具(PyGMT，GMT 6 語法)、給予回饋的巨大阻礙。然而新需求、新方法總會不斷地出現，停留在舊版程式與腳本也遲早會遇到無法被新平臺、新工具支援的問題。如果我們持續停留在消耗別人既有成果而對於回饋開源專案這件事情不甚重視的話，未來能從它身上滿足的需求也將持續受限，無法跨出新的一步。

其他能做的事情，或許可以列舉：
- 整理以上支援受限的 module 資訊，包含其必要性 (是否優於替代方案)，回報給開發者社群。
- 瞭解該專案 alias 實作方式，將能夠理解且需要修改的部分整理成新的 Pull Request (當然開發者搶先一步修好是更值得慶幸的)。
- 整理分享更多樣的繪圖腳本紀錄供社群參考，以利讓更多人在有意願嘗試時有個參考依據。

對於那些前仆後繼在 GMT 相關專案投入大量心力改善軟體品質的前輩，個人也給予深深的尊敬與肯定，自己能做的除了以上那些，可能也只有幫忙打包新版的程式到自己習慣用的 Linux 發行版(Archlinux Arch Users Repository)，方便別人嘗鮮吧。
