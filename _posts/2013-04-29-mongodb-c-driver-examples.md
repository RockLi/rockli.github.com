---
layout: post
title: "MongoDB C Driver Examples"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Recent days, I'm working with MongoDB C driver to communicate MongoDB by using capital C. There's almost no detailed articles I can refer, and the documents in MongoDB offical site doesn't reflect the latest C driver code changes. So in this post, I will try to write some codes to demonstrate the basic usage for those new guys.

Background
===
[MongoDB](http://www.mongodb.org/) is a document orinted database, schema less, it comes with lots of client drivers but this article will only cover the [mongo-c-driver](https://github.com/mongodb/mongo-c-driver). I assume you have familar with MongoDB, but have no idea how to communicate with MongoDB by using the C driver or confused with the c driver.

We already know MongoDB use the bson format for its internal storage, so I begin with the bson examples first, then continue to demonstrate how to send queries to MongoDB , check errors and play with the results.

BTW: The reason I decide to use MongoDB C driver to write clients, first is one of my client will be written in pure C, and another thing is I compared with Python Driver with the C driver, in my scenario, by using C is 1 times faster than Python at least.


BSON
===






