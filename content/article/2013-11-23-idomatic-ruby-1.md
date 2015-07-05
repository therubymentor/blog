---
type: article
date: 2013-11-23
url: /idiomatic-Ruby-1/
read_in: 10 minutes
author: Jim OKelly
author_link: https://codementor.io/rubymentor
difficulty: Beginner
title: Introduction to Idiomatic Ruby
summary: Writing beautiful Ruby code starts with being IDIOMATIC
---

## Just because it runs AS Ruby doesn't mean it IS Ruby!

OK, so you are writing IN Ruby, but you probably aren’t writing Ruby. You are writing c/basic in Rubyish syntax. This is bad. Slap your hand bad. Makes Matz cry bad. You should be writing in *idiomatic Ruby*.

So what does this *idiomatic Ruby* I speak of look like?

What better way to present my case then by starting with some *unidiomatic Ruby*?

```Ruby
def some_long_method_with_many_problems(user, ids)
  vals = []
  ids.each do |id|
    vals << user.returns_a_hash(id)
  end

  if vals.any?
    retval = do_something_with_hashes(vals)
  else
    retval = do_something_without
  end
  retval
end
```

And then refactor it into this *idiomatic Ruby* we get much different looking code. Code that is clean and neat and terse. *Ruby code*!

Let’s start with what I would consider the *first* violation of *idiomatic* Ruby, the use of a local variable to be looped over and appended to or replaced based on the outcome of the loop.

```Ruby
vals = []
ids.each do |id|
  vals << user.returns_a_hash(id)
end
```

What we really have here is a *enumeration*. We are going to iterate over several objects and place the return of that object method call into an array.

So instead of looping, lets make it *idiomatic*.

```Ruby
vals = ids.map {|id| user.returns_a_hash(id) }
```

The second violation to *idiomatic* Ruby is the ‘inner’ assignment inside the conditional:

```Ruby
if vals.any?
  retval = do_something_with_hashes(vals)
else
  retval = do_something_without
end
```

The fix is simple, remove the duplication of the retval assignment by moving the assignup up ahead of the if:

```Ruby
retval = if vals.any?
  do_something_with_hashes(vals)
else
  do_something_without
end
```

But then we need to think about what we want this method to *return*. In this case it will be the return of one of the two methods in the conditional. Since this is *Ruby*, every method returns the last evaluation naturally, which is why a return statement at the end of a method in Ruby makes little sense.

Well, if the last evaluation is returned and the last evaluation is the return of one of the two methods, we don't need to store a retval variable at all!

```Ruby
if vals.any?
  do_something_with_hashes(vals)
else
  do_something_without
end
```

All together now!

```Ruby
def a_better_method(user, ids)
  vals = ids.map {|id| user.returns_a_hash(id) }
  if vals.any?
    do_something_with_hashes(vals)
  else
    do_something_without
  end
end
```

Please challenge every line of code you have to be *idiomatic*, *beautiful* Ruby code.
