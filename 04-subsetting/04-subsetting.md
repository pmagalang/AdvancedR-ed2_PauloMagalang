---
title: 'Chapter 4: Subsetting'
author: "Paulo Magalang"
date: "2022-09-13"
output: 
  html_document: 
    keep_md: yes
---

# Quiz

1. What is the result of subsetting a vector with positive integers, negative
integers, a logical vector, or a character vector?

* positive int: selects at position

* negative int: drops at position

* logical: selects when `TRUE`

* character: name must be matching

2. What is the difference between `[`, `[[`, and `$` when applied to a list?

* `[` returns a list

* `[[` returns the list element

* `$` is shorthand for `[[`

3. When should you use `drop = FALSE`?

* use when you want to preserve the original dimensions of the matrix or dataframe

4. If `x` is a matrix, what does `x[] <- 0` do? How is it different from `x <- 0`?

* `x[] <- 0` reassigns each matrix element to 0

* `x <- 0` replaces the matrix with the value 0

5. How can you use a named vector to relabel categorical variables?

* you can create a lookup table to rename categorical values with another character/string

# 4.2 Selecting multiple elements

* 6 ways to subset a vector:

  - positive integers return elements at a given index; can give dupes for indices
  and the floor of doubles are taken when input as an index
  
  - exclude elements with negative integers, but can't mix and match positive and 
  negative integers in one call
  
  - elements returned when corresponding logical value is `TRUE`, be careful of recycling
  
  - not including an index returns the original vector
  
  - zero returns a zero-length vector
  
  - can use character vectors to subset a named vector
  
## 4.2.2 Lists

* `[` gives a list output

* `[[` and `$` will pull an element of a list by index or name

## 4.2.3 Matrices and arrays

* can subset high-dimensional structures with: multiple vectors, one vector, a matrix

* subsetting with `[` drops the results to the lowest possible dimension


```r
a <- matrix(1:9, nrow = 3)
colnames(a) <- c("A", "B", "C")

a[1, ] # 1d output
```

```
## A B C 
## 1 4 7
```


```r
# subsetting with one vector
vals <- outer(1:5, 1:5, FUN = "paste", sep = ",")
vals
```

```
##      [,1]  [,2]  [,3]  [,4]  [,5] 
## [1,] "1,1" "1,2" "1,3" "1,4" "1,5"
## [2,] "2,1" "2,2" "2,3" "2,4" "2,5"
## [3,] "3,1" "3,2" "3,3" "3,4" "3,5"
## [4,] "4,1" "4,2" "4,3" "4,4" "4,5"
## [5,] "5,1" "5,2" "5,3" "5,4" "5,5"
```

```r
vals[c(4, 15)] # arrays are stored in column-major order (unlike python/java)
```

```
## [1] "4,1" "5,3"
```


```r
# subsetting with another matrix
# each row specifies a "coordinate" in the larger matrix

select <- matrix(ncol = 2, byrow = TRUE, c(
  1, 1, # row 1 col 1
  3, 1, # row 3 col 1
  2, 4 # row 2 col 4
))
vals[select]
```

```
## [1] "1,1" "3,1" "2,4"
```

## 4.2.4 Data frames and tibbles

* data frames act both like lists and matrices

  - list behavior with one index and selects by column
  
  - matrix behavior with two indices and selects by row and column


```r
df <- data.frame(x = 1:3, y = 3:1, z = letters[1:3])

# There's an important difference if you select a single 
# column: matrix subsetting simplifies by default, list 
# subsetting does not.
str(df["x"]) # still a list
```

```
## 'data.frame':	3 obs. of  1 variable:
##  $ x: int  1 2 3
```

```r
str(df[, "x"]) # simplified to 1d vector
```

```
##  int [1:3] 1 2 3
```

* subsetting a tibble with `[` will return a tibble

## 4.2.5 Preserving dimensionality

* subsetting a matrix or data frame with a single value will simplify the output,
can preserve dimensionality with `drop = FALSE`


```r
a <- matrix(1:4, nrow = 2)
str(a[1, ])
```

```
##  int [1:2] 1 3
```

```r
str(a[1, , drop = FALSE])
```

```
##  int [1, 1:2] 1 3
```


```r
df <- data.frame(a = 1:2, b = 1:2)
str(df[, "a"])
```

```
##  int [1:2] 1 2
```

```r
str(df[, "a", drop = FALSE])
```

