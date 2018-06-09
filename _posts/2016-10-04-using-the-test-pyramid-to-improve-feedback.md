---
layout: post
title:  "Using the test pyramid to have better feedback from your test suite"
date:   2016-10-04
author: Ulisses Almeida
categories: plataformatec-blog
lang: en
excerpt: "In this blog post, I’ll explain the problem of developers spending too much time trying to discover a problem in a failing test suite. A cause of this issue may be the lack of a good test feedback. A good feedback is one that happens fast with a helpful message."
image: /assets/test-pyramid.png
---

__This is a repost from [Plataformatec blog](http://blog.plataformatec.com.br/2016/10/using-the-test-pyramid-to-have-better-feedback-from-your-test-suite/).__

In this blog post, I’ll explain the problem of developers spending too much time trying to discover a problem in a failing test suite. A cause of this issue may be the lack of a good test feedback. A good feedback is one that happens fast with a helpful message.

To illustrate this problem, consider a simple application of storing and listing contacts. You have an actor user that can sign up, sign in, then can CRUD their contacts with first name, last name and phone number. We are in a context of multiple developers working together, programming and delivering features every day. For the examples below I’ll use Ruby, RSpec, Capybara and Ruby on Rails.

Suppose we have a UI testing using RSpec features test like this:

```ruby
#spec/features/contacts_spec.rb
feature 'User can create new contact' do
  scenario 'when user provides valid contact information' do
    login_with email: 'default@mail.com', password: 'default'

    click_link 'Contacts'

    expect(page).to have_content('Listing Contacts')
    expect(page).to have_content("There're no contacts.")

    click_link 'New Contact'

    fill_in 'First name', with: 'Jon'
    fill_in 'Last name', with: 'Doe'
    fill_in 'Phone number', with: '1199999999'

    click_on 'Create Contact'

    expect(page).to have_content('Listing Contacts')
    expect(page).to have_content('Jon Doe - (11) 9999-9999')
  end
end
```

Simple and straight forward. The good part of this test is it checks the entire application stack: authentication, rendering, storing stuff on the database and the logic of creating contacts. Some developer may be confident with this test and start to question: Should I add more unit tests? For phone number formatting? For displaying complete names? For user authentication? For contact validation and creation? If one of these features fail, this feature test will fail too. Why should I care to add more tests? Isn’t it redundancy? Waste of time? The short answer is you should add more unit and integration tests because of feedback. Now, let’s expand the topic.

## Speed matters

We know the UI test brings confidence since it tests the entire stack of your application. But overusing this type of tests will severely impact your test suite speed.

In our contacts creation test example, we have some edge cases that were not covered. What happens if we use a different size of numbers in the telephone number field? How will it be displayed? We can add a new UI test to cover this case, but a unit test suits better and faster. If the number formatting is a view helper we can check it with a helper test, for example:

```ruby
describe '#format_phone_number' do
  it 'formats cell phones numbers of 10 digits in "(XX) XXXXX-XXXX" format' do
    formatted_number = helper.format_phone_number('11999998888')
    expect(formatted_number).to eq '(11) 99999-8888'
  end

  it 'does not format others digits different of 10' do
    formatted_number = helper.format_phone_number('11888')
    expect(formatted_number).to eq '11888'
  end
end
```

This test is way easier to write, read, cheaper and faster than adding a new UI to test this specific case. Keeping this practice, you will have fewer UI tests and more integration and unit tests. Such distribution of the different kind of tests resembles the format of a pyramid:

![test-pyramid](/assets/test-pyramid.png)

Few UI tests on top and a solid base of unit tests. If you want more information about the test pyramid, I recommend you the following links:

* [The Forgotten Layer of the Test Automation Pyramid](https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid) by [Mike Cohn](https://twitter.com/mikewcohn)
* [The Test Pyramid](https://www.kenneth-truyers.net/2015/06/27/the-test-pyramid/) by [Kenneth Truyers](https://twitter.com/kennethtruyers)
* [Test Pyramid](http://martinfowler.com/bliki/TestPyramid.html) by [Martin Fowler](https://twitter.com/martinfowler)
* We have a [talk](https://speakerdeck.com/plataformatec/piramide-de-testes-escrevendo-testes-com-qualidade-at-rubyconf-2015) and a [video](https://www.youtube.com/watch?v=xgnF1hxwpBM) explaining the test pyramid in portuguese.

## Helpful messages

In the daily routine of a development team, the code is constantly evolving. The test server is running and warning when the test suite is passing or broken. A passing test suite is a requirement to deploy a new change in production in most of the clients that we have been working on. Then when the test suite fails, it is important for developers to act and fix the broken tests fast. Without helpful messages, it is critically hard.

Let’s use again our contacts UI test, let’s use a scenario where we just have that test. The application has received new changes and the test server warns the team that we have an error, and the output is:

```
  1) User can create new contact when user provides valid contact information
     Failure/Error: expect(page).to have_content("Jon Doe - (11) 9999-9999")
       expected to find text "Jon Doe - (11) 9999-9999" in "default@mail.com
       Log out Contacts Contact was successfully created. Listing Contacts
       Jon Doe - 1199999"
```

We know the test failed in the last step, the step of checking the recent contact content in the listing page. In this sample application, we have a small text to search what’s going on. In a real world application, this content can be a lot bigger and unproductive to read. From here the developer has a lot of possibilities to search what the root cause of the problem. For example:

* Is it in the creation?
* Is it in the contacts listing?
* Is it in the telephone format method?
* Is it in the test?
* Had been changes to asynchronous request and expectation didn’t have time to assert the new content?

The list can go on depending on your creativity. Now let’s build a scenario where we also have the format_phone_number helper unit test. The tests are failing, but now have this output:

```
1) User can create new contact when user provides valid contact information
     Failure/Error: expect(page).to have_content("Jon Doe - (11) 9999-9999")
       expected to find text "Jon Doe - (11) 9999-9999" in "default@mail.com
       Log out Contacts Contact was successfully created. Contact was successfully
       created. Listing Contacts Jon Doe - 1199999999 Show Edit Destroy New Contact"

2) ContactsHelper#format_phone_number formats cell phones numbers of 10 digits in "(XX) XXXXX-XXXX" format
   Failure/Error: expect(helper.format_phone_number('11999998888')).to eq '(11) 99999-8888'

     expected: "(11) 99999-8888"
          got: "11999998888"

     (compared using ==)
```

Now we have two failing tests. The second error message is way easier to guess why it is broken. Some change on the `format_phone_number` helper has broken the cell phone formatting. Knowing it, looking again at the first error message you may connect the dots and find out the errors are connected.

If two tests are failing for the same reason, this is not the same thing of worthless redundancy. Focused behavior tests bring you a lot more information for debugging, helping you solve problems fast. This is another benefit of the test pyramid.

## Conclusion

Some people tend to be relaxed about worry too much about creating focused tests. I think the main reason is they are very used to code base context. If something fails they can navigate, debug and figure out fast what happened. But unfortunately, it not always true for everyone, specially for the newcomers.

Another problem is when the team becomes careless about covering exceptional cases with unit and integrations tests. The test suite is always green but problems are increasing in production. Your test suites become less reliable and leaving the entire team insecure about deploying new changes.

To keep your test suite fast and worthy of trust I recommend you use the test pyramid technique. In Rails applications will be something like:

* Fewer UI tests with Capybara. Use them to test the user main journey and some exceptional cases.
* Test beyond the expected behavior. Exercise the exceptional behaviors with integration and unit tests. Write more models, controllers, requests, views, helpers, or mailers tests.
* Consider adding JavaScript unit tests in your test suite.

These practices help us a lot to maintain a healthy code base and allow us to adapt and be more agile to change. What do you think of this practices? Which techniques and practices are you doing to keep your test suite fast and with helpful messages? Let me know with a comment below.
