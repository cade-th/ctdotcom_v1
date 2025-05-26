---
title: Error Driven Development
date: 2025-03-29
description: Propagating errors in C
tag: C
author: Cade Thornton
---

### Rust is nice, and C is nice

But I want some stuff from rust in C. I think one of the best things rust offers off the bat is it's result<> type.
```
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```
Any error-prone function can return this type, which will result in a typed error (usually an enum) in the case of an error and some data in the case of a success.

So you usually start by creating an enum for possible errors some module can return:
```
enum lexer_error_t {
        INVALID_TOKEN,
        ERROR_1,
        ERROR_2,
        // etc
}
```

then have some function return a result with it:
```
fn lex(&self) -> Result<Tokens[], lexer_error_t> {}
```
and you can get the data (error or success) using an unwrap() method, or propagate the error to another function by returning it

This is really nice in something like a lexer because you know exactly which functions are error prone and what those errors might return just by their signature.

AND this is almost perfect in my opinion. 

But as I was writing my lexer in the Tourist's guide to Interpreters on this blog, I found that if a function returned a result and the result was an error, I generally wanted more than just either an error enum or an error value: I wanted *both*. If the lexer failed and a token was invalid, I would correctly get a invalid token error, but I wouldn't know where the error first propagated from or in the case of the lexer, what the invalid token is. This means that I want multiple types in my errors. Error metadata.

However, I had been reading stuff about Errlang recently and it got me thinking: what if I just assume every function will error. Wild idea I know, but it does leading to somewhere I don't think anyone has been. 

Instead of functions returning a result and differentiating the data based off an error or a sucess, we can instead create a different type called Error which unifies the data returned, but add optional metadata like the error type enum.

Ideally, we'd return something like Error<error_t, data_t> from functions, and they could propagate the error to/from each other while changing the data type accordinly. This would probably work in rust but I haven't tried yet.

```
// Doing this in C because I'm doing embedded systems work right now
#define ERROR(enumerator)	\
    typedef struct {		\
	enumerator type;	    \
	bool ok;		        \
	void *data;		        \
    }
```

However, you can't declare a type inside a return type for a function in C, so if you did this as just returning the Error type, you'd lose information from the function prototype about what the function returns and make whoever is using your API angry (which is what is happening in Linux kernel development between C and rust maintainers currently I believe).

So for right now, I'll just make the error a global variable and have functions set the -ok field when they return as an indicator of an error.

We can use these two macros to allows functions to modify the data of the error:
```
#define THROW(E,T,D)		\
    E.ok = false;		\
    E.data = malloc(sizeof(T)); \
    *(T *)E.data = D;

#define CATCH(T,E)		\
    *(T *)E.data
```
That way any error type can return any kind of data like an int, char, string, struct, etc.

One big benefit of this approach is that test runners basically write themselves and reduce the need for a debugger:
```
// Error types for the module
typedef enum {
        error_1,
        error_2,
        error_3,
        error_4,
} module_error_t;
// Error macro we wrote
ERROR(module_error_t) module_error;

// Declaring the error as a global variable
module_error mod_error = { .type = error_1, .ok = true, .data = NULL };

int function_1(int input) {
    // simulate failure
    if (input == 2) {
        mod_error.type = error_1;
        THROW(mod_error,int,input);
        return input; 
    }   
    else {
        return input;
    }
}

int function_2(char input) {
    if (input == '+') {
        mod_error.type = error_2;
        THROW(mod_error,int,input);
        return input;
    } else {
        return input;
    }
    
}

void function_3(int input) {

    int get_thing_1 = function_1(234);
    if (!mod_error.ok) {
        return;
    }
    int get_thing_2 = function_2('=');
    if (!mod_error.ok) {
        return;
    }
    if (input == 2) {
        mod_error.type = error_4;
        THROW(mod_error,int,input);
    }
    else {
        return;
    }
}

int test_runner(void) {

    function_3(282);    

    if(mod_error.ok) {
        printf("Test Success!\n");
        return 0;
    }
    else {
        printf("Test Failed:\n");
        switch (mod_error.type) {
            case error_1 :
                printf("Error 1 with value %d\n", CATCH(int,mod_error));
                break;
            case error_2 :
                printf("Error 2 with value '%c'\n", CATCH(char,mod_error));
                break;
            case error_3 :
                printf("Error 3 with value %d\n", CATCH(int,mod_error));
                break;
            case error_4 :
                printf("Error 4 with value %d\n", CATCH(int,mod_error));
                break;
        };
        return 1;
    }
}
 
// Output:
Test Failed:
Error 2 with value '='
```

So our test runner just becomes a single function (test_runner() here) that returns some indicator of success (with a value if you'd like) and a switch statement which covers every possible error of the module plus any data necessary to fix the bug. Using this method, adding more possible errors is as simple as adding to the error enum and the final switch statement, meaning you can start modules by thinking of its errors *first*, which I am now coining as *error driven development* lol. I'll be fleshing out this idea both in this article and further articles as I demonstrate it in my projects.



