---
title: 'Chapter 8: Conditions'
author: "Paulo Magalang"
date: "2022-11-03"
output: 
  html_document: 
    keep_md: yes
---

# 8.1 Quiz

1. What are the three most important types of condition?

2. What function do you use to ignore errors in block of code?

3. What is the main difference between `tryCatch()` and `withCallingHandlers()`?

4. Why might you want to create a custom error object?



```r
library(rlang)
```


# 8.2 Signalling conditions

* three condtions

  - errors: function cannot continue and stops execution
  
  - warnings: function can partially recover but something went wrong
  
  - messages

  - keyboard interrupt (Esc, CTRL + C), only interactively
  
## 8.2.1 Errors

* errors are thrown by `stop()`


```r
f <- function() g()
g <- function() h()
h <- function() stop("This is an error!")

f()
```


```r
h <- function() stop("This is an error!", call. = FALSE)
f()
```

* rlang equivalent is `rlang::abort()`


```r
h <- function() abort("This is an error!")
f()
```


## 8.2.2 Warnings

* can have multiple warnings from a single function call


```r
fw <- function() {
  cat("1\n")
  warning("W1")
  cat("2\n")
  warning("W2")
  cat("3\n")
  warning("W3")
}

fw()
```

```
## 1
```

```
## Warning in fw(): W1
```

```
## 2
```

```
## Warning in fw(): W2
```

```
## 3
```

```
## Warning in fw(): W3
```

* `options(warn = 1)` makes warnings appear immediately

* `options(warn = 2)` to turn warnings into errors

* `options(warn = 0)` is default behavior

* rlang equivalent is `rlang::warn()`

## 8.2.3 Messages


```r
fm <- function() {
  cat("1\n")
  message("M1")
  cat("2\n")
  message("M2")
  cat("3\n")
  message("M3")
}

fm()
```

```
## 1
```

```
## M1
```

```
## 2
```

```
## M2
```

```
## 3
```

```
## M3
```

* `cat()` is for the user, `message()` for the developer 

## 8.2.4 Exercises

1. Write a wrapper around `file.remove()` that throws an error if the file to be
deleted does not exist.


```r
file.remove.err <- function(file) {
    out <- file.remove(file)
    if (out == FALSE) {
        abort(paste0(file, " does not exist"))
    }
}
```

2. What does the `appendLF` argument to `message()` do? How is it related to `cat()`?

`appendLF` appends a newline to the end of the message.

# 8.3 Ignoring conditions

* ignore errors with `try()`

* ignore warnings with `suppressWarnings()`

* ignore messages with `suppressMessages()`


```r
f2 <- function(x) {
  try(log(x)) # throws an error but execution continues
  10
}
f2("a")
```

```
## Error in log(x) : non-numeric argument to mathematical function
```

```
## [1] 10
```

* can assign within a `try()` call to set a default value if the code is not successful


```r
default <- NULL
try(default <- read.csv("possibly-bad-input.csv"), silent = TRUE)

default
```


```r
suppressWarnings({
  warning("Uhoh!")
  warning("Another warning")
  1
})
```

```
## [1] 1
```

```r
suppressMessages({
  message("Hello there")
  2
})
```

```
## [1] 2
```

```r
suppressWarnings({
  message("You can still see me")
  3
})
```

```
## You can still see me
```

```
## [1] 3
```

# 8.4 Handling conditions

* handlers can temporarily ovrride or supplement default behaviors of conditions


```r
tryCatch(
  error = function(cnd) {
    # code to run when error is thrown
  },
  code_to_run_while_handlers_are_active
)

withCallingHandlers(
  warning = function(cnd) {
    # code to run when warning is signalled
  },
  message = function(cnd) {
    # code to run when message is signalled
  },
  code_to_run_while_handlers_are_active
)
```

* `tryCatch()` defines exiting handlers

* `withCallingHandlers()` defines calling handlers

## 8.4.1 Condition objects


```r
cnd <- catch_cnd(stop("An error"))
str(cnd)
```

```
## List of 2
##  $ message: chr "An error"
##  $ call   : language force(expr)
##  - attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

## 8.4.2 Exiting handlers


```r
# return NA instead of throwing an error
f3 <- function(x) {
  tryCatch(
    error = function(cnd) NA,
    log(x)
  )
}

