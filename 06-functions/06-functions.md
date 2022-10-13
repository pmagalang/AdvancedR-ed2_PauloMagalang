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

* body, argument, environment

2. What does the following code return?


```r
x <- 10
f1 <- function(x) {
  function() {
    x + 10
  }
}
f1(1)() # output is 11, x is masked and is equal to 1
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

```r
2 * 3 + 1
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

```r
# mean(x = c(1:10, NA), na.rm = TRUE)
```

5. Does the following code throw an error when executed? Why or why not?

No because b is never utilized in f2.


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

* infix function: function is called between arguments

* replacement function: function that replace values by assignment

7. How do you ensure that cleanup action occurs regardless of how a function exits?

* use `on.exit()`

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
## <bytecode: 0x0000016c98006d68>
## <environment: namespace:base>
```

```r
x <- mean
match.fun(x)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x0000016c98006d68>
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
param_values <- unlist(lapply(params, length)) # sapply will return vector instead of list
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

`&` will run all boolean comparisons.


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
## [1] "2022-10-13 16:42:04 PDT"
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

# 6.6 `...` (dot-dot-dot)

* `...` allows functions to take any number of additional arguments, varargs in other
languages; also can use it to pass arguments to other functions


```r
i01 <- function(y, z) {
  list(y = y, z = z)
}

i02 <- function(x, ...) {
  i01(...)
}

str(i02(x = 1, y = 2, z = 3))
```

```
## List of 2
##  $ y: num 2
##  $ z: num 3
```

* can use `..N` to refer to elements of `...` by index

*`list(...)` evaluates arguments and stores them as a list


```r
i04 <- function(...) {
  list(...)
}
str(i04(a = 1, b = 2))
```

```
## List of 2
##  $ a: num 1
##  $ b: num 2
```

* uses of `...`:

  - pass additional arguments to a function if the function is an argument
  

```r
x <- list(c(1, 3, NA), c(4, NA, 6))
str(lapply(x, mean, na.rm = TRUE)) # na.rm is an argument for mean()
```

```
## List of 2
##  $ : num 2
##  $ : num 5
```
  
  - certain functions can take an arbitrary amount of arguments (ie: print)
  

```r
print(factor(letters), max.levels = 4)
```

```
##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
## 26 Levels: a b c ... z
```

```r
print(y ~ x, showEnv = TRUE)
```

```
## y ~ x
## <environment: R_GlobalEnv>
```
  
* downsides of `...`:

  - must be explicit in where additional arguments are utilized
  
  - misspelled arguments will not raise errors
  
## 6.6.1 Exercises

1. Explain the following results:


```r
# sum(..., na.rm = FALSE)
# mean(x, trim = 0, na.rm = FALSE, ...)

sum(1, 2, 3) # expected behavior
```

```
## [1] 6
```

```r
mean(1, 2, 3) # x = 1, mean of 1 is 1, trim set to 2, third argument is not used at all
```

```
## [1] 1
```

```r
sum(1, 2, 3, na.omit = TRUE) # argument name is na.rm not na.omit, TRUE is cast to 1
```

```
## [1] 7
```

```r
sum(1, 2, 3, na.omit = FALSE)
```

```
## [1] 6
```

```r
mean(1, 2, 3, na.omit = TRUE) # same typo on argument name, na.omit is passed in ...
```

```
## [1] 1
```


```r
# mean testing

# second argument is a numeric, value is set to trim
mean(iris$Sepal.Length)
```

```
## [1] 5.843333
```

```r
mean(iris$Sepal.Length, trim = 0.3)
```

```
## [1] 5.815
```

```r
mean(iris$Sepal.Length, 0.3)
```

```
## [1] 5.815
```

```r
# output is the same with third argument, argument is passed in ...
mean(iris$Sepal.Length, 0.3, 3)
```

```
## [1] 5.815
```

2. Explain how to find the documentation for the named arguments in the following function
call:

Named arguments in `plot` can be found in the documentation for `par()`.


```r
plot(1:10, col = "red", pch = 20, xlab = "x", col.lab = "blue")
```

3. Why does `plot(1:10, col = "red")` only color the points, not the axes or labels?
Read the source code of `plot.default()` to find out.

To change the color of axes or labels, you have to use the `col.axis` and `col.lab`
arguments (found in the `par()` documentation).

# 6.7 Exiting a function

* functions exit by returning a value or throwing an error

## 6.7.1 Implicit versus explicit returns

* two ways a function can return a value:

  - implicitly: last evaluated expression is the return value
  

