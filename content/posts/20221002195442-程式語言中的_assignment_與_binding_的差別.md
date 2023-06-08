+++
title = "程式語言中的 Assignment 與 Binding 的差別"
publishDate = 2016-04-04
categories = ["觀想"]
draft = false
+++

所謂的 Functional Language 的變數不可改其實是不精確的。實際上它只是將變數分的更嚴謹，可修改，與不可修改。 先來說說什麼是 Binding。


## Binding {#binding}

Binding 指的是將一個符號綁定在一個值，綁定後就不可修改，這個符號稱之為變數。以自然語言來理解的話，其實就是代名詞。

> favonia: 一個變數沒有指到一個值。一個變數可以被一個值取代，但不會儲存什麼內容在裡面。

所以說下面這個例子， 必須理解為3.14的另一個稱呼叫做 `PI`, 而不是 `PI` 這個容器的值是 3.14。而 `1 + PI` 這個式子的另一個稱呼是 result, 而不是 `1 + PI` 運算後，將值放到 `result` 這個容器裡。 不管是 `PI` ，還是 `result` 都只是個代稱，而不是容器。

```sml
PI = 3.14
result = 1.0 + PI
```

接下來搭配 SML 來解釋什麼是 Assignment。


## Assignment {#assignment}

下面這段源碼是說 `PI` 是 `3.14` 的 Binding, 而 `result` 是 `1 + PI` 的 Binding。程式執行時（runtime） 會把 `PI` 這個變數代換成 `3.14` 。

```sml
val PI = 3.14
val result = 1.0 + PI
```

那如果需要可修改的變數怎麼辦，SML 可以讓變數綁在值的的指標上, `ref 3.14` 指的是 `3.14` 這個值得指標指標。而因為指標指定的值是可以變的，所以現在 `PI` 可以當成是可以修改的。 `PI := 4` 只是讓 `PI` 的值（也就是指標） 從 `3.14` 指到 `4` ，並不會產生新的 `Binding` 。

```sml
val PI = ref 3.14 # binding
val result = 1.0 + PI # binding
val PI := 4 # assign
```

因此，在所有的 Imperative Language 裡，不管是 Python, Ruby, C, Java, 變數其實都是被綁在指標上，這樣變數指到的值才能被修改。


## 結論 {#結論}

所以說，變數只是個值的代稱，如果值是指標，那麼就可以修改指標要指到哪個 值，反之則不行。但不管怎麼說，變數都不應該被當成是一個容器。
