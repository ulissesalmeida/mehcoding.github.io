---
layout: post
title:  A Quick Style Guide Update for Elixir 1.11
date:   2020-09-05
author: Ulisses Almeida
categories: Elixir 1.11 is coming and bringing new data constructors compile checks that will change a little bit the safest way to write Elixir code. The good news is: it is for the best. The update will make your structs attribute access less verbose and straightforward.
lang: en
excerpt: "Elixir 1.11 ."
image: /assets/elixir_guide.jpg
---

![elixir_guide](/assets/elixir_guide.jpg)

[Elixir 1.11](https://github.com/elixir-lang/elixir/blob/master/CHANGELOG.md#changelog-for-elixir-v111) is coming and bringing new data constructors compile checks that will change a little bit the safest way to write Elixir code. The good news is: it is for the best. The update will make your structs attribute access less verbose and straightforward.

Before Elixir 1.11, if we have a struct like this:

```elixir
defmodule User do
  defstruct :last_access
end
```

And if we use this way:

```elixir
user = %User{}
user.last_acess
```

We'll have a runtime error. Didn't notice the typo? No? Don't worry, neither Elixir compiler would catch. That error `last_acess` key doesn't exist error would only pop up during runtime. That's why my previous suggestions of Elixir code style was to write code like this:

```elixir
user = %User{}
%User{last_acess: last_access} = user
```

This time, the Elixir compiler will notice the not existing key and fail early, before any code run. I suggested this approach, but personally, I never loved it. It's more verbose, less straightforward, and a duplication of information. You need to tell the compiler twice that you're working the `%User{}` struct. But at the same time, we're helping the compiler catch silly mistakes as early as possible. But these days are over with the release of [Elixir 1.11 smarter compile check for data constructors](https://github.com/elixir-lang/elixir/blob/master/CHANGELOG.md#compiler-checks-data-constructors). Now, if you write that
first version:

```elixir
user = %User{}
user.last_acess
```

The Elixir 1.11 will detect that `last_acess` doesn't exist and fail during compilation phase. With this new feature, here are my new preferable styles:

```elixir
# (dynamic, preferable for test)
user.id

# (static, preferable for lib)
%User{} = user # tell compiler which struct you expect here
user.id # have fun with dot notation

# not preferred
def create_order!(item, user) do
  Repo.insert!(%Order{
    user_id: user.id,
    user_email: user.email,
    user_shipping_address_id: user.user_shipping_address_id,
    total_price: item.price,
    total_weight: item.weight,
    quantity: 1
  })
end

# preferred
# tell the compiler which structs you expect in your function signature
def create_order!(%Item{} = item, %User{} = user) do
  # have fun with dot notation
  Repo.insert!(%Order{
    user_id: user.id,
    user_email: user.email,
    user_shipping_address_id: user.user_shipping_address_id,
    total_price: item.price,
    total_weight: item.weight,
    quantity: 1
  })
end
```

That's all! I've also updated my [style guide post]({{ site.baseurl }}{% post_url 2020-02-01-elixir-style-for-maintanability %}) with the improved data constructors feature. I hope Elixir 1.11 helps you write beautiful code.

I know it's have been a time since the last post. When the mid-spring arrived, I confess it was hard to stay at home and write after a long winter and quarantine isolation. In Estonia, the quarantine period was fast, something like 2-3 months, and we could back to "normal life" and enjoy mid-spring and summer. Now, the autumn is coming, the cases are increasing again, and probably soon, we'll be stuck in quarantine again. Maybe, write more. Stay safe!
