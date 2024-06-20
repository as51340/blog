---
title: Testing distributed systems
type: page
description: My attempt to work with Jepsen, testing tool for distributed systems.
topic: Jepsen, distributed systems, safety
---

## Introduction

In the last couple of weeks I had the chance to explore Jepsen, a framework used for testing distributed systems. Jepsen cannot prove the
correctness of some distributed system but rather takes an approach of finding inconsistencies. Deep in the core, it uses txn dependency graphs
as described in the [Adya's PhD thesis](https://pmg.csail.mit.edu/papers/adya-phd.pdf). 

## Clojure

Jepsen is written in Clojure, functional language that is bidirectionally compatible with Java. One of the most important characteristics 
of Clojure is its functions. Functions are used as values, you can return them from other functions, compose them or pass them as arguments.
Clojure is classical when you consider referential transparency => if you provide functions with same inputs, they will always return the
same output.

## Jepsen

Jepsen is a testing tool for distributed systems. Its great feature is an ability to inject different kinds of faults in the system. These
faults can be on the instance, network, storage any many different levels. It works on binaries which means it greatly mimics production
situations. 

## Presentation

I had an opportunity to present Jepsen to the rest of the Memgraph team. You can find the link [here](/blog/pdfs/jepsen.pdf) with many 
diagrams and explanations of how I used Jepsen in reality.




