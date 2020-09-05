---
layout: post
title:  More Elixir Guidelines for Code Maintainability
date:   2020-02-19
author: Ulisses Almeida
categories: Elixir
lang: en
excerpt: "Today I would like to share my thoughts about some Elixir good practices in writing maintainable code. The guidelines below cover topics that aren't present in the currently available style guides. They can be counterintuitive or controversial; so talk it over with your team before adopting them."
image: /assets/elixir_guide.jpg
---

![elixir_guide](/assets/elixir_guide.jpg)

__*Revised by [Marcelo de Polli](https://twitter.com/mdepolli). Thank you for the awesome revision. ❤❤❤*__

Hello! Today I would like to share my thoughts about some Elixir good practices in writing maintainable code. Thanks to the Elixir ecosystem, I don't have to write too much about it. We have the useful [Elixir formatter](https://hexdocs.pm/mix/master/Mix.Tasks.Format.html), the awesome [Credo](https://github.com/rrrene/credo), and even various([here one](https://github.com/christopheradams/elixir_style_guide)) useful ([another one](https://github.com/rrrene/elixir-style-guide#readme))
guidelines ([one more](https://github.com/lexmag/elixir-style-guide#readme)) to improve your Elixir code style. But still, I would like to add a new one. The guidelines below cover topics that aren't present in the currently available style guides. They can be counterintuitive or controversial; so talk it over with your team before adopting them.

## Typespecs on public functions

> Elixir is a dynamically typed language, and as such, type specifications are
never used by the compiler to optimize or modify code. [Elixir lang](https://hexdocs.pm/elixir/typespecs.html)

The Elixir docs also explain that typespecs can be used with Dialyzer to do static analysis and catch some bugs. Honestly, I was very resistant to `typespecs` at the beginning of my journey in Elixir. But today, I can see it helps you understand the inputs and outputs of your functions in a small and clean way.

### Example

```elixir
# not preferred
def calculate(%Item{}) do
# ...
def calculate(%Product{}) do
# ...
def calculate(number) when is_integer(number) do
# ...

# preferred
@spec calculate(Item.t() | Product.t() | integer) :: integer
def calculate(%Item{}) do
# ...
```

In Elixir, you can have multiple clauses for the same function with pattern matching and guard clauses. Thanks to that, you can dynamically dispatch different expressions given different data types. But the downside is it's hard to figure out in a quick glance how many data types are supported. When you use typespecs, they provide an excellent introduction to your function definitions. Remember, you don't need to add type specifications to your private functions, but feel free to add them to most of your public functions. Once you adopt them, you start being more thoughtful of how your function interfaces are structured and how well your boundaries are defined.

## Elixirfy the external world

A task that I often do in my current job is integrating with external service providers. These third-party services usually offer some HTTP API with JSON objects. The biggest temptation for all Elixir developers is doing `Jason.parse!(payload)` and then working with a dynamic map through the rest of the application code. That's fine for a start, but it can easily make your code harder to understand after a few weeks.

### Example

```elixir
# not preferred
GitHub.get_repo!("ulissesalmeida/cpf")
# => %{id: 2323, name: "cpf", owner: "ulissesalmeida"}

# preferred
GitHub.get_repo!("ulissesalmeida/cpf")
# => %GitHub.Repo{id: 2323, name: "cpf", owner: "ulissesalmeida"}

# not preferred
def transaction_exists?(id),
  do: "transactions/#{id}" |> get!() |> Jason.parse!()

# preferred
def transaction_exists?(id) do
  case get("transactions/#{id}") do
    HTTP.Reponse{status: 200} -> {:ok, true}
    HTTP.Reponse{status: 404} -> {:ok, false}
    _ -> {:error, :bad_server}
  end
end
```

The preferable path here is to define a struct or to use an Elixir data type to represent that payload. This enables you to use typespecs to do static analysis, attribute access with a compile-time check, and your future self won't always need to read external documentation to understand which kind of data they are dealing with. I suggest you only map those attributes you need when the external service contains a very complex JSON.

## Use compile-friendly style in lib, dynamic style in test

Elixir compiles all the code that lives in the `lib/` folder by doing some checks, and if everything is ok, your software is ready to run. While your software is running, some errors still might appear, preventing that, developers do manual and automated testing. The last one, the automation code, usually lives in the `test/` folder. I suggest all we adopt a different style of coding, depending on the folder we working.

### Example

```elixir
# (dynamic, preferable for test)
user.id

# (static, preferable for lib)
%User{} = user
user.id

# (static, preferable when your Elixir's version is previous than 1.11 or
# short assignments)
%User{id: user_id} = user

# (dynamic, preferable for test)
user.status == :ACTIVE

# (static, preferable for lib)
User.active?(user)

defmodule User do
  alias User.Status
  # ...
  def active?(%__MODULE__{status: status}), do: status == Status.active()
end

defmodule User.Status do
  def active(), do: :ACTIVE
end

# (dynamic, preferable for test)
put_in(user.name, "Ulisses")

# (static, preferable for lib)
%User{} = user
%{user | name: "Ulisses"}

# (static, preferable when your Elixir's version is previous than 1.11 or
# short updates)
%User{user | name: "Ulisses"}
```

As you can see, the static version is more verbose, and I recommend that for the `lib/` folder. It can prevent silly typo mistakes where the automated test is not covering and simplify future refactoring. For the test code in the `/test` folder or any other script code, feel free to use the dynamic style. Why? Well, because the runtime of the test code is in CI or our machines. Usually, when we run `mix test` executes all test code, any error there we'll have instant feedback. Then, it's safe having a dynamic approach.

If we have unit tests, why bother too much with static analysis? I had the same resistance in the past, but now I can sleep better with these guarantees that it provides while the downside of being more verbose is not that bad. Being more explicit about the structures you are using helps you localize them when you want to refactor and read the details about them.

## Pattern matching in function definitions

If you start to use a compile-friendly style. Where to put it? I suggest you leave them in function clauses to leave your function body clean.

### Example

```elixir
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
def create_order!(%Item{} = item, %User{} = user) do
  Repo.insert!(%Order{
    user_id: user.id,
    user_email: user.email,
    user_shipping_address_id: user.user_shipping_address_id,
    total_price: item.price,
    total_weight: item.weight,
    quantity: 1
  })
end

# compile-friendly, preferred when your Elixir's version is previous than 1.11
def create_order!(item, user) do
  %User{
    id: user_id,
    email: user_email,
    shipping_address_id: user_shipping_address_id
  } = user

  %Item{
    price: item_price,
    weight: item_weight
  } = item

  Repo.insert!(%Order{
    user_id: user_id,
    user_email: user_email,
    user_shipping_address_id: user_shipping_address_id,
    total_price: item_price,
    total_weight: item_weight,
    quantity: 1
  })
end

# preferred when your Elixir's version is previous than 1.11
@spec create_order!(Item.t(), User.t()) :: Order.t()
def create_order!(
      %Item{
        price: item_price,
        weight: item_weight
      },
      %User{
        id: user_id,
        email: user_email,
        shipping_address_id: user_shipping_address_id
      }
    ) do
  Repo.insert!(%Order{
    user_id: user_id,
    user_email: user_email,
    user_shipping_address_id: user_shipping_address_id,
    total_price: item_price,
    total_weight: item_weight,
    quantity: 1
  })
end
```

The preferred version is a little bit more verbose than its dynamic version, but let's raise the advantages:

  * It is very explicit with which structure is expected from the arguments
  * The function's body is clean
  * The compiler can catch typo mistakes

The downside is when your Elixir versions is before the 1.11, the extended function definition that is not easy and fast to read. However, if we use `typespecs`, we can mitigate that problem. But I strong suggest you upgrade your Elixir to the newest version and enjoy the improvements of the compiler on
data constructors.

## Use `if`

I love Elixir's functional style to work with boolean conditionals, specially when I'm working with recursive functions. However, if we abuse the use of function clauses to control the flow of our code, we're adding a lot of unnecessary indirections and making the code harder to understand.

### Example

```elixir
# not preferred

def create_order(item, user) do
  may_create_order(item, user, User.active?(user))
end

defp may_create_order(item, user, false), do: {:error, :user_not_active}
defp may_create_order(item, user, true) do
  #...
end

# preferred

def create_order(item, user) do
  if User.active?(user) do
    # ...
  else
    {:error, :user_not_active}
  end
end
```

The not preferred version we need to write and read an extra function. This extra step forces us to jump to another function and to think about a name for it. Remember, naming things is hard! All that for what? To not write an `if`? Please, write the `if` for boolean conditionals! It makes your code intention clear.

## Use `case`

Very similar to the `if` scenario, thanks to Elixir pattern matching in function clauses, we can handle conditionals code based on the data structure. However, if we abuse it, we'll end up with unnecessary indirections. Use `case` and save jumps to unnecessary function clauses.

### Example

```elixir
# not preferred
def create_order(user, item) do
  %{user: user, item: item}
  |> PayPal.create_order()
  |> handle_create_order_response()
end

defp handle_create_order_response({:ok, json}),
  do: {:ok, Order.new(json)}
defp handle_create_order_response({:error, json}),
  do: {:ok, CreateOrderError.new(json)}

# preferred

def create_order(user, item) do
  case PayPal.create_order(%{user: user, item: item}) do
    {:ok, json} -> {:ok, Order.new(json)}
    {:error, json} -> {:ok, CreateOrderError.new(json)}
  end
end
```

Extra functions force the developer to jump while reading code and think about a new function name when developing. On top of that, the tuple argument of `handle_create_order_response/1` is very uncommon to see. You can fix all that by using the `case` statement. It removes all the problems of extra functions brings, and the code is way more straightforward.

## Use `cond`

Sometimes your boolean code might end up with more than two paths. Elixir's function clauses can help you with that, but these extra functions can leave the code with bad function names and unnecessary indirections.

### Example

```elixir
# not preferred
def create_order(user, item) do
  may_create_order(user, item, User.active?(user), Item.available?(item))
end

defp may_create_order(user, item, false, _), do: {:error, :user_not_active}
defp may_create_order(user, item, _, false), do: {:error, :item_not_available}
defp may_create_order(user, item, true, true) do
  # ...
end

# preferred
def create_order(user, item) do
  cond do
    not User.active?(user) -> {:error, :user_not_active}
    not Item.available?(item) -> {:error, :item_not_available}
    true -> # ...
  end
end
```

The function clauses style doesn't scale well with boolean conditionals; you'll end up with a lot of boolean arguments that are very hard to track. The `cond` statement is your friend, make the code easier to understand and without extra functions to handle.

## Use `with`

The `with` was the latest addition to the Elixir's statements family, and was designed to help you with a chain of conditionals operations. We have the Elixir's pipe operators(`|>`) that works well with pure functions or stable returns. But, when we need to deal with conditional returns, the pipe version will require extra functions to branch the code.

### Example

```elixir
# not preferable
def create_oder(user_id, item_id) do
  user_id
  |> find_user()
  |> find_item(item_id)
  |> may_create_order()
end

defp find_user(user_id), do: Users.find(user_id)

defp find_item({:ok, user}, item_id),
  do: {user, Item.find(item_id)}
defp find_item({:error, _reason} = error, _item_id), do: error

defp may_create_order({user, {:error, _reason} = error}), do: error
defp may_create_order({user, {:ok, item}}), do: #...

# preferable
def create_oder(user_id, item_id) do
  with {:ok, user} <- User.find(user_id),
       {:ok, item} <- Item.find(item_id) do
         # ...
  end
end
```

The `with` version is shorter, easier to read, and doesn't have extra unnecessary functions. While the other version tries to put everything under the friendly pipe operator, but the result is convoluted.

## Use `for`

The `for` statement is Elixir's list comprehension mechanism that can allow you to build compelling operations with few expressions. Every time you end up doing maps, filters and reduce, give a try to `for` and check if the result isn't easier to understand.

### Example

```elixir
# not preferable
products
|> Enum.filter(& &1["available"])
|> Enum.map(&Map.new(&1, fn {key, value} -> {String.upcase(key), value} end))

# preferable (but too verbose if there's no reusability)
products
|> Enum.filter(&by_availability/1)
|> Enum.map(&to_upcase_keys/1)

defp by_availability(product), do: product["available"]

defp to_upcase_keys(product), do:
  Map.new(product, fn {key, value} -> {String.upcase(key), value} end)

# preferable
for product <- products,
    product["available"],
    {key, value} <- product,
    into: %{},
    do: {String.upcase(key), value}
```

Maybe at this point, you're not sold to the `for` version. But, trust me, when you dominate the Elixir's `for`, it starts to look more pleasant and straightforward.

## Prefer `with` over `case` to bubble up errors.

When you're building a function that bubbles up errors, and you only care about the successful results, the `with` is your friend again. Every time you omit the `else` in `with`, the not matching result is returned automatically.

### Example

```elixir
# not preferable
def create_user(username) do
  case Repo.insert(%User{username: username}) do
    {:ok, %User{id: id}} -> {:ok, id}
    error -> error
  end
end

# preferable
def create_user(username) do
  with {:ok, %User{id: id}} <- Repo.insert(%User{username: username}) do
    {:ok, id}
  end
end
```

The case version, the `error -> error` clause does nothing; it only bubbles up the not matching result. Then, you can use `with` and save you one line of code. 🤣 It's a very straightforward approach when you have in mind that `with` always return the not matching clause.

## Wrap the errors with context

Sometimes you need handle different cases of failure scenarios when you are chaining function calls using `with`. However, when the errors have no context, the task can be impossible.

### Example

```elixir
# not preferable
def create_oder(user_id, item_id) do
  with {:ok, user} <- User.find(user_id),
       {:ok, item} <- Item.find(item_id) do
         # ...
  else
    {:error, :not_found} ->
      # What's not found, user or item ?
  end
end

# preferable
def create_oder(user_id, item_id) do
  with {:ok, user} <- User.find(user_id),
       {:ok, item} <- Item.find(item_id) do
         # ...
  else
    {:error, {:users, :not_found}} ->
      # ...
    {:error, {:items, :not_found}} ->
      # ...
  end
end

# preferable (specially if you have bang version `create_user!` that raises exception)
def create_oder(user_id, item_id) do
  with {:ok, user} <- User.find(user_id),
       {:ok, item} <- Item.find(item_id) do
         # ...
  else
    {:error, %UserNotFoundError{}} ->
      # ...
    {:error, %ItemNotFoundError{}}} ->
      # ...
  end
end
```

The preferred version shows how is trivial handle specific failures scenarios when you wrap errors in a context.

## Keep last expression of your functions explicit

The last expression of an Elixir function is very important because it returns the resulting value for the caller. When refactoring your code, keep in mind in not hide it too much; otherwise, you'll make it harder to have a quick read.

### Example

```elixir
# not preferable
def create_user(username) do
  case Repo.insert(%User{username: username}) do
    {:ok, user} -> handle_ok(user)
    error -> handle_error(error)
  end
end

defp handle_ok(%User{id: id}), do: {:ok, id}

defp handle_error({:error, reason}) do
  Logger.error("Couldn't insert user: #{inspect(reason)}")
  {:error, reason}
end

# preferable
def create_user(username) do
  case Repo.insert(%User{username: username}) do
    {:ok, %User{id: id}} -> {:ok, id}

    {:error, reason} = error ->
      Logger.error("Couldn't insert user: #{inspect(reason)}")
      error
  end
end
```

When we want to have a quick understanding of a function, fewer indirections jumps is better. The not preferable version is trying to make the `create_user/1` shorter, but not easy to understand. The public conclusion was abstracted in two more extra functions without hiding any substantial work or detail.

## Keep related code together

We already have lots of discussions about code organization in Elixir. It looks like the community is moving to the Phoenix Context Pattern of code organization. I like this approach, mainly because the idea is to keep the related code close. I believe we should apply a similar approach to the functions organization inside of a module.

### Example

```elixir
# not preferable
defmodule Orders do
  @doc """ pub_fun_a """
  @typespec pub_fun_a
  def pub_fun_a

  @doc """ pub_fun_b """
  @typespec pub_fun_b
  def pub_fun_b

  @doc """ pub_fun_c """
  @typespec pub_fun_c
  def pub_fun_c

  defp helper_of_pub_fun_a

  defp helper_of_pub_fun_b

  defp helper_of_pub_fun_c

  defp helper_of_pub_fun_a_2

  defp generic_helper

  defp helper_of_pub_fun_c_2
end

# preferable
defmodule Orders do
  @doc """ pub_fun_a """
  @typespec pub_fun_a
  def pub_fun_a

  defp helper_of_pub_fun_a

  defp helper_of_pub_fun_a_2

  @doc """ pub_fun_b """
  @typespec pub_fun_b
  def pub_fun_b

  defp helper_of_pub_fun_b

  @doc """ pub_fun_c """
  @typespec pub_fun_c
  def pub_fun_c

  defp helper_of_pub_fun_c

  defp helper_of_pub_fun_c_2

  defp generic_helper
end
```

It might sound counterintuitive because, in many languages, we usually put the public functions on top and the private ones on the bottom. But, this approach leads us to many unnecessary scrolls when navigating the file. Keeping the related private functions together reduces the need for scroll, and it's easier to refactor when you need to move it to another module.

### Wrapping up

I believe that it's essential to discuss how a code can be easier understood, fixed, and upgraded (that's maintainability). I know that the "understand" part sometimes can be very subjective. When we're refactoring a make a code to looks more beautiful, it doesn't mean that we're making it easier to understand. That's was a tough lesson that I learned in recent years.

Now, let's see an example of a code that's very hard to understand and two refactorings. The "preferable" version is applying everything that I suggested in this post. While the "not preferable" goes to the opposite direction.

## Example

```elixir
# code hard to understand
def create_oder(user_id, item_id) do
  case Users.find(user_id) do
    {:ok, user} ->
      if user.status == :ACTIVE do
        case Items.find(item_id) do
          {:ok, item} ->
            if item.stock_quantity > 0 do
              Repo.insert(Order%{user_id: user.id, item_id: item.id})
            else
              {:error, {:item_not_available}}
            end
          error -> error
        end
      else
        {:error, :user_not_active}
      end
    error -> error
  end
end

# not preferable refactoring
def create_oder(user_id, item_id) do
  user_id
  |> find_user()
  |> find_item(item_id)
  |> insert_order()
end

defp find_user(user_id), do: Users.find(user_id)

defp find_item({:ok, user}, item_id), do: find_item(user, user.status == :ACTIVE, item_id)
defp find_item(error, _item_id), do: error

defp find_item(user, false, item_id), do: {:error, :user_not_active}
defp find_item(user, true, item_id), do: {Items.find(item_id), user}

defp insert_order({{ "{{:ok, item" }} }, user}), do: insert_order(item, user, item.stock_quantity > 0)
defp insert_order({error, _user}), do: error

defp insert_order(item, user, false), do: {:error, :item_not_available}
defp insert_order(item, user, true), do: Repo.insert(%Order{user_id: user.id, item_id: item.id})

# preferable
@type id :: pos_integer
@type create_order_errors :: {:orders, :user_not_active | :item_not_available | Changeset.t()}

@spec create_order(user_id :: id, item_id :: id) ::
        {:ok, Order.t()}
        | {:error,
           Users.not_found_error()
           | Items.not_found_error()
           | create_order_errors()}
 def create_order(user_id, item_id) do
   with {:ok, user} <- Users.find(user_id),
        {:user_active?, true} <- {:user_active?, User.active?(user)},
        {:ok, item} <- Items.find(item_id),
        {:item_available?, true} <-
          {:item_available?, Item.available?(item)},
        {:ok, order} <-
          Repo.insert(%Order{user_id: user_id, item_id: item_id}) do
     {:ok, order}
   else
     {:user_active?, false} -> {:error, {:orders, :user_not_active}}
     {:item_available?, false} -> {:error, {:orders, :item_not_available}}
     {:error, %Changeset{} = changeset} -> {:error, {:orders, changeset}}
     error -> error
   end
 end
```

If we look at not preferable version, the `create_order/2` looks beautiful. It's straightforward to understand what are the steps. However, it's misleading, not always `find_item` or `insert_order` happens. If we try to read the private functions is very confusing. Meanwhile, the preferable version isn't pretty, but it makes all the steps and conditionals explicit and clear. It hides some details like how a user is active or how the item is available, but the main logic is clear.

In my opinion, a code is easier to understand when:

* You have a good knowledge of the language
* You read a function their flow and conclusion are clear
* You read a function and their basic expressions are close
* You read a function and details and reusable expressions are abstracted

When you have a good understanding of the language, you ending up learning new tricks that can make some expressions shorter. Here are my suggestions; it doesn't take them as a universal truth. Sometimes you'll find scenarios and cases that the opposite that I suggested is better. What do you think about these suggestions? Do you think it will make you write the maintainable Elixir code?
