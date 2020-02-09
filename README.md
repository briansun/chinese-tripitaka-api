# API for Chinese Tripitaka (deerpark.app)

[![CBETA version](https://img.shields.io/badge/CBETA-2019Q4-brightgreen.svg)](http://cbeta.org)

[「漢文大藏經」網站](https://deerpark.app)目前是由一個人在開發和維護，進度不會太快，有任何建議或意見請發到 brian@xmind.net 。謝謝支持！🙏

服務器目前放在 AWS Tokyo，運營成本由[「XMind 軟件公司」](https://www.xmind.net)供養。🙏

## 概念
本站的 API 看起來會像是 [CBETA API](https://cbdata.dila.edu.tw/v1.2/) 的一個簡化版本。

以下是一些概念：
* work：一部經或一部論，有的很長有600卷，有的只有一卷。
* juan：一卷。也就是一個 html 文件。
* lb：行號。在 html 文件中，會以 `<span class="lineInfo" line="0106c09"></span>` 的形式出現，一般只在每一段的開頭。


## All works
取回所有佛經的 meta data，目前有 4600+ 條。

### API:
```URL
https://deerpark.app/api/v1/allworks
```

### Sample Response:
除了 alias/alt 之外，其它信息是一定會有的。
```js
[
  {
    "id": "T0001", // 在 CBETA 中的編號，前一個或兩個字母表示藏經，具體可查閱 CBETA 文檔
    "title": "長阿含經",
    "byline": "後秦 佛陀耶舍共竺佛念譯", // 譯者或作者，有些是一人，有些是多人，有些是（失）
    "juans": [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22], // 大部分經是從1開始，有時候不是
    "chars": 198729, // 祛除標點符號後的字數

    // Optional
    "alias": "...", // 短名稱。大部分經沒有，少數有，如「法華經」
    "alt": "X0001 xxxxxx" // CBETA 其它編號和名稱。大部分經沒有，少數有，詳見 JB277。App 中不予以顯示。
  },
  ...
]
```


## Table of Contents
取回某部經典的目錄

### API:
```URL
https://deerpark.app/api/v1/toc/:id
```

### Sample Request:
```URL
https://deerpark.app/api/v1/toc/T0251
```

### Sample Response:
```js
{ // 返回兩個部分
  "juans": [ // 第一部分是分卷
    {
      "file": "T08n0251", // 用不上
      "lb": "0848c04",
      "juan": 1,
      "title": "第一"
    }
  ],
  "mulu": [ //第二部分是目錄
    {
      "indent": 1, // 縮進，1表示第一級，沒有0
      "title": "大明太祖高皇帝御製序",
      "juan": 1,
      "lb": "0848a03" // 行號
    },
    {
      "indent": 1,
      "title": "般若波羅蜜多心經序",
      "juan": 1,
      "lb": "0848b19"
    }
  ]
}
```


## Article in html
取回某卷經文的 html 版本。其中有少量的經文帶圖片，圖片是以 [Data URI](https://en.wikipedia.org/wiki/Data_URI_scheme) 的形式放在 html 裡的。

### API:
不帶卷號的 url 會自動轉向為該經的第一卷，如果沒有第一卷就往後遞推。
```URL
https://deerpark.app/api/v1/html/:id
https://deerpark.app/api/v1/html/:id/:juan
```

### Sample Request:
```URL
https://deerpark.app/api/v1/html/T0251
https://deerpark.app/api/v1/html/T0251/1
```

### Sample Response:
```html
<!doctype html>
<html lang="zh-TW">
<meta charset=utf-8>
...
<title>般若波羅蜜多心經</title>

<article>
...
</article>

</html>
```


## Download pdf/epub/mobi
最新更新：pdf/epub/mobi 已全部更新為最新版。今後應該能與 html 保持版本一致。

### API:
```URL
https://deerpark.app/api/v1/download/:type/:id
```

CBETA 提供了三種格式的文件下載，分別是：
* PDF （適合 A4 面幅打印）
* epub （開放格式，適合所有 epub 閱讀器，如 Apple Books）
* mobi （適合 Kindle 閱讀）

### Sample Request:
```URL
https://deerpark.app/api/v1/download/pdf/T0945
https://deerpark.app/api/v1/download/epub/T0945
https://deerpark.app/api/v1/download/mobi/T0945
```


## Full Text Search
搜索結果最多返回 100 個 items，目前還沒有做分頁，所以超過 100 個時，請增加搜索詞。

本 API 完全依賴 CBETA API 的 [kwic3](https://cbdata.dila.edu.tw/v1.2/static_pages/kwic3) 服務，所以有時會特別慢，有時結果中有錯，而且順序也有問題。此外，因為其它 API 如 allworks 是穩定在某個版本上，而 kwic3 服務會持續更新，極有可能不一致，有時會返回 allworks 中不存在的經典，也就是新整理的經典。（已被我去除，所以結果也可能少於 100 條，或少於 found。）

### API:
```URL
https://deerpark.app/api/v1/fullsearch/all/:term
https://deerpark.app/api/v1/fullsearch/:id/:term
```

### Sample Request:
```URL
https://deerpark.app/api/v1/fullsearch/all/一切有为法
https://deerpark.app/api/v1/fullsearch/T0251/般若
```

### Sample Response:
```js
{
  "found": 1006, // 一共有多少結果
  "results": [ // 最多 100 個結果
    {
      "id": "X0484",
      "title": "金剛經大意",
      "byline": "清 王起隆述",
      "total": 1, // 本經一共有多少卷
      "chars": 4819, // 本經一共有多少字
      "juan": 1, // 當前搜索結果是第幾卷
      "lb": "0362a01", // 行號
      "kwic": "。一重增勝。一重通為有相福德。較見性福德。實算數比喻不及。後四段。則佛隨順機緣。破遣凡夫依正報執。末一切有為法一偈。則總收全經。故不分正宗。不分流通。護念付囑。婆心痛切。庠序秩如似乎。貫華似乎。散華似乎。正智似" // 上下文
    },
    ...
  ]
}
```


## Reading lists

### API:
```URL
https://deerpark.app/api/v1/readinglist/:name
```

目前已有 4 個 reading list，分別是：
* 首頁常用經典（home）
* 更多常用經典（famous）
* 福嚴精舍三年閱經目錄（fuyan）
* 印順法師佛學著作集（yinshun）

### 閱讀列表有兩種格式：
1) 一種是數組，數組中的每個元素是一條數據。多數列表是這種格式。
```js
{ "items": [...] }
```
2) 另一種是分 category，每個 category 是一個數組。目前只有「福嚴精舍三年閱經目錄」是這種格式。
```js
{ 
  "items": [
    { 
      "title":"第一年·甲",
      "type": "category",
      "items":[...]
    },
    ...
  ]
}
```

### 單條數據有三種格式：
1) 整部經典。跟 allworks 中的格式一樣。
```js
{
  "id": "T0784",
  "title": "四十二章經",
  "byline": "後漢 迦葉摩騰共法蘭譯",
  "juans": [1],
  "chars": 2495
}
```
2) 某部經典中的一部分。
```js
{
  "special": true,
  "title": "華嚴經 十地品",
  "byline": [ // 不同於其它地方，這裡的 byline 是個數組
    "大方廣佛華嚴經 (八十華嚴)",
    "唐 實叉難陀譯",
    "CBETA T0279",
    "第34～39卷"
  ],
  "target": "T0279/34"
}
```
3) 外部網站連結。
```js
{
  "special": true,
  "external": true,
  "title": "學佛行儀",
  "byline": "善因法師述 | 外部網站",
  "target": "https://jnbooks.cn/search/學佛行儀"
}
```


### Sample Request:
```URL
https://deerpark.app/api/v1/readinglist/home
https://deerpark.app/api/v1/readinglist/famous
https://deerpark.app/api/v1/readinglist/fuyan
https://deerpark.app/api/v1/readinglist/yinshun
```

### Sample Response (home):
```js
{
  "items": [
    {
      "id": "T0784",
      "title": "四十二章經",
      "byline": "後漢 迦葉摩騰共法蘭譯",
      "juans": [1],
      "juan": 1,
      "chars": 2495
    },
    ...
  ]
}
```

# API for Buddhist Dictionary (New)

新增佛學辭典 API，內含五部辭典，分別是《陳義孝佛學常見辭彙》，《法相辭典》，《三藏法數》，《丁福保佛學大辭典》，《佛光大辭典》。

## Search Suggestion

### API:
```URL
https://deerpark.app/api/v1/dict/suggest/:term
```

### Sample Request:
如果用戶輸入的是簡體字，API 會自動轉為繁體。如果該簡體對應多個繁體，我們會盡量保證多個版本都有效。

```URL
https://deerpark.app/api/v1/dict/suggest/如來藏
```

最好在請求的關鍵詞中，只出現中文，不出現西文字符。 (Best Practice)

### Sample Response:
首先出現在結果中的，是和關鍵詞一模一樣的詞條，其次是以關鍵詞為首的詞條，最後是包含關鍵詞的詞條。最多返回 50 個結果。

```js
[
  "如來藏",
  "如來藏心",
  "如來藏緣起",
  "如來藏性",
  "如來藏經",
  "如來藏經十喻",
  "如來藏論",
  "一切有情是如來藏等",
  "三如來藏",
  "不空如來藏",
  "九喻──如來藏九喻",
  "二如來藏",
  "空如來藏",
  "大方廣如來藏經",
  "大方等如來藏經",
  "十種如來藏",
  "隱沒如來藏"
]
```

## Lookup

### API:
```URL
https://deerpark.app/api/v1/dict/lookup/:term
```
（PS. 在尾部加 `?html` 可以得到簡易版網頁）

### Sample Request:
注意：搜索詞必須是精確的，否則會 404。這裡不提供簡繁轉換。所以最好使用上面 Search Suggestion API 提供的結果。

```URL
https://deerpark.app/api/v1/dict/lookup/如來藏
```

### Sample Response:
```js
{
    "word": "如來藏", // 搜索詞
    "data": [ // 不超過 5 個結果，因為只有五部辭典，請勿依賴其順序
        {
            "dictid": 0,
            "dict": "陳義孝佛學常見辭彙", // 辭典名稱
            "expl": "<p>真如在煩惱中，攝藏如來一切果地上的功德，名如來藏，若出了煩惱，即名法身。</p>"
        },
        ...
    ]
}
```
如果 dictid = 3 （丁福保佛學大辭典），expl 會出現：
* `<a href="bword://如來藏">如來藏</a>` （此處可按需要將 bword:// 替換為真實連結）
