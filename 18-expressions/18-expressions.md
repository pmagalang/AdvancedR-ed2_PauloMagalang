---
title: 'Chapter 18: Expressions'
author: "Paulo Magalang"
date: "2023-07-05"
output: 
  html_document: 
    keep_md: yes
---


```r
library(lobstr)
library(rlang)
```

# 18.1 Introduction

* way to separate expressions vs execution of expression? -> use `rlang::expr()`


```r
z <- rlang::expr(y <- x * 10)
z
```

```
## y <- x * 10
```


```r
x <- 4
eval(z) # evaluate expressions with `base::eval()`
y
```

```
## [1] 40
```

# 18.2 Abstract syntax trees (ASTs)

## 18.2.1 Drawing

* can draw ASTs with `lobstr::ast()`


```r
lobstr::ast(f(x, "y", 1))
```

```
## █─f 
## ├─x 
## ├─"y" 
## └─1
```

* leaves of tree are symbols (in purple and rounded corners) or constants (black 
borders and square corners)

* branches of tree are objects (ie: function calls) and represented as orange rectangles


```r
lobstr::ast(f(g(1, 2), h(3, 4, i())))
```

```
## █─f 
## ├─█─g 
## │ ├─1 
## │ └─2 
## └─█─h 
##   ├─3 
##   ├─4 
##   └─█─i
```

## 18.2.2 Non-code components

* only one place where whitespace affects ASTs


```r
lobstr::ast(y <- x)
```

```
## █─`<-` 
## ├─y 
## └─x
```


```r
lobstr::ast(y < -x)
```

```
## █─`<` 
## ├─y 
## └─█─`-` 
##   └─x
```

## 18.2.3 Infix calls


```r
# recall both lines of code are equivalent
y <- x * 10
`<-`(y, `*`(x, 10))
```



```r
lobstr::ast(y <- x * 10)
```

```
## █─`<-` 
## ├─y 
## └─█─`*` 
##   ├─x 
##   └─10
```

## 18.2.4 Exercises

1. Reconstruct the code represented by the trees below:


```r
lobstr::ast(f(g(h())))
```

```
## █─f 
## └─█─g 
##   └─█─h
```


```r
lobstr::ast(1 + 2 + 3)
```

```
## █─`+` 
## ├─█─`+` 
## │ ├─1 
## │ └─2 
## └─3
```

```r
# how are parentheses included in ASTs?
#lobstr::ast((1 + 2) + 3)
```


```r
lobstr::ast((x + y) * z)
```

