---
title: 'Chapter 7: Environments'
author: "Paulo Magalang"
date: "2022-10-18"
output: 
  html_document: 
    keep_md: yes
---


```r
library(rlang)
```

# 7.1 Introduction

## Quiz

1. List at least three ways that an environment differs from a list?

2. What is the parent of the global environment? What is the only environment
that doesn't have a parent?

3. What is the enclosing environment of a function? Why is it important?

4. How do you determine the environment from which a function was called?

5. How are `<-` and `<<-` different?

# 7.2 Environment basics

* environments are similar to a named list except:

  - every name is unique
  
  - names are unordered
  
  - has a parent
  
  - no copy on modify
  
## 7.2.1 Basics


```r
# create environments

# with rlang::env
e1 <- env(
  a = FALSE,
  b = "a",
  c = 2.3,
  d = 1:3,
)

# envrionments can contain themselves
e1$d <- e1

# with base::new.env (from 1st edition of the book)
e <- new.env() # init environment
e$a <- FALSE
e$b <- "a"
e$c <- 2.3
e$d <- 1:3
```


```r
# memory addresses
e1 # just prints address
```

```
## <environment: 0x00000208ca1eff88>
```

```r
env_print(e1) # has more info
```

```
## <environment: 0x00000208ca1eff88>
## Parent: <environment: global>
## Bindings:
## • a: <lgl>
## • b: <chr>
## • c: <dbl>
## • d: <env>
```


```r
# print names
env_names(e1)
```

```
## [1] "a" "b" "c" "d"
```

```r
names(e1)
```

```
## [1] "a" "b" "c" "d"
```

```r
ls(e1, all.names = TRUE)
```

```
## [1] "a" "b" "c" "d"
```

## 7.2.2 Important environments

* current environment `current_env()`: environment where code is currently executing

* global environment `global_env()`: environment where interactive computation takes place

* access global env with `globalenv()` and current env with `environment()`


```r
# environment comparison: use identical()
identical(global_env(), current_env())
```

```
## [1] TRUE
```

```r
#global_env() == current_env()
```

## 7.2.3 Parents

* every environment has a parent environment, used to implement lexical scoping


```r
e2a <- env(d = 4, e = 5)
e2b <- env(e2a, a = 1, b = 2, c = 3)
```


```r
# find parent of an environment with env_parent()
env_parent(e2b)
```

```
## <environment: 0x00000208c8dd1790>
```

```r
env_parent(e2a)
```

```
## <environment: R_GlobalEnv>
```

* empty environment does not have a parent


```r
e2c <- env(empty_env(), d = 4, e = 5)
e2d <- env(e2c, a = 1, b = 2, c = 3)
```


```r
# can see ancestors with env_parents()
env_parents(e2b) # by default stops at global environment
```

```
## [[1]]   <env: 0x00000208c8dd1790>
## [[2]] $ <env: global>
```

```r
env_parents(e2d) # ancestors terminate at empty environment
```

```
## [[1]]   <env: 0x00000208c9274bd8>
## [[2]] $ <env: empty>
```


```r
# ancestors of global environment includes all attached packages
env_parents(e2b, last = empty_env())
```

```
##  [[1]]   <env: 0x00000208c8dd1790>
##  [[2]] $ <env: global>
##  [[3]] $ <env: package:rlang>
##  [[4]] $ <env: package:stats>
##  [[5]] $ <env: package:graphics>
##  [[6]] $ <env: package:grDevices>
##  [[7]] $ <env: package:utils>
##  [[8]] $ <env: package:datasets>
##  [[9]] $ <env: package:methods>
## [[10]] $ <env: Autoloads>
## [[11]] $ <env: package:base>
## [[12]] $ <env: empty>
```

* `parent.env()` finds parent of environment

## 7.2.4 Super assignment `<<-`

* `<-` creates a variable in the current environment, `<<-` modifies an existing variable
in a parent env

* if a variable is not found using `<<-`, one is created in the global environment


