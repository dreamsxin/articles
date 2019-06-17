# Laravel 套件開發小提醒

因 Laravel 5.0 開始的小版號調整並不完全往下相容，這與 Composer 的更新規則（`^`）不同。一般在開發專案並不會有太大問題，因為可以直接鎖定小版號，甚至直接指定版號即可。但套件通常會想在某個範圍的版本正常使用，以追求更大的相容性。

## 目錄

* [Arr 的版本差異](arr-class-diff.md)
* [Container 差異](container-diff.md)