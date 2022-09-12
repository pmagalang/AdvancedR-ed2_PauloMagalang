---
title: 'Chapter 3: Vectors'
author: "Paulo Magalang"
date: "2022-09-08"
output: 
  html_document: 
    keep_md: yes
---


```r
library(lobstr)
```

# 3.1 Introduction

* two types of vectors: atomic and list

  * atomic: all elements have to be the same type
    
  * list: elements can be different types
    
  * NULL: zero length vector
  
* vector attributes (metadata)

  * dimension: turns vectors into matrices
  
  * class: powers the S3 system  (factors, data frames, tibbles)
  
## Quiz

Quiz was done after going through the chapter.

1. What are the four common types of atomic vectors? What are the two rare types?

* common: integer, double, logical, character

* rare: raw, complex

2. What are attributes? How do you get them and set them?

* attributes are object metadata

* get and set individual attributes with `attr()` or many with `attributes()`

3. How is a list different from an atomic vector? How is a matrix different from a data frame?

* list elements can be any type while atomic vector elements must be the same type

* data frame columns can by different types while matrix elements must be the same type

4. Can you have a list that is a matrix? Can a data frame have a column that is a matrix?

* a list can be a matrix if dimensions are assigned to the list

* it is possible to have a column of a data frame be a matrix if the number of rows of
the matrix and dataframe are the same

5. How do tibbles behave differently from data frames?

# 3.2 Atomic Vectors

* 4 types of atomic vectors: logical, integer, double, character

* numeric vectors: integer and double

* rare types of vectors: complex (not needed in stats) and raw (for binary data)


## 3.2.1 Scalars

* scalar: special syntax to create an individual value

  * logical scalar: TRUE/FALSE, T/F
  
  * double scalar: decimal, scientific, hex; special values Inf, -Inf, NaN
  
  * integer scalar: numeric value followed by an L
  
  * string scalar: surrounded by " or '; escape special characters with \
  
## 3.2.2 Making longer vectors with `c()`

* `c()` to combine

* combining atomic vectors will create another atomic vector (flattening)

* `typeof()`: determine type of vector

* `length()`: determine length of vector

## 3.2.3 Missing Values

* `NA`: missing values

* infectious, computations involving a missing value will return another missing
value


```r
# exceptions

NA ^ 0
```

```
## [1] 1
```

```r
NA | TRUE
```

```
## [1] TRUE
```

```r
NA & FALSE
```

```
## [1] FALSE
```

* use `is.na()` to determine NA in a vector

## 3.2.4 Testing and coercion

* test if a vector is a given type with `is.logical()`, `is.double()`, 
`is.integer()`, `is.character()`, be careful with `is.vector()`, `is.atomic()`,
and `is.numeric()`

* combining different types of atomic vectors will cause elements to be cast/coerced
in a fixed order: character -> double -> integer -> logical

* coerce by using `as.integer()`, `as.double()`, `as.logical()`, `as.chacater()`

## 3.2.5 Exercises

1. How do you create raw and complex scalars?

`raw()` creates a raw vector and `complex()` creates a complex vector.

2. Test your knowledge of the vector coercion rules by predicting the output of the following 
uses of `c()`: 


```r
c(1, FALSE) # 1, 0
```

```
## [1] 1 0
```

```r
c("a", 1) # "a", "1"
```

```
## [1] "a" "1"
```

```r
c(TRUE, 1L) # 1, 1
```

```
## [1] 1 1
```

3. Why is `1 == "1"` true? Why is `-1 < FALSE` true? Why is `"one" < 2` false?


```r
1 == "1" # 1 was cast into "1", "1" == "1" is true
```

```
## [1] TRUE
```

```r
-1 < FALSE # FALSE was cast into 0, -1 < 0 is true
```

```
## [1] TRUE
```

```r
"one" < 2 # 2 was cast into "2", something about ASCII values probably
```

```
## [1] FALSE
```

4. Why is the default missing value, `NA`, a logical vector? What's special about logical
vectors? (Hint: think about `c(FALSE, NA_character_)`.)

* NA will be automatically coerced to the correct type when needed.

* logical values have the lowest priority coercion-wise

* The default for NA is a logical vector because it is able to be coerced into
any other type.

5. Precisely what do `is.atomic()`, `is.numeric()`, and `is.vector()` test for?

* Objects can be atomic but not necessarily vectors (from `is.atomic()` docs)