```r
j01 <- function(x) {
  if (x < 10) {
    0
  } else {
    10
  }
}
j01(5)
```

```
## [1] 0
```

```r
j01(15)
```

```
## [1] 10
```
   - explicitly: call `return()`
   

```r
j02 <- function(x) {
  if (x < 10) {
    return(0)
  } else {
    return(10)
  }
}
```
   

## 6.7.2 Invisible values

* can prevent automatic printing when invoking a function with `invisible()`


```r
j04 <- function() invisible(1)
j04()
```


```r
# check that the function output actually exists

print(j04())
```

```
## [1] 1
```

```r
(j04())
```

```
## [1] 1
```


```r
# withVisible() returns the value and visibility flag
str(withVisible(j04()))
```

```
## List of 2
##  $ value  : num 1
##  $ visible: logi FALSE
```

* `<-` returns invisibly

* please don't chain variable assignments

## 6.7.3 Errors

* `stop()` throws errors, will terminate function execution


```r
j05 <- function() {
  stop("I'm an error")
  return(10)
}
j05()
```

## 6.7.4 Exit handlers

* `on.exit()` to set up an exit handler: changes to the global environment from a
function are reset


```r
j06 <- function(x) {
  cat("Hello\n")
  on.exit(cat("Goodbye!\n"), add = TRUE) 
  # add = TRUE necessary to retain previous exit handlers
  
  if (x) {
    return(10)
  } else {
    stop("Error")
  }
}

j06(TRUE)
j06(FALSE)
```


```r
cleanup <- function(dir, code) {
  old_dir <- setwd(dir)
  on.exit(setwd(old_dir), add = TRUE)
  
  old_opt <- options(stringsAsFactors = FALSE)
  on.exit(options(old_opt), add = TRUE)
}
```


```r
with_dir <- function(dir, code) {
  old <- setwd(dir)
  on.exit(setwd(old), add = TRUE)

  force(code)
}

getwd()
```

```
## [1] "C:/Users/paulo/Documents/Maloof/rclub/AdvancedR-ed2_PauloMagalang/06-functions"
```

```r
with_dir("~", getwd())
```

```
## [1] "C:/Users/paulo/Documents"
```


```r
j08 <- function() {
  on.exit(message("a"), add = TRUE)
  on.exit(message("b"), add = TRUE)
}
j08()
```

```
## a
```

```
## b
```


```r
j09 <- function() {
  on.exit(message("a"), add = TRUE, after = FALSE)
  on.exit(message("b"), add = TRUE, after = FALSE)
}
j09()
```

```
## b
```

```
## a
```

## 6.7.5 Exercises

1. What does `load()` return? Why don't you normally see these values?

`load()` loads in objects saved in `.RData` files. From the source code, it returns
another `load()` call wrapped in `.Internal()`. Not exactly sure how the recursive
call is broken but it probably has something to do with `.Internal()`. Values are
not printed since `verbose = FALSE` is default.

2. What does `write.table()` return? What would be more useful?

`write.table()` returns `NULL`. It would be more useful to print out some sort of
confirmation that the data frame was successfully written.


```r
df <- data.frame(x = 1, y = 2)
(write.table(df))
```

```
## "x" "y"
## "1" 1 2
```

```
## NULL
```

3. How does the `chdir` parameter of `source()` compare to `with_dir()`? Why might
you prefer one to the other?

`chdir` is a Boolean that is set to `FALSE` by default. When `chdir` is `TRUE`, the
working directory is changed to the filepath given by the `file` argument. `with_dir()`
does the same thing, but the filepath is explicitly defined by the user instead of
being taken by another function argument. If the code needs to be executed in a different
directory than the R script is in, than `with_dir()` is preferred since it is flexible.

4. Write a function that opens a graphics device, runs the supplied code, and closes
the graphics device (always, regardless of whether or not the plotting code works). 


```r
q4_fxn <- function() {
    windows()
    on.exit(dev.off(), add = TRUE)
}
#q4_fxn()
```

5. We can use `on.exit()` to implement a simple version of `capture.output()`.


```r
capture.output2 <- function(code) {
  temp <- tempfile() # generate path to tempfile
  on.exit(unlink(temp), add = TRUE, after = TRUE) # delete tempfile

  sink(temp) # generate the tempfile, redirect console outputs to tempfile
  on.exit(sink(), add = TRUE, after = TRUE) # outputs redirected back to console

  force(code)
  readLines(temp)
}
capture.output2(cat("a", "b", "c", sep = "\n"))
```

