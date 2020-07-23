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

```ucm:hide:all
.> cd .foldl.trunk
```

```unison:hide
nums : [Nat]
nums = range 1 101
```

```ucm:hide:all
.foldl.trunk> add
```

`count` and `sum` are a couple of the simplest folds that you can perform.

```unison
use foldl.trunk

> fold count nums

> fold sum nums
```

You can compose folds using [applicative](http://hackage.haskell.org/package/base-4.14.0.0/docs/Control-Applicative.html)-style composition. The combined results of these folds will be calculated in a single pass of the data.

```unison
> fold (pair <$> sum <*> count) nums

averageNat : Fold Nat Nat
averageNat = (/) <$> sum <*> count

> fold averageNat nums
```

There are a couple of issues with this `averageNat` function. Since it is using `Nat` division, The actual result `50.5` is being truncated to `50`. Also it will throw a "divide by zero" error if you pass in an empty list. You are better off using the `average` function provided by this library. The `average` function expects the input to have type `Float`, which we can accomplish with a `premap` on the input to the `average` fold.

```unison
> fold (premap toFloat average) nums
```

The result type is `Optional Float`, because we might not be able to calculate the average if there are zero items in the input. If you want to map the output of the fold in addition to pre-mapping the input, you can use `dimap` and supply both. Let's map the output to return `0.0` instead of `None` if there are no items in the input.

```unison
natAverageOrZero : Fold Nat Float
natAverageOrZero = dimap Nat.toFloat (orDefault 0.0) average

> fold natAverageOrZero nums

> fold natAverageOrZero []
```

# module API

Below is a listing of the types and functions provided by this module:

```ucm
.foldl.trunk> find
```
