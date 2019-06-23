# Unix-like 終端機相關筆記

> 在寫 Command line 相關的應用程式，所以研究了一下。

終端機介面又稱 TTY 介面，它有兩種模式：正規（canonical）和非正規（non-canonical）。

| 模式 | 終端設備行為 | 範例 |
| --- | --- | --- |
| canonical（也稱 cooked 模式） | 終端設備會處理特殊字元，且會以一次「一列」的方式將輸入傳給應用程式 | 如 Shell 指令，或是一般寫指令都會使用這個模式 |
| non-canonical（也稱 raw 模式） | 終端設備「不會」處理特殊字元，且會以一次「一個字元」的方式將輸入傳給應用程式 | 如 Vim，或是有些指令安裝會有選單介面，都會使用這個模式 |

> [stty 指令參考](https://codingstandards.iteye.com/blog/826924)，[tput 指令參考](https://blog.csdn.net/fdipzone/article/details/9993961)

## References

* [What is tty?](https://flykof.pixnet.net/blog/post/24277709-%5B%E8%BD%89%E8%BC%89%E6%95%B4%E7%90%86%5Dwhat-is-tty%3F)
* [Linux RS-232 程式設計](https://blog.xuite.net/uwlib_mud/twblog/108242774-Linux+RS-232+%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88)

因相關的中文資料難找，因此另外找了寫 command line 的套件參考：

* [Symfony Console](https://github.com/symfony/Console)
* [Hoa\Console](https://github.com/hoaproject/Console)
* [CLImate](https://github.com/thephpleague/climate)
* [ink](https://github.com/vadimdemedes/ink)

另外也有直接操控 terminal 的套件可以參考：

* [Terminal Utility](https://github.com/php-school/terminal)
* [termbox-go](https://github.com/nsf/termbox-go)
* [`termios.h`](https://github.com/torvalds/linux/blob/master/include/uapi/asm-generic/termios.h)
