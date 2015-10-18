---
layout: post
title: Handling Errors in Elixir, Who said Monad
description: Why the Internet everywhere is not the same as the Internet anywhere
date: 2015-10-15 14:56:05
tags:
- Elixir
- Functional Programming
- Monad
author: Peter Saxton
---

### Being Contrary

This post is to extend a throwaway comment I made on [Twitter]() about returning tagged tuples as the result of any expression in Elixir.

**Tempted so say all elixir funcs should return result tuple. `{:ok, 4} = 2 + 2`**

Obviously several people disagreed and a tweet was never going to be enough to explain my reasoning. So here goes... (or something like that??)

### Some background

- *Tuple:* A data structure consisting of multiple parts.
- *Tagged Tuple:* A tuple where the first part is an atom that acts as a label describing the remaining contents in the tuple.

Elixir has inherited the tagged tuple error handling convention from Erlang. Let's take a function that has a chance of failing to computer. If all goes well, the function will return a tuple tagged with the `:ok` atom and a value. If the function could not return a value then it returns a tuple tagged `:error` and the reason. In your code you can then use pattern matching on the tag to call the behaviour that you want.

{% highlight elixir %}
```elixir
case MyModule.flaky_method do
  {:ok, value} -> IO.puts "All good value was: #{value}."
  {:error, reason} -> IO.puts "Uh oh! Failed due to #{reason}."
end
```
{% endhighlight %}

This is only a convention and so can be ignored but the situation is a bit more complicated in the real world:

- Some functions simply return `nil` or `:error` instead of a tuple and a reason.
- Some functions can't fail so just return a value. As in my original tweet, adding 2 numbers always works and so `2 + 2` will only return `4`.
- Some functions borrow from Ruby conventions and have methods that can throw errors which may or may not end in an exclamation mark.

### A problem to solve

So before presenting you with a solution it's good practise dealing with a problem:

Imagine I have a JSON file of information about people in my company. I want to fetch all the data relating to me in this file.

To do this I first read the file, then parse the JSON, and finally search for the details relating to me. Ideally our code will resemble an intuitive description of the problem; we described this problem as a sequence of steps and so our code should read like a sequence of steps.
Given that it is possible for each of our steps to fail, we need to prepare to handle those errors. The error handling can pollute our nice list of instructions and result in code littered with conditionals.

{% highlight elixir %}
```elixir
result = case File.read("my_company/employees.json") do
  {:ok, contents} ->
    case Poison.decode(contents) do
      {:ok, json} ->
        case Dict.fetch(json, "Peter") do
          {:ok, data} -> {:ok, data}
          :error -> {:error, :key_not_found}
          # Notice the Dict API doesn't return error with reason.
          # So we add a reason to make it conform.
        end
      {:error, reason} -> {:error, reason}
    end
  {:error, reason} -> {:error, reason}
end
```
{% endhighlight %}

Well that's ugly. Can we do this better?

Thinking about it a bit harder we can see a common problem in the code:
If the result of a step is successful, we want to take the value of the result and use it in the next step. But if the result was an error then we want to stop processing and return.

### How about a Result module?

Let's create a module that has the responsibilty of handling this conventional error pattern. The `Result` module handles passing values from `:ok` tagged tuples to a given function. But if the result is an `:error` tagged tuple then the function will simply be ignored.

The method of interest is `bind`, so-called bind to make reading the code reading easier. It has the following behaviour:

- If the result given is a success, call the next function with the returned value.
- If the result is a failure, don't invoke the function.

{% highlight elixir %}
```elixir
defmodule Result do
  def bind({:ok, value}, func) when is_function(func, 1), do: func.(value)
  def bind(failure = {:error, _}, _func), do: failure
end
```
{% endhighlight %}

With the `bind` method we can then return to a linear set of steps.

{% highlight elixir %}
```elixir
import Result

{:ok, "my_company/employees.json"}
|> bind(&File.read/1)
|> bind(&Poison.decode/1)
|> bind(fn
  (%{"Peter": data}) -> {:ok, data}
  (_) -> {:error, :key_not_found}
  # This noise here is only necessary as the Dict returns :error without a reason
end)
```
{% endhighlight %}

Not bad! We have a single list of steps, but having the `bind` method everywhere and the function capture still makes this a bit messy.

Can we go better again?

### There is a Monad in the house

This combination of tagged tuples can be viewed as a result monad. The result monad is a monad because it has features in common with other monads.

Monads are a very abstract concept and as we are not trying to generalise any further we do not need to investigate them any further. Using the result tuple as a monad allows us to use the experience of people who have thought more about monads.

For this reason I am working on an extension to the result module that will handle tagged tuples.
I want it to be able to handle `{:ok, value} | {:error, reason}` only so that it provides guidance on the API for functions.

Once I have finished reading [Metaprogramming Elixir]()(great book) then our example problem can be refactored too.

Still, at in it's current form, this code is simpler than what we started with.

*I have assumed the API for dict is updated*

{% highlight elixir %}
```elixir
use Result

def get_employee_data(file, name) do
  {:ok, file}
  |r> File.read
  |r> Poison.decode
  |r> Dict.fetch(name)
end

def handle_user_data({:ok, data}), do: IO.puts("Contact at #{data["email"]}")
def handle_user_data({:error, :enoent}), do: IO.puts("File not found")
def handle_user_data({:error, {:invalid, _}}), do: IO.puts("Invalid JSON")
def handle_user_data({:error, :key_not_found}), do: IO.puts("Could not find employee")

get_employee_data("my_company/employees.json")
|> handle_user_data
```
{% endhighlight %}

### Conclusion

At the end of the last example we used a regular pipe to pass the tuple to the function instead of to the extracted value.
In practice this often happens as the last step.
A good thing as confident code should end by handling the errors it can resolve.
As a convention my functions that expect a tuple I will name handle_something.
I also explicitly match error in the handle function, this is so an error I do not have a handling strategy for will crash the process.

Does considering monads in a pipeline help or hinder your thinking?
Would you use a bind pipeline for a result tuple?
Should I finish my `Result` macro?

### Resources
- tom stuart
- monads in pictures
