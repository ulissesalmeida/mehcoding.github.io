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

Designing your APIs outputs has huge impact on your client development. You need to consider some tradeoffs before make any decision. Although, for a future proof API, you should describe your output data with all information you can for clients has the flexibility to format any way they can.

We have a lot of API output discussion, some of them are about API with hypermedia links or not. This post is about the resource attributes data format. If you want to know more about the hypermedia and resource attributes convention take e a look on the JSON API spec.

First, see the API output below. Let's call it the "raw data output":

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

Second, see this another way to return API result. Let's call it the "formatted data output":


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

Compare both approaches, try to figure out the information formatting flow on
from the "raw" version to the "formatted". We are going to discuss the advantages and disvantages of each approach.

## The Formatted Version

On the formatted version all the data was prepared for the clients just show the information. We are reducing the client work and trying to centralize how the information should be presented. This approach has some benefits:

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
to separate a lot of time of API development.

You can mitigate this problem, passing some param to API, like "language", for example.