* `is.vector()` checks the attributes of an object and will return `FALSE` if the object
has any other attributes other than names.

* `is.numeric()` checks to see if the object is `double` or `integer` and values look
numeric (arithmetic, comparisons).


# 3.3 Attributes

## 3.3.1 Getting and setting

* attributes are namve-value pairs that attach metadata onto an object

* `attr()`: retrieve and modify attributes

* `attributes()` and `structure()` retrieve and modify many attributes respectively


```r
a <- 1:3
attr(a, "x") <- "abcdef"
attr(a, "x")
```

```
## [1] "abcdef"
```

```r
attr(a, "y") <- 4:6
str(attributes(a))
```

```
## List of 2
##  $ x: chr "abcdef"
##  $ y: int [1:3] 4 5 6
```

```r
# Or equivalently
a <- structure(
  1:3, 
  x = "abcdef",
  y = 4:6
)
str(attributes(a))
```

```
## List of 2
##  $ x: chr "abcdef"
##  $ y: int [1:3] 4 5 6
```

* most attributes are lost by operations, non permanent


```r
attributes(a[1])
```

```
## NULL
```

```r
attributes(sum(a))
```

```
## NULL
```

* names and dim are attributes that are preserved

## 3.3.2 Names 


```r
# 3 ways to name a vector

# When creating it: 
x <- c(a = 1, b = 2, c = 3)

# By assigning a character vector to names()
x <- 1:3
names(x) <- c("a", "b", "c")

# Inline, with setNames():
x <- setNames(1:3, c("a", "b", "c"))
```

* remove names with `x <- unname(x)` or `names(x) <- NULL`

* names should be unique and non-missing (but not enforced), missing names
can be either empty strings or NA

## 3.3.3 Dimensions

* adding `dim` attribute to a vector makes it behave like a 2D/multidim array


```r
# Two scalar arguments specify row and column sizes
x <- matrix(1:6, nrow = 2, ncol = 3)
x
```

```
##      [,1] [,2] [,3]
## [1,]    1    3    5
## [2,]    2    4    6
```

```r
# One vector argument to describe all dimensions
y <- array(1:12, c(2, 3, 2))
y
```

```
## , , 1
## 
##      [,1] [,2] [,3]
## [1,]    1    3    5
## [2,]    2    4    6
## 
## , , 2
## 
##      [,1] [,2] [,3]
## [1,]    7    9   11
## [2,]    8   10   12
```

```r
# You can also modify an object in place by setting dim()
z <- 1:6
dim(z) <- c(3, 2)
z
```

```
##      [,1] [,2]
## [1,]    1    4
## [2,]    2    5
## [3,]    3    6
```


```r
# vector without dim attribute has NULL dimensions
str(1:3)                   # 1d vector
```

```
##  int [1:3] 1 2 3
```

```r
# matrix with single row or col
str(matrix(1:3, ncol = 1)) # column vector
```

```
##  int [1:3, 1] 1 2 3
```

```r
str(matrix(1:3, nrow = 1)) # row vector
```

```
##  int [1, 1:3] 1 2 3
```

```r
# array with single dimension
str(array(1:3, 3))         # "array" vector
```

```
##  int [1:3(1d)] 1 2 3
```

## 3.3.4 Exercises

1. How is `setNames()` implemented? How is `unname()` implemented? Read the source code.


```r
setNames
```

```
## function (object = nm, nm) 
## {
##     names(object) <- nm
##     object
## }
## <bytecode: 0x000002bb4fd50428>
## <environment: namespace:stats>
```

```r
unname
```

```
## function (obj, force = FALSE) 
## {
##     if (!is.null(names(obj))) 
##         names(obj) <- NULL
##     if (!is.null(dimnames(obj)) && (force || !is.data.frame(obj))) 
##         dimnames(obj) <- NULL
##     obj
## }
## <bytecode: 0x000002bb50107d78>
## <environment: namespace:base>
```

* `setNames()` uses the assignment form of `names()` to name a vector.

* `unname()` checks to see if the object is a vector or matrix/array and uses
the assignment form of `names()` and `dimnames()` to set the name to NULL respectively.

2. What does `dim()` return when applied to a 1-dimensional vector? When might you use
`NROW()` or `NCOL()`?


```r
a <- 1:3
dim(a) # will return NULL
```

```
## NULL
```

```r
# NROW and NCOL treats vectors as 1-column matrices
```

3. How would you describe the following three objects? What makes them different from `1:5`?


