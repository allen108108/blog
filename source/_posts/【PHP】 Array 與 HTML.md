---
title: "【PHP】 Array 與 HTML"
date: 2020-09-14 23:03:31
categories:
- 程式設計 Programming
- PHP
image: https://i.imgur.com/ZWBPXbd.png
mathjax: true
---


PHP 之所以好用，一來是其與 HTML 的高度相容，可以在 HTML 中任意的地方插入 PHP 程式碼，另外一部分就是 PHP 作為 Web 應用的 Programming Language，它可以做為與資料庫的互動極為好用的工具之一。而這樣的特性，在 IOT 領域上獲得了非常多的應用。

<!-- more -->

Array 陣列
---

### Array 的型態

從 Python 的角度來看， PHP 的 array 可以視為`list列表`, `array陣列` 與 `dictionary字典` 的結合。它的型態可以是一維的線性型態、可以是二維的型態也可以是包含著 array 的 array 型態。PHP 中的陣列沒有一個很嚴謹的型態告訴使用者什麼叫做 array，也相對來說在應用上具有很高的彈性。以下是官方對於陣列的解釋 : 

> An array in PHP is actually an ordered map. A map is a type that associates values to keys. This type is optimized for several different uses; it can be treated as an array, list (vector), hash table (an implementation of a map), dictionary, collection, stack, queue, and probably more. As array values can be other arrays, trees and multidimensional arrays are also possible.
> 

以下是幾種陣列型態的舉例 : 

```php=
<?php
//一維陣列
$arr = array("Michael", "Jackson");
// output :
// Array ( [0] => Michael [1] => Jackson )
?>
```
上面的例子可以知道，其實即使是單一元素的陣列，它仍是一個有序的 Key-Value pair，只是以 index 做為 Key 而已。

```php=
<?php
//二維陣列
$arr = array("First_Name"=>"Michael", 
             "Last_Name"=>"Jackson");
// output :
// Array ( [First_Name] => Michael [Last_Name] => Jackson )
?>
```

```php=
<?php
//陣列包含陣列
$arr = array(
          "Idol_Name" =>array("First_Name"=>"Michael", 
                         "Last_Name"=>"Jackson"),
          "Idol_Album" =>array("Album_Name"=>"Dangerous", 
                         "Released"=>"November 26, 1991",
                         "Length"=>"77:03"),
          "Idol_Movie" =>array("Movie_Name"=>"Moonwalker", 
                         "Directed_By"=>"Jerry Kramer",
                         "Written_By"=>"David Newman",
                         "Distributed_By"=>"Warner Bros.")
);
// output :
// Array ( [Idol_Name] => Array ( [First_Name] => Michael 
//                                [Last_Name] => Jackson ) 
//         [Idol_Album] => Array ( [Album_Name] => Dangerous 
//                                 [Released] => November 26, 1991 
//                                 [Length] => 77:03 ) 
//         [Idol_Movie] => Array ( [Movie_Name] => Moonwalker 
//                                 [Directed_By] => Jerry Kramer 
//                                 [Written_By] => David Newman 
//                                 [Distributed_By] => Warner Bros. ) )
?>
```

從上面的例子可以發現 PHP 中 array 展現的多樣性，而這樣的彈性會為我們後面要介紹的資料庫存取有很大的關連性。


PS : 這裡我們先暫時不管 output 是怎麼出來的，留待後面的部分再進行介紹。但是當我們初次接觸 PHP 時 (尤其像筆者是從高階語言跳過來學習 PHP 的人)，要習慣在每一個語句的後面都要加上 `;` ，而且變數必須在前面加上 `$`，不然一定會報錯。 ( 不過跟其他語言一樣，如果會看得懂 Error message 就應該可以抓到問題點 )

### Array 的新增

PHP 作為使用者與後台資料庫的介接，不管是使用者輸入參數要藉由 PHP 來對資料庫撈資料，或是面對撈出來的資料要整理出來呈現給使用者，這中間都會使用到 array 來作為資料的型態進行傳遞。有鑑於上面所談到 array 的多樣結構，這對於結構性資料的呈現非常有幫助，因此我們必須要知道如何新增這些資料進 array 之中。

#### 單一元素的新增

單一元素的新增方法其實有蠻多的方法，比較正規的大概是利用 `array_push` function 來新增元素

`方法一`
```php=
<?php
$list = array("a", "b");
array_push($list, , "c", "d", "e");

print_r($list)

// output :
//Array([0] => a
//      [1] => b
//      [2] => c
//      [3] => d
//      [4] => e)
?>
```
但筆者在現實狀況下比較常用的是另一種新增方式，下面的方式如果遇到較多資料的新增，就會讓整個程式碼變得冗長，不過這樣的方式在運用的不同場景下會有更多的彈性 ( 例如 : 會需要利用迴圈來進行新增 )

