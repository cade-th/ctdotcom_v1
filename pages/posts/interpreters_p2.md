---
title: A Tourist's Guide to Interpreters Part II
date: 2025-03-28
description: Making an interpreter in C++
tag: C++
author: Cade Thornton
---

Our problem has been simplified now into parsing a single string that represents either a file or the user's shell input. We will tackle this by create a new class a token in Cade_Lang.h:

```
class Token {
        public:
                Token();
        private:
                std::string data;
};
```
With this setup, we can actually start to incorporate some object-oriented features into our program as we will have different token types that will inherit from the base token class and also overload its functions. There will just be these examples for now:

```
class Operator : Token {
        public:
                Operator();
        private:
};

class Label : Token {
        public:
                Label();
        private:
};
```

We'll add a tokenize() function to our Cade_Lang class that will take a string and output a dynamic array of tokens:

```
std::vector<Token> Cade_Lang::tokenize(std::string input);
```
There is a pretty simple approach for this by just separating tokens by whitespace, which we will do by converting the input string to a stream that we can than use the " >> " operator to iterate by detected whitespace like so:

```
std::vector<Token> Cade_Lang::tokenize(std::string input) {
    std::vector<Token> tokens;
    std::stringstream ss(input);
    std::string buffer;

    while (ss >> buffer) {
        Token temp(buffer);
        tokens.push_back(temp);
    }

    return tokens;
}
```

It has dawned on me that I have not included tests up until this point, and interpreters are an excellent use case for test-driven development because they ought to be as deterministic as possible. Therefore, we will also setup a test suite using Google Test (which seems to be the most popular cpp testing framework). This adds a bunch more pain to our CMakeLists.txt, and we'll need to add some static libraries to /usr/lib because gtest doesn't come prebuilt with debian's package manager

We can set up some simple tests using a Test Fixture because our entire program is one object so we'll need some state in between tests:

``` 
// Cade_Lang_Test.cpp
class CadeLangTestFixture : public ::testing::Test {
protected:
    Cade_Lang* cade_lang = nullptr;
    std::vector<char*> argv;
    int argc;

    void SetUp() override {
        argc = 2;
        // Without these strdup the compiler complains
        argv = {strdup("Cade_Lang"), strdup("../tests/test_file.txt")};
        cade_lang = new Cade_Lang(argc, argv.data());
    }

    void TearDown() override {
        delete cade_lang;
        cade_lang = nullptr;
        for (char* arg : argv) {
            free(arg);
        }
    }
};

TEST_F(CadeLangTestFixture, num_token_test) {
        std::string test_string = "hello world";
        std::vector<Token> tokens = cade_lang->tokenize(test_string);
        int num_tokens = tokens.size();
        int num_tokens_expected = 2;
        EXPECT_EQ(num_tokens, num_tokens_expected);
}

TEST_F(CadeLangTestFixture, num_token_test_2) {
        std::string test_string = "hello world this should be longer";
        std::vector<Token> tokens = cade_lang->tokenize(test_string);
        int num_tokens = tokens.size();
        int num_tokens_expected = 6;
        EXPECT_EQ(num_tokens, num_tokens_expected);
}
```

And with that, we have some passing tests with just a whitespace delimiter (I've added more tests, like for multiple spaces etc. but I don't want to clog up this page with them:

```
Running main() from ./googletest/src/gtest_main.cc
[==========] Running 4 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 4 tests from CadeLangTestFixture
[ RUN      ] CadeLangTestFixture.num_token_test
[       OK ] CadeLangTestFixture.num_token_test (0 ms)
[ RUN      ] CadeLangTestFixture.num_token_test_2
[       OK ] CadeLangTestFixture.num_token_test_2 (0 ms)
[ RUN      ] CadeLangTestFixture.num_token_test_3
[       OK ] CadeLangTestFixture.num_token_test_3 (0 ms)
[ RUN      ] CadeLangTestFixture.num_token_test_spaces
[       OK ] CadeLangTestFixture.num_token_test_spaces (0 ms)
[----------] 4 tests from CadeLangTestFixture (0 ms total)

[----------] Global test environment tear-down
[==========] 4 tests from 1 test suite ran. (0 ms total)
[  PASSED  ] 4 tests.

```

Continuing...




