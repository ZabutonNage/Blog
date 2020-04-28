---
meta:
  - name: title
    content: One Tick To Rule Them All
  - property: author
    content: ZabutonNage
  - name: description
    content: Ending the seemingly eternal battle of single quotes vs double quotes. There is an alternative outdoing both of them.
  - name: keywords
    content: javascript es6 single double quotes backticks

  - name: twitter:card
    content: summary
  - name: twitter:title
    content: One Tick To Rule Them All
  - name: twitter:creator
    content: "@ZabutonNage"
  - name: twitter:description
    content: Ending the seemingly eternal battle of single quotes vs double quotes. There is an alternative outdoing both of them.
  - name: twitter:image
    content: https://www.zabutonnage.net/avtr.png

  - property: og:title
    content: One Tick To Rule Them All
  - property: og:type
    content: article
  - property: og:image
    content: https://www.zabutonnage.net/avtr.png
  - property: og:description
    content: Ending the seemingly eternal battle of single quotes vs double quotes. There is an alternative outdoing both of them.
---

# One Tick To Rule Them All

<p style="font-style:italic; color:#bbb;">Originally posted in 2020-04</p>

Single quotes or double quotes? Which do you prefer?

## Two choices

Both have their own benefits so whichever you choose, you miss out on the other's. Let me show you what I mean.

Double quotes are useful for human language.
```js
const good = "Oh well, c'mon everybody and let's get together tonight.";
```

This is rather awkward with single quotes.
```js
const ugh = 'Oh well, c\'mon everybody and let\'s get together tonight.';
```

Single quotes on the other hand shine when you need to put HTML into a string.
```js
element.innerHTML = '<input id="name" placeholder="Name" />';
```

Imagine this with double quotes. Yikes!
```js
element.innerHTML = "<input id=\"name\" placeholder=\"Name\" />";
```

Obviously the best thing to do is to not constrain yourself to one or the other but instead pick the right one for the right task. Which would be:
- single quotes: HTML
- double quotes: human language

Now what do you do when these two come together?
```js
element.innerHTML = '<textarea id="the-text">Oh well, c\'mon everybody and let\'s get together tonight.</textarea>';
element.innerHTML = "<textarea id=\"the-text\">Oh well, c'mon everybody and let's get together tonight.</textarea>";
```

You lose either way.

## ES6 offers an alternative

Since ES6 there is a **third** kind of quote. The backtick `` ` ``. Its most common use, I would guess, is string interpolation. I'm sure you are already using it. It also has multi-line support.

```js
const text = `Hi!
My name is ${firstName} ${lastName}.`;
```

Remember that you aren't obliged to do string interpolation when using backticks. Neither are you required to have a multi-line string. So you could just....

```js
element.innerHTML = `<textarea id="the-text">Oh well, c'mon everybody and let's get together tonight.</textarea>`;
```

Beautiful! No escaping necessary!

So do we use single quotes for HTML, double quotes for human language and backticks when we have both single and double quotes in the string from now? No! That is way too complicated! More rules means more complexity which leads to more arguments and lower productivity.

Make it simple. Make it good!

Fewer rules means fewer exceptions. Fewer exceptions means less complexity. Less complexity means more available brain capacity to tackle truly complex problems.

::: tip The consequence
Use backticks everywhere!
:::

## Watch out

There are some restrictions, however. But your JavaScript parser will happily yell at you if you violate them.

For example, you can't use backticks in `import` statements, although you can in `require` because it is a regular function call.

```js
import * as Foo from `my-module`;  // illegal

const Foo = require(`my-module`);  // works just fine
```

You also can't use them directly for object keys, but it is possible when assigning them dynamically because, again, there it is basically just a regular argument to a function.

```js
{ `nope`: `value is ${okay} of course` }  // key is illegal, while value is ok

{ [`dynamic-${key}`]: `works like a charm` }  // everything's fine here
```

Simply pick your favourite quote for these cases.


## The benefits

I have been using _"backticks-everywhere"_ for a couple of years in production now. Teams I introduced them to picked them up quite quickly and there hasn't been a discussion about _single vs double_ ever since. To put the benefits into a list:

- no arguments over _single vs double_
- decision of which quote to pick is a no-brainer because there is no decision to be made
- no escaping of other quotes necessary
- string interpolation and multi-line support included
- convenient location on keyboard (at least on mine where it is next to `P`)

**Go give it a try!**

And if you like, find me on Twitter to tell me about your experience.