f3("x")
```

```
## [1] NA
```


```r
# code executes normally if signaled condition does not match
tryCatch(
  error = function(cnd) 10,
  1 + 1
)
```

```
## [1] 2
```

```r
tryCatch(
  error = function(cnd) 10,
  {
    message("Hi!")
    1 + 1
  }
)
```

```
## Hi!
```

```
## [1] 2
```

* are exiting handlers because after the condition is signaled, control passes
to the handler and never returns to the original code


```r
tryCatch(
  message = function(cnd) "There",
  {
    message("Here")
    stop("This code is never run!")
  }
)
```

```
## [1] "There"
```

* `cnd` is the argument for the handler function


```r
tryCatch(
  error = function(cnd) {
    paste0("--", conditionMessage(cnd), "--")
  },
  stop("This is an error")
)
```

```
## [1] "--This is an error--"
```

* `finally` in `tryCatch()` specifies a block of code to run regardless of
whether the initial expression succeeds or fails


```r
path <- tempfile()
tryCatch(
  {
    writeLines("Hi!", path)
    # ...
  },
  finally = {
    # always run
    unlink(path)
  }
)
```

## 8.4.3 Calling handlers

* code execution continues normally once the handler returns in calling handlers


```r
tryCatch(
  message = function(cnd) cat("Caught a message!\n"), 
  {
    message("Someone there?")
    message("Why, yes!")
  }
)
```

```
## Caught a message!
```

```r
withCallingHandlers(
  message = function(cnd) cat("Caught a message!\n"), 
  {
    message("Someone there?")
    message("Why, yes!")
  }
)
```

```
## Caught a message!
```

```
## Someone there?
```

```
## Caught a message!
```

```
## Why, yes!
```


```r
withCallingHandlers(
  message = function(cnd) message("Second message"),
  message("First message")
)
```

```
## Second message
```

```
## First message
```


```r
# Bubbles all the way up to default handler which generates the message
withCallingHandlers(
  message = function(cnd) cat("Level 2\n"),
  withCallingHandlers(
    message = function(cnd) cat("Level 1\n"),
    message("Hello")
  )
)
```

```
## Level 1
## Level 2
```

```
## Hello
```

```r
# Bubbles up to tryCatch
tryCatch(
  message = function(cnd) cat("Level 2\n"),
  withCallingHandlers(
    message = function(cnd) cat("Level 1\n"),
    message("Hello")
  )
)
```

```
## Level 1
## Level 2
```


```r
# Muffles the default handler which prints the messages
withCallingHandlers(
  message = function(cnd) {
    cat("Level 2\n")
    cnd_muffle(cnd) # stop here
  },
  withCallingHandlers(
    message = function(cnd) cat("Level 1\n"),
    message("Hello")
  )
)
```

```
## Level 1
## Level 2
```

```r
# Muffles level 2 handler and the default handler
withCallingHandlers(
  message = function(cnd) cat("Level 2\n"),
  withCallingHandlers(
    message = function(cnd) {
      cat("Level 1\n")
      cnd_muffle(cnd) # stop here
    },
    message("Hello")
  )
)
```

```
## Level 1
```

## 8.4.4 Call stacks

* call stacks of exiting vs calling handlers


```r
f <- function() g()
g <- function() h()
h <- function() message("!")
```


```r
# in context of the call that signalled the condition
withCallingHandlers(f(), message = function(cnd) {
  lobstr::cst()
  cnd_muffle(cnd)
})
```

```
##      ▆
##   1. ├─base::withCallingHandlers(...)
##   2. ├─global f()
##   3. │ └─global g()
##   4. │   └─global h()
##   5. │     └─base::message("!")
##   6. │       ├─base::withRestarts(...)
##   7. │       │ └─base (local) withOneRestart(expr, restarts[[1L]])
##   8. │       │   └─base (local) doWithOneRestart(return(expr), restart)
##   9. │       └─base::signalCondition(cond)
##  10. └─global `<fn>`(`<smplMssg>`)
##  11.   └─lobstr::cst()
```


```r
# in context of the call to tryCatch()
tryCatch(f(), message = function(cnd) lobstr::cst())
```

```
##     ▆
##  1. └─base::tryCatch(f(), message = function(cnd) lobstr::cst())
##  2.   └─base (local) tryCatchList(expr, classes, parentenv, handlers)
##  3.     └─base (local) tryCatchOne(expr, names, parentenv, handlers[[1L]])
##  4.       └─value[[3L]](cond)
##  5.         └─lobstr::cst()
```

## 8.4.5 Exercises

1. What extra information does the condition generated by `abort()` contain
compared to the condition generated by `stop()` ie. what's the difference between
these two objects? Read the help for `?abort` to learn more.

`abort()` saves the backtrace of the error condition.


```r
stop_cnd <- catch_cnd(stop("An error"))
abort_cnd <- catch_cnd(abort("An error"))
str(stop_cnd)
```

```
## List of 2
##  $ message: chr "An error"
##  $ call   : language force(expr)
##  - attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

