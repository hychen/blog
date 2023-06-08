+++
title = """
  如何解決 org-helm-bibtext 跳出 "scan-error "Unbalanced parentheses" 的錯誤？
  """
publishDate = 2022-05-20
tags = ["Emacs"]
categories = ["問題與解法"]
draft = false
+++

在 Emacs 中可以用 [org-ref](https://github.com/jkitchin/org-ref) 跟 [org-helm-bibtext](https://github.com/tmalsburg/helm-bibtex) 搜尋、引用文獻，並且可以直接導入 [Zotero](https://www.zotero.org/) 輸出的 [Bibtex](https://www.bibtex.org/) 檔案，使用起來相當方便，但偶而我們會遇到 Bibtex  檔案解析失敗，跳出 `scan-error "scan-error "Unbalanced parentheses" 195830 512550"` 類似的錯誤訊息，這代表什麼問題呢？本文簡略說明問題成因跟解法。

通常會出現這狀況，通常是
Bibtex 檔案裡面的內容有不匹配的括號，導致 Regex 的解析錯誤。解決方式有兩種，一種是用 debug mode 啟動 Emacs，看 backtrace 是哪一個檔案導致的問題，另一種是打開 biblatex檔，透過 bibtext-validate 除錯。


## 第一種方式 {#第一種方式}

首先用下面令查看 backgrace， 看是哪一個解析哪一的檔案導致的問題。

```sh
emacs --debug-init
```

根據錯誤訊息 `scan-error "scan-error "Unbalanced parentheses" 195830 512550"` ，我們知道出錯的字元位置是在 **195830** ，所以找到是哪一檔案後，直接用 `goto-char` (M-g c) 跳到出錯的地方看是怎樣的內容導致錯誤：

```lisp
goto-char 195830
```

執行後發現是因為 title 的 「自主學習-如何成為未來職場需要的自由人材(上）」 個字元是 `(` 跟 `）` 一個是半形，一個是全形，導致括號沒有正確匹配，將 ( 改成全形 `（` 的即可。

```bibtex
@video{bestianZiZhuXueXiRuHeChengWeiWeiLaiZhiChangXuYaoDeZiYouRenCaiShang2012,
  title = {自主學習-如何成為未來職場需要的自由人材(上）},
  editor = {{Bestian}},
  date = {2012-12-27},
  url = {https://www.youtube.com/watch?v=ZPzqw3x05-g},
  urldate = {2022-04-05},
  abstract = {自主學習促進會-遠距系列講座之四：如何成為未來職場需要的自由人才 中集網址：http://www.youtube.com/watch?v=Yc-fHn... 下集網址：http://www.youtube.com/watch?v=IqQTLQ... 自學出身和本科系出身的人材有無不同？  面試錄取者和未錄取的主要差異是？  軟體設計在學校學習有用嗎？還有哪些管道？ 面對大的挑戰時，如何避免自己嚇自己？如何評估挑戰的價值？ 如何面對理想與現實的拉扯？ 如何落實創意生活？在經常移動的工作中維持安定感與家庭相處品質？ 如何改變習慣？如何做出特出的事？ 如何決定是否要換工作？時間點該怎麼抓？ 如果以上正是您的疑問，本講座就是為您準備的心靈禮物。  本次講座由資深自由軟體工作者唐鳳(Audrey Tang)主講，天水工作室製作。},
  editortype = {director},
  keywords = {@DONE}
}
```

除這以外，還有另一種出錯，就是《 、 `「` 、 `{` 這類兩兩一對的括號都不能單獨存在，否則也會出現匹配錯誤，出錯的例子如下：

```bibtex
@online{k3vin-wang.ethWebGTMCeLue2022,
  title = {Web 3.0 {{GTM}} 策略：新思維、策略、指標},
  author = {family=wang.eth, prefix=k3vin-, useprefix=true},
  date = {2022-03-26T23:41:19},
  url = {https://medium.com/digital-discovery/web-3-0-gtm-%E7%AD%96%E7%95%A5-%E6%96%B0%E6%80%9D%E7%B6%AD-%E7%AD%96%E7%95%A5-%E6%8C%87%E6%A8%99-66dabd819d34},
  urldate = {2022-05-01},
  abstract = {新創通常產出的項目和市場上的成熟的項目相較之下的不確定性會比較大，因此一個新創如何解決 『冷啟動問題 Cold Start Problem …},
  langid = {english},
  organization = {{Digital Discovery}},
}
```

這邊可以看到，簡介內的文字裡的 `『冷啟動問題` 的 `「` 單獨存在，導致語法錯誤。


## 第二種方式 {#第二種方式}

如果已經懷疑 "scan-error" 是因為 bibtext 檔案的格式出錯，bibtext 本身提供 lint 工具，其實可以用該命令直接檢查。執行該指令後，會直接顯示哪一行出現錯誤。

```lisp
bibtext-validate
```

惟須注意的是，有時候 bibtext-validate 也抓不到左括號右掛號不匹配的問題，我多是仰賴第一種方式進行。


## 小結 {#小結}

本篇文章說明了使用 org-helm-bibtext 常見的問題，以及兩種解決方式。