```
## 'data.frame':	2 obs. of  1 variable:
##  $ a: int  1 2
```

* `drop = TRUE` is the source of many bugs, get in the habit of using `drop = FALSE` when
subsetting 2D objects

* `drop = FALSE` is default with tibbles

* `drop` with factors controls if levels are preserved, `FALSE` by default

## 4.2.6 Exercises

1. Fix each of the following common data frame subsetting errors:


```r
#mtcars[mtcars$cyl = 4, ]
mtcars[mtcars$cyl == 4, ]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
#mtcars[-1:4, ]
mtcars[-c(1:4), ]
```

```
##                      mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Hornet Sportabout   18.7   8 360.0 175 3.15 3.440 17.02  0  0    3    2
## Valiant             18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
## Duster 360          14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
## Merc 240D           24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230            22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Merc 280            19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
## Merc 280C           17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
## Merc 450SE          16.4   8 275.8 180 3.07 4.070 17.40  0  0    3    3
## Merc 450SL          17.3   8 275.8 180 3.07 3.730 17.60  0  0    3    3
## Merc 450SLC         15.2   8 275.8 180 3.07 3.780 18.00  0  0    3    3
## Cadillac Fleetwood  10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
## Lincoln Continental 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
## Chrysler Imperial   14.7   8 440.0 230 3.23 5.345 17.42  0  0    3    4
## Fiat 128            32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic         30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla      33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona       21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Dodge Challenger    15.5   8 318.0 150 2.76 3.520 16.87  0  0    3    2
## AMC Javelin         15.2   8 304.0 150 3.15 3.435 17.30  0  0    3    2
## Camaro Z28          13.3   8 350.0 245 3.73 3.840 15.41  0  0    3    4
## Pontiac Firebird    19.2   8 400.0 175 3.08 3.845 17.05  0  0    3    2
## Fiat X1-9           27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2       26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa        30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Ford Pantera L      15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4
## Ferrari Dino        19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
## Maserati Bora       15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8
## Volvo 142E          21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
#mtcars[mtcars$cyl <= 5]
mtcars[mtcars$cyl <= 5, ]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
#mtcars[mtcars$cyl == 4 | 6, ]
mtcars[mtcars$cyl == 4 | mtcars$cyl == 6, ]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4      21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag  21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive 21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
## Valiant        18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Merc 280       19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
## Merc 280C      17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Ferrari Dino   19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

2. What does the following code yield five missing values? (Hint: why is it different from
`x[NA_real_]`?)


```r
x <- 1:5
x[NA]
```

```
## [1] NA NA NA NA NA
```

```r
x[NA_real_]
```

```
## [1] NA
```

```r
# NA is a logical by default and is recycled when subsetting using x[NA]
# NA_real_ is a double and doubles are truncated when used to subset a vector which is
# why there is only one output

typeof(NA)
```

```
## [1] "logical"
```

```r
typeof(NA_real_)
```

```
## [1] "double"
```

3. What does `upper.tri()` return? How does subsetting a matrix with it work? Do we
need any additional subsetting rules to describe its behavior?


```r
x <- outer(1:5, 1:5, FUN = "*")
x[upper.tri(x)]
```

```
##  [1]  2  3  6  4  8 12  5 10 15 20
```

`upper.tri()` returns a logical matrix with the elements in the upper triangular of the matrix 
true (excluding the main diagonal). Subsetting a matrix with a logical matrix will pull all of
the indices from the matrix that are `TRUE` in the corresponding logical matrix. Subsetting
this way will return a vector instead of a matrix, however.

4. Why does `mtcars[1:20]` return an error? How does it differ form the similar `mtcars[1:20, ]`?


```r
dim(mtcars)
```

```
## [1] 32 11
```

`mtcars[1:20]` will return the first 20 columns of `mtcars`. However, looking at the dimensions of
`mtcars` there are only 11 columns present. `mtcars[1:20, ]` will return the first 20 rows of the
data frame.

5. Implement your own function that extracts the diagonal entries from a matrix (it should
behave like `diag(x)` where `x` is a matrix).


```r
get_diag <- function(x) {
    # can't assume x is a square matrix, take smallest dimension of x if rectangular matrix
    # generate vector of indices for selection matrix
    indices <- rep(1:min(dim(x)), each = length(dim(x)))
    
    # subset matrix by matrix
    select <- matrix(ncol = length(dim(x)), byrow = TRUE, indices)
    x[select]
}

