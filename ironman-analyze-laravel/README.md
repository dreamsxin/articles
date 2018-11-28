# Laravel 原始碼分析

Laravel 是目前 PHP 熱門的框架之一；它一定是好用，才會受到大家關注；那對開發者而言，什麼才是好用呢？具備「快速驗證」、「簡潔的程式碼」、「豐富的套件生態系」、「客製化容易」等特性的語言或框架，開發者肯定都會躍躍欲試。未來三十天，筆者將會試著分析 Laravel 原始碼，讓讀者了解什麼是好的架構，並在未來開發設計有好的方向可以參考。

## 前言

其實去年就有想過要寫 Laravel 相關的原始碼分析。在 Laravel 開發過程中，有時會遇到困難或瓶頸，不知如何是好時，這時翻原始碼追原因後，都會有頓悟的感覺。通常是自己耍笨，或是覺得 Laravel 怎麼有這樣的神設計。

因此，會想把這個追原始碼與了解設計的過程筆記起來。在以後設計系統甚至是框架的時候，都能回頭省思現在自己的作品到底是好或不好。

同時，也希望可以幫助更多開發者，無論有沒有用 Laravel ，都可以了解什麼是「比較好的」設計，同時也就能避免寫出難以維護的程式碼。不僅自己開心，其他共同維護者也會很開心。

## 目錄

* [Day 1 - 簡介](day01.md)
* [Day 2 - 分析 bootstrap 流程](day02.md)
* [Day 3 - 分析 Container（1）](day03.md)
* [Day 4 - 分析 Container（2）](day04.md)
* [Day 5 - 分析 Application](day05.md)
* [Day 6 - 分析 Config](day06.md)
* [Day 7 - 分析 Pipeline（1）](day07.md)
* [Day 8 - 分析 Pipeline（2）](day08.md)
* [Day 9 - 分析 Cookie](day09.md)
* [Day 10 - 分析 Session（1）](day10.md)
* [Day 11 - 分析 Session（2）](day11.md)
* [Day 12 - 分析 Routing（1）](day12.md)
* [Day 13 - 分析 Routing（2）](day13.md)
* [Day 14 - 分析 Routing（3）](day14.md)
* [Day 15 - 分析 Routing（4）](day15.md)
* [Day 16 - 分析 Routing（5）](day16.md)
* [Day 17 - 分析 Routing（6）](day17.md)
* [Day 18 - 分析 Routing（7）](day18.md)
* [Day 19 - 分析 Marcoable](day19.md)
* [Day 20 - 解析 Middleware 的實作細節](day20.md)
* [Day 21 - 分析 Log](day21.md)
* [Day 22 - 分析 Facade](day22.md)
* [Day 23 - 分析 AliasLoader](day23.md)
* [Day 24 - 分析 Auth（1）](day24.md)
* [Day 25 - 分析 Auth（2）](day25.md)
* [Day 26 - 分析 Auth（3）－－客製化驗證機制](day26.md)
* [Day 27 - 分析 Auth（4）－－Authorization](day27.md)
* [Day 28 - 分析 Auth（5）－－Authorization](day28.md)
* [Day 29 - 分析 Auth（6）－－Authorization](day29.md)
* [Day 30 - 總結](day30.md)

---

* [Day 31 - 分析自定義錯誤頁](day31.md)
* [Day 32 - Redirector 與 UrlGenerator 的關係](day32.md)
* [Day 33 - 如何正確地在 Response 加 Header（1）](day33.md)
* [Day 34 - 如何正確地在 Response 加 Header（2）](day34.md)
* [Day 35 - 自定義 bootstrapper](day35.md)
* [Day 36 - array_get()、data_get() 與 object_get() 的差異](day36.md)
* [Day 37 - 分析 Collection（1）](day37.md)
* [Day 38 - 分析 Collection（2）](day38.md)
* [Day 39 - 分析 Collection（3）－－Higher Order Messages](day39.md)
* [Day 40 - 再看 tap()](day40.md)

---

* [Day 41 - Lumen 簡介](day41.md)
* [Day 42 - 分析 bootstrap 流程－－Lumen 篇](day42.md)
* [Day 43 - 分析 Lumen Application－－dispatch() 上篇](day43.md)
* [Day 44 - 分析 Lumen Application－－dispatch() 下篇](day44.md)

## 誌謝

* 感謝老婆支持我寫作。
