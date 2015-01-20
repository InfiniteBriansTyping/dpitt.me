---
layout: post
title:  "Assertion Avoidance in Go"
date:   2015-01-03 12:43:56
---

  Like many statically typed languages, Go gives the user options when the exact type of an object may not be known.  I'm going to cover two of the three primary ways of doing this in Go, and _then_ discuss avoidance in the context of these methods, and why this avoidance can be advantageous.  To start, here is a traditional Go interface:

{% highlight go %}
type Painter struct {}
type Scraper struct {}

type Doer interface {
	Do()
}

func (p *Painter) Do() {
	// do dtuff
}

func (s *Scraper) Do() {
	// do dtuff
}

func doWork(d Doer)
	d.Do()
}
{% endhighlight %}

  Here the function `doWork` needs to call `Do()` on whatever is passed to it.  It does not need to know the actual type of `d`, but only that it has a method called `Do`.  By creating the interface type `Doer`, and then passing a variable typed to that interface, all `doWork` has to do is execute `Do()` provided by the interface, and does not need to know the underlying type.  Flexiblilty is added in that I don't have to create separate methods for doPaintingWork and doScrapingWork with separate signatures.  Since they both implement `Do()`, we can use an interface.

Go offers two other ways to handle unknown types, and I'm only going to cover one of them.  The remaining type related behavior in Go is _reflection_, which is out of scope for this post, and something I'd generally say to avoid if possible.  Reflection has negative effects on the performance of your program, as it circumvents the complier's type system.

### Assertion

From the Godocs:

[Assertion](https://golang.org/ref/spec#Type_assertions): For an expression x of interface type and a type T, the primary expression

```
x.(T)
```

asserts that x is not nil and that the value stored in x is of type T. The notation x.(T) is called a type assertion.


That is to say, if we go back to our types `Scraper` and `Painter` above:

{% highlight go %}

func (s *Scraper) Dont() {
	// dont
}

func dontScrape(d Doer) {
	s := d.(Scraper)
	s.Dont()
}
{% endhighlight %}

By determining the type of object I'm being given via the interface is in fact a `Scraper`, I can now execute methods which are not implemented on the incoming interface by treating my new object `s` as a `Scraper`, because, beneath the interface, it actually is.

Assertion is less performance inhibitive than reflection, but nonetheless, it is still less than ideal to cirumvent the type system.  That being said, using assertion I have found to be pretty unavoidable in Go programming, especially when making use of many interfaces.

### Avoidance

Before diving into the pattern I'm suggesting, I want share a little about what I mean by _avoidance_.  In [Coodination Avoidance in Database Systems](http://www.bailis.org/papers/ca-vldb2015.pdf) by Peter Bailis, et al the authors discuss the trade off between an application's correctness and its scalability.  I.e. the C and the A in the CAP theorem.  This trade off is typically mitigated by choosing one or the other when designing a system.  In the paper, the authors develop a formal system which they call invariant confluence to achieve (some) of the best of both worlds.  Invariant confluence is essentially used to determine whether or not coordination is _necessary_ for correctness.  Using this system to only coordinate when necessary, allows other actors in the system continue without blocking (or coordinating) when making changes to shared state.  I wrote a bit more about this [here](2014/05/03/coordination-avoidance-systems/).

### Assertion Avoidance

As for _assertion_ avoidance, I am suggesting the use of less a formal system, more a conscious consideration when developing using interfaces.

Using the pattern below, when receiving one of these objects, I can route based on a string comparison, rather than via the [type assertion switch](https://golang.org/ref/spec#Switch_statements):

{% highlight go %}

type MessageBody interface {
	// your common usage message methods
}

type Message struct {
	Key string
	Body MessageBody
}
{% endhighlight %}


{% highlight go %}
func routeMessage(m Message) {

	switch m.Key {
	case "doer.painter":
		m.Do()
	// ...
{% endhighlight %}

Now lets consider the avoidance in this context.  As I stated above, switching over the key above does avoid asserting the type, but I'm going to take the avoidance one step further.  Bear with me through this longer example, I will explain in detail below.

{% highlight go %}
package main

import "fmt"

type Message struct {
	Key string
	Body MessageBody
}

type MessageBody interface {
	Do() string
}

type Painter struct {
	Job string
	Tool string
}

type Scraper struct {
	Job string
}

func (p Painter) Do() string {
	return p.Job
}

func (s Scraper) Do() string {
	return s.Job
}

func paint() {
	p := Painter{"painting", "brush"}
	DoWork(Message{"doer.painter", p})
}

func DoWork(m Message) {
	switch m.Key {
	case "doer.scraper":
		fmt.Println(m.Body.Do())
		// here I just react.  I don't care about the data inside m,
		// I just know that when scraping is done, I need to paint.
		// This means I can avoid asserting m's type
		paint()
	case "doer.painter":
		p := m.Body.(Painter)
		fmt.Println(m.Body.Do() + " with " + p.Tool)
	}
}

func main() {
	s := Scraper{"scraping"}
	m := Message{"doer.scraper", s}
	DoWork(m)
}
{% endhighlight %}
Run it yourself [here](http://play.golang.org/p/CmWxap-7KY).

This example takes our `Painter` and `Scraper` types and uses the objects themselves as payloads to a message struct.  We expect the body to implement the interface `MessageBody` which we called `Doer` before.  By considering what the outcome of scraping in our program would be, we can avoid asserting `m` altogether, as we know that no matter what, when scraping is done, we need to paint.  We do this regardless of what may be contained in the message itself.

Discerning this type of avoidance is trival in the example above; in a complex system this can very quickly become non-trival.  The primary consideration in Assertion Avoidance is a strong separation of concerns between different modules within your program.  These separate parts need to be guarded by interfaces which contain only the necessary parts.  When each module (or package) is self-contained, then the cross communication can be simple, and in effect, better performing.  Coupling the pieces of your program in such a way that the only link is an interface like `MessageBody` (or even a struct like `Event`) can benefit not only the readability of your program through the inherent modularity that comes with SoC, but can actually have positive performance outcomes.  [This benchmark test](https://gist.github.com/danielscottt/ea9e16112f13c78783df) yields the following results:
```
BenchmarkString 2000000000           0.26 ns/op
BenchmarkAssert 1000000000           2.30 ns/op
```
