# The Interpreter for Z-Language, written in Scheme0

## What's Scheme0?

It is a toy language from the Chapter 5 of an (awesome)
[book](https://www.itu.dk/people/sestoft/pebook/)
on partial evaluation, for which the authors of the book have published a
partial evaluator, written in, well, Scheme0.

## Why would you even care about Scheme0?

Well, having a partial evaluator for a language, written in the language allows
you to unleash the full power of the
[Futamura Projections](https://fi.ftmr.info/), which we well provide here for
the sake of completeness.

### Futamura Projections

**Pre-reqs**

`mix`: A Partial Evaluator (a.k.a. Specializer):
- taking programs in a language S0
- and a part of their input;
- outputting programs in S0;
- and ideally itself written in S0.

`p_S0`: A Program:
- written in S0;
- taking some input `i`, that can be split into `i_1` and `i_2`
(feel free to leave one of these empty);
- producing some `output`.

Like this:

```
output = p_S0(i_1, i_2)
```

`z_int`: An interpreter for some language Z:
- written in S0;
- taking some program in language Z as input;
- also taking the program's input `i`;
- producing the `result` of running `p_z` on `i`.

Just like that:

```
result = z_int(p_z, i)
```

**The Specialization Equation**

```
output = p_S0(i_1, i_2) = mix(p_S0, i_1) (i_2)
```

**The First Futamura Projection**

```
result = z_int(p_z, i) = mix(z_int, p_z) (i)

p_z_C = mix(z_int, p_z)
result = p_z_C(i)
```

So, according to definition of `mix`, `p_z_C` is a program in S0, which has the
same behavior as `p_z`, which was written in Z. Whoa, we just compiled our
program from Z to S0! Wait, there's more.

**The Second Futamura Projection**

```
p_z_C = mix(z_int, p_z) = mix(mix, z_int) (p_z)

z_compiler = mix(mix, z_int)
p_z_C = z_compiler(p_z)
```

Looks like we have a compiler now! But wait, there's even more!

**The Third Futamura Projection**

```
z_compiler = mix(mix, z_int) = mix(mix, mix) (z_int)

compiler_generator = mix(mix, mix)
z_compiler = compiler_generator(z_int)
py_compiler = compiler_generator(py_int)  # py_int must be written in S0 though
```

So, `mix(mix, mix)` is a compiler generator. Totally makes sense.

## What's your Z language then?

You can check out the semantics in [zlang.tex](./zlang.tex).
To compile it, you will
need a `defs.tex` from [here](https://www.cs.ubc.ca/~rxg/cpsc509/), and a bunch
of LaTeX packages, installable via CTAN. The code for the interpreter is,
obviously, is in [zlang.s0](./zlang.s0).

## Okay, cool, now I see. How do I try it for myself?

1. Download [Gambit Scheme](http://gambitscheme.org/wiki/index.php/Main_Page).
You can use `brew install gambit-scheme` on the Mac, but it doesn't link the
interpreter into `/usr/local/bin`, so you will have to locate the executable,
which is called `gsi`. 

2. Download the source code for Scheme0 and the Scheme0 Specializer from the
[book's webpage](https://www.itu.dk/people/sestoft/pebook/).

3. Probably copy `zlang.s0` into the main directory of the book code.

4. Go to that directory.

5. Run `gsi` (you'll figure something out).

```scheme
(load "scheme0.ss")
(create)
(load "zlang.s0")

;; You now have access to Z-interpreter as z-int
;; btw. SORRY for - vs _ inconsistencies :(
;; And it's code as z-int-code

;; enough boilerplate!
;; let's define a Z program
(define p_z '(++ 4 (i-t-e x (++ y 5) (++ y x))))

;; run it!
(z-int p_z '(x y) '(4 5))  ;; pass a list of var names and their vals
;; Should output something like 14 maybe...

;; Now do the Futamura Projections (1)

;; This says that this should be specializable on the program code
;; and the list of variable names
;; while taking the variable values as the dynamic input
(define z-int-SSD (monotate z-int-code '(S S D)))

;; Now compile the p_z
(define p_z_C-code (spec z-int-SSD (list p_z '(x y))))
;; You can check out the code and see what's what
(make 'p_z_C p_z_C-code)
(p_z_C '(4 5))
;; Should still be 14...

;; Now to the Futamura 2
(define spec-SD (monotate specializer '(S D)))
(define z_compiler-code (spec spec-SD (list z-int-SSD)))
(make 'z_compiler z_compiler-code)
(make 'p_z_C_too (z_compiler (list p_z '(x y))))
(p_z_C_too '(4 5))
;; should be 14

;; And the third one!
(make 'cogen (spec spec-SD (list spec-SD)))
(make 'z_compiler_too (cogen (list z-int-SSD)))
(make 'p_z_C_too_2 (z_compiler_too (list p_z '(x y))))
(p_z_C_too_2 '(4 5))
;; 14? idk...
```

There is also a file, cryptically called `run.121002-1`, in which the authors
of Scheme0 provide some of their own examples.

**Thanks for reading :)**