```r
x <- 0
f <- function() {
  x <<- 1
}
f()
x
```

```
## [1] 1
```

## 7.2.5 Getting and setting

* get and set elements of an environment with `$` and `[[`, but cannot use numeric
indices


```r
e3 <- env(x = 1, y = 2)
e3$x
```

```
## [1] 1
```

```r
e3$z <- 3
e3[["z"]]
```

```
## [1] 3
```

* if binding does not exist, will return `NULL` but can change default value in
`env_get()` or just throw an error

* `env_poke()` takes a name and value to add additional bindings


```r
env_poke(e3, "a", 100)
e3$a
```

```
## [1] 100
```

* `env_bind` to add multiple bindings


```r
env_bind(e3, a = 10, b = 20)
env_names(e3)
```

```
## [1] "x" "y" "z" "a" "b"
```

* `env_has()` to check if binding exists in an env

* `env_unbind()` to unbind objects from env


```r
e3$a <- NULL
env_has(e3, "a")
```

```
##    a 
## TRUE
```

```r
env_unbind(e3, "a")
env_has(e3, "a")
```

```
##     a 
## FALSE
```

* be careful with `get`, `assign`, `exists`, and `rm`: designed to be used interactively

## 7.2.6 Advanced bindings

* `env_bind_lazy()` creates delayed bindings - bindings that are evaluated the first
time they are accessed, useful for accessing data from R packages


```r
env_bind_lazy(current_env(), b = {Sys.sleep(1); 1}) # b not evaluated

system.time(print(b)) # now evaluated
```

```
## [1] 1
```

```
##    user  system elapsed 
##    0.00    0.00    1.03
```

```r
system.time(print(b)) # already loaded in, no need to reevaluate
```

```
## [1] 1
```

```
##    user  system elapsed 
##       0       0       0
```

* `env_bind_active()` creates active bindings - bindings that are recomputed every
time they are accessed, used for R6 active fields


```r
env_bind_active(current_env(), z1 = function(val) runif(1))

z1
```

```
## [1] 0.06715805
```

```r
z1
```

```
## [1] 0.6608591
```

## 7.2.7 Exercises

1. List three ways in which an environment differs from a list.

* environments not copied on modification, cannot be subset using numerical
indices, and each environment has a parent

2. Create an environment as illustrated by this picture.


```r
loop_env <- env()
loop_env$loop <- loop_env
```

3. Create a pair of environments as illustrated by this picture.


```r
deloop_env <- env()
loop_env <- env()

loop_env$loop <- deloop_env
deloop_env$deloop <- loop_env
```

4. Explain why `e[[1]]` and `e[c("a", "b")]` don't make sense when `e` is an environment.

The book mentioned that environments cannot be subset using numeric indices. This
makes sense knowing that environment names are not ordered, unlike lists. 

I'm not sure why accessing multiple environment names would not work.

5. Create a version of `env_poke()` that will only bind new names, never re-bind
old names. Some programming languages only do this, and are known as single assignment
languages.


```r
env_poke_SA <- function(environment, name, value) {
    # check to see if binding in the environment exists
    if(!env_has(environment, name)) { # if binding does not exist
        environment[[name]] <- value # create binding
    }
}

# test with existing e3
env_names(e3)
```

```
## [1] "x" "y" "z" "b"
```

```r
e3$x
```

```
## [1] 1
```

```r
env_poke_SA(e3, "x", 2)
e3$x
```

```
## [1] 1
```

6. What does this function do? How does it differ from `<<-` and why might you
prefer it?

`rebind()` recursively checks through each parent environment to see if `name` exists.
If `name` is found in a parent environment, it is reassigned `value`. If not, it will
check the subsequent parent environment up until the empty environment. `<<-` only
stops at the global environment, not the empty environment, and will create a new
binding in the global environment if the name does not exist.