```r
x1 <- array(1:5, c(1, 1, 5)) # values filled  along 3rd dimension in the array
x2 <- array(1:5, c(1, 5, 1)) # values filled as a single row in the array
x3 <- array(1:5, c(5, 1, 1)) # values filled as a single column in the array
```

* The three objects are arrays while `1:5` is a vector.

4. An early draft used this code to illustrate `structure()`:


```r
z <- structure(1:5, comment = "my attribute", comment2 = "hey")
z
```

```
## [1] 1 2 3 4 5
## attr(,"comment2")
## [1] "hey"
```

```r
str(attributes(z)) # attribute is there but hidden
```

```
## List of 2
##  $ comment : chr "my attribute"
##  $ comment2: chr "hey"
```

But when you print that object you don’t see the comment attribute. Why? Is the attribute missing, or is there something else special about it? (Hint: try using help.)

Seems like `comment()` is a function and the `comment` attribute is not printed to the console (from `?comment`).

# 3.4 S3 Atomic Vectors

* `class` vector attribute turns an object to an S3 object, behaves differently when passed into a generic
function

## 3.4.1 Factors

* factors are vectors that contain predefined values, categorical data

* factors built on top of int vectors with:

  - factor: a `class` that makes it behave differently from int vectors
  
  - levels: defines the values stored


```r
x <- factor(c("a", "b", "b", "a"))
x
```

```
## [1] a b b a
## Levels: a b
```

```r
typeof(x)
```

```
## [1] "integer"
```

```r
attributes(x)
```

```
## $levels
## [1] "a" "b"
## 
## $class
## [1] "factor"
```


```r
sex_char <- c("m", "m", "m")
sex_factor <- factor(sex_char, levels = c("m", "f"))

table(sex_char)
```

```
## sex_char
## m 
## 3
```

```r
table(sex_factor)
```

```
## sex_factor
## m f 
## 3 0
```

* ordered factors: behave like regular factors but the order of levels is meaningful


```r
grade <- ordered(c("b", "b", "a", "c"), levels = c("c", "b", "a"))
grade
```

```
## [1] b b a c
## Levels: c < b < a
```

* use `stringAsFactors = FALSE` to suppress base R functions that automatically coerce
strings to factors

* factors are built on top of integers, string methods will automatically coerce into
strings; explicitly coerce factors into char if need string-like behavior

## 3.4.2 Dates

* built on double vectors


```r
today <- Sys.Date()

typeof(today)
```

```
## [1] "double"
```

```r
attributes(today)
```

```
## $class
## [1] "Date"
```


```r
date <- as.Date("1970-02-01")
unclass(date)
```

```
## [1] 31
```

## 3.4.3 Date-times

* two ways of storing date-time info: POSIXct and POSIXlt

* POSIX: Portable Operating System Interface

* ct: calendar time

* lt: local time


```r
now_ct <- as.POSIXct("2018-08-01 22:00", tz = "UTC")
now_ct
```

```
## [1] "2018-08-01 22:00:00 UTC"
```

```r
typeof(now_ct)
```

```
## [1] "double"
```

```r
attributes(now_ct)
```

```
## $class
## [1] "POSIXct" "POSIXt" 
## 
## $tzone
## [1] "UTC"
```


```r
structure(now_ct, tzone = "Asia/Tokyo")
```

```
## [1] "2018-08-02 07:00:00 JST"
```

```r
structure(now_ct, tzone = "America/New_York")
```

```
## [1] "2018-08-01 18:00:00 EDT"
```

```r
structure(now_ct, tzone = "Australia/Lord_Howe")
```

```
## [1] "2018-08-02 08:30:00 +1030"
```

```r
structure(now_ct, tzone = "Europe/Paris")
```

```
## [1] "2018-08-02 CEST"
```

## 3.4.4 Durations

* difftimes: represent amount of time between pairs of dates or date-times; built on
doubles, `units` attribute that represents the units of time


```r
one_week_1 <- as.difftime(1, units = "weeks")
one_week_1
```

```
## Time difference of 1 weeks
```

```r
typeof(one_week_1)
```

```
## [1] "double"
```

```r
attributes(one_week_1)
```

```
## $class
## [1] "difftime"
## 
## $units
## [1] "weeks"
```

```r
one_week_2 <- as.difftime(7, units = "days")
one_week_2
```

```
## Time difference of 7 days
```

```r
typeof(one_week_2)
```

```
## [1] "double"
```

