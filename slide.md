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
  | ELam [a] (Expr a)
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
(output, stack, dump, heap, stats)
```

  * 然後舉一兩個例子。

---

# the G-machine

```
(output, stack, dump, heap, stats)
```

  * 舉很多例子。

---

# the G-machine

```
(output, stack, vstack, dump, heap, stats)
```

  * 舉一個 int 例子，一個 boolean 例子。

---

# STG and STGi

---

# References

  * a

  * b

  * c

---

class: inverse, center, middle

# fin

