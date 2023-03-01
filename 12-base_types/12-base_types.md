---
title: "Object Oriented Programming Intro and Base Types"
author: "Paulo Magalang"
date: "2023-03-01"
output: 
  html_document: 
    keep_md: yes
---

# Object-oriented Programming Introduction

* OOP systems: S3, R6, S4

* S3 and S4 use generic function OOP, not encapsulated OOP (aka python, java)

* functional programming more important in R since it uses generic function OOP

## OOP systems

* polymorphism: function's interface is seperate from implementation

```
// simple example of static polymorphism in java
// example from https://stackify.com/oop-concept-polymorphism/

// create a CoffeeMachine object and a method to brew coffees
// how many coffees to brew? one or many?

public class BasicCoffeeMachine {
    public Coffee brewCoffee(CoffeeSelection selection) throws CoffeeException {
    // brewCoffee() implementation to brew only one selection of coffee
        switch (selection) {
        case FILTER_COFFEE:
            return brewFilterCoffee();
        default:
            throw new CoffeeException(
                "CoffeeSelection ["+selection+"] not supported!");
        }   
    }
  
    public List brewCoffee(CoffeeSelection selection, int number) throws CoffeeException {
    // brewCoffee() implementation to brew multiple cups of coffee defined by number
        List coffees = new ArrayList(number);
        for (int i=0; i<number; i++) {
            coffees.add(brewCoffee(selection));
        }
        return coffees;
    }
}
```

* example of polymorphism in R


```r
diamonds <- ggplot2::diamonds

summary(diamonds$carat) # diamonds$carat is doubles
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.2000  0.4000  0.7000  0.7979  1.0400  5.0100
```

```r
summary(diamonds$cut) # diamonds$cut is factors
```

```
##      Fair      Good Very Good   Premium     Ideal 
##      1610      4906     12082     13791     21551
```

* classes define the object (like the fields of an object), methods define what objects can do

* method dispatch defines how methods are inherited from parent classes

* encapsulated OOP: methods belong to objects or classes, `object.method(arg1, arg2)`

* functional OOP: methods belong to generic functions, `generic(object, arg2, arg3)`


## OOP in R

* S3: informal implementation of functional OOP

* S4: formal and rigorous rewrite of S3; provides more guarantees and encapsulation

* RC: encapsulated OO; special type of S4 objects that are mutable

* R6: encapsulated OOP like RC but resolves some issues described in 14.5

* R.oo: mutable S3 objects

* proto: blurs distinctions between classes and instances of classes


## sloop


```r
library(sloop)

otype(1:10)
```

```
## [1] "base"
```

```r
otype(mtcars)
```

```
## [1] "S3"
```

```r
mle_obj <- stats4::mle(function(x = 1) (x - 2) ^ 2)
otype(mle_obj)
```

```
## [1] "S4"
```

# 12. Base Types

## 12.2 Base versus OO objects


```r
# A base object:
is.object(1:10)
```

```
## [1] FALSE
```

```r
sloop::otype(1:10)
```

```
## [1] "base"
```

```r
# An OO object
is.object(mtcars)
```

```
## [1] TRUE
```

```r
sloop::otype(mtcars)
```

```
## [1] "S3"
```


```r
# OO object have "class" attribute
attr(1:10, "class")
```

```
## NULL
```

```r
attr(mtcars, "class")
```

```
## [1] "data.frame"
```


```r
# class() safe for S3 and S4 objects, but misleading for base objects
x <- matrix(1:4, nrow = 2)
class(x)
```

```
## [1] "matrix" "array"
```

```r
sloop::s3_class(x)
```

```
## [1] "matrix"  "integer" "numeric"
```

## 12.3 Base types


```r
# every object has a base type
typeof(1:10)
```

```
## [1] "integer"
```

```r
typeof(mtcars)
```

```
## [1] "list"
```

* vectors: `NULL`, `logical`, `integer`, `double`, `complex`, `character`, `list`, and `raw`


```r
typeof(NULL)
```

```
## [1] "NULL"
```

```r
typeof(1L)
```

```
## [1] "integer"
```

```r
typeof(1i)
```

```
## [1] "complex"
```

* functions: `closure`, `special`, `builtin`


```r
typeof(mean)
```

```
## [1] "closure"
```

```r
typeof(`[`)
```

```
## [1] "special"
```

```r
typeof(sum)    
```

```
## [1] "builtin"
```

* environments: `environment`


```r
typeof(globalenv())
```

```
## [1] "environment"
```

* `S4`: S4 classes that don't inherit from base type


```r
mle_obj <- stats4::mle(function(x = 1) (x - 2) ^ 2)
typeof(mle_obj)
```

```
## [1] "S4"
```

* language components: `symbol`, `language`, `pairlist`, `expression`


```r
typeof(quote(a))
```

```
## [1] "symbol"
```

```r
typeof(quote(a + 1))
```

```
## [1] "language"
```

```r
typeof(formals(mean))
```

```
## [1] "pairlist"
```

* rarely seen in R: `externalptr`, `weakref`, `bytecode`, `promise`, `...`, `any`

### 12.3.1 Numeric type

* 3 different meanings of "numeric"

1. alias for double type

2. in S3/S4, numeric means either integer or double type


```r
sloop::s3_class(1)
```

```
## [1] "double"  "numeric"
```

```r
sloop::s3_class(1L)
```

```
## [1] "integer" "numeric"
```

3. `is.numeric()` tests for objects that behave like numbers, ex: factors


```r
typeof(factor("x"))
```

```
## [1] "integer"
```

```r
is.numeric(factor("x"))
```

```
## [1] FALSE
```