```r
attributes(one_week_2)
```

```
## $class
## [1] "difftime"
## 
## $units
## [1] "days"
```

## 3.4.5 Exercises

1. What sort of object does `table()` return? What is its type? What attributes does it have? How does the dimensionality change as you tabulate more variables?



```r
sex_char <- c("m", "m", "m")
sex_factor <- factor(sex_char, levels = c("m", "f"))

typeof(table(sex_factor)) # built on top of int vectors which is expected
```

```
## [1] "integer"
```

```r
attributes(table(sex_factor)) # ldim, dimnames, dimnames$sex_factor, class
```

```
## $dim
## [1] 2
## 
## $dimnames
## $dimnames$sex_factor
## [1] "m" "f"
## 
## 
## $class
## [1] "table"
```

```r
sex_factor <- factor(sex_char, levels = c("m", "f", "a", "b", "c")) # just adding random levels
attributes(table(sex_factor)) # dimensions increase with amount of levels defined
```

```
## $dim
## [1] 5
## 
## $dimnames
## $dimnames$sex_factor
## [1] "m" "f" "a" "b" "c"
## 
## 
## $class
## [1] "table"
```

2. What happens to a factor when you modify its levels?


```r
f1 <- factor(letters)
levels(f1) <- rev(levels(f1))
```

Modifying a factors levels will rearrange the vector based on the new ordering of the levels.

3. What does this code do? How do `f2` and `f3` differ from `f1`?


```r
f2 <- rev(factor(letters))

f3 <- factor(letters, levels = rev(letters)) # vector values in same order but levels are reversed
```


# 3.5 Lists

* list elements can be any time because each list element is a reference to another
object

## 3.5.1 Creating


```r
l1 <- list(
  1:3, 
  "a", 
  c(TRUE, FALSE, TRUE), 
  c(2.3, 5.9)
)

typeof(l1)
```

```
## [1] "list"
```

```r
str(l1)
```

```
## List of 4
##  $ : int [1:3] 1 2 3
##  $ : chr "a"
##  $ : logi [1:3] TRUE FALSE TRUE
##  $ : num [1:2] 2.3 5.9
```

* lists are sometimes called recursive vectors since lists can contain othter lists


```r
l3 <- list(list(list(1)))
str(l3)
```

```
## List of 1
##  $ :List of 1
##   ..$ :List of 1
##   .. ..$ : num 1
```

* use `c()` to combine several lists into one list but will coerce vectors to lists
before combining


```r
l4 <- list(list(1, 2), c(3, 4))
l5 <- c(list(1, 2), c(3, 4))

str(l4) # nested list
```

```
## List of 2
##  $ :List of 2
##   ..$ : num 1
##   ..$ : num 2
##  $ : num [1:2] 3 4
```

```r
str(l5) # flattened
```

```
## List of 4
##  $ : num 1
##  $ : num 2
##  $ : num 3
##  $ : num 4
```

## 3.5.2 Testing and coercion

* test for list with `is.list()` and coerce with `as.list()`

* turn any list into an atomic vector with `unlist()`


```r
list(1:3)
```

```
## [[1]]
## [1] 1 2 3
```

```r
as.list(1:3)
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 2
## 
## [[3]]
## [1] 3
```

## 3.5.3 Matrices and arrays

* `dim` attribute can create list-matrices or list-arrays


```r
l <- list(1:3, "a", TRUE, 1.0)
dim(l) <- c(2, 2)
l
```

```
##      [,1]      [,2]
## [1,] integer,3 TRUE
## [2,] "a"       1
```

```r
l[[1, 1]]
```

```
## [1] 1 2 3
```

## 3.5.4 Exercises

1. List all the ways that a lists differs from an atomic vector.

* list elements are pointers to objects so every object can be different types

* lists have the `dim` attribute

* lists are recursive

* lists take up less memory than atomic vectors

2. Why do you need to use `unlist()` to convert a list to an atomic vector? Why
doesn't `as.vector()` work?

Lists are already vectors but `unlist()` will convert it to an atomic vector.


```r
l1 <- list(
  1:3, 
  "a", 
  c(TRUE, FALSE, TRUE), 
  c(2.3, 5.9)
)

unlist(l1)
```

```
## [1] "1"     "2"     "3"     "a"     "TRUE"  "FALSE" "TRUE"  "2.3"   "5.9"
```

```r
as.vector(l1)
```

