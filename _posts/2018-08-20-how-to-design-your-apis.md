---
layout: post
title:  Designing Your HTTP JSON APIs Output
date:   2018-08-20
author: Ulisses Almeida
categories: development
lang: en
excerpt: "A long time ago, I participated in a discussion about HTTP JSON API design with mobile developers that will use an API provided from us(the backend team). One side was defending reusability, the other hand was defending independence. I'll present you some notes from that discussion. Tradeoffs that we need to consider before picking an option. Today, I'm still holding my point of client independence for a future-proof API. I support that describing and generalizing your data is the best option for an API, this way, the clients have more opportunities to evolve. But, before digging on that comparison, let's recap a little bit what we have today."
image: /assets/apis.jpg
---

![apis](/assets/apis.jpg)

A long time ago, I participated in a discussion about HTTP JSON API design with mobile developers that will use an API provided from us(the backend team). One side was defending reusability, the other hand was defending independence. I'll present you some notes from that discussion. Tradeoffs that we need to consider before picking an option. Today, I'm still holding my point of client independence for a future-proof API. I support that describing and generalizing your data is the best option for an API, this way, the clients have more opportunities to evolve. But, before digging on that comparison, let's recap a little bit what we have today.

## How that discussion started

Recent years, we had a lot of API patterns and format discussions: self-discovery API, camelcase attributes, pagination params, objects reference. From these debates came [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS), [some people love it](https://keyholesoftware.com/2016/02/29/dont-hate-the-hateoas/),  [other people not too much](https://jeffknupp.com/blog/2014/06/03/why-i-hate-hateoas/). A valuable API design resource [JSON API](http://jsonapi.org/) came out, it has a lot of valuable information to inspire us to build consistent APIs. Finally, we have [GraphQL](https://graphql.org/learn/), another API pattern, that simplifies some stuff and complicates other.

However, our discussion wasn't about GraphQL vs. REST. It was something more specific, the resource attributes. No matter which pattern you're using, you need to decide if your present your money data in a formatted string like `"R$ 50,00"` or a money object with `currency` and `amount` attributes. In that time, it was an obvious answer for me, we should always pick the money object instead of a formatted string.

Oh, problems in the paradise. In that time some developers had a strong position to use a formatted string from the backend service, and they came out with good arguments. In the first conversation, I had to retreat. I had to think more about why I was against that. Then, we had another time talking, this time I was prepared with better arguments than "It's what I have been doing in all my life and it works." Let's see their position, the formatted data from backend arguments.

## Serving formatted data from API

The primary purpose of serving formatted data from the backend is centralizing the formatting work on the backend. This way, you achieve more reusability to other clients and can simplify the maintenance since one fix, fix them all. Let's illustrate in an example:

```json
{
  "id": 40078,
  "name": "Game of Thrones, Vol 1",
  "price": "R$ 12,44",
  "quantity": "250 unidades",
  "weight": "1.2kg",
  "published_at": "10/05/2016"
}
```

Using this approach, you can reduce the clients work of formatting, taking all the job to the API server. The client work is focused on displaying the information.

### Benefits

* The code to format the data will be reused in all clients
* Enforce the client a consistent presentation
* One fix, fix them all

In mobile applications context, a simple fix can take hours or days to be launched on the store. It happens because some stores have a strict process of evaluating the applications versions before launch. It is understandable why this approach can be appealing to mobile developers. But, if we go all way and make this approach, we are coupling the client presentation to API.

### Disadvantages

* All the clients must use the same data format
* The presentation evolution impact API and mobile developers
* The presentation definition needs to start early

If your application clients do not follow the same data format, you're bringing pain to some clients developers. You may be adding an extra parsing code to some clients, for example, to format in English the `R$ 12.44` the Portuguese version of price `R$ 12,44` requires an extra parsing. Some clients might need to parse the weight `1.2kg` if they want to present in grams. It would simple to handle with better and generic data representation.

It's rare for a presentation layer of an application be stuck on time. The presentation will need to change, and it will be hard to communicate for every change in mobile apps needs to change the backend. Communication is essential in software development, we separate software in layers not just to have a better organization because it also simplifies when talking about the system. If the product people want to change how a date is displayed on the Android app, they will naturally ask the Android team.

Changes need more coordination using this approach. If the team wish the date period to be as "month/Year" on Android, the Android team neither API team can just change it. Retake a look at `published_at` attribute, if we change that it will impact the clients that need the old format. What should we do? Should we add a new field? Create a new version of the resource? An accidental change can be a disaster. Of course, we should not drive our decisions based only on fear of future requirements, but, making a decision that a simple change will bring a lot of pain, work, and versioning, is better think twice.

If your user stories do not have a defined layout early, your APIs will take long to start the development. It happens because the APIs developers needs to know how the data will be presented in details. Depending on how your team is comfortable to work, it can impact your features delivery lead time.

You can reduce some work and API inconsistency, allowing some params to API that give some tips on how the data should be presented. For example, you can have a "language" header, then the API now how to format the data according. Although, if you rely internationalization on API if the API doesn't envolve fast enough, can be a bottleneck in your software development process.

## Serving generic data from API

The alternative approach of serving formatted data is presenting the data in a generic/raw/programmable form. The good part of this approach is the independence of the clients, each client can work with data in different ways without extra magic parsing. For example:

```json
{
  "id": 40078,
  "name": "GAME OF THRONES, Vol 1",
  "price": {
    "currency": "BRL",
    "amount": 1244,
  },
  "quantity": 250,
  "weight": 1200,
  "published_at": "2016-05-10T19:20:30-02:00"
}
```

In this approach, the API will not have any assumption on how the data should be used. It uncouples the presentation responsibility from the API and let the clients do all the presentation work. This approach has some benefits and also its price. Let's see.

## Benefits

* Each client has the flexibility of formatting anyway is needed
* Design or layout changes not require API changes
* Coherent architecture where data formatting happen on client presentation layer
* No information loss with formatting

If we go all way and make this approach, your client applications will have additional development to formatting and present the data. This extra work will be duplicated among all these clients. All this extra and similar work can slow down the client side development.

The good part of this approach is you have built your API focusing on a robust data representation allowing client flexibility. It's like, go slow now to be very fast in the future. The number of attributes adding and versioning you won't need to do might worth this slowdown.

We can't forget the mobile applications. A small change to fix some format can take hours or days to deliver depending on the store. In my experience, most of the presentation fixes are at low risk for users but can be harmful to the company's brand. You can mitigate this risk by improving your development and validation process.

## Which one should I pick?

After this lengthy discussion, we can see that both approaches have benefits and downsides. I would say, you should take a hybrid approach. Some date needs to be consistent across all clients and platforms, some data needs to evolve differently. After accepting this fact, my approach to the mobile developers was different, the discussion is now which data must be consistent, which kind of computation is hard for them that would be awesome to be on the server to be reused. Here's some data types and ideas to discuss with your team:

* __Text values__, some text might have some letters capitalized, and others don't. The formatted approach can keep it consistent with some casing, like title case. It looks ok. However, we're losing information despite being the same text. The information that is missing is the original text the user has typed. It can sound silly, but it is essential in some contexts, for example, in an edit form. Some users will think that is a buggy if they type text in one way and see the result differently.

* __Number values__, if we present the number in a formatted string we can complicate some client jobs. The client will face some extra and annoying parsing to extract that number and format it differently or do some calculation. Represent number as numbers is often a better approach.

* __Date and DateTime values__, when we provide the user's formatted version we lose the time accuracy. Prefer the ISO 8601 format or the Unix timestamp.

I'm still a defender of providing a programmable friendly format on APIs output. However, now, I can see the value in putting some complicated formatting logic on API. You can, for example, provide the formatted attribute and their raw version in the same API. It might sound data redundancy in some cases, but can be useful to simplify the client application requirements.

How do you like to design your APIs? Which approach does work better for you?
