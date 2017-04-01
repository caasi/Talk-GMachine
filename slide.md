# G-machine: the stack machine inside a pure functional language

## lambda calculus

### reduction rules

* α-conversion

* β-reduction

* η-conversion

### lambda terms

```haskell
data Term =
  | Var String
  | Lam String Term
  | App Term Term
```

```
-- id
λa.a
```

```
-- id
\a a
```

## sugared lambda

### Peter Landin & ISWIN

### let

```
let x = 3 in id x
```

```
(\x id x)(3)
```

### where

```
id x where x = 3
```

```
(\x id x)(3)
```

### data constructor and `case ... of`

```
-- data Boolean = True | False
(\x \y y)
(\x \y x)
```

```
case ... of
  True  -> ...
  False -> ...
```

```
-- data List a = Nil | Cons a List
(\a \b a)
(\x \xs \a \b b x xs)
```

```
(\Nil
(\Cons
  (\list
    case list of
      Nil -> 0
      Cons x xs -> 1
  )(Cons 1 (Cons 1 (Cons 2 (Cons 3 Nil))))
)(\x \xs \a \b b x xs)
)(\a \b a)
```

```
-- 吃了兩個參數的 Cons
(\a \b b x xs)
```

```
(\Nil
(\Cons
  (\list
    list
      (0)
      (\x \xs 1)
  )(Cons 1 (Cons 1 (Cons 2 (Cons 3 Nil))))
)(\x \xs \a \b b x xs)
)(\a \b a)
```

### let(rec) with only 1 argument

```
-- length
(\xs
  (xs
    (\nil 0)
    (\y \ys (+ 1 (length ys))) 
  )
)
```

```
(\length \xs
  (xs
    (\nil 0)
    (\y \ys (+ 1 (length length ys)))
  )
)
(\length \xs ...)
```

```
(\f f f)
(\length \xs
  (xs
    (\nil 0)
    (\y \ys (+ 1 (length length ys)))
  )
)
```

```
(\f f f)
(\length
  (\g \xs
    (xs
      (\nil 0)
      (\y \ys (+ 1 (g ys)))
    )
  )
  (length length)
)
```

```
-- eta conversion
(\h
  (\f f f)
  (\length
    h
    (length length)
  )
)
(\g \xs
  (xs
    (\nil 0)
    (\y \ys (+ 1 (g ys)))
  )
)
```

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
-- Y
(\f
  (\x f (x x)) (\x f (x x))
)
```

## the Core language(1987)

回到 Core 的寫法，而不使用前面的寫法。

```haskell
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

```
-- EVar Name
id

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

```
-- ELet IsRec [(a, Expr a)] (Expr a)
let(rec)
  a = ...
  b = ...
in
  body
```

```
-- ECase (Expr a) [Alter a]
case ... of
  <0> -> ...
  <1> x xs -> ...
```

```
-- ELam [a] (Expr a)
\x.x
\x.y.x
\x.y.y
\f.g.x.f x (g x)
\f.g.x.f (g x)
\f.f f
```

## naive way: template instantiation

```
(output, stack, dump, heap, stats)
```

然後舉一兩個例子。

## the G-machine

```
(output, stack, dump, heap, stats)
```

舉很多例子。

```
(output, stack, vstack, dump, heap, stats)
```

舉一個 int 例子，一個 boolean 例子。

## STG and STGi

## References