`方法二`
```php=
<?php
$list = array("a", "b");
$list[] = "c";
$list[] = "d";
$list[] = "e";

// output :
//Array([0] => a
//      [1] => b
//      [2] => c
//      [3] => d
//      [4] => e)
?>
```
無論是方法一或二，都可以知道，新增單一元素其實也會自動新增 Key 值進去。如果原本的陣列就已經有其 Key 值，單純新增元素進去，Key 值會從 0 開始新增 

```php=
<?php
$list = array("A" => "a", 
              "B" => "b");
$list[] = "c";
$list[] = "d";
$list[] = "e";

// output :
//Array([A] => a
//      [B] => b
//      [0] => c
//      [1] => d
//      [2] => e)
?>
```


#### Key-Value pair 的新增

如果要新增一個 Key-Value pair，方法類似於 Python 字典的新增方式

```php=
<?php
$list = array("A" => "a", 
              "B" => "b");
$list["C"] = "c";
$list["D"] = "d";
$list["E"] = "e";

// output :
//Array([A] => a
//      [B] => b
//      [C] => c
//      [D] => d
//      [E] => e)
?>
```

### Array 的展示

在 PHP 中，要進行印出的工作，大多使用 `echo`, `var_dump`, `print`以及 `print_r`，當我們試圖要印出 array 時，建議使用 `var_dump` 以及 `print_r`。

當我們使用 `echo` 或 `print` 要印出 array 會遇到錯誤訊息

```
Notice:  Array to string conversion in <file_path> on <line_number> 
```

這個錯誤的原因是因為 `echo` 或 `print` 就是一個輸出字串的語言結構 (Language Structures)，我們可以說，它就是將看到的『字串』印出來而已，但 Array 是一個資料結構，並非字串符，當然會報錯。

利用 `print_r` 印出 Array，是一種比較簡潔的呈現方式，它只會將 Array 中的 Key-Value Pair 印出，正如上面所有例子中的 output 呈現方式

```php=
<?php
$list = array(0, 0.0, false, "");

print_r($list);
var_dump($list);

// output :
//Array([0] => 0
//      [1] => 0
//      [2] => 
//      [3] => )
//array(4) {[0]=>int(0)
//          [1]=>float(0)
//          [2]=>bool(false)
//          [3]=>string(0) ""}
?>
```
這裡比較要注意的部分是，`print_r` 對於某些型態的值是無法判別的，例如 `0` 與 `0.0` / `false` 與 `""`

```php=
<?php
$list = array("A" => "a", 
              "B" => "b");
$list["C"] = "c";
$list["D"] = "d";
$list["E"] = "e";

var_dump($list)

//output : 
//array(5) {
//  ["A"]=>
//  string(1) "a"
//  ["B"]=>
// string(1) "b"
//  ["C"]=>
//  string(1) "c"
//  ["D"]=>
//  string(1) "d"
//  ["E"]=>
//  string(1) "e"
//}
?>
```


`var_dump` 不僅呈現 Array 中的 Key-Value 關係，更會檢查所有值的轉存 (dump) 的型態

```php=
<?php
$list = array("A" => "a", 
              "B" => "b");
$list["C"] = "c";
$list["D"] = "d";
$list["E"] = "e";

var_dump($list)

//output : 
//array(5) {
//  ["A"]=>
//  string(1) "a"
//  ["B"]=>
// string(1) "b"
//  ["C"]=>
//  string(1) "c"
//  ["D"]=>
//  string(1) "d"
//  ["E"]=>
//  string(1) "e"
//}
?>
```
在 debug 階段，`var_dump` 往往是一個不錯的選擇，尤其當我們可能不清楚手上拿到的 Array 裡面元素的形態時。

Array Function
---