```
## [[1]]
## [1] 1 2 3
## 
## [[2]]
## [1] "a"
## 
## [[3]]
## [1]  TRUE FALSE  TRUE
## 
## [[4]]
## [1] 2.3 5.9
```

3. Compare and contrast `c()` and `unlist()` when combining a date and date-time
into a single vector.


```r
date <- as.Date("1970-01-01")
date_time <- as.POSIXct("1970-01-02 22:00", tz = "UTC")

c(date, date_time) # time zone and time is removed when combined
```

```
## [1] "1970-01-01" "1970-01-02"
```

```r
unlist(list(date, date_time)) 
```

```
## [1]      0 165600
```

```r
# dates and date-times are internally doubles, but not sure what units
# the values are in, seconds?
```

# 3.6 Data frames and tibbles

* data frames and tibbles are built on top of lists


```r
df1 <- data.frame(x = 1:3, y = letters[1:3])
typeof(df1)
```

```
## [1] "list"
```

```r
attributes(df1)
```

```
## $names
## [1] "x" "y"
## 
## $class
## [1] "data.frame"
## 
## $row.names
## [1] 1 2 3
```

* lengths of each vector in a df must be the same

* `names()` returns col names of df

* `length()` returns # of cols in a df


```r
library(tibble)

df2 <- tibble(x = 1:3, y = letters[1:3])
typeof(df2)
```

```
## [1] "list"
```

```r
attributes(df2)
```

```
## $class
## [1] "tbl_df"     "tbl"        "data.frame"
## 
## $row.names
## [1] 1 2 3
## 
## $names
## [1] "x" "y"
```

## 3.6.1 Creating


```r
df <- data.frame(
  x = 1:3, 
  y = c("a", "b", "c")
)
str(df)
```

```
## 'data.frame':	3 obs. of  2 variables:
##  $ x: int  1 2 3
##  $ y: chr  "a" "b" "c"
```


```r
# stringAsFactors is FALSE by default from the documentation

df1 <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"),
  stringsAsFactors = FALSE
)
str(df1)
```

```
## 'data.frame':	3 obs. of  2 variables:
##  $ x: int  1 2 3
##  $ y: chr  "a" "b" "c"
```


```r
# tibbles do not coerce their input
df2 <- tibble(
  x = 1:3, 
  y = c("a", "b", "c")
)
str(df2)
```

```
## tibble [3 × 2] (S3: tbl_df/tbl/data.frame)
##  $ x: int [1:3] 1 2 3
##  $ y: chr [1:3] "a" "b" "c"
```


```r
# tibbles do not automatically transform non-syntactic names
names(data.frame(`1` = 1))
```

```
## [1] "X1"
```

```r
names(tibble(`1` = 1))
```

```
## [1] "1"
```

* `data.frame()` and `tibble()` recycle inputs


```r
data.frame(x = 1:4, y = 1:2)
data.frame(x = 1:4, y = 1:3) # must be multiples of each other to be recycled

tibble(x = 1:4, y = 1) # tibbles recycle vectors of length one
tibble(x = 1:4, y = 1:2)
```



```r
# can refer to variables during tibble construction
tibble(
  x = 1:3,
  y = x * 2
)
```

```
## # A tibble: 3 × 2
##       x     y
##   <int> <dbl>
## 1     1     2
## 2     2     4
## 3     3     6
```

## 3.6.2 Row names


```r
df3 <- data.frame(
  age = c(35, 27, 18),
  hair = c("blond", "brown", "black"),
  row.names = c("Bob", "Susan", "Sam")
)
df3
```

```
##       age  hair
## Bob    35 blond
## Susan  27 brown
## Sam    18 black
```


```r
rownames(df3)
```

```
## [1] "Bob"   "Susan" "Sam"
```

```r
df3["Bob", ]
```

```
##     age  hair
## Bob  35 blond
```

* transposing data frames is not allowed

* row names are undesirable:

  - storing metadata differently than the rest of the data is inconvenient
  
  - row names can only be stored as a single string
  
  - row names must be unique

* can convert row names to columns using `rownames_to_column()` or `rownames` in
`as.tibble()`


```r
as_tibble(df3, rownames = "name")
```

```
## # A tibble: 3 × 3
##   name    age hair 
##   <chr> <dbl> <chr>
## 1 Bob      35 blond
## 2 Susan    27 brown
## 3 Sam      18 black
```

## 3.6.4 Subsetting

* subestting columns in a df will output a vector if one variable
is selected, otherwise a df is output

