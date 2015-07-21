---
type: article
date: 2015-06-05
url: /idiomatic-ruby-2/
series: ["Idiomatic Ruby"]
read_in: 10 minutes
author: Jim OKelly
title: Arrays, Multiple Assiment, and Splatting
summary: A basic look into Idiomatic Ruby Arrays
---

## In Ruby, the Array is King - Thanks Matz!

I have dug into the C source code to look at how Array was implemented since
early on I could tell that Ruby didn't really allocate Arrays in the normal way,
ie.. start with what you have, data size-wise, and then *grow* the Array from
there.

If it worked in the simplistic form I thought about back in 2009, it would be
very, very inefficient, so I went digging. Here is what I found... I give you
the TLDR version of the internals of Array memory management in Ruby.....

Matz and co have *highly optimized* Arrays in Ruby so there is actually a lot
of linked lists and pointer math going on under the hood so you do can work
with Arrays at such a high level, with little care to how you do it. (for the 
most part)

## Use the Array, Luke.. Jedi Ruby Array Tricks

Since Matz and co have gone through such lengths to make Arrays optimal and as
fast as something is going to be on a VM like MRI, I make liberal use of them
in my programs. Due to this need, Ruby has a bunch of 'shortcuts' to help us
create and use Arrays.


Want to create an Array of Strings? Coooooooo man, we got that:

```ruby
%w[John Susan Mary Black Red Heart Spade]
=> ["John", "Susan", "Mary", "Black", "Red", "Heart", "Spade"]
```

What if you need spaces in those strings? Well, then you could do this:

```ruby
%w[John Susan Mary Black Red Heart Spade 8\ of\ spades]
```

This works because `\` is an *escape* character. In this case it *escapes*
the 'specialness' of the space character in the context of `%w` which, if
you remember, uses spaces as an Array delimeter, saving us the 'effort' of
using commas and quotes.

Another common need in Ruby is to create an Array of `Symbols`:

```ruby
%i[hearts spades diamonds clubs]
=> [:hearts, :spades, :diamonds, :clubs]
```

As you can see, that works just like `%w` except we get a `Symbol` instead
of a `String`.

Here are a couple more:

```ruby
%r{ ^http://regex\. }
%{ String can have 'single' or "double" quotes }
```

## Array Maths - If you have 3 oranges and I give you 1 of mine

Let's tell this story using code since this *is* Ruby:

```ruby
mine  = 5.times.map { :orange }
=> [:orange, :orange, :orange, :orange, :orange]

yours = 3.times.map { :orange }
=> [:orange, :orange, :orange]

given = [:orange]
=> [:orange]
```

It's dangerous to go alone! Take this 🍊 

```ruby
yours = yours + mine.pop # .pop, pops one off _mine_
=> [:orange, :orange, :orange, :orange]
```

Still following? I just gave you an orange bro (or broette)! What about my oranges now?

```ruby
mine
=> [:orange, :orange, :orange, :orange]
```

Normally I prefer to *not mutate* stuff willie-nillie, but since these are 🍊 s and not
numbers like 1 2 3 4, we can't just do `mine - [:orange]` (you can try it) because it
will take ALL the 🍊 s out of my bag.

## Multiple assignment and splatting

The last two Array tricks I will show you are multiple assignment and the splat, or 
deconstructive operator.

Multiple assignment is useful in a number of ways, and rather than give another long
winded expaination of it, let's look at some code!

```ruby
def initialize(val1, val2, val3)
  @val1, @val2, @val3 = val1, val2, val3
end
```

I have seen this in a lot of Ruby code and I will call it shit, at least in this form. As
soon as someone needs to add another value to change the order, things get ugly pretty
fast. Let's change the arguments to x, y, and z:

```ruby
def initialize(x, y, z)
  @x, @y, @z = x, y, z
end
```

In this case it isn't extremely terrible from where I sit. As these are now 3d coordinates
their order and grouping makes a lot more sense. So in short, use multiple assignment like
this when the variables are closely related, but a better idea would be to make an object
to 'contain' them and just pass that into this method. Then we don't have the issues of
three params being passed in or what order they need to be in.

But there is another form of multiple assignment:

```ruby
# swap around values
x, y = y, x
```

I like this swapping idiom very much when it is useful to the problem I am solving. Another
one would be decontructing multiple return values:

```ruby
def some_method
  [x, y, z]
end

x, y, z = some_method
```

Since this method returns an Array, we can 'deconstruct' that array into the three values
Here is another example.

```ruby
head, *tail = %w[fred susan bobby]
head
=> "fred"

tail
=> ["susan", "bobbby"]
```

Whoa!!! Pointer magic in Ruby?

Well, no. That is the Splat, or deconstructor operator I elluded to before. Do you see how it
works? `head` gets the first value of the Array, while `tail` gets the rest of the elements,
so we end up with a head with 1 element, and a tail with the rest.

That is all I have for you today. Soon I will start showing off some functional principles
that will make use of what you learned in the first 2 of this series, "Idiomatic Ruby".

See you soon!
