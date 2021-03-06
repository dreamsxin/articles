# 簡介

未來三十天的過程，會帶著讀者一起分析一個開發成熟的原始碼。筆者會假設讀者具備下列基礎知識，以方便未來說明分析的過程：

* PHP 語言基礎，如變數、類別、trait 宣告與操作等
* Composer 使用，如安裝套件、定義自動載入等
* 使用過需配合 Apache rewrite 的 PHP MVC 框架任一種，如 [CodeIgniter][] 等

## 專有名詞對照表

因許多專有名詞都是英文為主，但大部分的專有名詞，中文都有通用翻譯。如果描述需要參照原始碼，只能用英文，因此為了避免中英混雜的情況太嚴重，未來的文章會盡可能把這些常見的專有名詞轉換成中文。

| 英文 | 中文 |
|---|---|
| variable | 變數 |
| function | 函數 |
| class | 類別 |
| method | 方法 |
| property | 屬性 |
| construct | 建構子 |
| instance | 實例 |
| abstract | 抽象 |

## 討論範圍

一般提到 Laravel，可能會想到的是它的[首頁][Laravel]，以及上面打的那句話：

> The PHP Framework For Web Artisans

它也有許多相關原始碼如 [Laravel Echo][] 或是 [Laravel Nova][] 等。因此第一天，要先來定義未來三十天要討論的原始碼是什麼。

預計主要會討論的將會是 Laravel 核心－－[Laravel Framework][laravel/framework]，另外還有大家第一次用 Laravel 會使用的安裝工具 [`laravel/installer`][laravel/installer] 會下載的原始碼－－[`laravel/laravel`][laravel/laravel]；另外在討論 Laravel Framework 時，也會配合 subtree 切分出來 [`illuminate`][illuminate] 裡的獨立元件做說明。

採用的版本如下：
 
