---
layout: post
title:  Programming Ecto Review
date:   2019-07-29
author: Ulisses Almeida
categories: Book Reviews
lang: en
excerpt: "Last year I was granted an opportunity to review the Programming Ecto book, published by The Pragmatic Bookshelf, and written by Darin Wilson and Eric Meadows-Jönsson. Today I want to share my impressions about the book and help you decide if Programming Ecto is for you."
image: /assets/programming-ecto.jpg
---

![ecto](/assets/programming-ecto.jpg)

__*Revised by [Marcelo de Polli](https://twitter.com/mdepolli). Thank you for the awesome revision. ❤❤❤*__

Hello, my beloved readers! Last year I was granted an opportunity to review the [*Programming Ecto*](https://pragprog.com/book/wmecto/programming-ecto) book, published by The Pragmatic Bookshelf, and written by [Darin Wilson](https://twitter.com/darinwilson) and [Eric Meadows-Jönsson](https://twitter.com/emjii) (note: Eric is the creator of Ecto and an Elixir core member). It took me a few days to read it through and add all my comments, and I can spoil this: it's a great book. Recently, I got the final printed version, and I reread it. Today I want to share my impressions about the book and help you decide if *Programming Ecto* is for you.

If you're not aware, Ecto is the database library to go with Elixir. It will be hard to find an Elixir project that touches a database that doesn't use Ecto. This is due to the excellent support Ecto has received from the developer community. For instance, even José Valim, the creator of Elixir, has active participation in it. Another reason for the current Ecto popularity: it was one of the first libraries to show up. It means this library has had years of development, testing, and running in real-world applications. We can say confidently that Ecto is battle-tested.

Darin and Eric start the book displaying what the core principles of Ecto are by showing how it borrowed a lot of great ideas from other database libraries and repacked them in a consistent, explicit, flexible and very Elixir-like style. Ecto isn't a framework. Or, using the author's words: it's a suite of tools for databases. Then, the authors end the first chapter showing you how Ecto is organized, how the Repository pattern works, and how you can use the `Repo` module.

In the following chapter, the authors move on by showing how you can query your database by running the most common SQL query expressions using the Ecto query macros. It really feels like the union of two worlds has been accomplished: you have a query syntax that is very close to SQL but still is Elixir. I love how Ecto does that. You'll play with many queries without using any Ecto schema structs, which just emphasizes how Ecto is flexible and how it does not force you into a single way of doing things. In Chapter 3, the book shows you can easily cut down on tons of query code by using Ecto schema structs and associations.

After playing with a lot of repositories and queries, Darin and Eric show you how to work with changesets in Chapter 4. Changesets are a powerful Ecto feature that allows you to cast and validate parameters from external input (either from a user or external systems). They show you how to validate data without going through a schema that represents your database. They also show how to glue your database schema and associations with your validation process. The book makes it clear that schema or schemaless approaches are achievable with Ecto.

In Chapter 5, the authors dive into database transactions with Ecto and how to write clean transaction code by using Multi. The highlight here is how to wrap database and non-database operations in the same database transaction. Finally, the authors wrap up the first part of the book by showing you how to make changes to the database structure by using Ecto migrations.

The first part of the book is a guided introduction to Ecto. It's important to get everyone on the same page so we can move on to the second part. The second part is split into independent topics that give the reader the flexibility to read them in any order. It means you can just pick whatever topic is more appealing to you at the moment. You can do that without any fear of feeling lost due to missing  other chapters in the second part. I'll give my impressions on these chapters in the same order they appear in the book:

*__Adding Ecto to an Elixir Application Without Phoenix__*. At first, this can bring out some reactions along the lines of "what does a Web framework have to do with a database library?"  Well, if you don't know, Phoenix is the most popular Web framework in Elixir, and it comes integrated with Ecto by default, without any additional code from the developers. The question "how can I use Ecto without Phoenix?" is one we get more often than you imagine. It's a straightforward chapter that teaches how to add Ecto to an Elixir project and put your Repo module under your application's supervision tree.

*__Working with Changesets and Phoenix forms__*. Generating forms with your schema? Check! Need casting or validation errors? Check! Validate one and multiple associations? Check that one too!

__*Testing with Sandboxes__*. It's a fascinating topic -- how to run asynchronous tests and share connections between processes. Must read!

__*Creating and Using Custom Types__*. It's an excellent topic. The highlight here is getting to know in greater detail how Ecto SQL interfaces with your database driver.

__*Inserting and Updating with Upserts__*. This topic is more geared towards a specific scenario when you want to insert data that might already be in the database. It's a cool feature to get rid of some conditional code and do atomic operations that avoid race conditions.

__*Optimizing Your Application Design__*. A great introduction to contexts and how to associate data from different contexts. In case you ever feel like watching a never-ending discussion, just get some developers together and have them debate how contexts should be organized in your application. Everybody has a different idea or taste, but this chapter brings you a pragmatic approach.

__*Working with Embedded Schemas__*. A straightforward chapter that shows the whys and hows of working with data associations that aren't references to other tables.

__*Creating Polymorphic Associations__*. My favorite part of the book. It shows you four different ways of implementing polymorphism in your database, comparing the pros and cons of each approach. It was a delightful and insightful read.

__*Optimizing IEx for Ecto__*. This one was not for me, but you might like adding some Ecto shortcuts for your IEx session.

__*Using Schemas Without Tables__*., It shows how you can create a schema struct that's not backed by a database schema. Very useful to create form validations that might trigger one or more services or repo changes.

__*Tuning for Performance__*. The authors show you some trade-offs you might want to be aware of when optimizing your application for performance. They show you some ways of querying with Ecto that saves tons of application memory.

These are all the advanced topics that I'm sure many developers will benefit from. I believe many of these topics would be helpful to any developer, not just the Elixir crowd.

As you could see, to read this book, you need a solid grasp of Elixir as well as some database knowledge. You don't need to be a DBA expert, but you need the basics. The second part will trigger the curiosity of experienced developers. If you have years of working with Ecto and databases, maybe this book is not for you. However, I learned Ecto by reading the official documentation and practicing in production applications, and I can say that reading this was an enlightening journey. I went back to the basics, I solidified my knowledge, and I learned a lot of new tricks.

If you also decide to read it, let me know your thoughts. Happy reading!
