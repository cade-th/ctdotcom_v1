---
title: A Tourists Guide to C++ Interpreters, Part I
date: 2025-03-29
description: Making an interpreter in C++
tag: C++
author: Cade Thornton
---

# Interpreters!

Recently, I had the pleasure of seeing a great movie called "The Menu". A fascinating blend of thrill/suspense, satire, and philosophy, this film was well worth its hour and a half-ish runtime

![TheMenu](./theMenu.jpeg)

Without revealing too much of the movie's plot, the story is centered around a ravish dining experience led by an imposing head chef and his legion of cooks who serve an array of pompous upper-class guests who don't care too much for their food (except only to criticize it whenever possible).
One of the story's main characters, Tyler, is a walking Dunning-Kruger of a man played by Nicholus Hoult

![Nicholas](./nicholas.jpeg)

Tyler, unlike the other guests, has an extreme and sometimes violent reverence for the chef and his restaurant. He frequently describes every ingredient and aroma of his food in shakespearean terms and makes it a point to show the chef his enormous amount of knowledge of their menu and of food/cooking in general.  

Throughout the film's runtime, the chef pretends to be flattered by Tyler's frequent "pick me" moments and repeated attempts at impressing him. However, in a climatic turn of events, the chef instructs Tyler to come to the open-air kitchen in front of all the guests and put on a chef's apron. Tyler obliges, his enthusiasm still sky-high but a little shake in his seemingly unbreakable confidence now becoming visible. With Tyler clearly having everything he could possibly need to cook literally any dish ever, the Chef then gives Tyler a very simple directive:


![Cook](./cook.jpeg)

*"Cook."*

Tyler freezes for a second, trying to reboot his brain's suddenly over-burdened operating system after a crash. He then begins to haggle together ingredients at random around the kitchen, pretends to cut meat very carefully while his hands shake at a gigahertz frequency, haphazardly throws some butter into a iron skillet, and "cooks" his dish. All this while every member of the kitchen observes intensely, satirically pretending to "learn" from Tyler's brilliance.  

This culminates in his dish's name aptly titled below:

![Tylers](./tyler's.jpeg)

Many a time, I will be sitting at my desk and a sudden wave of imposter syndrome will tumble over me like a suprise tsunami:

*"If you asked to write code right now, what would you do?"*

*"What could you do?"*

*"What do you actually know how to do?"*

My first ideas were usually like this:

1. Throw some javascript framework and postgres garage in the same vein of Tyler's Bullshit and call it a webapp
2. Pull out the Arduino IDE and make an LED blink
3. Starting typing away in bash and make the observer think I'm a super hacker-man
4. Install openbsd on a libre-booted thinkpad
5. Vomit out my wide array of tech buzzwords learned during my computer engineering degree like "C++ Template Metaprogramming" or "Clock Domain Crossing" or "Metal oxide semiconductor field effect transistor" or "Compile time polymorphism" or "source and load impedance matching" or "surface mount soldering" or "TCP/IP stack" or "Laplace domain analysis" or "SPI/UART/I2C serial communication" or "Device tree phandle array" or "hypercube based network topology" or "I haven't actually made anything but here's a fancy gatekeeping word to make you THINK that I have"

For a long time, this was mostly all I could think to do given the directive "Write code". 

And as an aspiring code chef looking to escape the LLM copy-paste apocalypse, I needed to do something better to avoid being a dunning-kruger like Tyler.

So, where to begin?

Well, we could definitely start where everyone probably should start with the simplest possible "Hello World" example. 

For me, in 2020 while in my freshman year in college, my Introduction to Computer Science One course did this with Java, and *DEAR GOD* is that the worst possible way to introduce people to programming. To illustrate, let's compare the Java "hello world" to the python and C "hello world":

```
// Java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Nothing says programming is fun like forcing your student's to learn 9 different keywords and basic OOP and type theory *JUST* to do what is literally this in python:

```
print("hello world");
```

and this in C:
```
#include <stdio.h>
  void main() {
    println("hello world");
  }
```

Tangents about computer science education aside, if told to "Code." I would start with the standard c++ hello world:

``` 
#include <iostream>

int main() {
  std::cout << "Hello, World!" << std::endl; 
  return 0;
}
```

After, of course, undergoing the hideous nightmare of setting up C/C++ build systems and reading an entire goddamn book about CMake so I can write code for my fucking meta-build system to generate my make-based build system that generates my final shit-based build system that runs shell commands that call Clang or gcc in whatever way with which monopolistic hardware vendors want to slam unsuspecting victims with patent trolls so that not even the biggest company in the world can build a cellular modem - wait, what was I talking about again?

A tourist's guide to Interpreters in C++, ah.

This C++ program obviously doesn't do much and would only impress your local bean-counter, so let's improve it by doing the next logical step up: a simple command line program.

```
#include <iostream>

int main(int argc, char* argv[]) {
    for (int i = 0; i < argc; i++) {
        std::cout << argv[i] << "\n";
    }
}
```

There. Instead of printing a string to the console, our program now prints to the console whatever you enter as a command line argument.
But what we have here is really still about equivalent to Tyler's Bullshit, so let's take it the next logical step up and add what everyone who doesn't go outside much loves: a shell.




