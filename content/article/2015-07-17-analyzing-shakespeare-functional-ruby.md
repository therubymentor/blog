---
type: article
date: 2015-07-17
url: /analyzing-shakespeare-functional-ruby/
series: ["Exploring Functional Ruby"]
read_in: 15 minutes
author: Jim OKelly
title: Analyzing Shakespeare using Functional Ruby
summary: Seriously, Enumerable is the shit!
---

## A code kata from some kool people

This kata comes from Thoutbot's Upcase program. A student of mine sent it in to me a year ago and I have used it since as a teaching tool to introduce strictly Object Orientated Programmers to face the strange... Chachachachanges.....

## But first, some Tenderlove parsing!

Nokogiri is the long pause in your `gem install` regimen. That pause is some c extentions being built which are needed to make it the speedy and useful tool it is.

The excersize at hand is to take some XML from this [www.ibiblio.org/xml/examples/shakespeare/macbeth.xml](file) and count out the spoken lines for each character appearing.

## Counting in the counting house... counting out...

To get that count we will be writing a Unix program, meaning a script that is executable, takes input, returns output. We do it with Ruby and a shebang.

```ruby
#!/usr/bin/env ruby
# cat macbeth.xml | xargs -0 ./lc
```

The second line there is really just a comment on how to use the program, but the first line tells the managing process which interpreter to use. This is done because the name of the utility `lc` does not have an *rb* extension.

## Getting the XML in question

We can either get the xml *realtime* and *pipe* it into our utility, or download the xml and then use `cat` to send it into our pipe.

The first option looks like this:

```bash
curl http://www.ibiblio.org/xml/examples/shakespeare/macbeth.xml | xargs -c ./lc
```

The second:

```bash
curl http://www.ibiblio.org/xml/examples/shakespeare/macbeth.xml > macbeth.xml
cat macbeth.xml | xargs -c ./lc
```

Notice the common demominator here is the *./lc*.

## Creating a UNIX utility commmand

