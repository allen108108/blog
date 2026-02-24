---
title: "Android 初探 (二) ： 從 Hi, Android ! 來認識 Activity 與 UI"
date: 2020-03-02 15:35:14
categories:
- 系統相關 OS
- Android
image: https://i.imgur.com/7YeW7Qb.png
mathjax: true
---

前言
---

上一篇文章 " [Android 初探 (一) ： Android 初探 (一) ： 從 Hello  World ! 認識 Android 專案開發](https://bit.ly/3bouPaB) " 簡單介紹了一個 Android 專案的主要架構以及如何在模擬器上 run 出整個結果，這一篇會著重在一個 Android app是怎麼運作的，以及可以怎麼簡單設計使用者介面。

<!-- more -->

這兩篇文章主要都是以 [Android Developers Documentation](https://developer.android.com/docs) 為主，另外補充一些資料進來，如果已有紮實的基礎，或許可以直接跟著 Android Developers Documentation 進行也是不錯的方向。

Activity
---

Android apps 基本上可以說是由各種可被獨立調用的不同部件所組合而成，其中一種與使用者最直接相關的就是 `Activity`。

我們可以將 `Activity` 視為電腦中「視窗」( window ) 的概念，一般來說，當你按下 Android app 圖示時會跳出一個 `main activity` 作為主畫面，在這個 `main activity` 視窗中，我們可以藉由一些操作連結到其他的 `Activity`，每一個 `Activity` 都會有一個新的「視窗」顯示在使用者螢幕上。又因為每一個 `Activity` 都是獨立的，因此我們可以跨 app 執行 `Activity`。

舉例來說，當我們點開一個 Email app，出現的第一個畫面即為`main activity`，然後我們可以進行特定的操作，每一個操作 (寫信、寄信、...) 都是 `Ａctivity`，而寫、寄信也可以從別的 app 中連結過來。



User Interface
---

使用者介面 (UI, Use Interface) 顧名思義就是使用者在使用這個 app 時可以見到的所有東西都是 UI 的範疇，其中包括了可以跟 app 互動的方式。延續上個部分將 `Activity` 是為視窗的概念，那麼 UI 則是每一個視窗內的整體配置 (layout)。

在 Android Studio 裡面，我們可以在 `layout` 資料夾中每一個 `Activity` xml 檔案中進行配置。

Veiw & ViewGroup
---

剛才提到將 `Activity` 視為「視窗」，但是必須要了解，視窗本身是不會對使用者顯示任何訊息的，真正顯示在「視窗」上面的是現在即將提到的 `View` ，而在 Android 中，UI 是建立在 `View` 與 `ViewGroup` 之上。

![](https://i.imgur.com/kFA6Ybj.png)


`View` 是所有 UI 的組件的基礎，不管是畫面上看到的文字部分 (`TextView`)、按鈕(`Button`)、可以輸入文字的部分 (`EditView`)、．．．等等都是在 `View` 底下實現，一個`View`就會佔據渲染並處理畫面中的某一個區域所發生的事件。`ViewGroup` 也是歸類在 `View` 底下，其作用在容納並管理下一層的 `View`。

總的來說，`Activity`、`View`與 windows的關係可以這樣解釋 ： 不可視的`Activity` 必須經由 windows 來對可視的 `View` 做管理，而`View` 需要利用 windows 來展示於 `Activity` 上。


從 Hello World 變成 Hi Android 
---

簡單了解 Android UI 的運作後，我們可以實際應用在專案上。

`app > res > layout > activity_main.xml` 存放的就是前面提到的 `main activity`，現在裡面應該只有 "Hello World!" 的字樣。

在 `activity_main.xml` 中我們選擇 `design` 後，我們可以從 Android Studio 看到以下的區塊

![](https://i.imgur.com/3iAgWFW.png)

`Palette` 存放了我們可以配置在畫面中的所有元組件，`Component Tree` 顯示了整個畫面中元件的階層結構，中間的 `Design Editor` 呈現了元件配置的視覺效果，最右邊的 `Attributes` 可以調整每一個元件的細節特性。

### 添加元件並設定固定間距

先將原本開啟新專案時「附贈」給我們的 Hello World 從 `Component Tree` 刪除，在這個步驟，我們目標是加入一個可以輸入文字的 `EditText` 以及可以送出文字的 `Bottun`，且間距固定為 16dp。

#### 關閉自動約束 (Autoconnect)，設定預設間距

在 `Design Editor` 上方有一個馬蹄鐵形狀的圖示，如果有一條斜線表示已經開啟自動約束，自動約束的意義在於，當我們將元件放置在版面中，系統就會自動加上約束。不過在這裡我們希望可以自行手動設置約束，因此先將其關閉 ( 沒有斜線的馬蹄鐵圖示 )。

再來將馬蹄鐵圖示旁邊的 `Default Margin` 設定為 16dp。

我們將 `Palette > Text > Plain Text` 拖曳至 `Design Editor` 



<img width=500 src="https://i.imgur.com/G9udy32.png" >




我們可以看見每一個頂點上會有一個方形的 handler 可以調整其大小，然後每一條邊上也有一個圓形的錨點是用來進行約束的。

我們將上面及左邊的錨點各自往上面及左邊的 layout 邊緣拉過去即會產生約束。 這個約束也就是我們剛才設定的 default margin 16dp。


<img width=500 src="https://i.imgur.com/ONwhkTZ.png" > 




利用相同的方法我們可以拖曳一個 `Button` 到輸入框的右邊，並且讓 `Button` 右側與 `EditText` 左側形成約束，間距一樣是 16dp。此外，在 `Button` 按下滑鼠右鍵選擇 `Show Baseline` 即會出現 Baseline Bar，拖曳此 Baseline Bar 到輸入框就可以讓兩筐進行 Baseline 水平對齊。

![](https://i.imgur.com/hw3WrV2.png)

#### 定義字串

在 Android 專案中，將每個元件所使用的字串通通定義在 `app > res > values > strings.xml` 中，這樣的好處是未來如果要進行 app 字串的更改，只要更改從這裡就可以進行統一更改，不用耗時去尋找應該更改的地方。

在上述檔案中右上方在上述檔案中右上方按下 `Open editor` 

![](https://i.imgur.com/diwgXEG.png)


在按下左上方的『＋』號即可進行字串的新增

![](https://i.imgur.com/RlthrqR.png)


首先輸入 `Key` 為 `edit_name`，而 `default value` (會顯示在視窗上) 為 `Enter a name ~`，按下 OK 後即新增一個新的字串資料


<img width=500 src="https://i.imgur.com/qLxsVk6.png" > 




利用同樣方法新增一個 `Key` 為 `button_send`，而 `default value` 為 `Send` 的字串資料。新增完成之後，我們就可以在剛剛設置的 layout 上面進行這些字串的配置。

選取 `EditText` 後，至右側的 `Attributes > text` 刪除其值 "Name"，再至 `Attributes > hint` 按下右側長條選擇我們剛剛新增的字串 `edit_name`


<img width=500 src="https://i.imgur.com/kheJjIg.png" > 



同樣的方式我們可以選擇 `Button` 的 `Attributes > text` 之值為剛剛新增的 `button_send` 字串。


#### 利用 chain 進行配置

`chain` 的功用在於將多個 `view` 進行剩餘空間的平均分配

![](https://i.imgur.com/j0kkrQe.gif)

在建造 `chain` 時並不會影響原來的約束，所以由上圖可以看到原本左右的約束仍在。我們利用相同的 `horizontal chain` 來對 `EditText` 與 `Button` 最水平方向的分配。



![](https://i.imgur.com/6mRX0XB.png)



我們再經由 `Attributes > Layout` 面板來紅框處調整為 `Match Constraints` 強制將 `EditText` 左側約束均維持在 16dp。

`Button` 右側也加上 16dp 的約束，有趣的地方是，如果將紅框處一樣設定為 `Match Constraints` 那麼 `EditText` 與 `Button` 的寬度會變得一樣寬。( 這樣的版面配置實在不是太好看，因此設定在 `Wrap Content` 即可 )


<img width=500 src="https://i.imgur.com/7rFWhVT.png" > 




而這也表示整個文字框寬度會變寬。

![](https://i.imgur.com/Gt764WQ.png)

### Activity 設定

#### Main Activity 定義操作

我們上面的種種動作，都僅只是在對整個畫面作配置，然而這樣並不能讓整個 app 作動，要完成這樣的工作，就必須要做 Activity 的設置。

首先，我們先針對 `main activity` 來做設定，至 `app > java > com.example.myfirstapp > MainActivity`


```java=
public class MainActivity extends AppCompatActivity {
    //加入 EXTRA_MESSAGE constant//
    public static final String EXTRA_MESSAGE = "com.example.myfirstandroidapp.MESSAGE";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    //定義當我們按下 Button 鍵後的行為//
    public void sendMessage(View view) {
        //意圖 (intent) 連結 main activity (this) 與另一個新的 activity (DisplayMessageActivity.class)//
        Intent intent = new Intent(this, DisplayMessageActivity.class);
        //定義 editText 是什麼//
        EditText editText = findViewById(R.id.editText);
        //從 editText 抓出我們輸入的訊息並定義其為 message//
        String message = editText.getText().toString();
        //將 message 加入 intent 之中//
        intent.putExtra(EXTRA_MESSAGE, message);
        startActivity(intent);
    }
}
```

當你加入這些程式碼之後，會有一些地方出現紅字報錯

![](https://i.imgur.com/JogDUW9.png)


絕大多數都是使用到了未知的符號，這時候可以使用 `Alt+Enter` 來對這些部分進行 `import`，算是蠻方便的做法。 

![](https://i.imgur.com/rBJqMEq.png)

不過，`DisplayMessageActivity` 的 error 比較不一樣，我們使用 `Alt+Enter`  時並不會出現 `import class` 的選項，最主要的原因是因為我們還沒有一個對應的 `Activity` ，這個將是接下來要完成的工作。

當我們定義了`Button`鍵按下以後的行為後，我們要回到 `activity_main.xml` ，從 `Attribute > onClick` 選擇 `sendMessage`，便可將這個行為加到 `Button` 中


#### 開啟新 Activity

左側的專案面板上，於 `app` 資料夾上按滑鼠右鍵，選擇 `New > Activity > Empty Activity` 來新增新的 `Activity`

![](https://i.imgur.com/9ZZVLmV.jpg)

之後將新的 `Activity` 命名為 `DisplayMessageActivity`，其餘均使用預設設定。

新增 `DisplayMessageActivity` 後整個專案有三個部份會有更動

1. `manifests > AndroidManifest.xml` 裡面會有新的 `activity` 元素
2. `java > com.example.myfirstandroidapp` 內會新增一個 `DisplayMessageActivity.xml` 檔
3. `res > layout` 內會新增一個 `activity_display_message.xml` 檔

![](https://i.imgur.com/Xphzd2T.png)

#### 配置新的視窗

前面我們針對 `main activity` 進行視窗的配置了，現在要進行 `DisplayMessageActivity` 的視窗配置。

`app > res > layout > activity_display_message.xml` 底下，先選擇 `Enable Autoconnection to Parent` (有斜線的馬蹄鐵圖示)，這樣可以確保系統可以自動約束。

然後從 `Palette` 面板中拖曳 `TextView` 到 `Design Editor` 中，由於我們開啟了自動約束，因此這個新增的 `View` 會自行約束，讓它水平置中。

但要注意的是，上方還是要利用錨點來進行 margin 16dp 的約束。

![](https://i.imgur.com/1XH0PBj.jpg)

#### New Activity 定義操作

至 `java > com.example.myfirstandroidapp > DisplayMessageActivity.xml` 中加入以下的程式碼

```java=
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_display_message);
    
    //接收到新的 intent 後取出收到的 message //
    Intent intent = getIntent();
    String message = intent.getStringExtra(MainActivity.EXTRA_MESSAGE);

    //
    //將收到的 message 放到 View 中//
    TextView textView = findViewById(R.id.textView);
    textView.setText("Hi, " + message);
}
```

遇到紅色報錯仍然使用 `Alt+Enter` 進行除錯。

### 設定導向

在 Android app 使用中，當我們進行一個動作，開啟一個新的視窗後，必須要讓使用者有能夠回到上一頁的導向方式，因此我們要針對各個 `Activity` 進行設定，要讓系統之道上一頁應該回到哪一個頁面、哪一個 `Activity`

至 `manifests > AndroidManifest.xml` 中，新增的 `<activity>`  標籤進行修改

```java=
<activity android:name=".DisplayMessageActivity"
          android:parentActivityName=".MainActivity">
    <!-- The meta-data tag is required if you support API level 15 and lower -->
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity" />
</activity>
```

這樣就可以將頁面導回至 `main activity` ，我們也完成了最後一個步驟了。


### Hi, Android 


![](https://i.imgur.com/rHt0iU3.png)


後記
---

這一個部份如果沒有基礎如我，其實會看得很吃力，不過多看幾次、多嘗試幾次後，其實整個運作過程會越來越清晰。

每一個使用者看到的視窗，都有一個 xml 檔在 `layout` 資料夾中，專責畫面的配置以及動作的連結，另外會有一個 xml 檔在 `com.example.<project_name>` 資料夾中負責定義每一個動作的細節。

了解這樣的架構後，回過頭來看上面的種種步驟就會顯得簡單明瞭不少。




參考資料
---

1. [Android Developers Documentation](https://developer.android.com/docs)
2. [[译] ConstraintLayout基础系列之Chains链](https://biaomingzhong.github.io/2017/constraintlayout-basics-chains-2/)

系列文章
---

* [Android 初探 (一) ： Android 初探 (一) ： 從 Hello  World ! 認識 Android 專案開發](https://bit.ly/3bouPaB)
* [Android 初探 (二) ： 從 Hi, Android ! 了解 Activity 與 UI](https://bit.ly/2vOoFlb)

