---
layout: post
title:  Announcing Elixir CPF 1.0.0
date:   2019-10-31
author: Ulisses Almeida
categories: Elixir
lang: en
excerpt: "Hello, my dearest readers. I'm here to let you know I've been working in a tiny CPF validation library for Elixir. Let's start by telling you where the motivation to write a new library came from."
image: /assets/cpf.jpg
---

![cpf](/assets/cpf.jpg)

Hello, my dearest readers. I'm here to let you know I've been working in a [tiny CPF validation library for Elixir](https://hex.pm/packages/cpf). Let's start by telling you where the motivation to write a new library came from.

This year I visited [ElixirConf EU 2019](https://www.elixirconf.eu/elixirconfeu2019).
It was my first time visiting a programming conference outside of Brazil. It was very cool to meet many developers and speakers from different countries. In that event had fantastic talks; one was very special for me: [Bridging the Divide A Philosophy of Library Design](https://www.youtube.com/watch?v=fxe5NzHDxpk) by Brooklin Zelenka. That talk gave me the inspiration to try creating a [cpf library](https://github.com/ulissesalmeida/cpf) following the concepts that she talked. And it's done. Ok, I know; software is never really done. But I believe I reached a point that is good for me, and probably it can be useful for more people.

## What the hell is CPF?

CPF is a customer ID number of Brazilians. It's possible to verify if the given number is valid by using the check digit algorithm similar to [ISBN 10](https://en.wikipedia.org/wiki/Check_digit#ISBN_10).

## Why a new CPF validation library?

Recently, working for an international company, we had to validate the CPF of Brazilian customers. We dug out some Elixir libraries and missed a library with:

* License (seriously, your open source library must be under some license)
* English documentation (international companies usually prefer to have stuff in English)
* A library that provides functions that follows the usability principles of Elixir core

One option would be to request changes in the existent libraries. But I didn't choose this option because these libraries in their core took a different approach from what I wanted. Then, I had a feeling that to exercise the concepts of Zelenka's talk; I would have to create a new CPF library. I liked the results, maybe more people might like it too. That's why I'm writing this blog post.

## What I like about `cpf` library?

In Zelenka's talk, she mentions that the Programming Language design principles should extend in the libraries. If a library breaks the balance of following principles, it ends up not playing nice with the language itself and other libraries. When developers try to plug pieces of code that doesn't play well together leads to a big frustration. With that in mind, I tried to build this library following the Elixir and other famous libraries like `ecto` design principles. For example, it's widespread in Elixir to have quick benefit from a library or module by using the language primitives. For example, the `CPF.valid?` function:

```elixir
CPF.valid?("247.999.735-92")
# true
CPF.valid?("24799973592")
# true
CPF.valid?(24799973592)
# true
CPF.valid?("abliidebob")
# false
```

You can quickly check if a positive integer or string is a valid CPF. No extra structs or dependencies are needed.

Another example is when you might want to do additional operations in a valid CPF. In Elixir,  we usually have helper functions that let you build a correct type that you can work. Check Elixir `Integer`, `DateTime` modules as an example. In `cpf` library, you can use `CPF.parse!` or `CPF.parse`:

```elixir
"24799973592" |> CPF.parse!() |> CPF.format()
# "247.999.735-92"
"247.999.735-92" |> CPF.parse!() |> to_string()
# "24799973592"
```

You can also use the CPF generator function from the command line:

```console
$ mix cpf.gen
194.925.115-25

$ mix cpf.gen --format=digits --count=2
19492511525
65313188640
```

Finally, the last example from Zelenka's talks is about a library should extend another library by respecting its principles. If you are using `ecto`, you can use CPF as a type, and put it as field type of your schema like any other `ecto` types. Take a look:

```elixir
# in your schema module
import CPF.Ecto.Type

schema "profiles" do
  field :cpf, cpf_type(:string)
end

# in other point of your app
{:error, changeset} =
  MySchema.new()
  |> MySchema.changeset(%{cpf: "abilidebob"})
  |> MyApp.Repo.insert()

Keyword.get(changeset.errors, :cpf)
# {"is invalid", [reason: :invalid_format]}
```

If you think that adding a type is too strict, you can use validations with `CPF.Ecto.Changeset.validate_cpf/2`:

```elixir
{:error, changeset} =
  MySchema.new()
  |> MySchema.changeset(%{cpf: "abilidebob"})
  |> CPF.Ecto.Changeset.validate_cpf(:cpf)
  |> MyApp.Repo.insert()

Keyword.get(changeset.errors, :cpf)
# {"is invalid", [reason: :invalid_format]}
```

This `ecto` integration was the last feature added to release the `v1.0.0`. Now, I can say the library is in parity on number features compared with others that exist.

## Future projects and libraries

I don't have big plans for `cpf` because the scope of the project is tiny. My goal was to do something that I like and exercise the concepts that I learned from Zelenka's talk. There's not much more that we can do here. However, I'm planning to release his sister, the `CNPJ` library.

The goal of the `CNPJ` library is education. I mean, for the next library, I'll do video streaming through [Twitch](https://www.twitch.tv/anizark) and build together with the community sharing everything that I learned by building the CPF. I believe it will be fun. It's very similar to what [Philip](https://www.twitch.tv/philipsampaio) does with his [Floki](https://github.com/philss/floki) library. The streamings will start in January 2020, then, stay tuned.

Have you tried `cpf` library? Did you like it? Let your impressions here.
