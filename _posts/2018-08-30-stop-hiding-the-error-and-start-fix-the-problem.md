---
layout: post
title:  Stop hiding the error and start fixing the problem
date:   2018-08-30
author: Ulisses Almeida
categories: plataformatec-blog
lang: en
excerpt: "I’ve been working on Plataformatec for 5 years and one common mistake that I see developers making is 
hiding the error, instead of fixing the problem. This kind of behaviour can turn your product full of problems 
quickly by having a codebase with unnecessary defensive programming. Let’s explore that by taking a look at an example 
from the real world."
image: /assets/targets.jpg
---

![longprojects](/assets/targets.jpg)

__This is a repost from [Plataformatec blog](http://blog.plataformatec.com.br/2018/07/stop-hiding-the-error-and-start-fixing-the-problem/).__

I’ve been working on Plataformatec for 5 years and one common mistake that I see developers making is hiding the error, 
instead of fixing the problem. This kind of behaviour can turn your product full of problems quickly by having a codebase 
with unnecessary defensive programming. Let’s explore that by taking a look at an example from the real world.

In some period of time, we’re working on an address autocomplete feature using Google places. Suddenly, we 
receive an error report complaining about an undefined object in Javascript. Something like this:

```
Uncaught TypeError: Cannot read property 'maps' of undefined
````

A developer looked at the stacktrace, then, found the problematic line. The line was something like this:

```
const autocomplete = new google.maps.places.Autocomplete(input, options);

After a moment, the developer opened a Pull Request like this:

// Sometimes google is undefined
if (google) {
const autocomplete = new google.maps.places.Autocomplete(input, options);
}
```

Can you say if it’s a good solution? I can say it’s an answer. Sometimes this kind of code can be a solution. 
However, when I see a solution like this, too easy, it sets my spidey sense off. Let’s see why it’s not good.

That code initializes the `Autocomplete` widget. It means, when google is undefined, the address suggestion is 
not working. The if might prevent an error report, but it won’t fix the problem. This version now is worse than 
the previous one. Why? Now, users still can’t have the address suggestion and the team will not be aware of 
the problem. The code is hiding the error, not fixing the users’ problem.

You might think it was a junior mistake, but I can tell you, I could see many developers with many years of 
experience doing that. Everybody can make a mistake, that’s why we have a process of reviewing other people’s 
code to prevent errors like this.

A bad context can make a good professional produce poor results. If the leader presses the team to make the errors 
reports go down fast, mistakes like that become more common. People start finding the easiest way of stopping that 
error of being reported.

<blockquote class="twitter-tweet" data-lang="pt"><p lang="en" dir="ltr">“People with targets […] will probably meet the targets - even if they have to destroy the enterprise to do it.” – Deming <a href="https://t.co/EJZLIZ50zT">pic.twitter.com/EJZLIZ50zT</a></p>&mdash; Lindsay Holmwood 🖤🧡❤️ (@auxesis) <a href="https://twitter.com/auxesis/status/828205291728625664?ref_src=twsrc%5Etfw">5 de fevereiro de 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The target should not be reducing the errors reported. The goal should be fixing the users’ problems. For every bug reported 
by a bug track tool, ask yourself: Which problem is it generating for users? Then, make that problem your target to fix, 
not the error reported. This mindset that generates better solutions, a better product, a better codebase.

If you are curious to know a good solution for the example problem, I’ll explain to you. That `Autocomplete` class is provided 
by a third party script included in the HTML document. The problem was a race condition. Our code was executing faster than 
the script being loaded and available. A good fix for that would be only executing that code after the Google’s script 
initialization. Thankfully, you can pass a function to be executed as an argument to Google’s script inclusion. Example:

```
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places&callback=initAutocomplete" async defer></script>
```

We passed the functioninitAutocomplete to initialize our address suggestion widget. It’s a good answer that fixes the user’s 
problem. It does not hide the error.

Every time you see a solution for an error, ask yourself: is that really fixing the problem? Try to figure out how that 
error impacts the end users. Remember, is better having errors reported than invisible problems. Avoid defensive code that 
hides errors instead of fixing them.
