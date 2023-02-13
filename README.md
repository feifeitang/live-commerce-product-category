## 描述
透過蝦皮上 25 萬多個商品的名稱及其對應的大類別及小類別，利用文字探勘的技巧，對蝦皮上的商品名稱先做基本的前處理後，嘗試用任意的演算法，對蝦皮的商品建立一個分類模型（大類、小類皆要）。
接著以直播平台上的商品名稱及販售的直播主，運用上面建立的分類模型，預測直播平台上的商品分類，預測完成後，在原資料右方在新增兩個欄位，分別存放預測出來的大類及小類。

## 蝦皮資料集
共有 253,285 筆資料，大類有 27 類，小類有 715 類。
大類與小類有階層關係，如「女生包包/精品」又可分為中夾/短夾、手提包、手提斜背筆電包等等。

## 直播平台資料集
約有 1 萬 8 千多筆商品名稱，且沒有標註類別。

## 解題想法
- 使用 jieba 與 CKIPTagger 進型斷詞，觀察兩者差異
- 斷詞結果以空格分隔，以便做文字詞向量，並能提高模型準確率
- 分別預測大類、小類及大小類合併，比較結果的差別

## 流程

| 建立模型流程 | jieba 模型 | CKIP 模型 |
| -------- | -------- | -------- |
| 蝦皮資料抓取 | 從各個小類中抓取 50 筆資料，共 3,1600 筆 | 同左 |
| 蝦皮資料前處理 | - 去除各式括號內容 <br> - 移除停用詞：空格、標點符號、單位及與產品無關的詞、國家名稱）<br> - 處理非中文資料：判斷含有 t 的字詞是否與衣服相關（t 恤、素 t）、移除英文及數字 | 同左 |
| 蝦皮資料斷詞 | jieba | CKIPTagger |
| 蝦皮預測模型建立 | MultinomialNB 多項貝氏分類器 | 同左 |
| jambo 資料前處理 | - 去除各式括號內容 <br> - 移除停用詞：空格、標點符號、單位及與產品無關的詞<br> - 處理非中文資料：判斷含有 t 的字詞是否與衣服相關（t 恤、素 t）、移除英文及數字 | 同左 |
| jambo 資料斷詞 | jieba | CKIPTagger |
| 大類訓練準確率 | 0.7618 | 0.7833 |
| 小類訓練準確率 | 0.5427 | 0.5774 |
| 合併訓練準確率 | 0.5349 | 0.5706 |
| 大類測試準確率（計算方式：取前 150 筆人工判斷正確與否）| 0.66 | 0.6733 |
| 小類測試準確率（計算方式：取前 150 筆人工判斷正確與否）| 0.5 | 0.5333 |
| 合併測試準確率（計算方式：取前 150 筆人工判斷正確與否）| 0.4666 | 0.5133 |

## 結果分析
CKIP 的準確率皆略高於 jieba，我們認為是因為 CKIP 的斷詞結果**較為精確**。但詳細觀察結果後可以發現，因為斷詞結果不同的關係，有時候也會有 jieba 預測得較好的情形發生。

例子一：CKIP 有斷出「掛勾」所以有成功分類小類，jieba 則無。
jieba：
![](https://i.imgur.com/Xmwpz7F.png)
CKIP：
![](https://i.imgur.com/SgK0WFp.png)

例子二：jieba 有認出「托特 包」所以有成功分類，CKIP 則無。
jieba：
![](https://i.imgur.com/QhnttKK.png)
CKIP： 
![](https://i.imgur.com/ZEb8ok5.png)

在大類、小類及大小類合併預測的部分，大類的準確率皆高於其他二者，我們認為是因為大類的類別數較少；小類的類別數高達 600 多項，因此模型預測較不準確；合併預測的類別數與小類類別數相同，所以準確率也較低，且預測結果多與小類相同。不過也會發生因為未合併預測而導致大類分類錯誤的情形發生。

例子一：大類預測正確，小類預測錯誤，合併預測錯誤，小類、合併結果相同。
![](https://i.imgur.com/i7hAVW1.png)
![](https://i.imgur.com/zNpzdM8.png)

例子二：大類預測錯誤，小類預測正確，合併預測正確，小類、合併結果相同。
![](https://i.imgur.com/3FUN9Fp.png)

測試準確率透過抽取 150 筆資料後再由人工判斷來計算。計算結果，jieba 模型，150 筆資料中大類有 99 筆是正確的，大類測試準確率為 0.66；小類有 75 筆是正確的，小類測試準確率為 0.5；大小類合併有 70 筆是正確的，合併測試準確率為 0.4666。CKIP 模型，150 筆資料中大類有 101 筆是正確的，大類測試準確率為 0.6733；小類有 80 筆是正確的，小類測試準確率為 0.5333；大小類合併有 77 筆是正確的，合併測試準確率為 0.5133。