```
## [1] "a" "b" "c"
```

Compare `capture.output()` to `capture.output2()`. How do the functions differ?
What features have I removed to make the key ideas easier to see? How have I rewritten
the key ideas so they're easier to understand?


```r
capture.output
```

```
## function (..., file = NULL, append = FALSE, type = c("output", 
##     "message"), split = FALSE) 
## {
##     type <- match.arg(type)
##     rval <- NULL
##     closeit <- TRUE
##     if (is.null(file)) 
##         file <- textConnection("rval", "w", local = TRUE)
##     else if (is.character(file)) 
##         file <- file(file, if (append) 
##             "a"
##         else "w")
##     else if (inherits(file, "connection")) {
##         if (!isOpen(file)) 
##             open(file, if (append) 
##                 "a"
##             else "w")
##         else closeit <- FALSE
##     }
##     else stop("'file' must be NULL, a character string or a connection")
##     sink(file, type = type, split = split)
##     on.exit({
##         sink(type = type, split = split)
##         if (closeit) close(file)
##     })
##     for (i in seq_len(...length())) {
##         out <- withVisible(...elt(i))
##         if (out$visible) 
##             print(out$value)
##     }
##     on.exit()
##     sink(type = type, split = split)
##     if (closeit) 
##         close(file)
##     if (is.null(rval)) 
##         invisible(NULL)
##     else rval
## }
## <bytecode: 0x0000016c98153370>
## <environment: namespace:utils>
```

# 6.8 Function forms

* 4 types of function calls:

  - prefix: function name comes before arguments
  
  - infix: function name comes between arguments
  
  - replacement: functions that replace values by assignment
  
  - special: examples - `[[`, `if`, `for`
  
## 6.8.1 Rewriting to prefix form


```r
x + y
`+`(x, y)

names(df) <- c("x", "y", "z")
`names<-`(df, c("x", "y", "z"))

for(i in 1:10) print(i)
`for`(i, 1:10, print(i))
```


```r
# equivalent outputs

add <- function(x, y) x + y # custom function
lapply(list(1:3, 4:5), add, 3)
```

```
## [[1]]
## [1] 4 5 6
## 
## [[2]]
## [1] 7 8
```

```r
lapply(list(1:3, 4:5), `+`, 3) # using existing `+`
```

```
## [[1]]
## [1] 4 5 6
## 
## [[2]]
## [1] 7 8
```

## 6.8.2 Prefix form

* specify arguments in prefix form in 3 ways:

  - by position
  
  - using partial matching, can output waring with `warnPartialMatchArgs` option
  
  - by name
  

```r
k01 <- function(abcdef, bcde1, bcde2) {
  list(a = abcdef, b1 = bcde1, b2 = bcde2)
}

# by position
str(k01(1, 2, 3))
```

```
## List of 3
##  $ a : num 1
##  $ b1: num 2
##  $ b2: num 3
```

```r
# by name: match has priority
str(k01(2, 3, abcdef = 1))
```

```
## List of 3
##  $ a : num 1
##  $ b1: num 2
##  $ b2: num 3
```

```r
# partial matching: match has priority
str(k01(2, 3, a = 1))
```

```
## List of 3
##  $ a : num 1
##  $ b1: num 2
##  $ b2: num 3
```

```r
# But this doesn't work because abbreviation is ambiguous
#str(k01(1, 3, b = 1))
```

## 6.8.3 Infix functions

* can create custom infix functions that start and end with `%`, can escape special
characters with `\` when defining the function


```r
`%+%` <- function(a, b) paste0(a, b)
"new " %+% "string"
```

```
## [1] "new string"
```


```r
`% %` <- function(a, b) paste(a, b)
`%/\\%` <- function(a, b) paste(a, b)

"a" % % "b"
```

```
## [1] "a b"
```

```r
"a" %/\% "b"
```

```
## [1] "a b"
```

* infix operators are always composed left to right

* `+` and `-` can be called with a single argument

## 6.8.4 Replacement functions

* functions that modify arguments in place, have name `xxx<-`, have
arguments `x` and `value`, and return modified object


```r
`second<-` <- function(x, value) {
  x[2] <- value
  x
}

x <- 1:10
second(x) <- 5L # second element modified
x
```

```
##  [1]  1  5  3  4  5  6  7  8  9 10
```

* copy on modify behavior


```r
library(lobstr)

