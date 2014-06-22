---
layout: post
title:  "Private Methods in Javascript, and why I'll never use them"
date:   2013-12-04 08:59:20
categories: Javascript
---

I'm a big fan of privatizing utility methods in an object.  In fact, I'm often one to go back and privatize methods that aren't accessed publicly in some of my teammates' code.  That being said, and to continue on my previous OO in Javascript post, I want to talk about privates in Javascript.

Douglas Crockford's short post on method scoping in js is the core reference here: [Private Members in Javascript](http://javascript.crockford.com/private.html).

Crockford lays it out simply:

####Public

{% highlight javascript %}
function Cheese() {}

Cheese.prototype.stinks = function() {
  return true
}
 {% endhighlight %}
####Private

 {% highlight javascript %}
function Cheese() {
  function stinks() {
    return true
  }
  stinks();
}
 {% endhighlight %}

The tricky part about pure private `vars` and functions here that they _do not_ have access to `this`, or the public methods.  To bridge the gap, a privileged  method can be created which has access to both.

####Privileged

 {% highlight javascript %}
function Cheese() {
  function stinks() {
    return true;
  }
  this.stinky = function() {
    return stinks();
  }
}
 {% endhighlight %}

So no, this method is not in the prototype, but it is accessible to `this` and public methods.  However, it also exposes the (now uselessly) private method to any instances of this object.  This isn't the worst of it.  Here's the accepted solution to 'privatizing' methods:

 {% highlight javascript %}
function Cheese() {
  var accessed = 3;
  function stinks() {
    if (accessed > 0) {
      accessed -= 1;
      return true;
    }
    else {
      return false;
    }
  }
  this.stinky = function() {
    if (stinks()) {
      return true;
    }
    else {
      return null;
    }
  }
}
 {% endhighlight %}

Yep.

This exposes `.stinky()` 3 times per instance of `Cheese`, after that it will return `null`.  

So -- The solution is to count how many times your utility methods will be used per instance, and only expose your privates for that many 'accesses'.  This is just plain silly.  And luckily, it doesn't stop there.

This pattern results in some potentially serious performance issues:

> This is a private method all right, but will tend to use up a lot more memory than an usual prototype method, especially if you are creating a lot of these objects. For every object instance, it creates a separate function bound to the object and not the class. Also, this does not get garbage collected until the object itself is destroyed. -- [SO user Arindam](http://stackoverflow.com/questions/55611/javascript-private-methods)

You also run into inheritance issues.  If an object is created to inherit from `Cheese`, overriding the private methods is useless, as it will call the parent in any public methods that try and use it.  You also, because they're private, don't have access to the parent's private methods.

###Ok, so...

Everything about OO in javascript seems like a hack to me, and it gets scarier the more I understand about it.  The above pattern is certainly a hack, this one just happens to be exceptionally ugly.  I will be prefixing methods with an underscore to signify "not for public use", not run into memory issues, and not worry about future inheritance weirdness.  A prefix should be signal enough, as "we _are_ all adults here."
