---
layout: post
title:  "CSP and You"
date:   2014-06-20 08:59:20
categories: concurrency
---

I've been writing a lot of Golang lately; enough, in fact, to push it on several of my coworkers.  The majority of our system is built around RabbitMQ, where we pass JSON around, and I've been using this as an argument to switch languages.  The first order of business, was to create a nice [abstraction to the Golang AMQP gem](https://github.com/danielscottt/amqp-controller).  In doing this, I came to know Go's [chans](http://golang.org/doc/effective_go.html#channels).  Go's cross-concurrent-process message passing is based on CSP: Communcating Sequential Processes.  Today, CSP is a formal algebraic language for expressing cross-process message passing, but it has evolved quite a bit since its inception.

Let's go back:

It all starts with a fellow named Carl Hewitt, who, in 1974, was writing working paper 83: _PROTECTION and SYNCHRONIZATION
in ACTOR SYSTEMS_.  Working paper 83 discusses message passing in the Actor model.
