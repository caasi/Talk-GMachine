class: inverse, center, middle

# G-machine
## the stack machine inside a pure functional language
### caasih

2017.04.06

---

# lambda calculus

```
data Term =
  | Var String
  | Lam String Term
  | App Term Term
```

  * lambda terms

    * variable

      ```
      x
      ```

    * lambda abstraction

      ```
      λx.x
      ```

    * lambda application

      ```
      f x
      ```

---

# lambda calculus

  * reduction rules

    * α-conversion (alpha-conversion)

      ```
      λx.x
      λy.y
      ```

    * β-reduction (beta-reduction)

      ```
      (λx.x) y
      y
      ```

    * η-conversion (eta-conversion)

      ```
      (λx.f x)
      f
      ```

      或反過來

---

# sugared lambda

  * Peter Landin & ISWIN

---

# sugared lambda

以下是對「加糖」前後的 lambda ，一種直觀的對照

為了方便，

```
λx.x
```

都寫做

```
\x x
```

---

# sugared lambda

### `let` & `where`

```
let x = 3 in id x
id x where x = 3
```

```
(\x    -- let x
  id x --   in id x
)(3)   -- = 3
```

---

# sugared lambda

### `letrec`

```
infinite n =
  letrec
    ns = cons n ns
  in
    ns
```

```
(\n
  (\ns                        -- letrec ns
    ns                        --   in ns
  )(\Y                        --
    Y (\ns cons n ns)         -- = cons n ns
  )(\f \x (f (x x))(f (x x)))
)
```

---

# sugared lambda

### data constructors

```
-- data Bool = True | False
(\x \y y)
(\x \y x)
```

```
-- data List a = Nil | Cons a List
(\a \b a)
(\x \xs (\a \b b x xs))
```

---

# sugared lambda

### `case ... of`

```
let
  list = (Cons 1 (Cons 1 (Cons 2 (Cons 3 Nil))))
in
  case list of
    Nil       -> 0
    Cons x xs -> 1
```

```
(\Nil
(\Cons
  (\list
    list         -- `case ... of` are embedded in functions
      0          -- Nil       -> 0
      (\x \xs 1) -- Cons x xs -> 1
  )(Cons 1 (Cons 1 (Cons 2 (Cons 3 Nil))))
)(\x \xs \a \b b x xs)
)(\a \b a)
```

---

# sugared lambda

### get `Y` from a `letrec` with only 1 argument

```
(\xs
  (xs
    0
    (\y \ys (+ 1 (length ys))) -- where is `length`?
  )
)
```

```
(\length \xs
  (xs
    0
    (\y \ys (+ 1 (length length ys))) -- dum dum
  )
)
(\length \xs ...) -- duplicated :(
```

---

# sugared lambda

```haskell
(\f f f)
(\length \xs
  (xs
    0
    (\y \ys (+ 1 (length length ys)))
  )
)
```

```
(\f f f)
(\length
  (\g \xs
    (xs
      0
      (\y \ys (+ 1 (g ys)))
    )
  )
  (length length)
)
```

---

# sugared lambda

```
(\h
  (\f f f)
  (\length
    h
    (length length)
  )
)
(\g \xs
  (xs
    0
    (\y \ys (+ 1 (g ys)))
  )
)
```

---

# sugared lambda

```
(\h
  (\f f f)
  (\g h (g g))
)
```

```
(\f
  (\y y y)
  (\x f (x x))
)
```

```
(\f
  (\x f (x x)) (\x f (x x))
)
```

---

# Core (1987)

```
data Expr a
  = EVar Name
  | ENum Int
  | EConstr Int Int
  | EAp (Expr a) (Expr a)
  | ELet
      IsRec
      [(a, Expr a)]
      (Expr a)
  | ECase
      (Expr a)
      [Alter a]
  | ELam [a] (Expr a)o

type Name = String
type CoreExpr = Expr Name
```

---

# Core

```
-- EVar Name
id
```

```
-- ENum Int
42
```

```
-- EConstr Int Int
Pack{tag, arity}
```

  * `tag` 出現了！ `tag` 可以說是 sum type 的 id

  * 在一個 type check 過的程式中，只要自己的 type 裡 tag 不衝突就好了

```
-- EAp (Expr a) (Expr a)
f x
```

另外 Core 是有 infix operators ，像是 `+` 啊 `*`

---

# Core

```
-- ELet IsRec [(a, Expr a)] (Expr a)
let
  a = ... ;
  b = ...
in
  body
```

```
-- ECase (Expr a) [Alter a]
case ... of
  <0>      -> ... ;
  <1> x xs -> ...
```

---

# Core

