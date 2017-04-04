# G-machine: the stack machine inside a pure functional language

An one hour talk for [Functional Thursday #50][funth-50].

[funth-50]: https://www.meetup.com/Functional-Thursday/events/238314171/

## Description

「函數式語言是怎麼運作的呢？」

帶這這樣的疑問, 講者在網路上胡亂地翻到了 [The Implementation of Functional Programming Languages][spj-1987] 還有 [Implementing functional languages: a tutorial][spj-lester-1992] 這兩本書. 聽了 SPJ 在 Erlang User Conf 2016 的演講 [Into the Core][spj-2016].

發現可以從 λ calculus 開始, 看看一個簡單的函數式語言（Core）與 λ calculus 之間的關聯, 並以 G-machine 來「計算」用 Core 寫的程式.

這次的分享以 [Implementing functional languages: a tutorial][spj-lester-1992] 的內容為主,「不」包含 STG machine, typeclass, Functor, Applicative, Monad, 請安心服用 :D

[spj-1987]: https://www.microsoft.com/en-us/research/publication/the-implementation-of-functional-programming-languages/
[spj-lester-1992]: https://www.microsoft.com/en-us/research/publication/implementing-functional-languages-a-tutorial/
[spj-2016]: https://www.youtube.com/watch?v=uR_VzYxvbxg

