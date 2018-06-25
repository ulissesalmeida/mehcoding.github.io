---
layout: post
title:  Inner Layouts With Phoenix
date:   2018-05-16
author: Ulisses Almeida
categories: plataformatec-blog
lang: en
excerpt: "Over the last few weeks, we have been building a web application in one of our clients and ended up duplicating some template code. These new pages had something in common between them, but not with the rest of the application. We needed an inner layout to reuse the template code between these pages, however, Phoenix doesn’t come with this feature. In this post, you’ll learn how you can build nested layouts in Phoenix and when you should use them."
image: /assets/inner-layout-diagram.png
---

![inner layout diagram](/assets/inner-layout-diagram.png)

__This is a repost from [Plataformatec blog](http://blog.plataformatec.com.br/2018/05/nested-layouts-with-phoenix/).__

Over the last few weeks, we have been building a web application in one of our clients and ended up duplicating some template code. These new pages had something in common between them, but not with the rest of the application. We needed an inner layout to reuse the template code between these pages, however, Phoenix doesn’t come with this feature. In this post, you’ll learn how you can build nested layouts in Phoenix and when you should use them.

## Why are they useful?

Inner (or nested) layouts are useful to reuse template code when you have subsections on your website. Usually, in Phoenix applications, we have `/templates/layout/app.html.eex` that shares template code between all pages. For example:

![layout](/assets/no-inner-layout.png)

In the example above, we want to reuse the application logo, title, and navigation menu across all pages. We can solve that by using the Phoenix default layout and it works great. However, sometimes you need to create a new website section inside the parent layout. For example, imagine we want to add a help section to our Phoenix website. Look at the image below:

![inner layout](/assets/inner-layout.png)

We still want to reuse the layout header across all pages, but we also want to reuse the left navigation and main content layout in all pages inside the help section. Let’s see how we can do that using Phoenix.

## The Inner Layout Solution

I tried some solutions by looking at some examples online. After discussing with the Plataformatec team, we came up with an approach that’s very simple and extensible, thanks to Phoenix explicit layout rendering. The solution works like this:

![inner layout diagram](/assets/inner-layout-diagram.png)

* You create the nested layout template.
* In the nested layout, you set the parent template by calling a function.
* The parent layout is aware that is possible to have an inner layout and will render it.
* You can invoke your nested layout by using the Phoenix `put_layout/2` function.

The main goal here is to make the nested layout work like any Phoenix layout. This will make it familiar to any Phoenix developer.

## Preparing The Parent Template

First, let’s add a function that allows a template to render the parent layout:

```elixir
# views/layout_view.ex
defmodule YourAppWeb.LayoutView do
  use YourAppWeb, :view

  def render_layout(layout, assigns, do: content) do
    render(layout, Map.put(assigns, :inner_layout, content))
  end
end
```

The `render_layout/3` will render the given layout by assigning the contents of the inner layout given in the do argument. Now, in the parent layout, we need to render the inner layout contents or the controller view contents. Look at how you can do it:

```eex
<%# templates/layout/app.html.eex %>

<%# ... %>

  <%= Map.get(assigns, :inner_layout) || render @view_module, @view_template, assigns %>

<%# ... %>
```

With the code above, our parent layout is aware that there may be an inner layout and it should render its contents when available. That’s it! Now let’s see how we can use it.

## Using the Nested Layout

You can create a nested layout template in the `layouts` folder and use it like this:

```eex
<%# templates/layout/nested_layout.html.eex %>

<%= render_layout "app.html", assigns do %>
  <%# Your HTML markup here %>
  <%= render @view_module, @view_template, assigns %>
  <%# More HTML markup here %>
<% end %>
```

We render the parent and put the inner content in the do/end blocks in our nested layout template. Any content outside of the do/end blocks will not be rendered. Don’t forget to call `render @view_module, @view_template, assigns`, or the contents of your controller’s action template will not be rendered.

Now you can use our nested layout by invoking the plug `put_layout/2` in your controller or router. For example:

```elixir
defmodule YourAppWeb.NestedContentController do
  use YourAppWeb, :controller

  plug :put_layout, :nested_layout

  def index(conn, params) do
    # stuff
  end
end
```

You can organize your nested layout files in a different way. For example, imagine you have a help section in your website and you want to keep the nested layout file in the help folder. In your `HelpView` you’ll need to import the `render_layout/3` function, like this:

```elixir
# views/help_view.ex
defmodule YourAppWeb.HelpView do
  use YourAppWeb, :view
  import YourAppWeb.LayoutView, only: [render_layout: 3]
end
```

After that, you can put your nested layout template in `templates/help/layout.html.eex`. In the controller or router, you can invoke the nested layout like this:

```elixir
plug :put_layout, {YourAppWeb.HelpView, "layout.html"}
```

The code above will render your `layout.html.eex` template in `views/help` directory using `YourAppWeb.HelpView` module.

## Wrapping Up

Inner layouts come in handy to reuse template code of subsections of your web application. You learned how simple it is to build that in Phoenix. Just be aware of not creating inner layouts of inner layouts. If you do that, your codebase will start to be very hard to maintain. You can see and try by yourself a sample [Phoenix app](https://github.com/ulissesalmeida/nested_layout) running the inner layout example.

Have you implemented inner layout in a different way? Do you have any feature that you would like to see how we can build it in Phoenix? Let us know in your comments.