x <- 1:10
tracemem(x)
```

```
## [1] "<0000016C9C32D0A8>"
```

```r
second(x) <- 6L
```

```
## tracemem[0x0000016c9c32d0a8 -> 0x0000016c9bf98e18]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
## tracemem[0x0000016c9bf98e18 -> 0x0000016c9c255528]: second<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>
```

```r
untracemem(x)
```

* if additional arguments are needed for a replacement function, put them between
`x` and `value`


```r
`modify<-` <- function(x, position, value) {
  x[position] <- value
  x
}

modify(x, 1) <- 10
x
```

```
##  [1] 10  6  3  4  5  6  7  8  9 10
```

```r
# behind the scenes, call is converted to
x <- `modify<-`(x, 1, 10)
```

## 6.8.5 Special forms

* examples: parentheses, subsetting, control flow, `function` functions

* implemented in C

## 6.8.6 Exercises

1. Rewrite the following code snippets into prefix form:


```r
1 + 2 + 3
```

```
## [1] 6
```

```r
`+`(`+`(1, 2), 3)
```

```
## [1] 6
```

```r
1 + (2 + 3)
```

```
## [1] 6
```

```r
`+`(1, `+`(2, 3))
```

```
## [1] 6
```

```r
x <- 1:10
n <- 1
if (length(x) <= 5) x[[5]] else x[[n]]
```

```
## [1] 1
```

```r
`if`(length(x) <= 5, `[[`(x, 5), `[[`(x, n))
```

```
## [1] 1
```

2. Clarify the following list of odd function calls:


```r
#x <- sample(replace = TRUE, 20, x = c(1:10, NA))
x <- sample(x = c(1:10, NA), size = 20, replace = TRUE)

#y <- runif(min = 0, max = 1, 20)
y <- runif(n = 20, min = 0, max = 1)

#cor(m = "k", y = y, u = "p", x = x)
cor(x = x, y = y, use = "pairwise.complete.obs", method = "kendall")
```

```
## [1] -0.397236
```

3. Explain why the following code fails:


```r
`modify<-` <- function(x, position, value) {
  x[position] <- value
  x
}

modify(get("x"), 1) <- 10

# call is converted to:
get("x") <- `modify<-`(get("x"), 1, 10)

# I don't think get() is meant to be a replacement function
```

4. Create a replacement function that modifies a random location in a vector.


```r
`random_replace<-` <- function(x, value) {
    index <- sample(x, size = length(x))[1]
    x[index] <- value
    x
}

x <- 1:10
random_replace(x) <- 999
x
```

```
##  [1]   1   2   3 999   5   6   7   8   9  10
```

5. Write your own version of `+` that pastes its inputs together if they are character
vectors but behaves as usual otherwise.


```r
`+` <- function(a, b) {
    if (is.character(a) | is.character(b)) {
        return(paste0(a, b))
    }
    else {
        return(sum(a, b))
    }
}

1 + 2
```

```
## [1] 3
```

```r
"a" + "b"
```

```
## [1] "ab"
```

6. Create a list of all the replacement functions found in the base package. Which
ones are primitive functions? (Hint: use `apropos()`.)


```r
replacement_functions <- apropos("<-$", mode = "function")
is_replacement_function <- sapply(replacement_functions, is.primitive)

replacement_functions[is_replacement_function]
```

```
## character(0)
```

7. What are valid names for user-created infix functions?

Valid names for user-created infix functions are names that start and end with `%`.

8. Create an infix `xor()` operator.

XOR is defined as: (P OR Q) AND NOT (P AND Q) (from wikipedia)


```r
`%xor%` <- function(a, b) {
    (a | b) & !(a & b)
}

# test
FALSE %xor% FALSE
```

```
## [1] FALSE
```

```r
FALSE %xor% TRUE
```

```
## [1] TRUE
```

```r
TRUE %xor% FALSE
```

```
## [1] TRUE
```

```r
TRUE %xor% TRUE
```

```
## [1] FALSE
```

9. Create infix versions of the set functions `intersect()`, `union()`, and
`setdiff()`. You might call them `%n%`, `%u%`, and `%/%` to match conventions
from mathematics.


```r
`%n%` <- function(a, b) {
    intersect(a, b)
}

`%u%` <- function(a, b) {
    union(a, b)
}

`%/%` <- function(a, b) {
    setdiff(a, b)
}

a <- c(1, 2, 4, 5)
b <- c(2, 3, 4)

a %n% b
```

```
## [1] 2 4
```

```r
a %u% b
```

```
## [1] 1 2 4 5 3
```

```r
a %/% b
```

```
## [1] 1 5
```

```r
b %/% a
```

```
## [1] 3
```
