```
-- ELam [a] (Expr a)
\x . x
\x y . x
\x y . y
\f g x . f x (g x)
\f g x . f (g x)
\f . f f
```

---

# graph reduction

  * lambda lifting

  * supercombinator

---

# Template Instantiation

```
data Node
  = NAp Addr Addr        -- Application
  | NSupercomb           -- Supercombinator
      Name
      [Name]
      CoreExpr
  | NNum Int             -- A number
  | NInd Addr            -- Indirection
  | NPrim Name Primitive -- Primitive
  | NData Int [Addr]     -- Tag, list of components
  | NMarked Node         -- Marked node
  | NForward Addr        -- Address pointed to the to-space heap
```

---

# Template Instantiation

```
(output, stack, dump, heap, globals, stats)
```

  * output: 放輸出的字元陣列

  * stack: 放現在正在爬的樹

  * dump: 可以放爬到一半的樹

  * heap: 放 Addr 與 AST 節點對應的 Map ， (Addr, Node)

  * globals: 預先做好的環境常數們 (Name, Addr)

  * stats: 放 debug 訊息

---

# Template Instantiation

```
-- program
K x y = x ;
main = K 3 (K 4 5)
```

```
-- globals + env
"main" => Addr 1
"K"    => Addr 0
```

---

# Template Instantiation

```
-- heap
Addr 1 =>
  NSupercomb
    "main"
    []
    (EAp
      (EAp (EVar "K") (ENum 3))
      (EAp
        (EAp (EVar "K") (ENum 4))
        (ENum 5)))
Addr 0 =>
  NSupercomb
    "K"
    ["x", "y"]
    (EVar "x")
```

---

# Template Instantiation

```
-- stack
[ 1 ] -- NSupercomb "main" ...
```

  * 在 stack 中找到了 `Addr 1`

  * 發現對應的 `CoreExpr` 是 `(EAp (EAp (EVar "K") (ENum 3)) (EAp (EAp (EVar "K") (ENum 4)) (ENum 5)))`

  * 把 `ENum 4` 做（instantiate）出來吧，得到 `NNum 4` ，放在 `Addr 3`

  * 把 `EVar "K"` 做出來

  * 從 globals + env 中查到 `"K"` 對應到 `0`

  * 得到 `NAp (Addr 0) (Addr 3)` 放在 `Addr 4`

  * 得到 `NNum 5` 放在 `Addr 5` ， `NAp (Addr 4) (Addr 5)` 放在 `Addr 6` 、 `NNum 3` 放 `Addr 7` 、 `NAp (Addr 0) (Addr 7)` 放在 `Addr 8` ...

  * 最後得到 `NAp (Addr 8) (Addr 6)` 放在 `Addr 9`

---

# Template Instantiation

```
-- stack
[ 9   -- (NAp (Addr 8) (Addr 6))
, 8   -- (NAp (Addr 0) (Addr 7))
]
```

```
-- stack
[ 9   -- (NAp (Addr 8) (Addr 6))
, 8   -- (NAp (Addr 0) (Addr 7))
, 0   -- Supercomb "K" ["x", "y"] ...
]
```

  * 接著在 instantiate `K` 的過程中，會把 `K` 的 `x`, `y` 對應到 `Addr 7` 和 `Addr 6` ，成為新的 `env`

---

# Template Instantiation

  * 簡單來說...

  * template instantiation 靠查找 AST ，動態產生表示整個程式現況的圖

  * 才能化簡並且共用節點

  * 化簡的過程就是不斷爬樹

  * 順著 `NAp` 一直往左邊爬，直到遇上 supercombinater

  * 然後做出新的點來

```
-- stack
[ 9   -- (NAp (Addr 8) (Addr 6))      | spine
, 8   -- (NAp (Addr 0) (Addr 7))      |
, 0   -- Supercomb "K" ["x", "y"] ... |
]     --                              v
```

  * 這條爬著爬著會找到 supercombinator 和它的所有參數的路，就是 `spine`

---

# Template Instantiation

  * 易懂但是：

    * 得走過 `CoreExpr` ，配上現有的 `globals` 和 `env` 才能生出新的 `Node`

    * 沒法簡單做出 `case ... of`

  * 能不能直接操作 `stack` 裡面的值就好？

---

# the G-machine

```
type GmCode = [Instruction]

data Instruction
  = Unwind
  | Pushglobal Name
  | Pushint Int
  | Push Int
  | Mkap
  | Update Int
  | Pop Int
  | ...
```

```
(output, stack, dump, heap, globals, stats)
```

---

# the G-machine

