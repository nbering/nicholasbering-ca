---
title: "node-pg"
author: Brian Carlson
address: https://github.com/brianc/node-postgres
category: "Database Tools and Libraries"
---
This is the defacto standard for postgres drivers in Node. There are a number
of convenience layers built on top of node-pg, but I personally like to work
directly in raw SQL for my data access layer.

node-pg directly handles a number of essential features for a SQL database
driver, including query parameterization.

node-pg has support for both native bindings for libpq, and a 100% JavaScript
implementation which is nice because it doesn't require linked libraries or
compiling native code.
