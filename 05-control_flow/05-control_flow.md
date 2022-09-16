---
title: 'Chapter 5: Control Flow'
author: "Paulo Magalang"
date: "2022-09-16"
output: 
  html_document: 
    keep_md: yes
---

# Quiz

1. What is the difference between `if` and `ifelse()`?

* `if` works for single values

* `ifelse()` is the vectorised version and can work with vector inputs

2. In the following code, what will the value of `y` be if `x` is `TRUE`? What if `x`
is `FALSE`? What if `x` is `NA`?


```r
y <- if (x) 3
```

* if `x` is `TRUE`, `y` is 3

* if `x` is `FALSE`, `y` is `NULL`

* if `x` is `NA`, an error is thrown since pseudo-numeric values are expected by `if`

3. What does `switch("x", x = , y = 2, z = 3)` return?

* 2 is returned due to "fall through" behavior

# 5.2 Choices

* `if` will return values 


```r
x1 <- if (TRUE) 1 else 2
x2 <- if (FALSE) 1 else 2

c(x1, x2)
```

```
## [1] 1 2
```

* `if` returns `NULL` if the condition is `FALSE`

## 5.2.1 Invalid inputs


```r
# strange inputs to if() will throw errors
if ("x") 1
if (logical()) 1
if (NA) 1
```


```r
# logical vectors with length > 1 will throw a warning instead
if (c(T, F)) 1
```


```r
# can turn this behavior into an error by setting an environment var
Sys.setenv("_R_CHECK_LENGTH_1_CONDITION_" = "true")
if (c(TRUE, FALSE)) 1
```

## 5.2.2 Vectorised if

* `ifelse(test, yes, no)`

* `dplyr::case_when()` useful instead of nested `ifelse()` statements


```r
x <- 1:10
dplyr::case_when(
  x %% 35 == 0 ~ "fizz buzz",
  x %% 5 == 0 ~ "fizz",
  x %% 7 == 0 ~ "buzz",
  is.na(x) ~ "???",
  TRUE ~ as.character(x)
)
```

```
##  [1] "1"    "2"    "3"    "4"    "fizz" "6"    "buzz" "8"    "9"    "fizz"
```

## 5.2.3 `switch()` statement


```r
# instead of:
x_option <- function(x) {
  if (x == "a") {
    "option 1"
  } else if (x == "b") {
    "option 2" 
  } else if (x == "c") {
    "option 3"
  } else {
    stop("Invalid `x` value")
  }
}

# can use a switch statement to make the code more compact
x_option <- function(x) {
  switch(x,
    a = "option 1",
    b = "option 2",
    c = "option 3",
    stop("Invalid `x` value") # last component should throw an error since 
                              # unmatched inputs will return NULL
  )
}
```


```r
# similar to C's switch statements, outputs will "fall through" to next value
legs <- function(x) {
  switch(x,
    cow = , # 4
    horse = , # 4
    dog = 4,
    human = , # 2
    chicken = 2,
    plant = 0,
    stop("Unknown input")
  )
}
legs("cow")
```

```
## [1] 4
```

```r
legs("dog")
```

```
## [1] 4
```

* recommend to use `switch()` with character inputs

## 5.2.4 Exercises

1. What type of vector does each of the following calls to `ifelse()` return?


```r
ifelse(TRUE, 1, "no") # double
```

```
## [1] 1
```

```r
ifelse(FALSE, 1, "no") # character
```

```
## [1] "no"
```

```r
ifelse(NA, 1, "no") # logical, NA is logical by default
```

```
## [1] NA
```

2. Why does the following code work?


```r
x <- 1:10 # length of x != 0
if (length(x)) "not empty" else "empty"
```

```
## [1] "not empty"
```

```r
x <- numeric() # length of x = 0
if (length(x)) "not empty" else "empty"
```

```
## [1] "empty"
```

```r
# booleans can be interpreted as 1 (for TRUE) and 0 (for FALSE)
# I'm assuming if() expects any integer input and will output TRUE for any nonzero value

# will if() be happy with double input?

if(3.2) 1 else 0 # double inputs are rounded down to the nearest whole integer
```

```
## [1] 1
```

```r
if(0.1) 1 else 0
```

```
## [1] 1
```

```r
if(-1) 1 else 0 # negative values are nonzero
```

```
## [1] 1
```

# 5.3 Loops

* `next` exits the current iteration of the `for` loop

* `break` exits the entire `for` loop


```r
for (i in 1:10) {
  if (i < 3) 
    next

  print(i)
  
  if (i >= 5)
    break
}
```

```
## [1] 3
## [1] 4
## [1] 5
```

## 5.3.1 Common pitfalls

* preallocate output object of a for loop if data is being generated, can use `vector()`


```r
means <- c(1, 50, 20)
out <- vector("list", length(means))
for (i in 1:length(means)) {
  out[[i]] <- rnorm(10, means[[i]])
}
```

* be careful for iterating over `1:length(x)` if `x` has length 0, use `seq_along()` instead


```r
means <- c()
out <- vector("list", length(means))
for (i in 1:length(means)) {
  out[[i]] <- rnorm(10, means[[i]])
}
```


```r
seq_along(means)
```

```
## [1] 1 2 3
```

```r
out <- vector("list", length(means))
for (i in seq_along(means)) {
  out[[i]] <- rnorm(10, means[[i]])
}
```

* loops will strip attributes, work around this using `[[`


```r
xs <- as.Date(c("2020-01-01", "2010-01-01"))
for (x in xs) {
  print(x)
}
```

```
## [1] 18262
## [1] 14610
```


```r
for (i in seq_along(xs)) {
  print(xs[[i]])
}
```

```
## [1] "2020-01-01"
## [1] "2010-01-01"
```

## 5.3.2 Related tools

* `while(condition) action`

* `repeat(action)`: action repeated until `break`

* no equivalent for do while loops in R

## 5.3.3 Exercises

1. Why does this code succeed without errors or warnings?


```r
x <- numeric() # length(x) is 0
out <- vector("list", length(x)) # empty list is initialized
for (i in 1:length(x)) { # i = 1, 0
  out[i] <- x[i] ^ 2
  # when i = 1, out[i] is NULL and x[i] is NA, out[i] is assigned NA; 
  # no out of bound error thrown since out[i] returns a list, not an element
  
  # when i = 0, out[i] is list() (empty list) and x[i] is numeric(0),
  # assignment to an empty object does not change the object
}
out
```

```
## [[1]]
## [1] NA
```

2. When the following code is evaluated, what can you say about the vector being iterated?


```r
xs <- c(1, 2, 3)
for (x in xs) { # the only values that x can be are 1, 2, and 3; updating xs after this declaration
                # is not "seen" after the fact
  xs <- c(xs, x * 2)
}
xs
```

```
## [1] 1 2 3 2 4 6
```

3. What does the following code tell you about when the index is updated?


```r
for (i in 1:3) { # i can take on values 1, 2, and 3
  i <- i * 2 # the value of i is updated
  print(i) 
  
  # but the updated values are not kept
}
```

```
## [1] 2
## [1] 4
## [1] 6
```

```r
i # since i is 6 here, the values of i are reassigned in the beginning of each iteration
```

```
## [1] 6
```