```
I x = x ;
main = I 3
```

  * 以 `I x = x` 為例，如果 `stack` 中離你最近的參數 index 為 `0` ，那你要做的事情是：

    * 把 index `0` 的 `x` 放到 `stack` 頂端

    * 更新原本的 `Node` 成 `stack` 頂端的結果

    * 把 `stack` 中的值和參數都清掉

  * 於是 `GmCode` 是：

    ```
    [Push 0, Update 1, Pop 1, Unwind]
    ```

---

# the G-machine

  * 再來看看 `main = I 3`

  * 寫成 `CoreExpr` 是 `EAp (EVar "I") (ENum 3)`

  * `I` 和 `3` 都不是 `main` 的參數，故 `GmCode` 是：

    ```
    [Pushint 3, Pushglobal "I", Mkap, Update 0, Pop 0, Unwind]
    ```

---

# the G-machine

```
-- stack
[ NGlobal
    0
    [Pushint 3, Pushglobal "I", Mkap, Update 0, Pop 0, Unwind]
]
-- instructions
[Unwind]
```

---

# the G-machine

```
-- stack
[ NGlobal 0 [...]
]
-- instructions
[Pushint 3, Pushglobal "I", Mkap, Update 0, Pop 0, Unwind]
```

---

# the G-machine

```
-- stack
[ NGlobal 0 [...]
, NNum 3
]
-- instructions
[Pushglobal "I", Mkap, Update 0, Pop 0, Unwind]
```

---

# the G-machine

```
-- stack
[ NGlobal 0 [...]
, NNum 3
, NGlobal 1 [...]
]
-- instructions
[Mkap, Update 0, Pop 0, Unwind]
```

---

# the G-machine

```
-- stack
[ NGlobal 0 [...]
, NAp
    NGlobal 1 [...]
    NNum 3
]
-- instructions
[Update 0, Pop 0, Unwind]
```

---

# the G-machine

```
-- stack
[ NInd
    NAp
      NGlobal 1 [...]
      NNum 3
]
-- instructions
[Unwind]
```

---

# the G-machine

```
-- stack
[ NAp
    NGlobal 1 [...]
    NNum 3
]
-- instructions
[Unwind]
```

---

# the G-machine

```
-- stack
[ NAp
    NGlobal 1 [...]
    NNum 3
, NNum 3
]
-- instructions
[Unwind]
```

---

# the G-machine

```
-- stack
[ NAp
    NGlobal 1 [...]
    NNum 3
, NNum 3
]
-- instrucitons
[Push 0, Update 1, Pop 1, Unwind]
```

---

# the G-machine

```
-- stack
[ NAp ...
, NNum 3
, NNum 3
]
-- instructions
[Update 1, Pop 1, Unwind]
```

---

# the G-machine

```
-- stack
[ NInd (NNum 3)
, Num 3
]
-- instructions
[Pop 1, Unwind]
```

---

# the G-machine

```
-- stack
[ NInd (NNum 3)
]
-- instructions
[Unwind]
```

---

# the G-machine

```
-- stack
[ NNum 3
]
-- instructions
[Unwind]
```

  * 跑完了，結果是 `3`

---

# the G-machine

  * 接著我們來看看：

    ```
    main = let three = 3 in three
    ```

---

# the G-machine

  * 來看看：

    ```
    Y f = letrec x = f x in x
    ```

---

# the G-machine

```
data Instruction
  = ...
  | Pushbasic Int
  | Mkbool
  | Mkint
  | Get
```

```
(output, stack, vstack, dump, heap, globals, stats)
```

  * 舉一個 int 例子，一個 boolean 例子。

---

# the G-machine

```
type GmCode = [Instruction]

data Instruction
  = Unwind
  | Pushglobal Name
  | Puthint Int
  | Push Int
  | Mkap
  | Slide Int
  | Alloc Int
  | Update Int
  | Pop Int
  | Eval
  | Add | Sub | Mul | Div | Neg
  | Eq | Ne | Lt | Le | Gt | Ge
  | Cond GmCode GmCode
  | Pack Int Int
  | Casejump [(Int, GmCode)]
  | Split Int
  | Print
```

---

# STG and STGi

---

# References

  * The Implementation of Functional Programming Languages

  * Implementing functional languages: a tutorial

  * [Into the Core](https://www.youtube.com/watch?v=uR_VzYxvbxg)

  * The next 700 programming languages

  * [Functional and low-level: watching the STG execute](https://skillsmatter.com/skillscasts/8800-functional-and-low-level-watching-the-stg-execute)

  * [workshop-2015.9.24.md](https://github.com/CindyLinz/BYOHC-Workshop/blob/master/workshop-2015.9.24.md)

---

class: inverse, center, middle

# fin

