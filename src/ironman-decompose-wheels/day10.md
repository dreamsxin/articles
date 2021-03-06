# Faker（5）－－Provider 們不要起爭議

Faker 提供的 Provider 非常多，除了預設之外，還有不同語系實作。

不過我們先來解決[昨天][Day 9]的疑惑：這些 Provider 到底是如何使用 `Generator` 的？

## Magic 的中介層設計

搜尋了一下，會發現 Provider 大部分會使用 `Generator` 的 `parse()` 方法，而 [Day 7][] 有提到，它的本質是 `format()`。換句話說，Provider 會經由 `Generator` 來存取其他 Provider。

這個設計有點類似 *Mediator Pattern*，它們的關係如下：

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuNBEIImkLd3EoKpDAu5od1ABKnMgkHGKb1NIK_DIYn9Byeki5DnXJAvQgBg0eloop9JK8aCqlX6KZz01CLv1rmv936oBJOskBf8vc696N70T2ZQw4ATPAVXcfcI23K580ir6c8Cah8iaOSJba9gN0lGr0000)

```
@startuml
Class Client
Class Base {
  # generator: Generator
}
Class Provider1
Class Provider2
Class Generator
Client -> Generator
Base <|-- Provider1
Base <|-- Provider2
Generator <- Base : Midiator
Generator -> Provider1
Generator ---> Provider2
@enduml
```

Provider 要使用 `Generator` 當 Mediator 時，必須小心循環呼叫的問題，比方說 A Provider 呼叫 B Provider，而 B Provider 又要呼叫 A Provider。

## 基礎是非常重要的

[Day 8][] 提到 `Provider\Base` 類別提供非常多基本亂數取樣方法，今天就派得上用場了！

Provider 最常用到的肯定是 `randomElement()`，不同領域的 Provider 通常都會有自己的口袋名單，要從口袋名單裡隨便選一個，當然就是用它。`numberBetween()` 也是個常用到方法，因為不同領域的 Provider 值域都不大一樣。

如果有仔細觀察，會發現 Provider 非常多方法都有用到 `Provider\Base` 的亂數取樣方法。

## 語系擴充

Provider 語系擴充的設計是，繼承的時候覆寫對應的口袋名單、樣版或是產生的方法即可。

比方說 [`Provider\zh_TW\Person`](https://github.com/fzaninotto/Faker/blob/v1.7.1/src/Faker/Provider/zh_TW/Person.php) 類別，它繼承自 [`Provider\Person`](https://github.com/fzaninotto/Faker/blob/v1.7.1/src/Faker/Provider/Person.php)，覆寫 `$maleNameFormats` 與 `$femaleNameFormats` 樣版，因為台灣名字的顯示慣例先姓後名：

```php
protected static $maleNameFormats = array(
    '{{lastName}}{{firstNameMale}}',
);

protected static $femaleNameFormats = array(
    '{{lastName}}{{firstNameFemale}}',
);
```

口袋名單 `$lastName`、`$characterMale` 與 `$characterFemale` 等，當然也會覆寫：

```php
protected static $lastName = array(
    // ...
);

protected static $characterMale = array(
    // ...
);

protected static $characterFemale = array(
    // ...
);
```

產生名字的方法也覆寫了，因為台灣大部分是一個姓配兩個名：

```php
public static function firstNameMale()
{
    return static::randomName(static::$characterMale, mt_rand(1, 2));
}

public static function firstNameFemale()
{
    return static::randomName(static::$characterFemale, mt_rand(1, 2));
}
```

`Provider]\Person` 的 `name()` 方法並沒有被覆寫，[原始碼](https://github.com/fzaninotto/Faker/blob/v1.7.1/src/Faker/Provider/Person.php#L47-L58)如下：

```php
public function name($gender = null)
{
    if ($gender === static::GENDER_MALE) {
        $format = static::randomElement(static::$maleNameFormats);
    } elseif ($gender === static::GENDER_FEMALE) {
        $format = static::randomElement(static::$femaleNameFormats);
    } else {
        $format = static::randomElement(array_merge(static::$maleNameFormats, static::$femaleNameFormats));
    }

    return $this->generator->parse($format);
}
```

因此，當我們使用 `zh_TW` 語系，在呼叫 `$generator->name` 的時候，事實上它會先跑 `Provider\Person` 的 `name()` 方法，然後使用 `$generator->parse()` 把「覆寫的樣板」放進去，接著再把「覆寫的口袋名單」與呼叫「覆寫產生名的方法」，最後組合出回傳結果。

> 這種覆寫的方法也是 Laravel 常見擴充寫法。

了解它的設計後，後面要擴充自己的假資料清單就會非常容易了。

---

Faker 主框架差不多介紹完了，明天來試試自定義 Provider。

## 參考資料

* [Mediator 模式](https://openhome.cc/Gossip/DesignPattern/MediatorPattern.htm) | 良葛格學習筆記

[Day 7]: day07.md
[Day 8]: day08.md
[Day 9]: day09.md