```
## █─`*` 
## ├─█─`(` 
## │ └─█─`+` 
## │   ├─x 
## │   └─y 
## └─z
```

2. Draw the following trees by hand and then check your answers with `lobstr::ast()`.


```r
f(g(h(i(1, 2, 3))))
f(1, g(2, h(3, i())))
f(g(1, 2), h(3, i(4, 5)))
```


3. What's happening with the ASTs below? (Hint: carefully read `?"^"`.)


```r
lobstr::ast(`x` + `y`)
```

```
## █─`+` 
## ├─x 
## └─y
```

```r
# in prefix form
```


```r
lobstr::ast(x ** y)
```

```
## █─`^` 
## ├─x 
## └─y
```

```r
# `**` and `^` are equivalent
```


```r
lobstr::ast(1 -> x)
```

```
## █─`<-` 
## ├─x 
## └─1
```

```r
# x <- 1 and 1 -> x are equivalent
```

4. What is special about the AST below?


```r
lobstr::ast(function(x = 1, y = 2) {})
```

```
## █─`function` 
## ├─█─x = 1 
## │ └─y = 2 
## ├─█─`{` 
## └─<inline srcref>
```

```r
# x and y are arguments for function
# {} contains the code for the function (is it hidden in the AST if not empty?)

lobstr::ast(function(x = 1, y = 2) {x + y})
```

```
## █─`function` 
## ├─█─x = 1 
## │ └─y = 2 
## ├─█─`{` 
## │ └─█─`+` 
## │   ├─x 
## │   └─y 
## └─<inline srcref>
```

```r
# not sure what `<inline srcref>` leaf refers to. source ref?
```

5. What does the call tree of an `if` statement with multiple `else if` conditions 
look like? Why?


```r
lobstr::ast(
  if (x == 0) {}
  else if (x < 1) {}
  else if (x >= 1) {}
)
```

```
## █─`if` 
## ├─█─`==` 
## │ ├─x 
## │ └─0 
## ├─█─`{` 
## └─█─`if` 
##   ├─█─`<` 
##   │ ├─x 
##   │ └─1 
##   ├─█─`{` 
##   └─█─`if` 
##     ├─█─`>=` 
##     │ ├─x 
##     │ └─1 
##     └─█─`{`
```

```r
# else if is equivalent to nested if statements
```


# 18.3 Expressions

## 18.3.1 Constants

* constants are either `NULL` or a length-1 atomic vector


```r
identical(expr(TRUE), TRUE)
```

```
## [1] TRUE
```

```r
identical(expr(1), 1)
```

```
## [1] TRUE
```

```r
identical(expr(2L), 2L)
```

```
## [1] TRUE
```

```r
identical(expr("x"), "x")
```

```
## [1] TRUE
```

## 18.3.2 Symbols

* symbols represent the name of an object

* can create symbols by capturing code that references an object with `expr()` or 
turning a string into a symbol with `rlang::sym()`


```r
expr(x)
```

```
## x
```

```r
sym("x")
```

```
## x
```

* can turn a symbol back into a string with `as.character()` or `rlang::as_string()`


```r
as_string(expr(x))
```

```
## [1] "x"
```


```r
str(expr(x))
```

```
##  symbol x
```

```r
is.symbol(expr(x))
```

```
## [1] TRUE
```

## 18.3.3 Calls

* call objects represent a captured function call, are special lists


```r
lobstr::ast(read.table("important.csv", row.names = FALSE))
```

```
## █─read.table 
## ├─"important.csv" 
## └─row.names = FALSE
```

```r
x <- expr(read.table("important.csv", row.names = FALSE))

typeof(x)
```

```
## [1] "language"
```

```r
is.call(x)
```

```
## [1] TRUE
```

## 18.3.3.1 Subsetting

* calls behave like lists


```r
x[[1]]
```

```
## read.table
```

```r
is.symbol(x[[1]])
```

```
## [1] TRUE
```


```r
as.list(x[-1])
```

```
## [[1]]
## [1] "important.csv"
## 
## $row.names
## [1] FALSE
```

```r
x[[2]]
```

```
## [1] "important.csv"
```

```r
x$row.names
```

```
## [1] FALSE
```

```r
length(x) - 1 # determine number of arguments
```

```
## [1] 2
```


```r
# standardize all arguments to use the full name, useful for extracting specific arguments from a call
rlang::call_standardise(x)
```

```
## read.table(file = "important.csv", row.names = FALSE)
```

## 18.3.3.2 Function position

* function position is the first element of the call object, contains the function that will 
be called


```r
lobstr::ast(foo())
```

```
## █─foo
```

```r
lobstr::ast("foo"())
```

```
## █─foo
```


```r
lobstr::ast(pkg::foo(1))
```

```
## █─█─`::` 
## │ ├─pkg 
## │ └─foo 
## └─1
```

```r
lobstr::ast(obj$foo(1))
```

```
## █─█─`$` 
## │ ├─obj 
## │ └─foo 
## └─1
```

```r
lobstr::ast(foo(1)(2))
```

```
## █─█─foo 
## │ └─1 
## └─2
```

## 18.3.3.3 Constructing

* can construct a call object using `rlang::call2()`


```r
call2("mean", x = expr(x), na.rm = TRUE)
```

```
## mean(x = x, na.rm = TRUE)
```

```r
call2(expr(base::mean), x = expr(x), na.rm = TRUE)
```

```
## base::mean(x = x, na.rm = TRUE)
```

```r
call2("<-", expr(x), 10)
```

```
## x <- 10
```

## 18.3.5 Exercises

1. Which two of the six types of atomic vector can't appear in an expression? Why?
Similarly, why can't you create an expression that contains an atomic vector of length
greater than one?

Complex and raw atomic vectors cannot appear in an expression.

You cannot create an expression that contains an atomic vector of length greater
than one because it is impossible to create an atomic vector of length greater than
one without a function call (ie: `c()`, `:`).


2. What happens when you subset a call object to remove the first element? Why?


```r
expr(read.csv("foo.csv", header = TRUE))[-1]
```

```
## "foo.csv"(header = TRUE)
```

The first element of the call object is the function position. Because we subset the call
object to remove the first element, "foo.csv" is treated as the function to be executed.

3. Describe the differences between the following call objects.


```r
x <- 1:10

call2(median, x, na.rm = TRUE)
```

```
## (function (x, na.rm = FALSE, ...) 
## UseMethod("median"))(1:10, na.rm = TRUE)
```

```r
call2(expr(median), x, na.rm = TRUE)
```

```
## median(1:10, na.rm = TRUE)
```

```r
call2(median, expr(x), na.rm = TRUE)
```

```
## (function (x, na.rm = FALSE, ...) 
## UseMethod("median"))(x, na.rm = TRUE)
```

```r
call2(expr(median), expr(x), na.rm = TRUE)
```

```
## median(x, na.rm = TRUE)
```

```r
# depends on what is being evaluated
# median: median function is evaluated which is why we see the extra
#         lines
# expr(median): median function is not evaluated
# x: argument becomes 1:10; expr(x): argument is x
```

4. `rlang::call_standardize()` doesn't work so well for the following calls. Why?
What makes `mean()` special?


```r
call_standardise(quote(mean(1:10, na.rm = TRUE)))
```

```
## mean(x = 1:10, na.rm = TRUE)
```

```r
call_standardise(quote(mean(n = T, 1:10)))
```

```
## mean(x = 1:10, n = T)
```

```r
call_standardise(quote(mean(x = 1:10, , TRUE)))
```

```
## mean(x = 1:10, , TRUE)
```


```r
mean
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000001f335f2a1b8>
## <environment: namespace:base>
```

`mean` has `...` as an argument so arguments are not lazily evaluated and can allow
more arguments.

5. Why does this code not make sense?


```r
x <- expr(foo(x = 1))
names(x)
```

```
## [1] ""  "x"
```

```r
names(x) <- c("x", "y")

x
```

```
## foo(y = 1)
```

```r
as.list(x)
```

```
## $x
## foo
## 
## $y
## [1] 1
```

`x` is now in the function position but the name of the function does not change.
`y` becomes the argument name.

5. Construct the expression `if(x > 1) "a" else "b"` using multiple calls to
`call2()`. How does the code structure reflect the structure of AST?


```r
lobstr::ast(expr(if(x > 1) "a" else "b"))
```

```
## █─expr 
## └─█─`if` 
##   ├─█─`>` 
##   │ ├─x 
##   │ └─1 
##   ├─"a" 
##   └─"b"
```


```r
call2("if", call2(">", expr(x), 1), "a", "b")
```

```
## if (x > 1) "a" else "b"
```

```r
# call2(function, param, param) params are leaves of tree
```

# 18.4 Parsing and grammar

## 18.4.1 Operator precednece

* operator precedence = PEMDAS in math


```r
lobstr::ast(1 + 2 * 3)
```

```
## █─`+` 
## ├─1 
## └─█─`*` 
##   ├─2 
##   └─3
```


```r
lobstr::ast(!x %in% y)
```

```
## █─`!` 
## └─█─`%in%` 
##   ├─x 
##   └─y
```

## 18.4.2 Associativity

* most operators are left-associative with exceptions in exponentiation and
assignment


```r
lobstr::ast(1 + 2 + 3)
```

```
## █─`+` 
## ├─█─`+` 
## │ ├─1 
## │ └─2 
## └─3
```

```r
lobstr::ast(2^2^3)
```

```
## █─`^` 
## ├─2 
## └─█─`^` 
##   ├─2 
##   └─3
```

```r
lobstr::ast(x <- y <- z)
```

```
## █─`<-` 
## ├─x 
## └─█─`<-` 
##   ├─y 
##   └─z
```

## 18.4.3 Parsing and deparsing

* can convert code stored as a string into a call with `rlang::parse_expr()`;
multiple expressions with `rlang::parse_exprs()`


```r
x1 <- "y <- x + 10"
x1
```

```
## [1] "y <- x + 10"
```

```r
is.call(x1)
```

```
## [1] FALSE
```

```r
x2 <- rlang::parse_expr(x1)
x2
```

```
## y <- x + 10
```

```r
is.call(x2)
```

```
## [1] TRUE
```

* base equivalent of `parse_exprs()` is `parse()`


```r
as.list(parse(text = x1))
```

```
## [[1]]
## y <- x + 10
```

* deparsing: turn an expression back into a string


```r
z <- expr(y <- x + 10)
expr_text(z)
```

```
## [1] "y <- x + 10"
```

## 18.4.4 Exercises

1. R uses parentheses in two slightly different ways as illustrated by these two
calls:


```r
f((1))
`(`(1 + 1)
```

Compare and contrast the two uses by referencing the AST.


```r
lobstr::ast(f((1)))
```

```
## █─f 
## └─█─`(` 
##   └─1
```


```r
lobstr::ast(`(`(1 + 1))
```

```
## █─`(` 
## └─█─`+` 
##   ├─1 
##   └─1
```


```r
# let's make it easier to compare both
lobstr::ast(((1 + 1)))
```

```
## █─`(` 
## └─█─`(` 
##   └─█─`+` 
##     ├─1 
##     └─1
```

There's an extra subtree in the third example which indicates that the `(` function
was called twice. This suggests that the inner parentheses in the second example
is only used for syntax.

2. `=` can also be used in two ways. Construct a simple example that shows both uses.


```r
# variable assignment (like with <-)
x = 1

# argument assignment in a function
mean(x = 1:10)
```

```
## [1] 5.5
```

3. Does `-2^2` yield 4 or -4? Why?


```r
-2^2
```

```
## [1] -4
```

```r
# will return -4 because exponentials are not left-associative
```

4. What does `!1 + !1` return? Why?


```r
!1 + !1
```

```
## [1] FALSE
```

```r
# returns FALSE because 1 = TRUE and 0 = FALSE, integer cast to logical with ! operator
```

5. Why does `x1 <- x2 <- x3 <- 0` work? Describe the two reasons.

* reason 1: assignment is not left-associative

not sure on the second reason

6. Compare ASTs of `x + y %+% z` and `x ^ y %+% z`. What have you learned about
the precedence of custom infix functions?


```r
lobstr::ast(x + y %+% z)
```

```
## █─`+` 
## ├─x 
## └─█─`%+%` 
##   ├─y 
##   └─z
```


```r
lobstr::ast(x ^ y %+% z)
```

```
## █─`%+%` 
## ├─█─`^` 
## │ ├─x 
## │ └─y 
## └─z
```

```r
# custom infix functions have less precedence than exponentiation
```

7. What happens if you call `parse_expr()` with a string that generates multiple
expressions? e.g. `parse_expr("x + 1; y + 1")`


```r
parse_expr("x + 1; y + 1")
# throws error: `x` must contain exactly 1 expression, not 2.
```

8. What happens if you attempt to parse an invalid expression? e.g. `"a +"` or `"f())"`


```r
parse_expr("a +")
# unexpected end of input error
```

9. `deparse()` produced vectors when the input is long. For example, the following call
produced a vector of length two:


```r
expr <- expr(g(a + b + c + d + e + f + g + h + i + j + k + l + 
  m + n + o + p + q + r + s + t + u + v + w + x + y + z))

deparse(expr)
```

```
## [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + "
## [2] "    p + q + r + s + t + u + v + w + x + y + z)"
```

What does `expr_text()` do instead?


```r
expr_text(expr)
```

```
## [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + \n    p + q + r + s + t + u + v + w + x + y + z)"
```

```r
# adds a newline
```

10. `pairwise.t.test()` assumes that `deparse()` always returns a length one character
vector. Can you construct an input that violates this expectation? What happens?


```r
pairwise.t.test(1, x +
                  y)
```

```
## 
## 	Pairwise comparisons using t tests with pooled SD 
## 
## data:  1 and x + y 
## 
## <0 x 0 matrix>
## 
## P value adjustment method: holm
```

```r
# looks like nothing broke...
```

# 18.5 Walking AST with recursive functions


```r
expr_type <- function(x) {
  if (rlang::is_syntactic_literal(x)) {
    "constant"
  } else if (is.symbol(x)) {
    "symbol"
  } else if (is.call(x)) {
    "call"
  } else if (is.pairlist(x)) {
    "pairlist"
  } else {
    typeof(x)
  }
}

switch_expr <- function(x, ...) {
  switch(expr_type(x),
    ...,
    stop("Don't know how to handle type ", typeof(x), call. = FALSE)
  )
}
```


## 18.5.1 Finding F and T


```r
logical_abbr_rec <- function(x) {
  switch_expr(x,
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T")
  )
}

logical_abbr_rec(expr(TRUE))
```

```
## [1] FALSE
```

```r
logical_abbr_rec(expr(T))
```

```
## [1] TRUE
```


```r
# wrapper
logical_abbr <- function(x) {
  logical_abbr_rec(enexpr(x))
}

logical_abbr(T)
```

```
## [1] TRUE
```

```r
logical_abbr(FALSE)
```

```
## [1] FALSE
```


```r
logical_abbr_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T"),

    # Recursive cases
    call = ,
    pairlist = purrr::some(x, logical_abbr_rec)
  )
}

logical_abbr(mean(x, na.rm = T))
```

```
## [1] TRUE
```

```r
logical_abbr(function(x, na.rm = T) FALSE)
```

```
## [1] TRUE
```

## 18.5.2 Finding all variables created by assignment


```r
find_assign_rec <- function(x) {
  switch_expr(x,
    constant = ,
    symbol = character()
  )
}
find_assign <- function(x) find_assign_rec(enexpr(x))

find_assign("x")
```

```
## character(0)
```

```r
find_assign(x)
```

```
## character(0)
```


```r
flat_map_chr <- function(.x, .f, ...) {
  purrr::flatten_chr(purrr::map(.x, .f, ...))
}

flat_map_chr(letters[1:3], ~ rep(., sample(3, 1)))
```

```
## [1] "a" "a" "a" "b" "c" "c"
```


```r
find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(as.list(x), find_assign_rec),
    call = {
      if (is_call(x, "<-")) {
        as_string(x[[2]]) # return second element of <- call
      } else {
        flat_map_chr(as.list(x), find_assign_rec)
      }
    }
  )
}

find_assign(a <- 1)
```

```
## [1] "a"
```

```r
find_assign({
  a <- 1
  {
    b <- 2
  }
})
```

```
## [1] "a" "b"
```


```r
# break the code
find_assign({
  a <- 1
  a <- 2
})
```

```
## [1] "a" "a"
```


```r
# wrapper to deal with duplicates
find_assign <- function(x) unique(find_assign_rec(enexpr(x)))

find_assign({
  a <- 1
  a <- 2
})
```

```
## [1] "a"
```


```r
# deal with chained <-
find_assign_call <- function(x) {
  if (is_call(x, "<-") && is_symbol(x[[2]])) {
    lhs <- as_string(x[[2]])
    children <- as.list(x)[-1]
  } else {
    lhs <- character()
    children <- as.list(x)
  }

  c(lhs, flat_map_chr(children, find_assign_rec))
}

find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(x, find_assign_rec),
    call = find_assign_call(x)
  )
}

find_assign(a <- b <- c <- 1)
```

```
## [1] "a" "b" "c"
```

```r
find_assign(system.time(x <- print(y <- 5)))
```

```
## [1] "x" "y"
```

## 18.5.3 Exercises

1. `logical_abbr()` returns `TRUE` for `T(1, 2, 3)`. How could you modify
`logical_abbr_rec()` so that it ignores function calls that use `T` or `F`?


```r
# need to add call case in recursion
T_call <- function(x) {
  if (is_call(x, "T") | is_call(x, "F")) { # check if T or F are used as function calls
    x <- as.list(x)[-1]
    purrr::some(x, logical_abbr_rec)
  } else { # treat same as pairlist
    purrr::some(x, logical_abbr_rec)
  }
}

logical_abbr_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T"),

    # Recursive cases
    call = T_call(x),
    pairlist = purrr::some(x, logical_abbr_rec)
  )
}
```

2. `logical_abbr()` works with expressions. It currently fails when you give it a
function. Why? How could you modify `logical_abbr()` to make it work? What components
of a function will you need to recurse over?


```r
logical_abbr_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T"),

    # Recursive cases
    call = ,
    pairlist = purrr::some(x, logical_abbr_rec)
  )
}

logical_abbr <- function(x) {
  logical_abbr_rec(enexpr(x))
}

# would need to add recursive case to iterate through each expression in a function
# i have no idea how to do that
```

3. Modify `find_assign` to also detect assignment using replacement functions,
i.e. `names(x) <- y`.


```r
x <- expr(names(x) <- y)
as.list(x)
```

```
## [[1]]
## `<-`
## 
## [[2]]
## names(x)
## 
## [[3]]
## y
```

```r
is_call(x[[2]])
```

```
## [1] TRUE
```



```r
find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(as.list(x), find_assign_rec),
    call = {
      if (is_call(x, "<-")) {
        # check if second element is a call
        if (is_call(x[[2]])) {
            as_string(x[[3]]) # return 3rd element if second element is a call
          } else {
            as_string(x[[2]])
          }
      } else {
        flat_map_chr(as.list(x), find_assign_rec)
      }
    }
  )
}

find_assign <- function(x) find_assign_rec(enexpr(x))
```

4. Write a function that extracts all calls to a specified function.


```r
x <- expr(sum(mean(1:4)))
as.list(x)
```

```
## [[1]]
## sum
## 
## [[2]]
## mean(1:4)
```

```r
as.character(as.list(x)[[2]]) # i want this as one string
```

```
## [1] "mean" "1:4"
```

```r
expr_text(as.list(x)[[2]])
```

```
## [1] "mean(1:4)"
```



```r
find_assign_call <- function(x) {
  if (is_call(x)) { # check if its any call
    lhs <- expr_text(as.list(x)[[2]])
    children <- as.list(x)[-1]
  } else {
    lhs <- character()
    children <- as.list(x)
  }

  c(lhs, flat_map_chr(children, find_assign_rec))
}

find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(x, find_assign_rec),
    call = find_assign_call(x)
  )
}
```




















