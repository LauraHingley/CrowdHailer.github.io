---
layout: post
title: Handling Errors in Elixir, No one say Monad.
description: Advice for handling errors in Elixir pipelines.
date: 2015-10-18 17:00:05
tags:
- Elixir
- Functional Programming
- Monad
author: Peter Saxton
---

### Being Contrary

This post is to extend a throwaway comment I made on [Twitter](https://twitter.com/CrowdHailer/status/654202422416556032) about returning tagged tuples as the result of any expression in Elixir.

**Tempted to say all elixir funcs should return a result tuple. `{:ok, 4} = 2 + 2`**

Obviously several people disagreed and a tweet was never going to be enough to explain my reasoning.
So here goes...

### Some background

- *Tuple:* A data structure consisting of multiple parts.
- *Tagged Tuple:* A tuple where the first part is an atom that acts as a label describing the remaining contents in the tuple.

Elixir has inherited the tagged tuple error handling convention from Erlang.
Let's take a function that has a chance of failing to compute.
If all goes well, the function will return a tuple tagged with the `:ok` atom and a value.
If the function could not return a value then it returns a tuple tagged `:error` and the reason.
Subsequent code can then pattern match on the tag to call the correct behaviour.

{% highlight elixir %}
case MyModule.flaky_method do
  {:ok, value} -> IO.puts "All good value was: #{value}."
  {:error, reason} -> IO.puts "Uh oh! Failed due to #{reason}."
end
{% endhighlight %}

Returning a tagged tuple is only a convention.
In the real world many functions do not adhere to this contract:

- Some functions simply return `nil` or `:error` instead of a tuple and a reason.
- Some functions can't fail so just return a value. As in my original tweet, adding 2 numbers always works and so `2 + 2` will only return `4`.
- Some functions borrow a Ruby convention and have functions that can throw errors and end with an exclamation mark.

### A problem to solve

So before presenting you with a solution it's good to have a problem:

Imagine I have a JSON file of information about people in my company.
I want to fetch all the data relating to me in this file.

To do this I first read the file, then parse the JSON, and finally search for the details relating to me.
Ideally our code will resemble an intuitive description of the problem; we described this problem as a sequence of steps and so our code should read as a sequence of steps.
Given that it is possible for each of our steps to fail, we need to prepare to handle those errors. The error handling can pollute our nice list of instructions and result in code littered with conditionals.

{% highlight elixir %}
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
{% endhighlight %}

Well that's ugly. Can we do this better?

Thinking about it a bit harder we can see a common pattern in the code:
If the result of a step is successful, we want to take the value of the result and use it in the next step.
But if the result was an error then we want to stop processing.

### How about a Result module?

Let's create a module that has the responsibilty of handling this conventional error pattern.
The `Result` module handles passing values from `:ok` tagged tuples to the next function.
But if the result is an `:error` tagged tuple then following functions will simply be ignored.

The function of interest is `bind`.
It has the following behaviour:

- If the result given is a success, call the next function with the returned value.
- If the result is a failure, don't invoke the next function.

{% highlight elixir %}
defmodule Result do
  def bind({:ok, value}, func) when is_function(func, 1), do: func.(value)
  def bind(failure = {:error, _}, _func), do: failure
end
{% endhighlight %}

With the `bind` method we can then return to a linear set of steps.

{% highlight elixir %}
import Result

{:ok, "my_company/employees.json"}
|> bind(&File.read/1)
|> bind(&Poison.decode/1)
|> bind(fn
  (%{"Peter": data}) -> {:ok, data}
  (_) -> {:error, :key_not_found}
  # This noise here is only necessary as the Dict returns :error without a reason
end)
{% endhighlight %}

Not bad! We have a single list of steps, but having the `bind` function and function capturing everywhere still makes this a bit messy.

Can we go better again?

### There is a monad in the house.

This combination of tagged tuples can be viewed as a result monad. The result monad is a monad because it has features in common with other monads.

Monads are a very abstract concept and as we are not trying to generalise any further we do not need to investigate them any further.
Using the result tuple as a monad allows us to use the experience of people who have thought more about monads.

For this reason I am working on an extension to the result module that will handle tagged tuples as monads.
I want it to be able to handle `{:ok, value} | {:error, reason}` only so that it provides guidance on the API for other functions.
The extension will be the ok-pipeline `|ok>` a combination of an elixir pipeline and the bind function.

With this ok-pipeline our example problem could be refactored even further.

*I have assumed the API for dict is updated*

{% highlight elixir %}
use Result

def get_employee_data(file, name) do
  {:ok, file}
  |ok> File.read
  |ok> Poison.decode
  |ok> Dict.fetch(name)
end

def handle_user_data({:ok, data}), do: IO.puts("Contact at #{data["email"]}")
def handle_user_data({:error, :enoent}), do: IO.puts("File not found")
def handle_user_data({:error, {:invalid, _}}), do: IO.puts("Invalid JSON")
def handle_user_data({:error, :key_not_found}), do: IO.puts("Could not find employee")

get_employee_data("my_company/employees.json")
|> handle_user_data
{% endhighlight %}

### Conclusion

At the end of the last example we used a regular pipe to pass the tuple to the function instead of to the extracted value.
In practice this often happens as the last step because we can't put off handling errors forever.
And nor should we, code is confident when we can handle errors all in one place.
This last function that knows about errors expects a tuple and not just the value  - to label this difference I normally name them `handle_something`.

So does considering monads in a pipeline help or hinder your thinking?
I think it helps and have managed to use a result pipeline in a couple of places so far.
If you are able to try it out do let me know how it goes.

### Resources
Finally a few excellent resources on monads if you want to read further but I stress that a full understanding is not necessary to enjoy the benefits.

- [Refactoring Ruby with Monads](https://www.youtube.com/watch?v=J1jYlPtkrqQ)  
  Whistle-stop tour of monads with [Tom Stuart](https://twitter.com/tomstuart). His `Optional` is the analogue of the Result monad.
- [Monads in Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)  
  Best resource for explaining monads, done with pictures. Thanks to [Aditya Bhargava](https://twitter.com/_egonschiele) for this one.
