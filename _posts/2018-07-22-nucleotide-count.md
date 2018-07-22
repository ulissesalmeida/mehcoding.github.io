---
layout: post
title:  Using `Enum.reduce/3` to improve Nucleotide Count performance.
date:   2018-07-22
author: Ulisses Almeida
categories: Elixir
lang: en
excerpt: "I was playing with Exercism, an awesome platform to practice and learn to code with exercises. While I was practicing, I found an interesting exercise called Nucleotide Count. First, I tried an obvious answer, but It leads me to a suboptimal solution in terms of speed. Let's see the error that I did and how we can improve the performance by using `Enum.reduce/3`."
image: /assets/nucleotide.png
---

![nucleotide](/assets/nucleotide.png)

I was playing with [Exercism](https://exercism.io/), an awesome platform to practice and learn to code with exercises. While I was practicing, I found an interesting exercise called [Nucleotide Count](https://github.com/exercism/elixir/tree/master/exercises/nucleotide-count). First, I tried an obvious answer, but It leads me to a suboptimal solution in terms of speed. Let's see the error that I did and how we can improve the performance by using `Enum.reduce/3`.

The __Nucleotide Count__ ask you to count the occurrences of some characters in a list characters. Theese characters are called nucleotides. For example:

```elixir
NucleotideCount.count('GTACG', ?T) == 1
# => 5
```

We count __one__ occurrence of nucleotide `T` in the character list `'GTACG'`. We use the `?` before of a letter to represent a character in Elixir. We can implement `NucleotideCount.count/2` function using Elixir's `Enum.count/2`. See:

```elixir
def count(strand, nucleotide) do
  Enum.count(strand, &(&1 == nucleotide))
end
```

I used an anonymous function to check if the current item of the iteration is equal to the given character.  `Enum.count/2` will increment one on the counter when that anonymous function returns a truthy value.

The next step of the exercise is build a histogram of a given sequence of characters. For example:

```elixir
 NucleotideCount.histogram('AGAGA')
# => %{?A => 3, ?T => 0, ?C => 0, ?G => 2}
```

This time, the function builds a map with 3 occurrences of the nucleotide `A`, 2 occurrences of `G`, and 0 for the remaining nucleotides. The obvious solution for me was to build a map using the `NucleotideCount.count/2` function. Something like:

```elixir
def histogram(strand) do
  %{
    ?A => count(strand, ?A),
    ?T => count(strand, ?T),
    ?C => count(strand, ?C),
    ?G => count(strand, ?G)
  }
end
```

The function builds a map counting each expected character. It works as expected and fulfills the specs. However, thinking a little bit about the performance, it is a suboptimal solution. Every time we use `NucleotideCount.count/2`, it iterates over the entire given list. It means the function will transverse the `strand` list __4__ times. Not cool. Shame on me. How can we avoid that? Let's build a solution that will read the list once.

The first thing we need is an initial histogram. See below:

```elixir
defp initial_histogram() do
  %{?A => 0, ?T => 0, ?C => 0, ?G => 0}
end
```

The second step is to iterate over the `strand` by updating the initial histogram with occurrences of nucleotides. Every time you need to iterate over an enumerable and manage a state, the `Enum.reduce/3` is a good candidate for the job. Let me show you how we can use it:

```elixir
def histogram(strand) do
  histogram = initial_histogram()
  Enum.reduce(strand, histogram, &update_histogram/2)
end
```

The `Enum.reduce/3` will iterate over all characters in `strand` by executing the `NucleotideCount.update_histogram/2`. The result of that function will be the new histogram that will be used as an argument for the next iterations. The last iteration will return the final histogram that also will be returned by the `Enum.reduce/3`.

The last piece of the puzzle is the `NucleotideCount.update_histogram/2`, let's see how we can implement that:

```elixir
def update_histogram(nucleotide, histogram) do
  Map.update!(histogram, nucleotide, &(&1 + 1))
end
```

The function receives a nucleotide from the `strand` list and the histogram. We use `Map.update!/3` to increment the counter for that nucleotide occurrence by 1 with the anonymous function `&(&1 + 1)`. Doing this way, we iterate over the entire `strand` list __once__. We're not adding any burden to our algorithm because the new operation `Map.update!/3` has a constant time.

Sometimes the obvious solution is not the best, it always good to check the algorithm complexity before move forward. I hope you liked my implementation of the `Nucleotide Count` exercise. If you like this exercise and want to try more, please visit the [Exercism](https://exercism.io/). If you are an expert programmer and want to help others to improve their self, you can participate in Exercism as a [mentor](http://mentoring.exercism.io/). If you have any question or other solution suggestion for exercise, let me know in the comments below.
