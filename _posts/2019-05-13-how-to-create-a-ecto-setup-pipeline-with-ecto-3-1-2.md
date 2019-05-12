---
layout: post
title:  How to create a `ecto.setup` pipeline with Ecto 3.1.2
date:   2019-05-13
author: Ulisses Almeida
categories: Elixir
lang: en
excerpt: "The `ecto.load` that allows you to get rid of the migrations files. However, that command is not easy to fit in all workflows. The Ecto SQL 3.1.2 added an option `--skip-if-loaded` that allows you to skip the database structure load in when it is loaded, allowing you to create a robust pipeline."
image: /assets/pipeline.jpg
---

![pipeline](/assets/pipeline.jpg)

There is a truth about software development: if your software is useful; it will change. If you're building or maintaining an application with a SQL database, you need a way to publish the structure changes. In Ecto SQL, we use the migrations files. These files, after a long time, can be painful to maintain. That's why provides `ecto.load` that allows you to get rid of the files. However, that command is not easy to fit in all workflows. The Ecto SQL 3.1.2 added an option `--skip-if-loaded` that allows you to skip the database structure load in when it is loaded, allowing you to create a robust pipeline. Let's see how.

## Love & Hate migrations

Before proceeding, let's give you more context and then we can be on the same page. Your application source code and your database live in a universe. When you run `git pull` in your terminal to get code updates, it doesn't automatically update your database changes. If you're using Ecto SQL, you also have to run `mix ecto.migrate`, then voilà, your database is now in the most recent version.

Looks like magic, but isn't. Somebody in your team had to write a migration file. The migrations files usually live on `priv/repo/migrations` of your application folder. These files names begin with a timestamp in their names, and Ecto uses that information to keep track of migration that had run or not in a table called `schema_migrations`. You can also check the migration status with `mix ecto.migrations`.

It's a very efficient way to organize and keep track of your database changes. However, if your software is a constant change, after a while, these migrations might not make sense anymore. For example:

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

In this example, without seeing the contents of the file, we can see that we created a table `flags` and we added `age` column to the table `users`. After a while, we deleted the table `flags` and the `age` from `users`. The last 2 files negate the effects of the first ones; it means we have 4 unnecessary files in our migrations folder that could be skipped or deleted.

The problem can be even worse if your old migrations files are referencing for some reason old modules functions. If you remove the old modules, you have to remove from these migration files too. You have to keep maintaining files that had run once in production; it never had to run there anymore. I don't know for you, but for me sound like a waste of time. No code run faster and is easier to maintain than any code.

## `ecto.load` and `ecto.dump` for the rescue!

Ecto SQL gives you a shine option to keep a `structure.sql` file in your source code that has the most recent structure of the database. Using this file, you can skip running all the migrations, and in a single operation, you can have the most recent structure of the database with all migrations tracked in the `schema_migrations` table. If you try to `ecto.migrate` in sequence, nothing will happen, because everything is there.

You can keep the `structure.sql` file updated by hooking the `ecto.migration`
and `ecto.rollback` commands to dump the structure of the database. For example, in your `mix.exs` file:

```elixir
defp aliases do
  [
    "ecto.migrate": ["ecto.migrate", "ecto.dump"],
    "ecto.rollback": ["ecto.rollback", "ecto.dump"]
  ]
end
```

If the code above, every time you run `mix ecto.migrate` or `mix ecto.rollback` it  updates your `structure.sql` file by dumping your database structure. For new systems, you can run `mix ecto.load`, it skips all the migrations, and run the `structure.sql` file. For existing systems, you can keep running `mix ecto.migrate` as usual.

## Something is rotten in the state of Denmark

There are two downsides of having `structure.sql`. First, it's a huge file and
can be a pain to solve branching conflicts in your source version control system. Second, you depend on two commands now, for new systems `mix ecto.load`, for old systems `mix ecto.migrate`, this two options can make hard integrate into some DevOps release strategies.

The first problem, if you're using `git`, you can use its power instead of
touching the `structure.sql`. You can run for example, `git checkout --ours`
on merging, or `git checkout --theirs` on rebasing, then run `mix ecto.migrate`, and you did it. You have the most recent `structure.sql`.

The second problem, you can create a single pipeline, for example:

```elixir
defp aliases do
  [
    "ecto.setup": ["ecto.create", "ecto.load", "ecto.migrate", "run priv/repo/seeds.exs"],
#...
```

If you run `mix ecto.setup`, Ecto tries to create the repository database. Then, load the database structure present in the `structure.sql` file, run any pending migration and finally run the seeds. But unfortunately, it doesn't work in the Ecto SQL version before `3.1.2`. Why?

The `ecto.create` command gracefully succeed if the database is already created. However, the `ecto.load`  loudly fails when it tries to load the file in a database that has a structure present. This failure interrupts the pipeline execution, and it does not run the following migrations and seed commands. This behaviour makes it impossible to have a single command that runs for old and new installations.

## `ecto.load --skip-if-loaded` saves the day

Ecto SQL team launched the `3.1.2` version that came with the `--skip-if-loaded` option. That option checks if your database contains a migrations table, if yes, it means a structure is living there, and the command succeeds and avoids to load it again. Then, our Ecto aliases can look like this:

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

Then, with `ecto.setup` can run on old systems, because it runs the
migrations and the seed file and skips the database creation and structure. It also works for new systems, since it creates the database, loads the structure, runs the seed and skips the migrations.

Now, it's up to you to decide how long you want to keep the migrations files.
Most of the cases, after you run the migrations successfully in a production environment, you can get rid of them. In my scenario, we have the same system installed in many environments, some of them our team don't control, then it might take some days(sometimes weeks) until we can safely delete old migration files.

## Wrapping up

We learned how migrations are useful and how they also can be a pain to maintain. We saw how keeping `structure.sql` can reduce the need of keeping the migration files and how Ecto SQL `3.1.2` allow us to have a single command to set up new systems or keep old systems up to date. How you keep your migrations files?
