---
title: "Node db-migrate"
author: Jeff Kunkle and Tobias Gurtzick
address: https://db-migrate.readthedocs.io/en/latest/
category: "Database Tools and Libraries"
---
Node db-migrate is a relatively light-weight utility for handling structured
version changes to your database schema. The paradigm is very similar to
migration authoring in .NET's Entity Framework, or Ruby's Active Record,
but without the heavy-weight Object Relational Mapper (ORM) component.

I generally prefer to write raw SQL for my data access layer, so this is a nice
solution for change management without tripping over an ORM.

Official support provided for Mysql, PostgreSQL, sqlite3, and MongoDB.
