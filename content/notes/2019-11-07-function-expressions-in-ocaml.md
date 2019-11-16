---
title: Function Expressions in Ocaml
author: Yifei Li
date: '2019-11-07'
slug: function-expressions-in-ocaml
categories: []
tags:
  - mcgill
  - Ocaml
draft: yes
---

Function plays a big role in Ocaml programming. What are the types of functions and how can we define a function in Ocaml? 

## Anonymous functions
**function** p -> expr \\
`function x -> x*x` (*expr being an expression,nothing fancy*)
`function x -> function y-> x+y` (*expr can also be a function expression*)

## Named functions


## Infix functions


## Higher Order functions
Let's look at an example and think about what type function `apply` is:
`let apply = function f -> function x -> f x` \\
The answer should be `('a -> 'b) -> 'a -> 'b =<fun>`