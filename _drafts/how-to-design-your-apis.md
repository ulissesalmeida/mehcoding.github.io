---
layout: post
title:  How to Design Your APIs Output
date:   2016-06-18
author: Ulisses Almeida
categories: rest-apis
lang: pt-BR
excerpt: ""
image: /assets/errorsarecoming.jpg
---

In the last past days, I have spent some time thinking how your REST APIs can impact your product development. We need to consider some tradeoffs before deciding the pattern of information we want to expose. I think, for a future proof API where the clients have the independence to evolve, describing and generalizing your data is the best option.

We have seen a lot of API patterns and format discussion: self-discovery API, camelcase attributes, pagination params, objects reference. From these debates came [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS), [some people love it](https://keyholesoftware.com/2016/02/29/dont-hate-the-hateoas/),  [others not too much](https://jeffknupp.com/blog/2014/06/03/why-i-hate-hateoas/). The thing today is the [JSON API](http://jsonapi.org/),  it has a lot of valuable information to inspire us to build consistent APIs.  

I'm writing this post to discuss how we should present the resource attributes. First, see two ways of formatting below:

__Generic:__

```json
{
  "id": 40078,
  "name": "GAME OF THRONES, Vol 1",
  "price_currency": "BRL",
  "price_in_cents": 1244,
  "quantity": 250,
  "weight": 1200,
  "published_at": "2016-05-10T19:20:30-02:00"
}
```

__Formatted:__
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

Compare both approaches. The first one present the raw data and the second show the data formatted to some client needs. Let's discuss advantages and disadvantages of both approaches; starting with the formatted version.

## The Formatted Version

When we choose this approach, we are reducing and centralizing the clients work of formatting to the API server. For the clients, their work is focused on showing the information.  I can list some benefits:

* The code to format the data will be reused on all clients
* Enforce the client a consistent presentation
* A fix on how the data is displayed will be automatically deployed for all clients

When we are talking about mobile applications, we know a simple fix can be take hours or days to be launched on the store. Because some applications stores has a strict process of evaluate the applications versions before launch. Since it's not the in the application developers control, a fix that can be launched anytime shiny the people in the team eyes.

If we go all way and do this approach, we are coupling the client presentation to API. In order to work without too much pain, if it is the projects restrictions:

* All the clients must use the same data format
* The client layout evolution should not impact on the data presentation
* All the presentation rules should start early for API developers

If your clients application does not follow these restrictions and you still insist on this approach you will may slow down your development. Let's see what happen if you break each of that restrictions.

If your clients does not follow the same data format some clients will face a real challenge. For example, is harder to format the Brazilian version of price, `R$ 12,44`, to english version `R$ 12.44`. You need to parse the weight `1.2kg` if you want present in grams. It would simple to handle with the raw data.

If your client layout evolution impact how the data will be shown, be prepared
to separate a lot of time of API development. If we have situation if the new layout should present dates as "month/Year", take a look at our `published_at` attribute. We can't just change it, it will impact with clients that needs the old format. Should we add a new attribute? Create a new version of the resource? Coupling too much with client will require a lot of API changes and versioning.

If your user stories not have a well defined layout early, your APIs will take long to start development. It happens because the APIs just not need what data need to be accessible, but they now need to know how it will be presented. Depending how your team are comfortable to work, it can impact in your features delivery lead time.

You can reduce some work and API inconsistency, allowing some params to API that give some tips how the data should be presented. For example you can create a "language" param, then the API now how format the data according. Although, your clients internatiolization with the API, if the API don't envolve fast and together it can be a bottleneck in your software development process.

## The Raw Version

The API raw version will show the data with the most information you can have. I will no create any assumption it how the data should be used. It uncouple the presentation responsibility from the API and let the clients do all the presentation work. This approach has some benefits:

* Each client has the flexibility of formatting anyway is needed
* Client layout evolutions almost not envolve API changes
* Coeherent archtecture where data formmating happen on client presentation layer
* New clients can be plugged to API with the flexibility to present anyway he want
* No data loss, allowing features with data computation for clients

If we go all way and do this approach, you client applications will need to do some development to formatting and present the data. It can slow down the development, depending of the frameworks and programming languages the clients are using.

But the other side is with this approach you almost have any restriction to worry. You've have build your API focusing on a solid data representation and flexibility. It's like, go slow now to be very fast in the future. The number of attributes adding and versioning you won't need to do, will worth this slow down.

By the way, on mobile applications, a small change to fix some format can take hours or days. Most of the presentaion fixes are low risk or hard to find, them I think it's not real matter to worry. You can mitigate this risk improving your development and validation process. Developming with explicit formmating requirements and accurate validation will prevent most of thess small bugs to happen.

## Reviweing

Formating your API output with presentation rules have some information loss. Review againg the formatted output and let's discuss some them:

* __text values__, the example has the name the raw version was inconsistent, some parts was capitilized and others don't. The formmatted version keep it consistent with the title case. It looks ok, which information are we missing? All the letters are there, is the same text. The information that is missing is the original text the user has acctually typed. It can sounds silly, but it is very important in some contexts, for example, in a edit form. It will looks buggy if the user type a text in one way and see the result different.

* __number values__, we don't have information loss here, but we are enforcing a BRL format. The client will face some extra and annoying parsing to format it differently or do some calucation with the value. Represent number as numbers and give some hints which value unit in different field

* __date and datetime values__, when we provide the formatted version despistes the extra work to parse and format it differently you have the information loss of what is the exactly time. The exactly time helps to inform the correct and give more information when it is needed without launch a new API. Prefer the isoxxx formand or the unixtimestamp.

## Wrapping up

You've have see two different approaches of formatting the data on API outputs. When your API developers and client developers have an awesome sinergy time you may want to experiment to couple the presentation layer with the API and do the formatting on API side.

If you want the client developers working with independency and

Let's mix
