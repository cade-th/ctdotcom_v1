---
title: A Tourist's Guide to C Interpreters Part II, Dynamic Arrays & Lexer
date: 2025-05-22
description: Making an interpreter in C
tag: C
author: Cade Thornton
---

### Oh the Pains of C

[Repository for this project](https://github.com/cade-th/interpreter_c)


A terrific reason interpreters are great because of test-driven development. TTD was something I never really got into during my degree or in my free time because things like graphics engines or many university projects don't really lend themselves towards the paradigm. However, an interpreter is ideally 100% deterministic, so TTD is *the best* place to both learn and demonstrate the approach. 

I'll begin by doing something I don't really see very often with programming projects, which is write a failing test that covers literally the entirety of the project. It'll be mostly pseudocode for now for reason's I'll explain in a second:
(I'm also using the Unity C testing framework, a very nice library consiting of just a single .c and header file that gives us some nice macros printing info about the tests)
```
void test_interpreter(void) {
        char *input = "1 + 1 * 2;"; 
        Lexer lexer = lexer_new(input);
        Tokens tokens[] = lex(&lexer);

        Parser parser = parser_new(tokens);
        AST tree = parse(&parser);

        char *output = evaluate(tree);
        assert(output, "4");
}

int main(void) {
        UNITY_BEGIN();
        RUN_TEST(test_interpreter):
        return UNITY_END();
}
```

So, our interpreter's structure is essentially exactly how this test function reads: we get the user input as a string from the shell/file (done in part 1), feed this into the lexer that outputs an array of tokens, feed that array into the parser which then builds a tree that we evaluate into something like an interger. Pretty simple. We're just doing basic arithmetic because this test serves to only illustrate the interpreter architecture. Further tests for the lexer/parser/evaluator will be more feature rich and specific.

At first glance, it seems we could move on now to constructing our lexer and trying to get some tokens made, but C is a what I call a fun langauge. 

The following does not work:
```
Tokens tokens[] = lex(&lexer);
```

For a couple reasons. In C, functions cannot return simple arrays, the size of an array most be known at compile time, and C does not have a type for types (i.e. generics) that we would need for our tokens to be returned from the lexer. Darn. Therefore, before we can create our black box for our lexer, we will first need to solve the quinessential problem of C that turns away so many people from the language: creating generic dynamic arrays.

A dynamic array, in our case, will just be a generic heap-based array with no fixed size. This usually looks like a vector in C++:
```
std::vector<Token>
```

But we don't have the nice generic type syntax of C++, so we'll need to use macros because type checking and debugging are for the weak. Basically, we just pass the argument of the macro to the constructor of the array through sizeof(), malloc it in dyn_array_init(), and cast the return pointer to the type we passed to the macro like so:
```
#define DYN_ARRAY(T) (T *)dyn_array_init(sizeof(T), INITIAL_CAPACITY);
```

dyn_array_init() will return something a little funky. In a lot of typical dynamic data structures in c, they begin with a struct defining the layout kind of like this:
```
typedef struct {
        int length;
        int capacity;
} Dyn_Array;
```

And this works fine for a lot of things, but a better way I found via [Dylan Falconer](https://www.bytesbeneath.com/p/dynamic-arrays-in-c) is to have the struct simply be a set of metadata (or a header, in other words) that takes up the first few bytes in the beginning of the dynamically allocated structure like this:
```
typedef struct {
        int length;
        int capacity;
} Dyn_Array_Header;
```

So just a name change. The init function calculates the initial size of the array plus the metadata/header to malloc(), then returns a pointer that is incremented past the metadata in order to start at the first index of the array:

```
void *dyn_array_init(int item_size, int capacity, Allocator *a) {
    void *ptr = 0;
    int size = item_size * capacity + sizeof(Array_Header);
    Array_Header *h = malloc(size);

    if(h) {
        h->capacity = capacity;
        h->length = 0;
        ptr = h + 1;
    }

    return ptr;
}
```

Theoretically, we now have a generic, heap allocated dynamic array. A simple push() and index_at() function will be enough for our lexer, but push() introduces another classic problem: ensuring the capacity of the array, and adjusting the capacity accordingly. Push() will need to be a macro as well because we're still dealing with generic types here:
```
#define ARRAY_PUSH(a,v) ( \
        (a) = array_ensure_capacity(a,1,sizeof(v)), \
        (a) = [array_header(a)->length] = (v), \
        &(a) = [array_header(a)->length++])

#define array_header(a) ((Array_Header *)(a) - 1)
#define array_length(a) (array_header(a)->length)
#define array_capacity(a) (array_header(a)->capacity)
```

ensure_capacity() is a bit of a gnarly function that I won't go into depth here, but it's really a new malloc() and a memcpy depending on if the array header indicates the size is too large. With that, we now have a dynamic, generic array in C with elements accessible via the "[]" syntax (which is syntactic sugar for *(array + index) or incrementing and dereferencing the array pointer

NOW we can begin our lexer. 
```
typedef enum {
        INT, 
        PLUS,
        ASTERISK,
        LET,
        SEMICOLON,
        ASSIGN,
        IDENT,
        ILLEGAL
} Token_t;

typedef struct {
        Token_t type;
        char *literal;
} Token;

typedef struct {
        char *input;
        int position,
        int read_position,
        char *ch;
} Lexer;
Lexer Lexer_new(char *input);
Token *lex(Lexer *self);
```

Below will be the test we'll try to make our lexer pass. It'll just be simple tokens for now:
```
void lexer_basic_test(void) {
    char* input = "=+(){},;";

    int expected_tokens_size = 9;
    Token expected_tokens[] = {
        {ASSIGN, "="},
        {PLUS, "+"},
        {LET, "("},
        {LET, ")"},
        {LET, "{"},
        {LET, "}"},
        {COMMA, ","},
        {SEMICOLON, ";"},
        {Eof, ""}
    };

    Lexer lexer = Lexer_new(input);
    Token *result = lex(&lexer);

    for (int i=0; i < expected_tokens_size; i++) {
        TEST_ASSERT(result[i].type == expected_tokens[i].type);        
        TEST_ASSERT(result[i].literal == expected_tokens[i].literal);        
    }

}
```
Not terrific error handling yet, but I'm considering adding a result type to be returned from each lexer function to mimic that error monad pattern in functional programming.








... [part 3](cadethornton.com/posts/interpreters_p3)



