---
layout: post
title:  "Some Readings on Mixins: Part 1"
date:   2014-05-27 08:59:20
categories: Mixins
---

To me, Mixins were always a simple, stateless collection of loosely related methods that could be _included_ in a class.  I mean _Stateless_ in that, they hold no state of their own, rather that they assume a given state, if anything.  This is the perspective Ruby takes on the topic.  As an example:

{% highlight ruby %}
module TestModule

  def test
    puts 'hi'
  end

end

class TestClass
  include TestModule
end
{% endhighlight %}

{% highlight ruby %}
irb> TestClass.new.test
irb> "hi"
{% endhighlight %}

Although quite simple, this perspective is misleading according to the classical definition of Mixins.  Modern mixins' original definition came from a [paper written by Gilad Bracha and William Cook](http://www.bracha.org/oopsla90.pdf), and their basis is this:

> A mixin is an abstract subclass; i.e. a subclass definition that may be applied to different superclasses to create a related family of modified classes. For example, a mixin might be defined that adds a border to a window class; this mixin could be applied to any kind of window to create a bordered-window class.

The bulk of this research came from a [paper that Richard P. Gabriel wrote](http://www.dreamsongs.com/Files/Incommensurability.pdf) that is actually about the philosophy of science / engineering, where context is all based around mixins.  Because I'm obsessed, I went and read the bulk of all the papers he cites while illustrating his, "engineering comes first in computer science" theories.  Although these posts won't cover _any_ of his fascinating perspectives on this philosophy, I'd still definitely recommend reading his paper.  He also [did a talk on the paper](http://www.infoq.com/presentations/Mixin-based-Inheritance), or at least what inspired him to write the paper: _incommensurability_.

To begin the delving into mixins, according to Gabriel, we have to start with Warren Teitelman, a PhD candidate at MIT in 1966. More then twenty years before the seminal Bracha and Cook paper, Teitelman wrote his [dissertation on a system called PILOT](http://dspace.mit.edu/handle/1721.1/6905). PILOT uses something called Teitelman called _advice_.  Here's a reductionist explanation taken from the paper's abstact:

> Advising consists of inserting new procedures at any or all of the entry or exit points to a particular procedure (or class of procedures).

Teitelman's thesis is verbose -- 200 pages, of which I did not read even half.  Though, In skimming through, I did stop and read _all_ of chapter 5: __Experiments With a Question Answer System__.  Using a couple of fairly complex logic problems based around a parallel to [McCarthy's Airport Problem](http://www-formal.stanford.edu/jmc/mcc59.pdf)*, Teitelman shows how tests and protections (for example) can be predefined as _advice_, and then a function can use this advice either before or after its invocation.  It is even possible for _advice_ block a function from being invoked at all.  The clearest example from the paper is still a little murky to me, even after a second and third read-though, so rather than attempting to delve into the weeds, I'll try and explain the best I can at a high level.

Let's assume a function whose job is to solve a problem by recursively running itself, and, if just the right input is given, it can get stuck in an infinite loop.  Using Teitelman's _advice_, a programmer can predetermine some invariants to run at the functions entry and exit points to test something like, "how may times have I called myself?" or "have I adequately solved this problem and can I stop the recursive calls?".  The programmer writes these _advice_ procedures in a generic way so that they can be reused by any number of functions who may meet the same requirements.

<pre>
(tell solution1, (before number advice),
If (countf history ((solution1 -)) is greater than 2, then quit)
</pre>

This is some code from the example above, where "number" is the number of times the recursive function has called itself:

> The user tells PILOT to modify SOLUTION1.  The phrase "(before number advice)" tells PILOT to insert this advice immediately before the advice containing the key word "number".

The first thought to came to mind after skimming this paper was Python decorators.  Though not exactly PILOT, I do see some strong similarities.  Lets take the example above, at least the part where we block a call to a function if its invocation count is greater than a pre-determined value.  This requires a bit of an understanding of Python and its decorators**, but lets just assume you "get it", now that I've briefly described PILOT.

First we'll set up a class whose constructor accepts a function and a count.  The class then sets these properties as instance variables.

{% highlight python %}
class Decorator(object):

    def __init__(self, func, count = 0):
        self.count = 0 or count
        self.func = func
{% endhighlight %}

Next, we'll override the `__call__()` method on the class.  This is the method that is invoked on a decorator.

{% highlight python %}
def __call__(self):
    self.count += 1
    if self.count < 5:
        self.func()
    else:
        print 'not calling'
{% endhighlight %}

And last, we define a recursive method, and decorate it with our `Decorator` class.

{% highlight python %}
@Decorator
def recurse():
    print 'recurse called'
    recurse()
{% endhighlight %}

As expected, the decorator blocks the call once `count` is greater than five.  Output:

<pre>
recurse called
recurse called
recurse called
recurse called
not calling
</pre>

[_See the whole thing here if you'd like (gist.github.com)_](https://gist.github.com/f1cd62a266566fe6b7cc)

On `recurse()`'s fifth invocation, it is not invoked, and the program exits.  To me, this is exactly the type of invariant that Teitelman is describing in the PILOT examples.  While not quite aligned with the modern perspective of multiple-inheritance based mixins, this is a clear precursor to the type of reusable, common functionality we understand as mixins today; at least on a much smaller (per function) scale.

(On a side note, how cool is that the same instance is passed to each call to `Decorator`.  I was surprised that it was this simple to implement.)

Up next in the mixin rabbit trail is _Flavors_ -- another Lisp based precursor to the Bracha and Cook paper, similar in pattern (before and after), but quite different in concept.



#### Footnotes:

Most of the research here should be pretty well cited in context, however, if you have any questions, please hit me up [on Twitter](https://twitter.com/danielscottt) -- I love to talk about computers.

<span class='footnote'>*This is not the googlable 'airport problem', this is why I've included the link to the paper where McCarthy first supposes the problem in 1959.  One of the first of its kind.</span>

<span class='footnote'>**Python decorators are this thing where functions can be passed as parameters to generic functions that utilize the `__call__()` method on the type `Function`.  Reminds me, in a way, to a Ruby function that takes a block (`Proc`).</span>
