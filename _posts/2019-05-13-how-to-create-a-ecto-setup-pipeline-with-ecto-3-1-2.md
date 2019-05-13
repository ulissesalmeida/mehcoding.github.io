---
layout: post
title:  How to create a `ecto.setup` pipeline with Ecto 3.1.2
date:   2019-05-13
author: Ulisses Almeida
categories: Elixir
lang: en
excerpt: "The `ecto.load` command allows you to get rid of migration files. However, that command is not easy to fit in all workflows. Ecto SQL 3.1.2 added a `--skip-if-loaded` option that skips the database structure load step  when it has already been loaded, allowing you to create a robust pipeline."
image: /assets/pipeline.jpg
---

![pipeline](/assets/pipeline.jpg)

__*Revised by [Marcelo de Polli](https://twitter.com/mdepolli). Thank you for the awesome review. ❤❤❤*__

There is a well-known truth about software development: if your software is useful, it will change. If you're building or maintaining an application that has a SQL database, you need a way to keep track of structural changes. In Ecto SQL, we use migration files. These files, after a long time, can be painful to maintain. That's why the library provides the `ecto.load` mix task, which allows you to get rid of an ever-increasing amount of files. However, that command is not easy to fit in all workflows. Ecto SQL 3.1.2 added a `--skip-if-loaded` option that skips the database structure load step when it has already been loaded, allowing you to create a robust pipeline. Let's see how.

## Migrations: a love and hate relationship

Before we move on, let's go through a little more context so we can be on the same page. Your application's source code and your database live in a different universe. When you run `git pull` in your terminal to get code updates, it doesn't automatically update your database changes. If you're using Ecto SQL, you also have to run `mix ecto.migrate` -- then voilà, your database is now in the most recent version.

It looks like magic, but it isn't. Someone in your team had to write a migration file. The migration files usually live on `priv/repo/migrations` of your application folder. These filenames begin with a timestamp, and Ecto uses that information to keep track of which migrations have or have not been run, and it does so in a table called `schema_migrations`. You can also check the migration status by doing `mix ecto.migrations`.

That's a very efficient way to organize and keep track of your database changes. However, if your software is a constant change, after a while, these migrations may stop making sense altogether. For example:

```
migrations/
  [timestamp]_create_users.exs
  [timestamp]_create_flags.exs
  [timestamp]_add_age_to_users.exs
  ...tons of files later
  [timestamp]_drop_flags.exs
  ...tons of files later
  [timestamp]_remove_age_from_users.exs
  ...tons of files
```

In this example, without looking at the contents of each file, we can see that we created a `flags` table and we added an `age` column to the `users` table. After a while, we deleted the `flags` table and the `age` column from `users`. The last two files negate the effects of the previous ones; it means we have four unnecessary files in our migrations folder that could be either skipped or deleted.

The problem can be even worse if your old migration files are referencing old module functions for some reason. If you removed the old modules, you'd have to remove those references from your migrations as well. That would mean maintaining files that were run only once in production; they don't even need to run there anymore. I don't know about you, but for me, that sounds like a waste of time. No code runs faster and is easier to maintain than any code.

## `ecto.load` and `ecto.dump` for the rescue!

Ecto SQL gives you the option to keep a `structure.sql` by running the `mix ecto.dump`  command. You have to keep that file in your source code, and that contains the most recent structure of your application database. By using running `mix ecto.load`, Ecto uses the `structure.sql` file to load the database structure, skipping running all of the migrations and, in the same operation, you can still have all migrations tracked in the `schema_migrations` table just as before. If you then try to `ecto.migrate` after loading the structure, nothing happens, because everything is already there.

You can keep the `structure.sql` file updated by hooking the `ecto.migration` and `ecto.rollback` commands to dump the structure of the database. For example, in your `mix.exs` file:

```elixir
defp aliases do
  [
    "ecto.migrate": ["ecto.migrate", "ecto.dump"],
    "ecto.rollback": ["ecto.rollback", "ecto.dump"]
  ]
end
```

If the code above, every time you run `mix ecto.migrate` or `mix ecto.rollback` it updates your `structure.sql` file by dumping your database structure. For brand new setups, you can run `mix ecto.load` and it will skip all the migrations and load up the `structure.sql` file. For existing setups, you can keep running `mix ecto.migrate` as usual.

## Something is rotten in the state of Denmark

There are two downsides to having a `structure.sql` file. First, it's a huge file, and any branching conflicts that happen in it can be painful to solve. Second, you'll have to depend on two commands now rather than one -- for new systems `mix ecto.load`, for old systems `mix ecto.migrate`, these two options can make it a bit harder to integrate into some DevOps release strategies.

As for the first problem, if you're using `git`, you can use its power instead of touching your `structure.sql`. You can run, for example, `git checkout --ours` when merging or `git checkout --theirs` when rebasing, followed by`mix ecto.migrate`, and that's it, you did it. You now have the most recent `structure.sql`.

For the second problem, you can create a single pipeline. For example:

```elixir
defp aliases do
  [
    "ecto.setup": ["ecto.create", "ecto.load", "ecto.migrate", "run priv/repo/seeds.exs"],
#...
```

If you run `mix ecto.setup`, Ecto tries to create the repository database, load the database structure present in the `structure.sql` file, run any pending migrations and finally run the seeds. However, unfortunately, it doesn't work in Ecto SQL versions before `3.1.2`. Why?

The `ecto.create` command gracefully succeeds if the database has already been created. However, `ecto.load` fails loudly when it tries to load the file in a database that has an already present structure. This failure interrupts the pipeline execution, preventing it from running the following migrations and seed commands. This behaviour makes it impossible to have a single command that works for both old and new installations.

## `ecto.load --skip-if-loaded` saves the day

Fortunately for us, the Ecto SQL team released the `3.1.2` version that comes with the `--skip-if-loaded` option. That option checks if your database contains a migrations table. If it does, that means a structure is living there, and the command succeeds and avoids loading it again. Then, our Ecto aliases can look like this:

```elixir
defp aliases do
  [
    "ecto.setup": [
      "ecto.create",
      "ecto.load --skip-if-loaded",
      "ecto.migrate",
      "run priv/repo/seeds.exs"
    ],
    "ecto.migrate": ["ecto.migrate", "ecto.dump"],
    "ecto.rollback": ["ecto.rollback", "ecto.dump"]
  ]
end
```

This way, `ecto.setup` can now be used on old systems as well, because it runs the migrations and the seeds file while skipping database creation and structure. It also works on new systems since it creates the database, loads the structure, runs the seeds and skips the migrations.

Now it's up to you to decide how long you want to keep the migration files. In most cases, after running the migrations successfully in a production environment, you can get rid of them. In my scenario, we have the same system installed in many environments, some of which our team does not control. We're delivering changes faster than people installing them in their environments. Then, it may take some days (or even weeks) until we can safely delete old migration files.

## Wrapping up

We learned how migrations are useful and how they also can be a pain to maintain. We saw how keeping `structure.sql` can reduce the need of keeping the migration files and how Ecto SQL `3.1.2` allows us to have a single command for setting up new systems and keeping old systems up to date. That's it; I hope you liked the post, and let me know, how do you keep your migration files?