ex <- outer(1:5, 1:5, FUN = "*")
get_diag(ex)
```

```
## [1]  1  4  9 16 25
```

6. What does `df[is.na(df)] <- 0` do? How does it work?

It replaces `NA` values in `df` with 0. `is.na(df)` outputs a logical
matrix where it is `TRUE` if there is an `NA` present in a given position
of the data frame. That logical matrix is used to subset the dataframe.

# 4.3 Selecting a single element

## 4.3.1 `[[` for single items

* subsetting a list with `[` will return a list, `[[` will return the element itself

* using a vector with `[[` will subset recursively, use `purr::pluck()` instead to pull
multiple elements from a list

* use `[[` with atomic vectors as well to reinforce single values are being extracted/changed

## 4.3.2 `$`

* `$` shorthand; example: `x$y` roughly equivalent to `x[["y"]]`

* use `[[` if trying to access a column but the colname is stored as a variable


```r
var <- "cyl"
# Doesn't work - mtcars$var translated to mtcars[["var"]]
mtcars$var
```

```
## NULL
```

```r
# Instead use [[
mtcars[[var]]
```

```
##  [1] 6 6 4 6 8 6 8 4 4 6 6 8 8 8 8 8 8 4 4 4 4 8 8 8 8 4 4 4 8 6 8 4
```

* `$` does partial matching, set `warnPartialMatchDollar = TRUE` under global options
to avoid this behavior

## 4.4.4 Missing and out-of-bounds indices

* `purrr::pluck()` always returns `NULL` and `purrr::chuck()` will throw an error


```r
x <- list(
  a = list(1, 2, 3),
  b = list(3, 4, 5)
)

purrr::pluck(x, "a", 1)
```

```
## [1] 1
```

```r
purrr::pluck(x, "c", 1)
```

```
## NULL
```

```r
purrr::pluck(x, "c", 1, .default = NA)
```

```
## [1] NA
```

## 4.3.4 `@` and `slot()`

* `@` is equivalent to `$` and `slot()` is equivalent to `[[` for S4 objects

## 4.3.5 Exercises

1. Brainstorm as many ways as possible to extract the thrid value from the `cyl` varialbe
in the `mtcars` dataset.


```r
mtcars[3, 2]
```

```
## [1] 4
```

```r
mtcars$cyl[[3]]
```

```
## [1] 4
```

```r
mtcars[, 2][[3]]
```

```
## [1] 4
```

```r
mtcars[, "cyl"][[3]]
```

```
## [1] 4
```

```r
mtcars[3, ]$cyl
```

```
## [1] 4
```

```r
mtcars[3, "cyl"]
```

```
## [1] 4
```

2. Given a linear model, e.g., `mod <- lm(mpg ~ wt, data = mtcars)`, extract the
residual degrees of freedom. Then extract the R squared from the model summary (`summary(mod)`)


```r
mod <- lm(mpg ~ wt, data = mtcars)

#str(mod)
mod$df.residual
```

```
## [1] 30
```

```r
#str(summary(mod))
summary(mod)$r.squared
```

```
## [1] 0.7528328
```

# 4.4 Subsetting and assignment

* subassignment: simultaneously subset and modify values of a vector, `x[i] <- value`


```r
x <- 1:5
x[c(1, 2)] <- c(101, 102) # reassign values to first two elements
x
```

```
## [1] 101 102   3   4   5
```

* recycling rules are messy so make sure `length(value) == length(x[i]` and `i` is unique

* use `x[[i]] <- NULL` to remove elements and `x[i] <- list(NULL)` to add `NULL` elements


```r
x <- list(a = 1, b = 2)
x[["b"]] <- NULL
str(x)
```

```
## List of 1
##  $ a: num 1
```

```r
y <- list(a = 1, b = 2)
y["b"] <- list(NULL)
str(y)
```

```
## List of 2
##  $ a: num 1
##  $ b: NULL
```

* subsetting with empty indices preserves the structure of the original object


```r
mtcars[] <- lapply(mtcars, as.integer)
is.data.frame(mtcars)
```

```
## [1] TRUE
```

```r
mtcars <- lapply(mtcars, as.integer)
is.data.frame(mtcars)
```

```
## [1] FALSE
```

# 4.5 Applications

## 4.5.1 Lookup tables (character subsetting)


```r
x <- c("m", "f", "u", "f", "f", "m", "m")

# create named vector
lookup <- c(m = "Male", f = "Female", u = NA)
lookup[x]
```

```
##        m        f        u        f        f        m        m 
##   "Male" "Female"       NA "Female" "Female"   "Male"   "Male"
```

```r
# unname() to remove names
unname(lookup[x])
```

```
## [1] "Male"   "Female" NA       "Female" "Female" "Male"   "Male"
```

## 4.5.2 Matching and merging by hand (integer subsetting)


```r
grades <- c(1, 2, 2, 3, 1)

# external key
info <- data.frame(
  grade = 3:1,
  desc = c("Excellent", "Good", "Poor"),
  fail = c(F, F, T)
)

# match(needle, haystack) to associate grades vector with the key
id <- match(grades, info$grade) 
id
```

```
## [1] 3 2 2 1 3
```

```r
info[id, ]
```

```
##     grade      desc  fail
## 3       1      Poor  TRUE
## 2       2      Good FALSE
## 2.1     2      Good FALSE
## 1       3 Excellent FALSE
## 3.1     1      Poor  TRUE
```

```r
# match() reminds me of family of join functions...
```

## 4.5.3 Random samples and bootstraps (integer subsetting)


```r
set.seed(95616)
df <- data.frame(x = c(1, 2, 3, 1, 2), y = 5:1, z = letters[1:5])

# Randomly reorder
df[sample(nrow(df)), ]
```

```
##   x y z
## 1 1 5 a
## 5 2 1 e
## 3 3 3 c
## 2 2 4 b
## 4 1 2 d
```

```r
# Select 3 random rows
df[sample(nrow(df), 3), ]
```

```
##   x y z
## 3 3 3 c
## 5 2 1 e
## 1 1 5 a
```

```r
# Select 6 bootstrap replicates
df[sample(nrow(df), 6, replace = TRUE), ] # allow dupes, take 6 samples
```

```
##     x y z
## 2   2 4 b
## 4   1 2 d
## 3   3 3 c
## 5   2 1 e
## 3.1 3 3 c
## 2.1 2 4 b
```

## 4.5.4 Ordering (integer subsetting)

* `order()` returns an int vector describing how to order the subsetted vector


```r
x <- c("b", "c", "a")
order(x)
```

```
## [1] 3 1 2
```

```r
x[order(x)]
```

```
## [1] "a" "b" "c"
```


```r
# Randomly reorder df
df2 <- df[sample(nrow(df)), 3:1]
df2
```

```
##   z y x
## 3 c 3 3
## 5 e 1 2
## 4 d 2 1
## 2 b 4 2
## 1 a 5 1
```

```r
df2[order(df2$x), ] # reorder rows by values in x
```

```
##   z y x
## 4 d 2 1
## 1 a 5 1
## 5 e 1 2
## 2 b 4 2
## 3 c 3 3
```

```r
df2[, order(names(df2))] # reorder columns by colnames
```

```
##   x y z
## 3 3 3 c
## 5 2 1 e
## 4 1 2 d
## 2 2 4 b
## 1 1 5 a
```

## 4.5.5 Expanding aggregated counts (integer subsetting)


```r
df <- data.frame(x = c(2, 4, 1), y = c(9, 11, 6), n = c(3, 5, 1))
rep(1:nrow(df), df$n) # repeat x[i] y[i] times
```

```
## [1] 1 1 1 2 2 2 2 2 3
```

```r
df[rep(1:nrow(df), df$n), ]
```

```
##     x  y n
## 1   2  9 3
## 1.1 2  9 3
## 1.2 2  9 3
## 2   4 11 5
## 2.1 4 11 5
## 2.2 4 11 5
## 2.3 4 11 5
## 2.4 4 11 5
## 3   1  6 1
```

## 4.5.6 Removing columns from data frames (character)


```r
# remove column by setting to NULL
df <- data.frame(x = 1:3, y = 3:1, z = letters[1:3])
df$z <- NULL
df
```

```
##   x y
## 1 1 3
## 2 2 2
## 3 3 1
```


```r
# exclude column by subsetting
df <- data.frame(x = 1:3, y = 3:1, z = letters[1:3])
df[c("x", "y")]
```

```
##   x y
## 1 1 3
## 2 2 2
## 3 3 1
```


```r
# can also use set operations
df[setdiff(names(df), "z")]
```

```
##   x y
## 1 1 3
## 2 2 2
## 3 3 1
```

## 4.5.7 Selecting rows based on a condition (logical subsetting)


```r
rm(mtcars) # mtcars was modified by a previous example
mtcars[mtcars$gear == 5, ]
```

```
##                 mpg cyl  disp  hp drat    wt qsec vs am gear carb
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.7  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.9  1  1    5    2
## Ford Pantera L 15.8   8 351.0 264 4.22 3.170 14.5  0  1    5    4
## Ferrari Dino   19.7   6 145.0 175 3.62 2.770 15.5  0  1    5    6
## Maserati Bora  15.0   8 301.0 335 3.54 3.570 14.6  0  1    5    8
```

```r
mtcars[mtcars$gear == 5 & mtcars$cyl == 4, ]
```

```
##                mpg cyl  disp  hp drat    wt qsec vs am gear carb
## Porsche 914-2 26.0   4 120.3  91 4.43 2.140 16.7  0  1    5    2
## Lotus Europa  30.4   4  95.1 113 3.77 1.513 16.9  1  1    5    2
```

## 4.5.8 Boolean algebra versus sets (logical and integer)

* set operations more effective when finding the first/last `TRUE` or when
there are few `TRUE`s and many `FALSE`s

* use `which()` to convert a boolean to integer


```r
x <- sample(10) < 4
which(x)
```

```
## [1] 1 3 4
```

```r
unwhich <- function(x, n) {
  out <- rep_len(FALSE, n)
  out[x] <- TRUE
  out
}
unwhich(which(x), 10)
```

```
##  [1]  TRUE FALSE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
```


```r
(x1 <- 1:10 %% 2 == 0)
```

```
##  [1] FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE
```

```r
(x2 <- which(x1))
```

```
## [1]  2  4  6  8 10
```

```r
(y1 <- 1:10 %% 5 == 0)
```

```
##  [1] FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE  TRUE
```

```r
(y2 <- which(y1))
```

```
## [1]  5 10
```

```r
# X & Y <-> intersect(x, y)
x1 & y1
```

```
##  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE
```

```r
intersect(x2, y2)
```

```
## [1] 10
```

```r
# X | Y <-> union(x, y)
x1 | y1
```

```
##  [1] FALSE  TRUE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE  TRUE
```

```r
union(x2, y2)
```

```
## [1]  2  4  6  8 10  5
```

```r
# X & !Y <-> setdiff(x, y)
x1 & !y1
```

```
##  [1] FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE FALSE
```

```r
setdiff(x2, y2)
```

```
## [1] 2 4 6 8
```

```r
# xor(X, Y) <-> setdiff(union(x, y), intersect(x, y))
xor(x1, y1)
```

```
##  [1] FALSE  TRUE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE FALSE
```

```r
setdiff(union(x2, y2), intersect(x2, y2))
```

```
## [1] 2 4 6 8 5
```

* use `x[y]` instead of `x[which(y)]` since:

  - `which()` will drop `NA` values
  
  - `x[-which(y)]` is not equivalent to `x[!y]`
  
## 4.5.9 Exercises

1. How would you randomly permute the columns of a data frame? (This is an
important technique in random forests.) Can you simultaneously permute the rows
and columns in one step?


```r
df <- data.frame(x = c(1, 2, 3, 1, 2), y = 5:1, z = letters[1:5])

# randomly permute columns
df[, sample(ncol(df))]
```

```
##   z y x
## 1 a 5 1
## 2 b 4 2
## 3 c 3 3
## 4 d 2 1
## 5 e 1 2
```

```r
# permute rows and columns
df[sample(nrow(df)), sample(ncol(df))]
```

```
##   x z y
## 2 2 b 4
## 4 1 d 2
## 3 3 c 3
## 1 1 a 5
## 5 2 e 1
```

2. How would you select a random sample of `m` rows from a data frame? What if
the sample had to be contiguous?


```r
df <- data.frame(x = c(1:10), y = (10:1))

m <- 3

# take random samples of m rows
df[sample(nrow(df), m), ]
```

```
##     x y
## 9   9 2
## 3   3 8
## 10 10 1
```

```r
# take contiguous samples
start <- sample(nrow(df) - m, 1) # take one random start site
contig_sample <- seq(start, start + m, by = 1) # generate vector that takes the sample
df[contig_sample, ]
```

```
##   x y
## 6 6 5
## 7 7 4
## 8 8 3
## 9 9 2
```