* Laravel Framework 與 illuminate 獨立元件：[`5.7.6`](https://github.com/laravel/framework/tree/v5.7.6)
* Laravel [5.7.0](https://github.com/laravel/laravel/tree/v5.7.0)

而上述各種套件之間的關係為：

* Laravel 使用 Laravel Framework 程式，實作了通用型的 Web 框架，包括基本常用的 config、route 與資料夾結構 
* Laravel Framework 的提供一個完整的元件庫，可實作各式各樣的應用，當然也包括 Web
* Laravel Framework 裡面也包含了一些可以獨立運作的元件，它使用 subtree 把各自獨立運作的元件額外切分至 `illuminate` 元件庫裡，包括 Auth、Database 等

知道這個關係後，接著就會知道如何使用它們：

* 如果沒有特別奇特的需求，可以直接使用 *Laravel*
* 如果有特別客製化需求，比方說：不使用 Artisan、不使用 View、不使用 Database、與其他框架共存，則可以使用 *Laravel Framework*
* 如果只是需要特定功能，如 Eloquent ORM，則可以去 *illuminate* 找需要的元件來用

## 獨立元件有哪些

Laravel 與 Laravel Framework 相信沒什麼問題了，接著來看獨立元件有哪些。我們可以從 [`composer.json`](https://github.com/laravel/framework/blob/v5.7.6/composer.json#L43-L70) 的設定查到有下面這些套件

* [`illuminate/auth`](https://github.com/illuminate/auth)
* [`illuminate/broadcasting`](https://github.com/illuminate/broadcasting)
* [`illuminate/bus`](https://github.com/illuminate/bus) 
* [`illuminate/cache`](https://github.com/illuminate/cache)
* [`illuminate/config`](https://github.com/illuminate/config)
* [`illuminate/console`](https://github.com/illuminate/console)
* [`illuminate/container`](https://github.com/illuminate/container)
* [`illuminate/contracts`](https://github.com/illuminate/contracts) 
* [`illuminate/cookie`](https://github.com/illuminate/cookie) 
* [`illuminate/database`](https://github.com/illuminate/database) 
* [`illuminate/encryption`](https://github.com/illuminate/encryption) 
* [`illuminate/events`](https://github.com/illuminate/events) 
* [`illuminate/filesystem`](https://github.com/illuminate/filesystem) 
* [`illuminate/hashing`](https://github.com/illuminate/hashing) 
* [`illuminate/http`](https://github.com/illuminate/http) 
* [`illuminate/log`](https://github.com/illuminate/log) 
* [`illuminate/mail`](https://github.com/illuminate/mail) 
* [`illuminate/notifications`](https://github.com/illuminate/notifications) 
* [`illuminate/pagination`](https://github.com/illuminate/pagination) 
* [`illuminate/pipeline`](https://github.com/illuminate/pipeline) 
* [`illuminate/queue`](https://github.com/illuminate/queue) 
* [`illuminate/redis`](https://github.com/illuminate/redis) 
* [`illuminate/routing`](https://github.com/illuminate/routing) 
* [`illuminate/session`](https://github.com/illuminate/session) 
* [`illuminate/support`](https://github.com/illuminate/support) 
* [`illuminate/translation`](https://github.com/illuminate/translation) 
* [`illuminate/validation`](https://github.com/illuminate/validation) 
* [`illuminate/view`](https://github.com/illuminate/view)

因為是用 subtree 切分的，所以它們在 Laravel Framework 的原始碼下也有對應的目錄，Laravel 文件可以找得到對應套件的說明（用法會有點不大一樣）。

唯一一個目錄是沒有被切出來的－－`Foundation`。其他元件在設計的時候，都盡可能讓它通用化，以便適應不同的場景。而 Laravel 框架也是一個使用場景，Foundation 是為了這個場景所寫的程式碼。更直白的說法就是：裡面有很多懶人包可以隨開即用，未來有機會都會說明。

最後，有一個很特別的元件 `Contracts`，正如其名，裡面有九成都是 `interface`，所有的元件都依賴 Contracts 元件，而元件跟元件之間的依賴會是以 Contracts 所提供的介面為主，甚至是介面依賴介面，這種設計方法就如同 Bridge Pattern：

> 將抽象和實作解耦合，使得兩者可以獨立地變化。

好處也很明顯，抽象和實作都可以很輕易地變化，而達到 開關原則（[Open-close principle][]）的目的。

## 這三十天將會做什麼

Laravel 核心有趣的地方在於：即使是不同框架，只要使用方法是符合它的標準，它一樣可以正常執行，甚至可以做到置換預先提供的功能。

前幾天曾看到 [PTT](https://www.ptt.cc/bbs/Soft_Job/M.1537805551.A.271.html) 有位 *benqhsia* 大大提到：

    在 clean code 這本書的前面幾個章節，有提到:
    
    程式碼最後失控，難道是 PM/PO/主管/客戶 的錯嗎？
    不，錯的是我們 RD，這都要怪我們「不夠專業」。

現在 PHP 都是團隊開發為主，因此不夠專業的 RD，將會大大影響整個團隊的產出。以筆者的經驗，程式碼失控通常都是設計太死、不夠靈活、測試難寫等，而這正好跟 Laravel 的特色相反。或許，我們可以先知道什麼樣是好的設計，進而去調整自己的程式，讓程式更加好調整，並成為一個夠專業的 RD。

三十天的時間當然是不夠的，所以筆者將會從 Laravel 預設是如何使用這個核心，在過程中也會盡可能說明各元件的設計。看完後，相信大家至少可以用 Laravel 用得很厲害！

[CodeIgniter]: https://www.codeigniter.com/
[Laravel]: https://laravel.com/
[Laravel Echo]: https://laravel.com/docs/5.7/broadcasting#installing-laravel-echo
[Laravel Nova]: https://nova.laravel.com/
[laravel/framework]: https://github.com/laravel/framework
[laravel/installer]: https://github.com/laravel/installer
[laravel/laravel]: https://github.com/laravel/laravel
[illuminate]: https://github.com/illuminate

[Open-close principle]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day08.md
