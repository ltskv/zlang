# The Interpreter for Z-Language, written in Scheme0

## What's Scheme0?

It is a toy language from the Chapter 5 of an (awesome)
[book](https://www.itu.dk/people/sestoft/pebook/)
on partial evaluation, for which the authors of the book have published a
partial evaluator, written in, you guessed it, Scheme0.

## Why would you even care about Scheme0?

Well, having a partial evaluator for a language, written in the language,
allows you to unleash the full power of the
[Futamura Projections](https://fi.ftmr.info/).

## Futamura Projections

**Pre-reqs**

`mix`: A Partial Evaluator (a.k.a. Specializer):
- receiving programs in a language S0
- and a part of their input;
- outputting programs in S0;
- and ideally itself written in S0.

`p_S0`: A Program:
- written in S0;
- taking some input `i`, that can be split into `i_1` and `i_2`
(feel free to leave one of these empty);
- producing some `output`.
- Like this:

```
output = p_S0(i_1, i_2)
```

`z_int`: An interpreter for some language Z:
- written in S0;
- taking some program in language Z as input;
- also taking the program's input `i`;
- producing the `result` of "running" `p_z` on `i`.
- Just like that:

```
result = z_int(p_z, i)
```

**A side note (you may skip it, because it might be dreadfully misguiding):**

You might be wondering what's so special about S0, that the S0-programs can
just "run", whereas Z-programs require an interpreter written in S0 to run.
The short answer is idk lol))0). A longer answer, *which I totally don't claim
to be correct and use it at your own risk you have been warned*, is that we
just happen to have access to a "runtime" for S0, that can
just "run" S0 programs. Such "runtime" might also exist for Z, but we
don't have access to it or don't want to use it. Here is a metaphor that helps
me wrap my mind around this concept: suppose you can email a code for an
S0-program and input for the program to an undergrad student, who happens to
understand S0, and who will email you back with a result. Suppose there is
also a student who understands Z, but that student is kinda busy with whatever
undergrad students may be busy with. But you totally need to run some Z code.
So you conjure up an interpreter for Z in S0, and send the helpful student this
interpreter with instructions to use your Z code and input to your Z code as
input for the Z interpreter (which is an S0 program). This will work, but might
seem as a bit of a hassle, as all you really care about is your Z code and not
the interpreter, so it's kinda silly to keep sending it every time you want
some Z code evaluated. Wouldn't it be nice to be able to automatically
translate Z code into equivalent S0 code, and just "run" it?

### The Specialization Equation

```
output = p_S0(i_1, i_2) = mix(p_S0, i_1) (i_2)
```

### The First Futamura Projection

```
result = z_int(p_z, i) = mix(z_int, p_z) (i)

p_z_C = mix(z_int, p_z)
result = p_z_C(i)
```

So, according to **The Specialization Equation**, `p_z_C` is a program in S0,
which has the same behavior as `p_z`. Whoa, we just compiled our program from Z
to S0! Wait, there's more.

### The Second Futamura Projection

```
p_z_C = mix(z_int, p_z) = mix(mix, z_int) (p_z)

z_compiler = mix(mix, z_int)
p_z_C = z_compiler(p_z)
```

Looks like we have a compiler now! But wait, there's even more!

### The Third Futamura Projection

```
z_compiler = mix(mix, z_int) = mix(mix, mix) (z_int)

compiler_generator = mix(mix, mix)
z_compiler = compiler_generator(z_int)
py_compiler = compiler_generator(py_int)  # py_int must be written in S0 though
```

So, `mix(mix, mix)` is a compiler generator. Totally makes sense.

## What's your Z language then?

You can check out the semantics in [zlang.tex](./zlang.tex).
To compile the LaTeX, you will
need a `defs.tex` from [here](https://www.cs.ubc.ca/~rxg/cpsc509/), and a bunch
of packages, installable via CTAN. The code for the interpreter is,
obviously, is in [zlang.s0](./zlang.s0).

## Now I want to try it myself!

1. Install [Gambit Scheme](http://gambitscheme.org/wiki/index.php/Main_Page).
You can use `brew install gambit-scheme` on the Mac, but it doesn't link the
needed binary into `/usr/local/bin`, so you will have to locate the executable,
which is called `gsi`, yourself (try `/usr/local/Cellar/gambit-scheme/...`). 
2. Download the source code for Scheme0 and the Scheme0 Specializer from the
[book's webpage](https://www.itu.dk/people/sestoft/pebook/) (Chapter 5).
3. Copy `zlang.s0` into the directory with the book code.
4. Go to that directory.
5. Run `gsi` and feel free to try these snippets:

```scheme
(load "scheme0.ss")
(create)
(load "zlang.s0")
;; enough boilerplate!

;; You now have access to Z-interpreter as z-int
;; And it's code as z-int-code
;; btw. SORRY for - vs _ inconsistencies :(

;; let's define a Z program
(define p_z '(++ 4 (i-t-e x (++ y 5) (++ y x))))

;; run it!
;; btw. the interpreter happens to need a list of variable names
;; and a list of the corresponding numerical values
;; in addition to the p_z source code
(z-int p_z '(x y) '(4 5))
;; Should output 14 maybe...

;; Now do the Futamura Projections

;; This has something to do with 'Annotations'
;; and 'Binding-Time Analysis', for which I will
;; just direct you to the Chapter 5 of The Book
(define z-int-SSD (monotate z-int-code '(S S D)))

;; Now compile the p_z
(define p_z_C-code (spec z-int-SSD (list p_z '(x y))))
p_z_C-code  ;; You can check out the code and see what's what

(make 'p_z_C p_z_C-code)
(p_z_C '(4 5))
;; Should still be 14...

;; Now to the Futamura 2
(define spec-SD (monotate specializer '(S D)))
(define z_compiler-code (spec spec-SD (list z-int-SSD)))
(make 'z_compiler z_compiler-code)
(make 'p_z_C_too (z_compiler (list p_z '(x y))))
(p_z_C_too '(4 5))
;; Should be 14

;; And the third one!
(make 'cogen (spec spec-SD (list spec-SD)))
(make 'z_compiler_too (cogen (list z-int-SSD)))
(make 'p_z_C_too_2 (z_compiler_too (list p_z '(x y))))
(p_z_C_too_2 '(4 5))
;; 14?
```

There is also a file in the book code archive, cryptically called
`run.121002-1`, in which the authors of Scheme0 provide some of their own
examples.

**Thanks for reading :)**