```r
str(abort_cnd)
```

```
## List of 4
##  $ message: chr "An error"
##  $ trace  :Classes 'rlang_trace', 'rlib_trace', 'tbl' and 'data.frame':	8 obs. of  6 variables:
##   ..$ call       :List of 8
##   .. ..$ : language catch_cnd(abort("An error"))
##   .. ..$ : language eval_bare(rlang::expr(tryCatch(!!!handlers, {     force(expr) ...
##   .. ..$ : language tryCatch(condition = `<fn>`, {     force(expr) ...
##   .. ..$ : language tryCatchList(expr, classes, parentenv, handlers)
##   .. ..$ : language tryCatchOne(expr, names, parentenv, handlers[[1L]])
##   .. ..$ : language doTryCatch(return(expr), name, parentenv, handler)
##   .. ..$ : language force(expr)
##   .. ..$ : language abort("An error")
##   ..$ parent     : int [1:8] 0 1 1 3 4 5 1 0
##   ..$ visible    : logi [1:8] FALSE FALSE FALSE FALSE FALSE FALSE ...
##   ..$ namespace  : chr [1:8] "rlang" "rlang" "base" "base" ...
##   ..$ scope      : chr [1:8] "::" "::" "::" "local" ...
##   ..$ error_frame: logi [1:8] FALSE FALSE FALSE FALSE FALSE FALSE ...
##   ..- attr(*, "version")= int 2
##  $ parent : NULL
##  $ call   : NULL
##  - attr(*, "class")= chr [1:3] "rlang_error" "error" "condition"
```

2. Predict the results of evaluating the following code


```r
show_condition <- function(code) {
  tryCatch(
    error = function(cnd) "error",
    warning = function(cnd) "warning",
    message = function(cnd) "message",
    {
      code
      NULL
    }
  )
}

show_condition(stop("!")) # error
```

```
## [1] "error"
```

```r
show_condition(10) # no matches, NULL
```

```
## NULL
```

```r
show_condition(warning("?!")) # warning
```

```
## [1] "warning"
```

```r
show_condition({ # message
  10
  message("?") # stops here
  warning("?!")
})
```

```
## [1] "message"
```

```r
show_condition({ # warning 
  10
  warning("?") # stops here
  message("?!")
})
```

```
## [1] "warning"
```

3. Explain the results of running this code:


```r
withCallingHandlers(
  message = function(cnd) message("b"),
  withCallingHandlers(
    message = function(cnd) message("a"),
    message("c")
  )
)
```

```
## b
```

```
## a
```

```
## b
```

```
## c
```

```r
# message("c") is run, bubbles up to outer calling handler
# b is printed, then the inner calling handler is run, a is printed
# now message("a") is evaluated which bubbles out to the outer calling handler
# b is printed
# the original message("c") is now evaluated
```

4. Read the source code for `catch_cnd()` and explain how it works.

skip


```r
catch_cnd <- function(expr, classes = "condition") {
  stopifnot(is_character(classes))
  handlers <- rep_named(classes, list(identity))

  eval_bare(rlang::expr(
    tryCatch(!!!handlers, { # !!! splice operator, injects list of arguments into function call
      force(expr) # force evaluation of input expression
      return(NULL)
    })
  ))
}
```

5. How could you rewrite `show_condition()` to use a single handler?


```r
show_condition <- function(code) {
  tryCatch(
    error = function(cnd) "error",
    warning = function(cnd) "warning",
    message = function(cnd) "message",
    {
      code
      NULL
    }
  )
}
```


```r
show_condition_q5 <- function(code) {
  tryCatch(
    condition = function(cnd) {
        cnd_var <<- cnd
        obj_attributes <- attributes(cnd) # will be set null for a non object
        if(!is.null(obj_attributes)) {
            # check class of object
            obj_class <- obj_attributes$class[2]
            
            if (obj_class == "error") {
                "error"
            } else if (obj_class == "warning") {
                "warning"
            } else if (obj_class == "message") {
                "message"
            }
        }
    },
    {
        code
        NULL
    }
  )
}
show_condition_q5(stop("!")) # error
```

```
## [1] "error"
```

```r
show_condition_q5(10) # no matches, NULL
```

```
## NULL
```

```r
show_condition_q5(warning("?!")) # warning
```

```
## [1] "warning"
```

```r
show_condition_q5({ # message
  10
  message("?") # stops here
  warning("?!")
})
```

```
## [1] "message"
```