The *./* in *./lc* means the current working directory, ie, the one you `cd` into. The filename of the command we are going to write is `lc`. Let's make that shit now.

 ```bash
vim lc
```

Or, create a file called `lc` in the directory you have open in the terminal. In it put:

```bash
#!/usr/bin/env ruby
# cat macbeth.xml | xargs -0 ./lc
```

## Making that shit executable

If you try to run it now:

```bash
./lc
```

You will get a permission error, that is because this file does not have permission to execute as per your login. To *set the executable flag*:

```bash
chmod a+x ./lc
```

Which means in *neckbeard*, make this file executable for *all* groups foo! Now run it

```bash
./lc
```

Nada! But that is what we want. Next add this line to the end of that `lc` file like so:

```bash
echo "puts 'Hello from UNIX bitches!'" >> ./lc
```

You are piping shit all over, welcome to the world of the programmer. /troll

Next time you run it, it will print out that statement, using the Ruby `puts` method, therefore it 'works'. Now let's start making it *useful*.

## Nokogiri search and Ruby map

Time to add some more code. Delete that last puts and make your `lc` file look like this:

```ruby
#!/usr/bin/env ruby
# cat macbeth.xml | xargs -0 ./lc

require 'nokogiri'

doc      = Nokogiri::XML(ARGV[0])
speeches = doc.search('SPEECH')
speakers = doc.search('SPEAKER').map(&:text).uniq
```

We `require`d nokogiri so we could reference it. Then we use nokogiri to parse the xml sent in as `ARGV[0]` into an tree based object model in order to easier extract the stuff we actually want from that giant xml file.

The next line get all the blocks of <SPEECH>, which we need:

```xml
<SPEECH>
  <SPEAKER>First Witch</SPEAKER>
  <LINE>When shall we three meet again</LINE>
  <LINE>In thunder, lightning, or in rain?</LINE>
</SPEECH>
<SPEECH>
  <SPEAKER>Second Witch</SPEAKER>
  <LINE>When the hurlyburly's done,</LINE>
  <LINE>When the battle's lost and won.</LINE>
</SPEECH>
<SPEECH>
  <SPEAKER>Third Witch</SPEAKER>
  <LINE>That will be ere the set of sun.</LINE>
</SPEECH>
<SPEECH>
  <SPEAKER>First Witch</SPEAKER>
  <LINE>Where the place?</LINE>
</SPEECH>
```

Next, we get a unique list of all the speakers, because we will need it later to *aggregate* the speaker lines:

```ruby
...
speeches = doc.search('SPEECH')
speakers = doc.search('SPEAKER').map(&:text).uniq
```

## Using Rubys #map function

I love ```#map``` and it's whole family of *higher order functions* like ```reduce```, and ```select``` and every Highschooler's favorite, ```reject```. This represent the best part of Ruby in my opinion.

We need to *transform* that ```Array``` of nokogiri xml to be in a format for friendly for our purposes, and I have decided that the ```Hash``` is the perfect data structure:

```ruby
{ name: "Bruce Lee", lines: [
  "Hey Chuck....",
  "Who's yo daddy?",
  "Come on Chucky... Say it"
] }
```

To do the transform we will use ```map```, which starts with an enumerable collection, an ```Array``` here, and for every item in the source ```Array```, a new item will be inserted into a *new* ```Array```, and that new ```Array``` will be returned.

That is what a ```map``` means in programming. Here is the code that will map that ugly xml to something more friendly:

```ruby
spoken_lines = speeches.map {|s| { name:  s.at('SPEAKER').text, 
                                   lines: s.search('LINE').map(&:text) } }
```

This is how we build the ```Array``` of ```Hash```s, one has per *row* essentially, and the hash defines the *columns*, much like a database in memory.

Now instead of a lot of data we don't want, in a format we don't like, we have a nice, homogonized block of data to play with.

## I'm getting Aggregated here!

What the hell is an "aggregation"? Well I learned the term doing SQL queries, which, by the way is a great, declarative, almost functional language. It might be easier to *show it*, than to *describe it*:

```ruby
[
  { name: "Bruce Lee", lines: ["Knock knock"] },
  { name: "Chuck Norris", lines: ["Who's there?"] },
  { name: "Bruce Lee", lines: ["Yo daaaadddddyyyyyy!"] }
]
```

This data can be aggregated to remove the *duplicate* information, which will make it a hell of a lot easier to get the count of any one speakers lines. But to do so we need to actually make a couple swipes at the problem to take it down.

We will be *iterating* over that list of speakers we got from *line 8*, and ```select``` the lines that belong to each speaker.

Imagine we are on the first *iteration*, the ```name``` variable would be "Bruce Lee":

```ruby
aggregated = speakers.map do |name|
  {
    name: name,
    line_count: spoken_lines.select {|l| l[:name] == name }.
                            reduce([]) {|acc, l| acc << l[:lines] }.
                            flatten.count
  }
end
=> 
```

## Hello Reduce, hope you guessed my name

This will require a bit-o-splain'. The ```map``` is just a map, and on each iteration we will be returning a single ```hash``` with the stuff inside we be a wantin, namely the name, and a line count.

The line count part gets a bit trickeh. Reduce is like a map, except that instead of building up a collection, it *reduces* down to a single value, even if that single value is an Array of things :)

We will be reducing the lines that are spead around in multiple key values, into a single key-value. Ie, line_count: 42. ```reduce``` takes a block and it has 2 *block variables*, the *accumulator* and the *iterator*.

The *accumulator* is the value that... yep, accumulates over the iterations. Think of it as a variable whos scope extends the lifetime of the whole iteration. Every time the iteration returns a value, that value can be 'mathed' against the accumulator and that result will continue to the next iteration as the accumulator. Easy, right?

```ruby
%i[bob sue jim harry].reduce("") {|acc, name| acc = name }
=> :harry

%i[bob sue jim harry].reduce("") {|acc, name| name }
=> ""

%w[bob sue jim harry].reduce(0) {|acc, name| acc + name.chars.count }
=> 14
```

The easiest way to splain this shit is to just walk through it. So here we go:

(iteration 1)
acc is 0, name is bob = 0 + bob.chars.count (3) = 3

(iteration 2)
acc is 3, name is sue = 3 + sue.chars.count (3) = 6

(iteration 3)
acc is 6, name is jim = 6 + jim.chars.count (3) = 9

(iteration 4)
acc is 9, name is harry = 9 + harry.chars.count (5) = 14

## Formatting and printing

Now that we have our values let's sort, format and print. We start with the sorting, which goes a little something like this:

```ruby
sorted = aggregated.sort {|a,b| b[:count] <=> a[:count] }
```

Then we format it for printing by returning the line count, followed by a tab character, followed by their name:

```ruby
formatted = sorted.map{|s| [s[:count], "\t", s[:name]].join }.
                   join("\n")
```

Printing is stupid simple at this point, and since we have run long I will just spit it out:

```ruby
puts formatted
```

## Back to the pipeline

Now, when we run our unix command pipeline:

```bash
curl http://www.ibiblio.org/xml/examples/shakespeare/macbeth.xml | xargs -c ./lc
```

Now we get the list of names and line counts. Isn't functional programming in Ruby awesome?
