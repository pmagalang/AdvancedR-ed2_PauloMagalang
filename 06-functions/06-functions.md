---
title: 'Chapter 6: Functions'
author: "Paulo Magalang"
date: "2022-09-23"
output: 
  html_document: 
    keep_md: yes
---

# 6.1 Introduction

## Quiz

1. What are the three componenets of a function?

2. What does the following code return?


```r
x <- 10
f1 <- function(x) {
  function() {
    x + 10
  }
}
f1(1)()
```

```
## [1] 11
```

3. How would you usually write this code?


```r
`+`(1, `*`(2, 3))
```

```
## [1] 7
```


4. How could you make this call easier to read?


```r
mean(, TRUE, x = c(1:10, NA))
```

```
## [1] 5.5
```

5. Does the following code throw an error when executed? Why or why not?


```r
f2 <- function(a, b) {
  a * 10
}
f2(10, stop("This is an error!"))
```

```
## [1] 100
```

6. What is an infix function? How do you write it? What's a replacement function?
How do you write it?

7. How do you ensure that cleanup action occurs regardless of how a function exits?

# 6.2 Function fundamentals

* 3 function componenets: arguments, body, environment

* functions are objects

## 6.2.1 Function componenets

* 3 parts:
 
  - `formals()`: list of arguments
  
  - `body()`: code in function
  
  - `environment()`: determines how function finds values associated with the names, based on
  where the function is defined
  

```r
f02 <- function(x, y) {
  # A comment
  x + y
}

formals(f02)
```

```
## $x
## 
## 
## $y
```

```r
body(f02)
```

```
## {
##     x + y
## }
```

```r
environment(f02)
```

```
## <environment: R_GlobalEnv>
```


```r
# functions are objects -> functions also have attributes
attr(f02, "srcref") # scref == source reference, ppoints to the source code
```

```
## function(x, y) {
##   # A comment
##   x + y
## }
```

## 6.2.2 Primitive functions

* primitive functions call C code directly, doesn't have 3 componenets


```r
sum
```

```
## function (..., na.rm = FALSE)  .Primitive("sum")
```

