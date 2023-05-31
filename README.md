# ⚠️Disclaimer: the following benchmarks are outdated. ⚠️

Functional Benchmarks
=====================

This repository contains a collection of benchmarks of functional programming languages and proof assistants. It is
split in two categories, Checker and Runtime, which measure the time it takes to type-check and run a program,
respectivelly. The main goal of this repository is to track [Kind2](https://github.com/kindelia/kind2)'s performance
compared to alternatives. 

Type Checker
------------

The tests below measure type-checking and theorem proving performance.

### nat_exp

Proof that `is-even (2 ^ N) == true`. Measures type-level evaluation speed.

```agda
add (a: Nat) (b: Nat) : Nat
add x zero     = x
add x (succ y) = succ (add x y)

mul (a: Nat) (b: Nat) : Nat
mul a zero     = zero
mul a (succ b) = add a (mul a b)

exp (a: Nat) (b: Nat) : Nat
exp a zero     = succ zero
exp a (succ b) = mul a (exp a b)

Main : Equal (is_even (exp N2 N)) true
Main = refl
```

![](image/checker_nat_exp_.png)

**Comments:** Kind2 is *many times* faster than alternatives, due to HVM's raw speed.

### tree_fold

Proof that `(tree-fold (full-tree N) and true) == true`. Measures type-level evaluation speed.

```agda
full_tree (d: Nat) : Tree
full_tree zero     = leaf
full_tree (succ d) = let branch = full_tree d; node branch branch

tree_fold (a: Tree) (p: Type) (n: p -> p -> p) (l: p) : p
tree_fold leaf       p n l = l
tree_fold (node a b) p n l = n (tree_fold a p n l) (tree_fold b p n l)

force_tree (a: Tree) : Bool
force_tree t = tree_fold t Bool (a => b => and a b) true

Main : Equal (force_tree (full_tree N)) true
Main = refl
```

![](image/checker_tree_fold_.png)

**Comments:** Kind2 is *many times* faster than alternatives, due to HVM's raw speed.

### nat_exp_church

Proof that `is-even (2 ^ N) == true`, Church encoded. Measures type-level evaluation asymptotics.

```agda
Church.add (a: Church.Nat) (b: Church.Nat) : Church.Nat
Church.add a b = p => f => z => a p f (b p f z)

Church.mul (a: Church.Nat) (b: Church.Nat) : Church.Nat
Church.mul a b = p => f => a p (b p f)

Church.exp (a: Church.Nat) (b: Church.Nat) : Church.Nat
Church.exp a b = p => b (p -> p) (a p)

Main : Equal (Church.is_even (Church.exp Church.N2 N)) Church.true
Main = refl
```

![](image/checker_nat_exp_church_.png)

**Comments:** Kind2 is *exponentially* faster than alternatives, due to HVM's optimal reduction.

### tree_fold_church

Proof that `(tree-fold (full-tree N) and true) == true`, Church encoded. Measures type-level evaluation asymptotics.

```agda
Church.full_tree (d: Church.Nat) : Church.Tree
Church.full_tree d = p => n => l => d p (t => n t t) l

Church.tree_fold (a: Church.Tree) (p: Type) (n: p -> p -> p) (l: p) : p
Church.tree_fold t p n l = t p n l

Church.force_tree (a: Church.Tree) : Church.Bool
Church.force_tree t = Church.tree_fold t Church.Bool (a => b => Church.and a b) Church.true

Main : Equal (Church.force_tree (Church.full_tree N)) Church.true
Main = refl
```

![](image/checker_tree_fold_church_.png)

**Comments:** Kind2 is *exponentially* faster than alternatives, due to HVM's optimal reduction.

### vector

Type-checks a huge vector with several meta variables. Measures parser and elaborator performance.

```
TODO
```

### STLC

Type-checks a simply typed λ-calculus interpreter. Measures parser and elaborator performance.

```
TODO
```

### RLP

Type-checks a RLP identity proof. Measures performance of heavy-weight equational reasoning.

```
TODO
```

Runtime
-------

The tests below measure runtime performance.

### list_fold

Creates a large list and folds over it with uint64 addition. Measures runtime evaluation speed for sequential recursion.

```agda
Fold <t: Type> (list: List t) <p: Type> (cons: t -> p -> p) (nil: p) : p
Fold t Nil         p c n = n
Fold t (Cons x xs) p c n = c x (Fold xs c n)

Range (n: U60) (list: List U60) : List U60
Range 0 xs = xs
Range n xs = let m = (- n 1); Range m (Cons m xs)
```

![](image/runtime_list_fold_.png)

**Comments:** Kind2 and Haskell have similar performance on sequential recursive algorithms.

### tree_fold

Creates a large list and folds over it with uint64 addition. Measures runtime evaluation speed for parallel recursion.

```agda
Gen (n: U60) : Tree
Gen 0 = Leaf 1
Gen n = Node (Gen (- n 1)) (Gen (- n 1))

Sum (tree: Tree) : U60
Sum (Leaf x)   = x
Sum (Node a b) = (+ (Sum a) (Sum b))
```

![](image/runtime_tree_fold_.png)

**Comments:** Kind2 outperforms Haskell on parallel recursive algorithms.

### quicksort

Creates a large list of uint64's and sorts it. Measures runtime evaluation speed for parallel recursion.

```
TODO
```

**Note:** benchmark already available on [HVM](https://github.com/kindelia/hvm)'s repo. Will be ported to Kind2 soon.

### composition

Composes a function an exponential amount of times. Measures runtime asymptotics.

```
TODO
```

**Note:** benchmark already available on [HVM](https://github.com/kindelia/hvm)'s repo. Will be ported to Kind2 soon.

### lambda_arithmetic

Performs arithmetic with λ-encoded ints. Measures runtime asymptotics.

```
TODO
```

**Note:** benchmark already available on [HVM](https://github.com/kindelia/hvm)'s repo. Will be ported to Kind2 soon.

Replicating
===========

To replicate these benchmarks, just run:

```
ulimit -s unlimited # give unlimited stack space
node benchmark.js
```

For specific commands, check the contents of [benchmark.js](benchmark.js).

Notes
=====

The original sources are available on [Checker](/Checker) and [Runtime](/Runtime). We attempt to make the test cases as
identical as possible. These tests are focused in comparing pure functional programming workloads, which mostly consist
of datatype allocation, pattern-matching, functions, closures and recursion. As such, it doesn't cover low-level
algorithms that are heavy on mutable array manipulation and in-place loops with mostly numeric workloads.

These benchmarks are focused on runtime and type-level evaluation performance. They do not cover elaboration (yet),
which is commonly a significant source of slowdown. Kind2 doesn't have a complex elaborator, thus, it doesn't suffer
from this problem; but, in turn, it is slightly more verbose than alternatives in certain areas. For a brilliantly
designed elaborator, check [smalltt](https://github.com/AndrasKovacs/smalltt) by András Kovács. Kind2 aims to, in a
future, incorporate insights from this work. Also, Kind2 does **not** have a totality checker (yet), so, it is arguably
doing less than alternatives; although the cost of the totality checker should be negligible in comparison to general
evaluation, but it is still worth noting.

Finally, these the type-checker all use Kind2's **Rust interpreter**. In theory, Kind2 could be compiled to machine
code, which would make it 3-4 faster, but the cost of C compilation would overshadow that. In a future, once we
implement incremental compilation, we might be able to pre-compile the type-checker, allowing only user-defined
functions to be injected. That would immediately make Kind2's numbers 3-4x lower. There are other incoming
optimizations on the making, including a complete GPU runtime!
