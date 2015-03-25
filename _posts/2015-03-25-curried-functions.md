---
title: Curried functions in short
category : posts
tags : [functional, curry]
layout: post
image: http://upload.wikimedia.org/wikipedia/commons/2/22/Curry_Ist.jpg
---

let multiply x y = x * y

```haskell
ghci> let multiply x y = x * y
```

We are used to see **multiply** as a single function with exactly 2 arguments. In this case the **multiply** function takes 2 numbers and yields its product. 
The first step to start thinking functional is to imagine the function as a series of functions with just one argument, so that if you call **multiply 2** you have a new function as result with one argument `y` and where `x` has been bound to 2. It's now possible to call the latter function to double a value.

```haskell
ghci> let double = multiply 2
ghci> double 20
40
```

The translation of **multiply** into a sequence of 2 functions each with one argument is called **currying** and every single application is called **partial application**.
If we call multiply with too few parameters we get back the partially applied function which is a function that takes as many parameters as we left out. This means we can think functions as factories that take some arguments and creates other functions.

```haskell
ghci> let multiplyByTen  = multiply 10

ghci> let multiplyByFive = multiply 5
```
The interesting thing is that functional languages such as Haskell, ML, etc... apply transparently the currification.

This behaviour influences the language's type system, in fact even though we expects the type of multiply should be `(a , a) -> a` (a function that takes two numbers and returns a number), the inferred type is `a -> a -> a` because the function is curried.

```haskell
ghci> :t multiplyByTen
multiplyByTen :: Num a => a -> a
```