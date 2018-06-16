---
layout: post
title:  Keep your Ruby On Rails app easy to update
date:   2016-05-12
author: Ulisses Almeida
categories: plataformatec-blog
lang: en
excerpt: "Keeping the application’s Rails version updated is a responsibility that should not be ignored since it brings improvements for your team and security for your users. I hope these tips help you keep your code easy to update."
image: https://upload.wikimedia.org/wikipedia/commons/6/62/Ruby_On_Rails_Logo.svg
---

![rubyonrails](https://upload.wikimedia.org/wikipedia/commons/6/62/Ruby_On_Rails_Logo.svg)

<p class="img-attribution">
  Rails Logo By Jamie Dihiansan http://weblog.rubyonrails.org/2016/1/19/new-rails-identity/2 (http://rubyonrails.org/) [CC0], via Wikimedia Commons
</p>

__This is a repost from [Plataformatec blog](http://blog.plataformatec.com.br/2016/05/keeping-your-ruby-on-rails-app-easy-to-update/).__

[The Rails 5 release candidate is out](http://weblog.rubyonrails.org/2016/5/6/this-week-in-rails-railsconf-recap-rails-5-0-rc-1-is-out/), bringing new improvements that will make your life as a developer easier. Probably you are excited to update your application to the new major Rails release, but you may have some concerns. It is normal, updating your application to fit the new version may bring an unknown number of changes to work. On this blog post, I’ll give you some tips to keep the workload of updating at a minimum.

You can try the new Rails version using the [published guides](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-4-2-to-rails-5-0) and report any problem you may find on [Rails issue tracker](https://github.com/rails/rails/milestones/5.0.0).

## Why update?

I think it is an important question you may need to answer to your team members. My favorite reasons are: security, bug fixes, and features.

Rails has supported versions to receive security fix releases. If you are in a version not supported, you may be vulnerable. You should read the [security policies](http://rubyonrails.org/security) and check if your app is using the supported versions.

The framework also has [maintenance policies](http://rubyonrails.org/maintenance/) that you should be aware of. Performance improvements and bug fixes you may miss if your app is too old. You need to code by yourself and do workarounds to have the same benefits, a code that would be more reliable being inside the framework.

We usually hear developers complaining about applications that use old versions of the framework. The reason is new versions of the tool usually bring improvements to make the developer’s life easier. In the developer’s perspective, it’s demotivating knowing there’s a robust, elegant and productive way to do things and they are not able to use it.

Keeping your Rails up to date will help your code base health and also can be a factor of motivation for your team.

## What I should do?

We have been maintaining and evolving several old Rails apps in different contexts for years, and we have seen some practices that make it easier to keep your app updated:

* strict use of gems
* avoid monkey patches
* keep a good test coverage

### Gems

Adding new dependencies using RubyGems and bundler is awesome, but overusing gems can not only slow your app down but slow you with the amount of work you need to update your Rails version. Some gems are coupled to the framework when you update it. These gems may break. When they break, you need to update them together.

I recommend you considering these points before putting a new gem in your Gemfile:

* __Is this gem adding great value to your project?__ Some gems increase your app security, solve complex problems and reduce a lot of worktimes. Other gems add less value and can easily be replaced with your own implementation. Sometimes, it’s worth doing your own implementation than depending on a third­party code.

* __Is this gem well maintained?__ It’s good checking if the commits are recent, the project is well documented, the changelog contains helpful messages between releases, the maintainers often close issues and answer them respectfully. These are good signs that the gem you are adding won’t give you trouble in the future.

Adding a gem to your project is adding a coupling. Software coupling has its downsides, for example, the ripple effect of changes. The downside of more work to update your dependencies will be worth it if you accurately added them. If you want to learn more about downsides of dependencies, you can read this blog post about Ruby dependencies and this one about Node.

### Avoid monkey patches

Monkey patches are often used to change the behavior of the code you don’t own. For example, rewriting or adding Ruby standard library classes or methods to fit your application needs. Careless monkey patches can bring you serious problems while updating Rails.

Adding code that changes the behavior of gem classes can result in hidden bugs and turn any upgrade into a painful experience. It’s often uncertain how many objects are depending on your change and predicting all the effects of the monkey patched method. For example, in a new version of the gem, your monkey patch can change a method that was updated or doesn’t even exist anymore.

We have seen some monkey patches that are a gem fix or enhancement. These changes may benefit others users of the gem and were hidden in application code. You may get in touch with gem maintainers and propose your changes. If your contribution is accepted, your fix will have a solid and proper place to be.

If your behavior is specific for your app, you should consider extending the gem and applying your wanted behavior using the conventional OO practices. For example, you can create an adapter class, subclass or composition. You may also take a look at Ruby Refinements. Using this structure, you can create scoped changes that must be used explicitly. Explicit code reduces costs to maintain the app and the effort of updating to a new Rails version.

After checking all previous solutions and you still think that a monkey patch better suits your needs, you should know there are organized ways of adding them. Regardless of coding, I recommend you to add a great documentation to keep it sane for your teammates and your future self. In this documentation, you should describe why it is necessary, when it can be removed and how it can be removed. It will help a better understanding of the app and provide useful information while updating your Rails app.

### Keep a good test coverage

We all need to pay special attention to the tests. Some changes while updating Rails will be required and having a trustful test suite will help you discover problems before deploying your app to your end users.

Having 100% of test coverage is [no guarantee of testing the correct behavior](http://martinfowler.com/bliki/TestCoverage.html). I would say the team should have a test coverage that they feel confident about.

Some symptoms that your application doesn’t have good coverage is when recurrent errors are found while the application is running and all your automated tests are passing. You may use a tool like [simplecov](https://github.com/colszowka/simplecov) to check your coverage rate, having a very low coverage is a bad sign.

Adding proper tests is the only solution for low test coverage. If you are in this situation, you should start adding tests focused on the most used and important features. Having a good test coverage is essential to evolve because its main purpose is providing fast feedback that previously working features are still working.

## Conclusion

Keeping the application’s Rails version updated is a responsibility that should not be ignored since it brings improvements for your team and security for your users.

I hope these tips help you keep your code easy to update.

Do you have more tips? Please leave comments below and tell us about your experience to keep your application up to date.