其實，筆者其實並沒有要詳細介紹所有的陣列函式，若讀者有興趣可以參閱 [PHP 官方文件 -- Array Functions](https://www.php.net/manual/en/ref.array.php)，裡面會介紹到許多的陣列函式及其應用方法。

這一段落，筆者想要介紹一個對於後續與網頁互動非常相關的函式 -- `array_keys` 與 `array_key_exists`


### HTTP Method : GET and POST 

筆者將以非常淺談的方式來介紹 GET 與 POST。

當我們利用 HTTP 要進行資料或是參數的傳送時，我們可以定義傳送這些資料的方法，如果我們定義以 GET Method 進行傳送，那麼傳送的資訊會以字串的方式利用 `?` 加在網址後面，並以 `&` 做區隔。

舉例來說，Google api 許多都是以類似的方法進行

```
https://maps.googleapis.com/maps/api/geocode/json？latlng=23.48386540,120.45358340
// ? 後面就是要查詢的經緯度
```

```
https://www.googleapis.com/youtube/v3/channels?part=contentDetails&id=UCMUnInmOkrWN4gof9KlhNmQ&key=Abccato
// ? 後面就是要查詢的影片頻道 ID 及使用者 KEY
```

相對於 GET，POST 傳送的資訊則是包在 message body 中以封包方式進行傳送。其實，也就是可見與不可見的差異，GET 方式會使要傳送的資料一覽無遺的呈現在網址中，而 POST 則較為隱密。

### `array_keys` /`array_key_exists`

搞清楚 HTTP 中傳送資料的方式後，接下來的問題是，PHP 要怎麼去知道使用者傳送了什麼資料 / 條件 / 參數 ? 我們必須先知道這些，才有辦法對後面的資料庫進行資料查詢。

如果今天是以 GET 方式傳遞，可以利用 `array_keys` 來得到參數的 Key-Value 結構，Key 指的是參數名稱，而 Value 則是其值。後續再以 `array_key_exists` 來進行參數判斷。

```php=
<?php
...
...
$arr_keys_in = array_keys($_GET); 
foreach ($arr_keys_in as $item){
    if (array_key_exists("latlng", $item)){
    ...
    }else (array_key_exists("address", $item)){
    ...
    }
}
...
...
?>
```
上面的方式便是我們可以判斷 Google api 中以經緯度或是地址來查詢地理資料分別可以進行不同的動作。


Array and Database
---

前面的部分，筆者從 Array 的基本介紹中，也帶出了許多 Array 在 HTTP 中的應用範疇，接下來，該是談談 Array 如何應用在 database 之中了。

### JSON

JSON ( JavaScript Object Notation，JavaScript物件表示法 )，是一種文本格式，之所以廣泛的應用於 HTTP 之中，最主要的原因是它具有結構性，對於存儲或是提取 SQL, MySQL 或 SQLite 這些關聯式資料庫十分適合。因此，利用 Array 作為 PHP 與 Database 之間的傳遞型態，但如果要跟 HTML 作互動，大多使用 JSON 格式來進行。

假如下列為 SQLite `Person.db` 資料庫中的某一個關聯表 `PersonTbl`

![](https://i.imgur.com/hze3kBt.png)

我們可以利用 SQLite PHP API 來進行讀取關聯表，並且存入陣列中 : 

```php=
<?php
$db = new SQLite3("./Person.db"); //讀取資料庫
$strSQL = "SELECT * FROM PersonTbl;" //對關聯表進行 Query
$results = $db->query($strSQL)

$list = array() //初始化陣列

//將每一個人的資料作為一個陣列添加進 $list array 之中
while ($row as DB_fetch_row($result)){
    $list[] = array("PersonID"=>$row["PersonID"],
                    "Name"=>$row["Name"],
                    "Age"=>$row["Age"],
                    "Gender"=>$row["Gender"],
                    "Profession"=>$row["Profession"]);
}

print_r($list);

//output : 
//Array([0] => Array([PersonID] => 90152
//                   [Name] => TuPac
//                   [Age] => 25
//                   [Gender] => Male
//                   [Profession] => Singer)
//
//      [1] => Array([PersonID] => 63254
//                   [Name] => Dreck
//                   [Age] => 32
//                   [Gender] => Male
//                   [Profession] => Teacher)
//
//      [2] => Array([PersonID] => 45369
//                   [Name] => Nicki
//                   [Age] => 28
//                   [Gender] => Female)
//
//      [3] => Array([PersonID] => 85624
//                   [Name] => Jay-Z
//                   [Age] => 40
//                   [Gender] => Male
//                   [Profession] => Driver))

?>
```
最後我們要轉換成 JSON 只需要利用 `json_encode` 就可以直接轉換

```php=
<?php
$res_obj = json_encode($list);
echo $res_obj;

//[
//    {"PersonID":"90152","Name":"TuPac","Age":"25","Gender":"Male","Profession":"Singer"},
//    {"PersonID":"63254","Name":"Dreck","Age":"32","Gender":"Male","Profession":"Teacher"},
//    {"PersonID":"45369","Name":"Nicki","Age":"28","Gender":"Female","Profession":"Artist"},
//    {"PersonID":"85624","Name":"Jay-Z","Age":"40","Gender":"Male","Profession":"Driver"}
//]
?>
```

後記
---

PHP 作為新手的第二 Programming Language 實在是不錯的選擇，其簡潔有力的寫法讓人可以很快的上手，另外它在網路上的廣泛應用，對於學習後端設計的新手來說也非常的有用。現在許多的網路應用 API 都會以 PHP 為架構進行編寫，希望筆者的文章可以幫助大家對 PHP 更有興趣，然後跟著筆者一起來學習吧~~~~~