* extracting a column with `df$x` when `x` does not exist in the df
will select any variable that starts with `x`; NULL output if no column name
starts with `x`

* subsetting tibbles with `[` always returns tibbles and `$` does partial matching

## 3.6.5 Testing and Coercing

* check if an object is a dataframe or tibble with `is.data.frame()`

* coerce with `as.data.frame()` or `as_tibble()`

## 3.6.6 List columns

* data frames can have a column that is a list to keep related objects together
in a row


```r
# list columns in dataframes
df <- data.frame(x = 1:3)
df$y <- list(1:2, 1:3, 1:4)

data.frame(
  x = 1:3, 
  y = I(list(1:2, 1:3, 1:4)) # wrap in I()
)
```

```
##   x          y
## 1 1       1, 2
## 2 2    1, 2, 3
## 3 3 1, 2, 3, 4
```


```r
# list columns in tibbles
tibble(
  x = 1:3, 
  y = list(1:2, 1:3, 1:4)
)
```

```
## # A tibble: 3 × 2
##       x y        
##   <int> <list>   
## 1     1 <int [2]>
## 2     2 <int [3]>
## 3     3 <int [4]>
```

## 3.6.7 Matrix and data frame columns

* possible to have a matrix or array as a column in a dataframe if the 
number of rows matches the data frame


```r
dfm <- data.frame(
  x = 1:3 * 10
)
dfm$y <- matrix(1:9, nrow = 3)
dfm$z <- data.frame(a = 3:1, b = letters[1:3], stringsAsFactors = FALSE)

str(dfm)
```

```
## 'data.frame':	3 obs. of  3 variables:
##  $ x: num  10 20 30
##  $ y: int [1:3, 1:3] 1 2 3 4 5 6 7 8 9
##  $ z:'data.frame':	3 obs. of  2 variables:
##   ..$ a: int  3 2 1
##   ..$ b: chr  "a" "b" "c"
```


```r
dfm[1, ] # no output?
```

```
##    x y.1 y.2 y.3 z.a z.b
## 1 10   1   4   7   3   a
```

## 3.6.8 Exercises

1. Can you have a data frame with zero rows? What about zero columns?


```r
# zero rows
a <- data.frame(x = character())
dim(a)
```

```
## [1] 0 1
```

```r
# zero columns
b <- data.frame(row.names = 1)
dim(b)
```

```
## [1] 1 0
```

2. What happens if you attempt to set rownames that are not unique?


```r
c <- data.frame(row.names = rep(1, 5))
names(c)
# cannot set non-unique rownames for dataframes, got an error
```

3. If `df` is a data frame, what can you say about `t(df)` and `t(t(df))`? Perform
some experiments, making sure to try different column types.


```r
df <- data.frame(a = 1:3, b = c("one", "two", "three"), c = c(T, F, F))
t(df)
```

```
##   [,1]   [,2]    [,3]   
## a "1"    "2"     "3"    
## b "one"  "two"   "three"
## c "TRUE" "FALSE" "FALSE"
```

```r
t(t(df))
```

```
##      a   b       c      
## [1,] "1" "one"   "TRUE" 
## [2,] "2" "two"   "FALSE"
## [3,] "3" "three" "FALSE"
```

```r
# matrix outputs, every element is cast to highest priority type (in this case character)
```

4. What does `as.matrix()` do when applied to a data frame with columns of different types?
How does it differ from `data.matrix()`?


```r
df <- data.frame(a = 1:3, b = c("one", "two", "three"), c = c(T, F, F), d = c(1.1, 1.2, 1.3))
as.matrix(df) # same findings from 3
```

```
##      a   b       c       d    
## [1,] "1" "one"   "TRUE"  "1.1"
## [2,] "2" "two"   "FALSE" "1.2"
## [3,] "3" "three" "FALSE" "1.3"
```

```r
data.matrix(df) # logical cast to integers, weird things going on with strings
```

```
##      a b c   d
## [1,] 1 1 1 1.1
## [2,] 2 3 0 1.2
## [3,] 3 2 0 1.3
```

# 3.7 `NULL`

* `NULL` has its own unique type, length zero, and does not have attributes


```r
typeof(NULL)

length(NULL)

x <- NULL
attr(x, "y") <- 1
```

* test for `NULL` with `is.null()`

* uses for `NULL`:

  - to represent an empty vector of arbitrary type
  
  - to represent an absent vector


























