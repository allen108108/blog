---
title: "【PHP】 require_once 與 __DIR__"
date: 2020-09-10 01:23:10
categories:
- 程式設計 Programming
- PHP
image: https://i.imgur.com/ZWBPXbd.png
mathjax: true
---

最近在進行 PHP 的一個開發項目，在測試時剛好遇到一個 Bug，測試了很久，始終找不到真正討人厭的那隻蟲，一直到最後回家後進行一些簡單的測試才知道問題出在哪裡。查了一下，發現其實不乏有人問相同的問題，因此想說隨手做個小短篇紀錄一下。

<!-- more -->

`require_once`
--

已經有了 Python 的基礎，要上手 PHP 並不算太難，我自己比較粗淺的理解就是 Python 中 `import` 的概念，或許在細節上的設計內涵有所不同，但大概念上就是一種「匯入」的概念。

在比較大型的專案中，通常會另外把用到的函式、特定的變數定義或是一些初始數值...等項目與真正的程式碼邏輯內容分開存放，這樣的好處一來是讓真正運作的部分顯得簡潔有力，二來就是當我們不同的程式碼檔案要用到同樣的函式、數值，就不用再重複寫入，這樣會讓整個開發效率快很多，當然，也讓後續維護的工作變得相對簡單。

這就是我們常聽到的『模組化』概念。

不管是 Python 中的 `import` ，還是 PHP 中的 `require_once` 都是方便我們進行模組化的絕佳工具。

### `require_once` / `require` / `include_once` / `include`


雖然筆者拿了 Python 的 `import` 來進行概念上的類比，但在實際的應用上， PHP 的 `require_once` 還是有些本質上的不同。最常見的就是 PHP 裡面類似的工具其實不只有 `require_once`，正如同標題上說到的，`require_once` / `require` / `include_once` / `include` 都有類似的功效，那究竟這中間有什麼差異呢 ?

### Differences

* `require` 在檔案引入的過程中，如果發生錯誤，會直接報 `fatal error` 且立刻中斷程式運作。
* `include` 在檔案引入的過程中，如果發生錯誤，會給予警告，但程式會繼續進行。
* `require` / `include` 在引入的過程中會檢查是否有重複引入的問題，若有，則會報錯。
* `require_once` / `include_once` 在引入的過程中若有重複引入的狀況，則會選擇不要重複引入。

上面的敘述大概可以掌握了其中的差異在哪邊，但是問題是，到底什麼情況要選擇什麼樣的工具呢 ? 筆者在這邊簡單的為大家列出幾個方向供讀者參考。

### Principles


* 絕大多數狀況建議使用 `require_once`。
* 匯入的模板或是檔案非必須，使用 `include`，若匯入的檔案極為重要，則建議使用 `require`。
* 極大型的專案，或是對於反應時間極為敏感的專案建議使用 `require` 來取代 `require_once`。

不過，在許多論壇及討論中，對於這些工具的使用時機仍然是爭論不休，筆者也僅是提供幾個大方向來給大家參考參考。

### 路徑問題

不管是 Python 還是 PHP，在檔案引入的時候，很常遇到環境、路徑上的問題導致檔案匯入發生問題，直接中斷程式運作，而這也是筆者在這次 PHP 的開發中遇到的問題。


![](https://i.imgur.com/JCxGfBi.png)

假設現在我們有一個檔案結構如上圖，`index.php` 要引用 `Utility_1.php` 及 `Utility_2.php`，而 `Utility_2.php` 也同時要引用 `Utility_1.php`，以下，筆者舉了一個簡單的例子 : 

`index.php`
```php=
<?php

require_once('./Utlis/Utility_1.php');
require_once('./Utlis/Utility_2.php');

$x = 12;
$y = 13;
$z = 2;

$sum = func($x, $y);
$result = multi($sum, $z);

echo($result)
?>
```


`Utility_1.php`
```php=
<?php
function func($a, $b){
    $res = multi($a, $b)+5;
    return $res;
}

?>
```



`Utility_2.php`
```php=
<?php
require_once(.'/Utility_1.php');

function multi($a, $b){
    $res = $a * $b;
    return $res;
}

?>
```

這樣的寫法導致筆者專案線上測試時，一直會出現 `500 Interal Server Error` 的錯誤，然而避開引用，將 `Utility_2.php` 的函數直接放進 `index.php` 運行則是沒有問題的。

看起來是引用 `Utility_2.php` 而導致報錯，而最後筆者利用上述例子使用 MAMP 來進行 debug 才確定了問題是出現在 `Utility_2.php` 中 `require_once` 的路徑設置出現問題。

當筆者利用上述例子，會得到這樣的錯誤

```php
Fatal error: require_once(): Failed opening required '/Utilis/Utility_1.php'
```
原來，當我們使用 `require_once` 進行引用時，就***意義上相當於把被引用的所有檔案置換進需要引用的檔案中***。

所以當我們用這種角度來看上面的例子，就會發現 `Utility_2.php` 的飲用路徑發生錯誤。最直覺的方法就是，我們依照要引用的檔案 (`index.php` ) 路徑來更改 : 

`Utility_2.php`
```php=
<?php
require_once(.'/Utilis/Utility_1.php');

...
...
...
?>
```

這樣的方法當然很直接，然而這樣的處理方式還是會有致命的錯誤，尤其是日後專案擴編、後續維護上都會有額外的副作用產生。

`__DIR__`
---

為什麼上面的改法會有問題呢 ? 上述的例子是非常簡單的範例，如果今天有很多不同層的檔案都要對 `Utility_2.php` 來做引用的話，我們根本不可能滿足所有檔案的，對吧 !!! 

也因次，上述的方法並不能根本性的解決問題，因此筆者建議使用 `__DIR__` 來設置路徑。簡單來說，`__DIR__` 可以***根據引用檔案的路徑來調整被引用的檔案的相對路徑***，換句話說，這是一種『動態』調整路徑的方法。

因此，我們要避免使用上述的方法來進行 `require_once` 路徑的設定，從舉例來看，我們應該這樣更改 : 


`Utility_2.php`
```php=
<?php
require_once(__DIR__.'/Utility_1.php');

...
...
...
?>
```


