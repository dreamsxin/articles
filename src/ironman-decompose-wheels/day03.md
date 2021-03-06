# Carbon（2）－－繼承並不萬惡

Carbon 本身並不複雜，它使用兩個物件，分別繼承了原生 PHP [`DateTime`][] 與 [`DateInterval`][] 類別，並實作了新的行為，讓它更好使用。

以下會翻 [Carbon 1.22.1](https://github.com/briannesbitt/Carbon/tree/1.22.1) 版來說做明。

## 繼承如何實作才安全

在學 Design Pattern 時，常會聽到要「多用組合，少用繼承」。繼承這麼可怕，怎麼至今大多數語言都支援呢？這表示，繼承雖然有風險，但能避開風險的話，它仍然是個好用的觀念。像 Carbon 就是一個很好的例子，使用繼承擴展功能後，反而受到大多數開發者的喜愛。

我們這幾天可以一起來看看 Carbon 怎麼安全地實作繼承。

### 繼承的潛在風險

物件導向設計原則中，其中有一個原則是－－[里氏替換原則][]，身為一個子類，如果要繼承家業的話，必須要把父類原本做的事做好才行，就算有想要改善或是調整，也不能破壞行為。

所以，首先我們來看 `Carbon` 類別繼承了 [`DateTime`][] 哪些實作，來了解它是改善調整，還是破壞行為。

使用 IDE 可以很清楚知道下面這些方法有做覆寫：

```php
public function __construct($time = null, $tz = null)
public static function createFromFormat($format, $time, $tz = null)
public static function getLastErrors()
public function setDate($year, $month, $day)
public function setTimezone($value)
public function modify($modify)
```

以下來看看這些方法到底做了哪些事：

#### `__construct`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L271-L292)

這裡可以注意到，建構子中間多了 [`if` 判斷](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L275)；後面在傳 `$tz` 前，還有做一層[手腳](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L291)，把這兩部分拿掉的話，就跟原本的 `DateTime` 完全一樣了。
 
其中，`if` 判斷主要的任務是為了在測試階段時，要把「現在」替換成指定的時間點。而指定的時間是放在靜態變數裡，建構時再去取得靜態變數（[`getTestNow()`](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L1045-L1048)）。

這要怎麼用呢？比方說，我們要測試跨年前 10 秒會不會自動啟動煙火機制，直接使用 `date` 指令調電腦時間實在是太蠢了，來看看 Carbon 怎麼做：

```php
$realNow = new Carbon();
echo "Real: $realNow\n";

Carbon::setTestNow('2017-12-31 23:59:50');

$mockNow = new Carbon();
echo "Mock: $mockNow\n";
```

輸出結果如下：

```
Real: 2017-12-22 18:51:32
Mock: 2017-12-31 23:59:50
```

這還有另一個更顯著的好處：測試時間再怎麼長，任何時間點拿的 `now` 都會是同一個時間。

```php
$realNow = new Carbon();
echo "Real: $realNow\n";

Carbon::setTestNow('2017-12-31 23:59:50');

$mockNow = new Carbon();
echo "Mock: $mockNow\n";

sleep(1);

$mockNow = new Carbon();
echo "Mock: $mockNow\n";

Carbon::setTestNow();

$cleanNow = new Carbon();
echo "Real: $cleanNow\n";
```

輸出結果如下：

```
Real: 2017-12-22 18:55:44
Mock: 2017-12-31 23:59:50
Mock: 2017-12-31 23:59:50
Real: 2017-12-22 18:55:45
```

如果程式需要依賴「現在」的話，將是非常好用的功能。

然而，它的啟動條件是先做設定「現在時間」（`Carbon::setTestNow()`），啟動前並不影響任何行為；啟動後則是位移時間，最終還是會傳正確 `$time` 格式給父類別。

[`safeCreateDateTimeZone()`](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L228-L256) 則是在做正規化 [`DateTimeZone`](http://php.net/manual/en/class.datetimezone.php) 和一些 TimeZone 格式錯誤時的錯誤處理，而且避開了 [Bug #52063](https://bugs.php.net/bug.php?id=52063)。

因此這兩個功能都有加強原建構子的功能，並沒有破壞行為。

#### `createFromFormat`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L568-L583)

開頭的 `if` 判斷和正規化 `DateTimeZone`，與建構子 `safeCreateDateTimeZone()` 在做的事類似。

這邊會執行父類別建立 DateTime 的方法，接著 `setLastErrors()` 是存放建立時遇到的錯誤（`parent::getLastErrors()`）。會這麼做的理由在最後面：因為 Carbon 設計這個 function 預期錯誤會丟例外，而不是 DataTime 回傳 `false`。

如果是丟例外的話，需要有個地方取得錯誤訊息。是的，所以需要覆寫 `getLastErrors`，來取得剛剛呼叫 `setLastErrors()` 時傳入的 `parent::getLastErrors()`。這些過程有點繞，總之，Carbon 的目的是為了要把它改成「錯誤丟例外」。

如果 DateTime 成功建立，則會使用 `instance()` 轉換成 Carbon，再回傳出去。

原則上，這是一個工廠方法，所以回傳的物件應該會是 Class 本身，因此行為有點不同（回傳的是 Carbon 而不是 DateTime），但使用上並不會有任何影響。

#### `getLastErrors`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L600-L603)很單純，會這樣寫的理由請參考 `createFromFormat` 覆寫的原因。

實際會使用到的時機是在接 `createFromFormat` 方法所丟出的例外：

```php
try {
    $carbon = Carbon::createFromFormat('Y/m/d', 'unknown');
} catch (Exception $e) {
    echo $e->getMessage();
}
```

輸出如下：

```
A four digit year could not be found
Data missing
```

#### `setDate`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L859-L864)註解有提到，這裡是為了 workaround 修[這個 Bug](https://bugs.php.net/bug.php?id=63863)

#### `setTimezone`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L944-L947)

正規化傳入的 Timezone，減少問題發生的機率。

#### `modify`

[原始碼](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/Carbon.php#L3312-L3324)

翻了一下 commit 記錄與 [issue](https://github.com/briannesbitt/Carbon/issues/88)，這是為了修 DateTime 的 bug。

另外還有一個 `CarbonInterval` 類別繼承 [`DateInterval`][]。它只有覆寫 [`__construct`](https://github.com/briannesbitt/Carbon/blob/1.22.1/src/Carbon/CarbonInterval.php#L120-L146) 的實作，換言之，它只改變了建立物件的方法，其他的行為都沒有改變。

## 行為不變最安全

由上面的分析看來，Carbon 並沒有改變原本物件的行為，因此我們甚至可以拿 Carbon 來取代任何需要使用 DateTime 的方法，達到里氏替換原則的精髓！

---

今天翻過它跟父類別，也就是跟原有功能有相關的程式，明天來看看擴充的功能有哪些，而這也正是 Carbon 吸引人的地方！

[DateTime]: http://php.net/manual/en/class.datetime.php
[DateInterval]: http://php.net/manual/en/class.dateinterval.php
[里氏替換原則]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day09.md
