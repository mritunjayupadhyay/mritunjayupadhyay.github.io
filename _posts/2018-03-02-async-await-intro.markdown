---
layout: post
title:  "async await introduction"
date:   2018-02-21 17:49:00 +0530
categories: javascript 2017
---


Today we will learn async-awiat to make asynchronus request. This is a new syntax to write previous way of writing (using `using .then() to resolve promise`). To understand this new syntax, we will write a function in `.then() format` and again write same function using `async-await format`. 

Suppose we are writing a function for getting all user from `myapi.com/user`.

{% highlight ruby %} 
  function prevFormat() {
      fetch(`myapi.com/user`)
      .then((res) => res.json())
      .then((json) => console.log(json));
  }
  prevFormat();
{% endhighlight %} 

Now same function with new syntax. Firstly we will write `async` before function's name in function's declaration and then `await` before any line that is executing asynchronously. 

{% highlight ruby %} 
 async function newFormat() {
      const res = await fetch(`myapi.com/user`);
      const json = await res.json();
      console.log(json);
  }
  newFormat();
{% endhighlight %} 