```r
rebind <- function(name, value, env = caller_env()) {
  if (identical(env, empty_env())) { # check to see if we went back to far, base case
    stop("Can't find `", name, "`", call. = FALSE)
  } else if (env_has(env, name)) {
    env_poke(env, name, value) # reassign value when it exists, terminate recursion
  } else {
    rebind(name, value, env_parent(env)) # recursive call, reduction step
  }
}

#rebind("a", 10)
a <- 5
rebind("a", 10)
a
```

```
## [1] 10
```

# 7.3 Recursing over environments

* `where()` looks for a name starting at a given environment


```r
where <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    # Base case
    stop("Can't find ", name, call. = FALSE)
  } else if (env_has(env, name)) {
    # Success case
    env
  } else {
    # Recursive case
    where(name, env_parent(env))
  }
}
```


```r
#where("yyy")

x <- 5
where("x")
```

```
## <environment: R_GlobalEnv>
```

```r
where("mean")
```

```
## <environment: base>
```

## 7.3.1 Exercises

1. Modify `where()` to return all environments that contain a binding for `name`.
Carefully think through what type of object the function will need to return.


```r
# kind of a cheat solution since function needs an empty vector called env_list to work

#env_list <- c() # init. empty vector
where_all <- function(name, env = caller_env(), env_list = c()) {
  if (identical(env, empty_env())) {
    # Base case, return env_list output
    return(env_list)
  } 
  else {
    if (env_has(env, name)) {
    # Success case, but do not terminate recursive loop
    env_list <- c(env_list, env) # concatenate to env_list
    }
      
    # Recursive call, must terminate at empty env
    where_all(name, env_parent(env), env_list) # also keep current copy of env_list to carry on
  }
}

# test
e4a <- env(empty_env(), a = 1, b = 2)
e4a
```

```
## <environment: 0x00000208c8d99908>
```

```r
e4b <- env(e4a, x = 10, a = 11)
e4b
```

```
## <environment: 0x00000208cdf68df8>
```

```r
where_all("a", env = e4b)
```

```
## [[1]]
## <environment: 0x00000208cdf68df8>
## 
## [[2]]
## <environment: 0x00000208c8d99908>
```

2. write a function called `fget()` that finds only function objects. It should
have two arguments, `name` and `env`, and should obey the regular scoping rules
for functions: if there's an object with a matching name that's not a function,
look in the parent. For an added challenge, also add an `inherits` argument
which controls whether the function recurses up the parents or only looks
in one environment.


```r
fget <- function(name, env, inherits = TRUE, func_list = c()) {
    # base case, got to empty_env
    if(identical(env, empty_env())) {
        return(func_list)
    } else {
        if(env_has(env, name) & is_function(name)) { # check if it exists in env and is a function
            func_list <- c(func_list, name) # success case
        }
        
        # before recursive call, check if inherits is FALSE
        if(inherits == FALSE) {
            return(func_list) # will break before recursion starts
        }
        
        fget(name, env_parent(env), inherits, func_list) # recursive call
    }
}
```


```r
fget <- function(name, env, inherits = TRUE) {
    # base case, at empty env
    if(identical(env, empty_env())) {
        stop("function not found")
    } else if(env_has(env, name)) {
        # check if object is a function
        object <- env_get(env, name)
        if(is.function(object)) {
            return(object) # terminate recursion
        }
        # if object is not a function, check to see if we need to make recursive call
        if(inherits == FALSE) {
            stop("function not found") # terminate recursion
        }
    }
    
    fget(name, env_parent(env)) # recursive call
}

# test
fget("mean", globalenv())
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x00000208ccca2d48>
## <environment: namespace:base>
```

```r
#fget("asdf", globalenv())

func_e1 <- env(a = mean, empty_env())
func_e2 <- env(a = 1, func_e1)

fget("a", func_e1, inherits = FALSE)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x00000208ccca2d48>
## <environment: namespace:base>
```

```r
#fget("a", func_e2, inherits = FALSE)
fget("a", func_e2, inherits = TRUE)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x00000208ccca2d48>
## <environment: namespace:base>
```


