```r
`[`
```

```
## .Primitive("[")
```


```r
typeof(sum) # builtin type
```

```
## [1] "builtin"
```

```r
typeof(`[`) # special type
```

```
## [1] "special"
```


```r
# primitive functions exist in C -> function componenets are NULL
formals(sum)
```

```
## NULL
```

```r
body(sum)
```

```
## NULL
```

```r
environment(sum)
```

```
## NULL
```

## 6.2.3 First-class functions

* first-classs functions: functions are objects (in R)


```r
# create function with function() and bind to a name (f01)
f01 <- function(x) {
  sin(1 / x ^ 2)
}
```

* anonymous function: a function without a name


```r
lapply(mtcars, function(x) length(unique(x)))
```

```
## $mpg
## [1] 25
## 
## $cyl
## [1] 3
## 
## $disp
## [1] 27
## 
## $hp
## [1] 22
## 
## $drat
## [1] 22
## 
## $wt
## [1] 29
## 
## $qsec
## [1] 30
## 
## $vs
## [1] 2
## 
## $am
## [1] 2
## 
## $gear
## [1] 3
## 
## $carb
## [1] 6
```

```r
Filter(function(x) !is.numeric(x), mtcars)
```

```
## data frame with 0 columns and 32 rows
```

```r
integrate(function(x) sin(x) ^ 2, 0, pi)
```

```
## 1.570796 with absolute error < 1.7e-14
```


```r
# can wrap functions into a list
funs <- list(
  half = function(x) x / 2,
  double = function(x) x * 2
)

funs$double(10)
```

```
## [1] 20
```

* closures: R functions capture their environments

## 6.2.4 Invoking a function

* use `do.call(function, arguments)` to invoke a function with the arguments in a data structure


```r
args <- list(1:10, na.rm = TRUE)
do.call(mean, args)
```

```
## [1] 5.5
```

## 6.2.5 Exercises

1. Given a name, like `"mean"`, `match.fun()` lets you find a funciton. Given a function,
can you find its name? Why doesn't that makes ense in R?


```r
match.fun(mean)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x0000029e3d41ae08>
## <environment: namespace:base>
```

```r
x <- mean
match.fun(x)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x0000029e3d41ae08>
## <environment: namespace:base>
```

```r
body(mean)
```

```
## UseMethod("mean")
```

```r
body(x)
```

```
## UseMethod("mean")
```

2. It's possible (although typically not useful) to call an anonymous function.
Which of the two approaches below is correct? Why?


```r
function(x) 3()
```

```
## function(x) 3()
```

```r
(function(x) 3)() # this approach is correct, function(x) 3 is the body,
```

```
## [1] 3
```

```r
                  # surrounding the body with parentheses and following it with
                  # () invokes the function
```

3. A good rule of thumb is that an anonymous function should fit on one line and shouldn’t need to use `{}`. 
Review your code. Where could you have used an anonymous function instead of a named function? 
Where should you have used a named function instead of an anonymous function?


```r
# goal was to extract read depths from a vcf file, very ugly nested str_remove call
# rip RDs from vcf
vcf.read_depths <- auto.vcf.data %>%
  mutate(across(all_of(sample_names), ~ as.integer(str_remove(str_remove(.x, "^[^:]*:"), ":.*$")))) %>%
  select(all_of(sample_names)) %>% pivot_longer(everything(), names_to = "sample", values_to = "read_depth")


# should rewrite like this
extract_read_depths <- function(format_str) {
  read_depth <- str_remove(format_str, "^[^:]*:") %>%
    str_remove(":.*$") %>% as.integer()
}
vcf.read_depths <- auto.vcf.data %>%
  mutate(across(all_of(sample_names), ~ extract_read_depths(.x))) %>%
  select(all_of(sample_names)) %>% pivot_longer(everything(), names_to = "sample", values_to = "read_depth")
# there might be a bug since .x can either be one string or a vector of strings, never tested
```

4. What function allows you to tell if an object is a function? What function allows
you to tell if a function is a primitive function?

* `is.function()` and `is.primitive()` will say

5. This code makes a list of all functions in the base package.


```r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.function, objs)
```

Use it to answer the following questions:

a. Which base function has the most arguments?
  

```r
params <- lapply(funs, formals)
param_values <- unlist(lapply(params, length))
which(param_values == max(param_values))
```

```
## scan 
## 1057
```

b. How many base functions have no arguments? What's special about those functions?


```r
sum(param_values == 0)
```

```
## [1] 253
```

```r
head(which(param_values == 0), n = 25)
```

```
##                    -                    !                   != 
##                    1                    4                    7 
##                    $                  $<-                   %% 
##                    8                   11                   13 
##                  %*%                  %/%                    & 
##                   14                   15                   19 
##                   &&                    (                    * 
##                   20                   23                   24 
##               ...elt            ...length             ...names 
##                   26                   27                   28 
##                   .C         .cache_class                .Call 
##                   35                   36                   37 
##       .Call.graphics              .class2            .External 
##                   38                   39                   56 
##   .External.graphics           .External2           .First.sys 
##                   57                   58                   59 
## .fixupGFortranStderr 
##                   60
```

```r
# Functions that have no arguments are all primitive
```

c. How could you adapt the code to find all primitive functions?


```r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.primitive, objs)

head(funs, n = 10)
```

```
## $`-`
## function (e1, e2)  .Primitive("-")
## 
## $`!`
## function (x)  .Primitive("!")
## 
## $`!=`
## function (e1, e2)  .Primitive("!=")
## 
## $`$`
## .Primitive("$")
## 
## $`$<-`
## .Primitive("$<-")
## 
## $`%%`
## function (e1, e2)  .Primitive("%%")
## 
## $`%*%`
## function (x, y)  .Primitive("%*%")
## 
## $`%/%`
## function (e1, e2)  .Primitive("%/%")
## 
## $`&`
## function (e1, e2)  .Primitive("&")
## 
## $`&&`
## .Primitive("&&")
```

6. What are the three important components of a function?

* `formals()`, `body()`, and `environment()`

7. When does printing a function not show the environment it was created in?

The environment is not shown for a function if it was defined in the global environment
or if the function is a primitive function.

# 6.3 Function composition

* nesting functions:

  - pros: concise
  
  - cons: long sequences hard to read, Dagwood sandwich problem
  
* save functions as intermediate objects:

  - pros: intermediates are important
  
  - cons: need to name objects, intermediates are not that important
  
* piping functions:

  - pros: read code in a left-to-right manner
  
  - cons: only use with linear sequences of transformations, assumes person reading
  your code understands piping

# 6.4 Lexical scoping

* lexical scoping in R follows 4 rules:

  - name masking
  
  - functions vs variables
  
  - a fresh start
  
  - dynamic lookup
  
## 6.4.1 Name masking

* name masking: names defined inside a function mask names defined out of a function


```r
x <- 10
y <- 20
g02 <- function() {
  x <- 1
  y <- 2
  c(x, y)
}
g02()
```

```
## [1] 1 2
```

```r
x
```

```
## [1] 10
```

```r
y
```

```
## [1] 20
```


```r
# names invoked within functions but are not defined said function are drawn from
# levels/layers outside of the function, up to the global environment and lastly
# loaded packages

x <- 2
g03 <- function() {
  y <- 1
  c(x, y)
}
g03()
```

```
## [1] 2 1
```

```r
y
```

```
## [1] 20
```


```r
x <- 1
g04 <- function() {
  y <- 2
  i <- function() {
    z <- 3
    c(x, y, z)
  }
  i() 
}
g04()
```

```
## [1] 1 2 3
```

```r
# 1. x is assigned 1 in the environment
# 2. function g04() is defined
# 3. function g04() is invoked
# 4. y is assigned 2 in g04()
# 5. function i() is defined
# 6. function i() is invoked
# 7. z is assigned 3 in i()
# 8. need vars x and y but they are not found within i(), move one level back
# 9. y is found but x is still needed, move one level back
# 10. x is found, output is 1 2 3
```

## 6.4.2 Functions versus variables

* scoping rules apply to functions as well


```r
g07 <- function(x) x + 1
g08 <- function() {
  g07 <- function(x) x + 100
  g07(10)
}
g08()
```

```
## [1] 110
```

* R ignores non-function objects when input as function arguments


```r
g09 <- function(x) x + 100
g10 <- function() {
  g09 <- 10
  g09(g09)
}
g10()
```

```
## [1] 110
```

## 6.4.3 A fresh start


```r
# wow its code that has similar behavior to other languages

g11 <- function() {
  if (!exists("a")) {
    a <- 1
  } else {
    a <- a + 1
  }
  a
}

g11()
```

```
## [1] 1
```

```r
g11()
```

```
## [1] 1
```

## 6.4.4 Dynamic lookup

* outputs of a function can differ depending on the objects outside the function's
scope


```r
# x as a global variable here

g12 <- function() x + 1
x <- 15
g12()
```

```
## [1] 16
```

```r
x <- 20
g12()
```

```
## [1] 21
```

* `codetools::findGlobals()` lists external dependencies within a function


```r
codetools::findGlobals(g12)
```

```
## [1] "+" "x"
```

* can empty the function's environment as a workaround


```r
environment(g12) <- emptyenv()
#g12()
```

## 6.4.5 Exercises

1. What does the following code return? Why? Describe how each of the three `c`'s is
interpreted.


```r
c <- 10
c(c = c)
```

```
##  c 
## 10
```

```r
# output is a named vector with an element 10 named c
# c() is the function call, c is the variable assigned the value 10, and
# the c parameter in c() is the name of the element
```

2. What are the four principles that govern how R looks for values?

name masking, prioritizing variable names, variables are instanced within functions
if it doesn't exist in the global environment, utilize global variables if invoked
in functions

3. What does the following function return? Make a prediction before running the code
yourself.


```r
f <- function(x) {
  f <- function(x) {
    f <- function() {
      x ^ 2
    }
    f() + 1 # look for x, evaluate x^2 + 1
  }
  f(x) * 2 # take previous output and multiply by 2
}
f(10)
```

```
## [1] 202
```

# 6.5 Lazy evaluation

* lazy evaluation: arguments are only evaluated if accessed


```r
h01 <- function(x) {
  10
}
h01(stop("This is an error!"))
```

```
## [1] 10
```

```r
h01() # argument x is never used in h01()
```

```
## [1] 10
```

## 6.5.1 Promises

* promises (or thunk) has 3 components:

  - an expression
  
  - an environment where the expression will be evaluated; when an argument is assigned
  inside a function, the variable is defined outside the function's environment
  
  - a value
  
## 6.5.2 Default arguments

* default function values can be defined in terms of other arguments or variables
defined later in the function


```r
h04 <- function(x = 1, y = x * 2, 
                z = a + b) { # this is annoying
  a <- 10
  b <- 100
  
  c(x, y, z)
}

h04()
```

```
## [1]   1   2 110
```


```r
h05 <- function(x = ls()) {
  a <- 1
  x
}

h05() # list objects in h05 environment
```

```
## [1] "a" "x"
```

```r
h05(ls()) # list objects in global environment
```

```
##  [1] "args"         "c"            "f"            "f01"          "f02"         
##  [6] "f1"           "f2"           "funs"         "g02"          "g03"         
## [11] "g04"          "g07"          "g08"          "g09"          "g10"         
## [16] "g11"          "g12"          "h01"          "h04"          "h05"         
## [21] "objs"         "param_values" "params"       "x"            "y"
```

## 6.5.3 Missing arguments

* use `missing()` to determine if an argument's value comes from the user or from a default
argument, but be careful with functions with many arguments


```r
h06 <- function(x = 10) {
  list(missing(x), x)
}
str(h06())
```

```
## List of 2
##  $ : logi TRUE
##  $ : num 10
```

```r
str(h06(10))
```

```
## List of 2
##  $ : logi FALSE
##  $ : num 10
```


```r
args(sample)
```

```
## function (x, size, replace = FALSE, prob = NULL) 
## NULL
```



```r
sample <- function(x, size = NULL, replace = FALSE, prob = NULL) {
  if (is.null(size)) {
    size <- length(x)
  }
  
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}

args(sample)
```

```
## function (x, size = NULL, replace = FALSE, prob = NULL) 
## NULL
```


```r
`%||%` <- function(lhs, rhs) {
  if (!is.null(lhs)) {
    lhs
  } else {
    rhs
  }
}

sample <- function(x, size = NULL, replace = FALSE, prob = NULL) {
  size <- size %||% length(x) # use LHS if not null, use right otherwise
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}
```

## 6.5.4 Exercises

1. What important property of `&&` makes `x_ok()` work?


```r
x_ok <- function(x) {
  !is.null(x) && length(x) == 1 && x > 0
}

x_ok(NULL)
```

```
## [1] FALSE
```

```r
x_ok(1)
```

```
## [1] TRUE
```

```r
x_ok(1:3)
```

```
## [1] FALSE
```


```r
x_ok <- function(x) {
  !is.null(x) & length(x) == 1 & x > 0
}

x_ok(NULL)
```

```
## logical(0)
```

```r
x_ok(1)
```

```
## [1] TRUE
```

```r
x_ok(1:3)
```

```
## [1] FALSE FALSE FALSE
```

`&` is the vectorized version of `&&` so a vector will always be output with `&`.


```r
# testing
x <- NULL

!is.null(x)
```

```
## [1] FALSE
```

```r
length(x) == 1
```

```
## [1] FALSE
```

```r
x > 0 # logical(0) output here
```

```
## logical(0)
```

```r
# mess with the order
!is.null(x) && length(x) == 1 && x > 0 # FALSE && FALSE && logical(0) --> FALSE
```

```
## [1] FALSE
```

```r
length(x) == 1 && !is.null(x) && x > 0 # same output as above
```

```
## [1] FALSE
```

```r
x > 0 && length(x) == 1 && !is.null(x) # same output
```

```
## [1] FALSE
```

```r
!is.null(x) & length(x) == 1 & x > 0
```

```
## logical(0)
```

```r
length(x) == 1 & !is.null(x) & x > 0
```

```
## logical(0)
```

```r
x > 0 & length(x) == 1 & !is.null(x)
```

```
## logical(0)
```

2. What does this function return? Why? Which principle does it illustrate?


```r
f2 <- function(x = z) {
  z <- 100
  x
}
f2()
```

```
## [1] 100
```

```r
# default value x was defined by a variable that will be defined in f2()
```

3. What does this function return? Why? Which principle does it illustrate?


```r
y <- 10

# f1() has its own function environment distinct from the global environment
f1 <- function(x = {y <- 1; 2}, # one liner to set y <- 1 and x = 2
               y = 0) { # default argument overwritten by `y <- 1` declaration
  c(x, y)
}

f1()
```

```
## [1] 2 1
```

```r
y # the global variable y has not been changed
```

```
## [1] 10
```

4. In `hist()`, the default value if `xlim` is `range(breaks)`, the default value for
`breaks` is `"Sturges"`, and


```r
range("Sturges")
```

```
## [1] "Sturges" "Sturges"
```

Explain how `hist()` works to get a correct `xlim` value.


5. Explain why this function works. Why is it confusing?


```r
show_time <- function(x = stop("Error!")) {
  stop <- function(...) Sys.time() # stop() redefined to call Sys.time() and not throw an error
  print(x)
}
show_time()
```

```
## [1] "2022-10-09 15:54:35 PDT"
```

6. How many arguments are required when calling `library()`?


```r
args(library)
```

```
## function (package, help, pos = 2, lib.loc = NULL, character.only = FALSE, 
##     logical.return = FALSE, warn.conflicts, quietly = FALSE, 
##     verbose = getOption("verbose"), mask.ok, exclude, include.only, 
##     attach.required = missing(include.only)) 
## NULL
```








