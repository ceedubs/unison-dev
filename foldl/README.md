# foldl

`foldl` is a Unison module that allows you to compose strict left folds such that their results can be combined in a single pass of input data.

This is a partial Unison port of the Haskell [foldl](https://hackage.haskell.org/package/foldl-1.1.2) module, and you'll find much better documentation there.

## getting the dependency

In a `ucm` session, run the following. Feel free to change `external.foldl` to whatever namespace you prefer.

```
.> pull https://github.com/ceedubs/unison-dev.git:.foldl.trunk external.foldl

```

## usage

First let's set up some test data: the numbers 1 through 100 (Unison's `range` is inclusive for the first value but exclusive for the second).

```unison
nums : [Nat]
nums = range 1 101
```

`count` and `sum` are a couple of the simplest folds that you can perform.

```unison
use foldl.trunk

> fold count nums

> fold sum nums
```

```ucm

  ✅
  
  scratch.u changed.
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    3 | > fold count nums
          ⧩
          100
  
    5 | > fold sum nums
          ⧩
          5050

```
You can compose folds using [applicative](http://hackage.haskell.org/package/base-4.14.0.0/docs/Control-Applicative.html)-style composition. The combined results of these folds will be calculated in a single pass of the data.

```unison
> fold (pair <$> sum <*> count) nums

averageNat : Fold Nat Nat
averageNat = (/) <$> sum <*> count

> fold averageNat nums
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      averageNat : Fold Nat Nat
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    1 | > fold (pair <$> sum <*> count) nums
          ⧩
          (5050, 100)
  
    6 | > fold averageNat nums
          ⧩
          50

```
There are a couple of issues with this `averageNat` function. Since it is using `Nat` division, The actual result `50.5` is being truncated to `50`. Also it will throw a "divide by zero" error if you pass in an empty list. You are better off using the `average` function provided by this library. The `average` function expects the input to have type `Float`, which we can accomplish with a `premap` on the input to the `average` fold.

```unison
> fold (premap toFloat average) nums
```

```ucm

  ✅
  
  scratch.u changed.
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    1 | > fold (premap toFloat average) nums
          ⧩
          Some 50.5

```
The result type is `Optional Float`, because we might not be able to calculate the average if there are zero items in the input. If you want to map the output of the fold in addition to pre-mapping the input, you can use `dimap` and supply both. Let's map the output to return `0.0` instead of `None` if there are no items in the input.

```unison
natAverageOrZero : Fold Nat Float
natAverageOrZero = dimap Nat.toFloat (orDefault 0.0) average

> fold natAverageOrZero nums

> fold natAverageOrZero []
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      natAverageOrZero : Fold Nat Float
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    4 | > fold natAverageOrZero nums
          ⧩
          50.5
  
    6 | > fold natAverageOrZero []
          ⧩
          0.0

```
# module API

Below is a listing of the types and functions provided by this module:

```ucm
.foldl.trunk> find

  1.  type Fold a b
  2.  type Fold' a b x
  3.  Fold'.Fold' : (x -> a -> x)
                    -> x
                    -> (x -> b)
                    -> Fold' a b x
  4.  Fold.<$> : (b -> c) -> Fold a b -> Fold a c
  5.  Fold.<*> : Fold a (b -> c) -> Fold a b -> Fold a c
  6.  Fold.Fold : (∀ r. (∀ x. Fold' a b x -> r) -> r)
                  -> Fold a b
  7.  Fold.List.fold : Fold a b -> [a] -> b
  8.  Fold.Set.fold : Fold a b -> Set a -> b
  9.  Fold.ap.tests.paired : [Result]
  10. Fold.dimap : (a -> b) -> (c -> d) -> Fold b c -> Fold a d
  11. Fold.dimap.tests.paired : [Result]
  12. Fold.foldSet.tests.sumNat : [Result]
  13. Fold.fromFold' : Fold' a b x -> Fold a b
  14. Fold.map : (b -> c) -> Fold a b -> Fold a c
  15. Fold.mkFold : (x -> a -> x) -> x -> (x -> b) -> Fold a b
  16. Fold.premap : (a -> b) -> Fold b c -> Fold a c
  17. Fold.runFoldl : (∀ a b. (b -> a -> b) -> b -> f a ->{𝕖} b)
                      -> Fold a b
                      -> f a
                      ->{𝕖} b
  18. find : (a -> Boolean) -> Fold a (Optional a)
  19. folds.Float.average : Fold Float (Optional Float)
  20. folds.Float.sum : Fold Float Float
  21. folds.Nat.count : Fold a Nat
  22. folds.Nat.sum : Fold Nat Nat
  23. folds.Nat.sum.tests.nonempty : [Result]
  24. folds.all : (a -> Boolean) -> Fold a Boolean
  25. folds.any : (a -> Boolean) -> Fold a Boolean
  26. nums : [Nat]
  

```
