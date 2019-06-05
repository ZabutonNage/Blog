---
meta:
  - name: title
    content: Optional Chaining or SafeAccess for JavaScript
  - property: author
    content: ZabutonNage
  - name: description
    content: This post talks about implementing optional chaining with plain JavaScript. It gets beefed up with default values and safe function application through monadic structures for an extra smooth developer experience.
  - name: keywords
    content: functional javascript optional chaining safeaccess es6 proxy

  - name: twitter:card
    content: summary
  - name: twitter:title
    content: Optional Chaining or SafeAccess for JavaScript
  - name: twitter:creator
    content: "@ZabutonNage"
  - name: twitter:description
    content: This post talks about implementing optional chaining with plain JavaScript. It gets beefed up with default values and safe function application through monadic structures for an extra smooth developer experience.
  - name: twitter:image
    content: https://www.zabutonnage.net/avtr.png

  - property: og:title
    content: Optional Chaining or SafeAccess for JavaScript
  - property: og:type
    content: article
  - property: og:image
    content: https://www.zabutonnage.net/avtr.png
  - property: og:description
    content: This post talks about implementing optional chaining with plain JavaScript. It gets beefed up with default values and safe function application through monadic structures for an extra smooth developer experience.
---

# Optional Chaining or SafeAccess for JavaScript

<p style="font-style:italic; color:#bbb;">Originally posted in 2019-03</p>

Let's discuss the concept of optional chaining in JavaScript. There is a **tc39 proposal** which we will take a brief look at. Then we will explore a few options you have to use this feature already today before it has become part of the standard. And lastly, we are going to implement it **ourselves**. But better! We will see that it is not always necessary to extend a language's standard because a lot of times, and with a little creativity, a feature can be implemented with already existing tools.

## What this is about

Oftentimes, when accessing nested properties of JavaScript objects, you have to verify the existence of all properties leading up to the one you are looking for.
```js
const street = user && user.address && user.address.street;
```
If you don't and you hit an `undefined` halfway, you get a nasty error message which you probably know very well.
```
Uncaught TypeError: Cannot read property 'address' of undefined
```

As of February 2019, there is a [Stage 1 proposal](https://github.com/tc39/proposal-optional-chaining) that tries to combat this issue. The **optional chaining operator**. Depending on your preferred platform you may know it as _"safe navigation operator"_, _"null-conditional operator"_ or perhaps _"existential operator."_ I have even heard it being called the _"Elvis operator"._ But that only seems to be used among C♯ folks. Here it goes:
```js
?.
```
The idea is that you can safely access an object's properties without that tedious validation and without being thrown exceptions at if things aren't were you expect them to be.
```js
const maybeStreet = user?.address?.street;
```
If `user` or `address` is `undefined`, the result of the expression and therefore `maybeStreet` will be `undefined`. Makes things much shorter and readable.

## Shortcomings

There are some shortcomings though. The main one I see is that we can't provide a default value for the case of unsuccessful access. The operator always defaults to `undefined`. So we still have to check whether our efforts of getting the value were actually successful or not. This requires us to assign an intermediary name even though we want to use the value right away. We don't need it to loiter around.
```js
const maybeStreet = user?.address?.street;
return maybeStreet !== undefined
    ? fetchHousesOnStreet(maybeStreet)
    : Promise.reject();
```
Compared to the old way:
```js
const maybeStreet = user && user.address && user.address.street;
return maybeStreet !== undefined
    ? fetchHousesOnStreet(maybeStreet)
    : Promise.reject();
```
Not much of a difference. It's a bit more readable and a few characters shorter but the repetitive assign/null-check/perform pattern remains unchanged. Cold comfort.

This pattern is ceremony that doesn't add to the expressivity of the code and it pollutes the current scope _and our brains_ with temporary names. We should always strive for writing code that is as **expressive** and **declarative** as possible. It helps others and your future self to get an idea of **what** this code is doing instead of **how** it is doing things.

As Eric Elliott put it in his excellent series on [Composing Software](https://medium.com/javascript-scene/composing-software-an-introduction-27b72500d6ea):
>More concise code expression leads to enhanced comprehension. Some code gives us useful information, and some code just takes up space.

Go check it out!

To improve the expressivity of the example above let's ask ourselves **what** we actually want to do here.

_Call `fetchHousesOnStreet` on `user.address.street` if `street` is there, otherwise return a rejected Promise._

Agree? Good! Now let's see what the code is telling us:
- get `user.address.street` and store in `maybeStreet`
- check if `maybeStreet` has a useful value
- if it has, call `fetchHousesOnStreet`
- otherwise reject

Too much information! Getting the value in a separate step and checking if we were successful are way too trivial to concern ourselves with. Let's get rid of them.

Now we could make `fetchHousesOnStreet` handle the check for `undefined`.
```js
function fetchHousesOnStreet(maybeStreet) {
    if (maybeStreet === undefined) return Promise.reject();

    // actual implementation
}
```
Then we could use it like this.
```js
return fetchHousesOnStreet(user?.address?.street);
```
Looks quite good. Problem is, to use this pattern the ceremonial check must now be placed in **every single function** that shall have this behaviour. We just relocated the unwanted check and are now polluting our functions with that ceremony. These functions can no longer exclusively focus on their actual task. In other words, they can no longer trust their input. This is actually worse than the first pattern. And thinking about refactoring this.... Let's try something else.

What about this?
```js
return fetchHousesOnStreet(user?.address?.street || Promise.reject());
```
Well, this works only if we consider all falsey values to be invalid. The empty string might be reasonably invalid in this very example but if we are, for instance, dealing with numeric values, `0` is very likely to be valid and shouldn't be replaced by the default value.

This isn't getting us anywhere. It seems we would have to introduce more utilities to make safe member access work **and** eliminate noisy null-checks.

So the proposed optional chaining operator simplifies only part of a common pattern. It is therefore questionable if it is worth introducing new syntax and making the learning curve steeper for beginners. Changing the standard should imply a considerable leap, not just a half-hearted tiny step. Because once it's in the standard, it's hard to get it out again.

**Now let's check out some possibilities we have today and how they are dealing with these shortcomings.**

## Your possibilities today

### Babel

Babel has an [experimental plugin](https://babeljs.io/docs/en/babel-plugin-proposal-optional-chaining) that enables you to use optional chaining as it is proposed. That means you still have to deal with noisy null-checks. And a transpiler. If you already have a build apparatus in place, it's probably super easy to implement. I'd rather do without.

### Lodash

[Lodash](https://lodash.com/docs/4.17.11#get) offers the `get` function:

```js
_.get(object, path, [defaultValue])
```
It goes like this:
```js
const object = { "a": [{ "b": { "c": 3, "foo": () => "bar" } }] };

_.get(object, ["a", "0", "b", "c"]);  // 3

_.get(object, "a[0].b.c");  // 3

_.get(object, "a.b.c");  // undefined

_.get(object, "a.b.c", "default");  // "default"

_.get(object, "a[0].b.foo", () => {})();  // "bar"
```
Here you specify your path to the desired property with a string. You can also optionally specify a default value. And it certainly does the job. Of course it  also works for safely invoking functions when we provide a no-op as the default value.

I just don't get warm with providing an array or a string as the path to the required property. Feels unwieldy and unnatural. The refactoring experience isn't too nice either.

### Ramda
Then there are [Ramda](https://ramdajs.com/docs/#path)'s `path` and `pathOr`.

This solution works about the same as Lodash's. The most prominent difference is the inverted order of arguments. While Lodash takes the object first, Ramda takes it last. Similarly, the default value comes first instead of last.
```js
R.path(path, object)
R.pathOr(defaultValue, path, object)
```
Lodash's approach makes sense if you think of a default value as optional. But usually it is better to explicitly state what you want to do and avoid optional arguments. It makes your intent more obvious and therefore reduces chances of people (including your future self) unfamiliar with your code wondering whether you forgot to provide a default value in one place when you provided one in all other places. This can't happen with Ramda's distinction between `path` and `pathOr`.

But more important is that the object to work on is provided last. This makes it very easy to partially apply the function and reuse it. When used frequently, it is the object that is the most likely to change. So it makes sense to take it last with less volatile arguments being stored for reuse. With Ramda's built-in currying you can do
```js
const getStreet = R.pathOr(`N/A`, [`address`, `street`]);
const streets = users.map(getStreet);
```
It is much more likely that you access the same property of hundreds of objects than accessing a hundred properties of only one object.

With Lodash you would have to write
```js
const getStreet = obj => _.get(obj, `address.street`, `N/A`);
const streets = users.map(getStreet);
```
Altogether Ramda is with its parameter ordering and its built-in currying more composition-friendly and thus allows for a nicer functional programming experience.

### Sanctuary

One more library I would like to mention is [Sanctuary](https://github.com/sanctuary-js/sanctuary). Its approach is similar to that of Ramda and it has all of its benefits. One big bonus you get on top is that it eliminates the need to check for `undefined` after operations that can fail. How does it do it?

It wraps results of such operations in a container called **Maybe** which has two possible states: `Nothing` meaning no valid value is contained, and `Just` with the requested result inside. The effect is that the getter function **always** returns something that you can work with. There is no need to check for `undefined` or whatever. Ever! And to access the value that is _maybe_ inside, you simply `map` over that container like you would map over an array. If you are not familiar with this kind of structure, you can indeed (as a start) think of it as an array that may have _one_ or _zero_ elements. But never more!
```js
const user = { name: `Lemmy`, address: { street: `Kilmister St` } };

const maybeStreet = S.gets (S.is ($.String)) ([`address`, `street`]) (user)
// Just ("Kilmister St")

S.map (S.toUpper) (maybeStreet)
// Just ("KILMISTER ST")

const maybeZip = S.gets (S.is ($.Number)) ([`address`, `zip`]) (user)
// Nothing

S.map (S.show) (maybeZip)
// Nothing
```
Sanctuary takes an additional predicate to determine if the found property is actually to be returned. Here we check the type of the found property. Nice!

Still, we have to provide the path to our property as an array of strings. Still feels unnatural. And refactoring support isn't that great. [WebStorm](https://www.jetbrains.com/webstorm/), my favourite IDE, does a pretty good job in that it recognises the properties in the path string and renames them, but after all it's just a textual find-and-replace occasionally resulting in renamed values all over your project. If only we could use native member access kind of like so:
```js
user.address.street.map(toUpper).default(`N/A`)
```


## Implementation with plain JavaScript

Now it's time to build it ourselves! Let's summarise the requirements:
- natively accessing an object's properties without verifying their existence
- no _Cannot access..._ exceptions
- no intermediary variables
- no checking if a value was successfully retrieved
- function application being skipped automatically when value retrieval failed

Considering the requirements of directly accessing properties without ever throwing, and instead returning a `Maybe` container for our requested value, this is what it is going to look like:
```js
SafeAccess(user).address.street
    .map(fetchHousesOnStreet)
    .fromMaybe(Promise.reject())
````
First, we put the `user` object in the `SafeAccess` context which we are going to explore in a minute. This allows us to natively access any property no matter at what level of nesting. So it is perfectly fine to do something like this:
```js
const users = [{ name: `Lemmy` }];
const maybeStreet = SafeAccess(users)[1].address.street.map(st => st.toUpperCase());
```
Usually, we would get the _Cannot access..._ exception on trying to access `address` because there is no object at index 1 of the `users` array. Not so when in the `SafeAccess` context.

Once we have reached the property we are looking for, we call `map`. We pass a function that will be applied to the property's value. That is, if there is one. Just like an array's `map` function it is only applied to elements that are actually in the array. As a result we receive a `Maybe` object that either contains the modified value or _Nothing_.

Now if we are fine to work with a `Maybe` value, we can stop here and move on. If, however, we need an "unboxed" value, we can call `fromMaybe` to exit the `Maybe` context. But as there might be _Nothing_ inside, we have to provide a default value. Otherwise we would be back to where we started checking for a value's existence. 

We also have the opportunity to exit the _Safe_ context without mapping. To do so we call `fromSafe` on the property we want to return. Again, we need to provide default value.

Here are a few samples to get familiar with `SafeAccess`:

```js
const users = [{ name: `Lemmy`, address: { street: `Kilmister St` } }];

SafeAccess(users)[0].name  // Safe(`Lemmy`)
SafeAccess(users)[0].name.fromSafe(`default`)  // `Lemmy`
SafeAccess(users)[2].name.fromSafe(`default`)  // `default`
SafeAccess(users)[0].address.zip  // Safe(<invalid>)
SafeAccess(users)[0].address.zip.fromSafe(`default`)  // `default`

const safeUsers = SafeAccess(users);

safeUsers[0].name.map(st => st.toUpperCase())  // Maybe(`LEMMY`)
safeUsers[0].address.zip.map(zip => zip.toString())  // Nothing

const safeAddress = safeUsers[0].address;

safeAddress.street.map(st => st.toUpperCase()).fromMaybe(`default`)  // `KILMISTER ST`
safeAddress.zip.map(zip => zip.toString()).fromMaybe(`default`)  // `default`
```

::: tip Wow!
Did you notice that you can store a partly traversed path in a variable and continue later on? Of course you can pass those properties wrapped in their _Safe_ context to functions, too. You can't do that with the libraries from the previous section.
:::

### Is this magic?

No. It's ES6. Nothing special going on, actually.

Let's inspect how `SafeAccess` works. The core feature we need is `Proxy`. As we are also going to need a few helpers and shared references, we put everything in a module.

First, here is the entire module. Take a look to get an overview of its workings. Afterwards, we are going to look at the individual parts.

```js
// SafeAccess.js
module.exports = SafeAccess;


const invalid = {};


function SafeAccess(obj) {
    const lastProxyable = { value: undefined };
    const handler = {
        get: (o, prop) => {
            if (prop === map.name) return map(o, lastProxyable);
            if (prop === fromSafe.name) return fromSafe(o, lastProxyable);

            const nextVal = o[prop];

            if (o === invalid || o === lastProxyable || !defined(nextVal)) {
                return new Proxy(invalid, handler);
            }

            if (typeof nextVal === `object`) {
                return new Proxy(nextVal, handler);
            }

            lastProxyable.value = nextVal;
            return new Proxy(lastProxyable, handler);
        }
    };

    return new Proxy(obj, handler);
}


function map(o, lastProxyable) {
    const Maybe = require(`./Maybe`);

    if (o === invalid) return () => Maybe();

    const val = o === lastProxyable
        ? lastProxyable.value
        : o;

    return f => Maybe(f(val));
}

function fromSafe(o, lastProxyable) {
    return val => o === invalid
        ? val
        : o === lastProxyable
            ? lastProxyable.value
            : o;
}


function defined(a) {
    return a !== undefined && a !== null && !Number.isNaN(a);
}
```

There is only one export: the `SafeAccess` function. It takes an object to be put in the _Safe_ context, defines a handler config object to be used by a `Proxy` and eventually creates a `Proxy` with the received object and the handler.

### The handler

The heart of a `Proxy` is of course its handler. Here, we define a trap for `get` operations. It receives the proxied object `o` and the property name `prop` being accessed. An example to illustrate this behaviour:
```js
new Proxy({ foo: `bar` }, {
    get: (o, prop) => { console.log(o, prop); }
}).foo
// {foo: "bar"} "foo"  <- logged
// undefined           <- returned value
```
As expected, the proxied object `{ foo: "bar" }` and the accessed member `"foo"` are logged. Our `get` handler doesn't explicitly return anything, therefore we get `undefined`.

There are three main ways our `get` handler can return. By the special property `map` which returns a function that, when called by the consumer, returns a `Maybe` object. By another special property `fromSafe` that returns a function which allows exiting the _Safe_ context directly. Or, in any other case, by returning another `Proxy`.

Note that it is our implementation that makes `map` and `fromSafe` special.

Recursively returning another `Proxy` attached with the same handler enables the consumer to access object members at any depth. Only when accessing `map` or `fromSafe` it is possible to exit the _Safe_ context and get a plain value.

While traversing an object's members it is important that we keep track of the current state of validity. For this purpose there are `lastProxyable` and the module-scoped object `invalid`. Let's investigate the handler's recursive part first and see how these two objects work together.

#### Three Proxies

```js
const nextVal = o[prop];

if (o === invalid || o === lastProxyable || !defined(nextVal)) {
    return new Proxy(invalid, handler);
}

if (typeof nextVal === `object`) {
    return new Proxy(nextVal, handler);
}

lastProxyable.value = nextVal;
return new Proxy(lastProxyable, handler);
```

There are three branches that all return a new `Proxy`. This means that in the next iteration we will still be in the _Safe_ context. The important difference is the object that we wrap in the `Proxy` before returning.

Let's assume for now the user wants to access a valid property. The first condition evaluates to `false` and we skip to the second. We check if `nextVal` is of type `object`. This is important as `Proxy` cannot be applied to **primitives**! Now if the next value is an object, we can simply wrap it in another `Proxy` and carry on. That is the most trivial case.

In case `nextVal` is NOT an object but a primitive, leading us to the third branch, we can't stick it in a `Proxy`. This would actually cause an exception! But one does not simply return a plain primitive and exit the _Safe_ context either. The end of the context would be unpredictable for the user and it would thus totally defeat the purpose of `SafeAccess`.

The solution is the `lastProxyable` object. As it is just a regular object, we can put it in a `Proxy`. We use it as a container for a non-proxyable primitive. At the same time it indicates your last chance to call `map` or `fromSafe` and get a value before going invalid.

That being said, let's take a look at the first branch. We can see that trying to access the next value from `lastProxyable` results in an invalid result. Remember that `lastProxyable` was defined outside the `get` trap and that we just passed it to a `Proxy` in the previous iteration. `o` is exactly that object so we can do a convenient check for referential equality. On all further attempts to access another property, `o` will be `invalid`.

For completeness' sake, we take a brief look at the last possibility resulting in an invalid result. And that is always when the accessed property's value is deemed not `defined`. This would typically be the case when the requested property does not exist on the preceding object.
```js
SafeAccess({ foo: `foo` }).foo.bar
// Safe(<invalid>)
```

#### The way out

As already stated above the only way out of the _Safe_ context is via call to `map` or `fromSafe`. We have two dedicated checks in the `get` handler:
```js
get: (o, prop) => {
    if (prop === map.name) return map(o, lastProxyable);
    if (prop === fromSafe.name) return fromSafe(o, lastProxyable);

    /* ... */
}
```
We simply check if the specified property name matches either of the special functions and, if it does, delegate to the relevant one. Notice that `prop === map.name` is used instead of `prop === "map"`. This is a nice way to ensure that the names of the function and the public property representing it are always in sync. Whenever you rename the `map` function, the property name will reflect it automatically.

##### map

```js
function map(o, lastProxyable) {
    const Maybe = require(`./Maybe`);

    if (o === invalid) return () => Maybe();

    const val = o === lastProxyable
        ? lastProxyable.value
        : o;

    return f => Maybe(f(val));
}
```
`map` takes the object `o` to be mapped over and the `lastProxyable` object. We import `Maybe`, a neighbouring module from the same repository `SafeAccess` is in. It's very small but we are skipping details here as it is not the focus of this post.

If `o` turns out to be `invalid`, we immediately return a function that ignores its inputs and yields _Nothing_ in the form of an empty `Maybe` object. This constitutes our requirement of skipping the mapping function when the _Safe_ context is invalid.

Remember that we are not mapping yet. We are producing the value for the property `map` on the proxied object. Therefore, we have to return a function in order to be able to call `map` from the outside:
```js
SafeAccess(obj).foo.bar.map(mappingFunction)
```

The next step, if `o` is not `invalid`, is to get the value to map over. In case we already arrived at `lastProxyable`, we have to unwrap it from the container that allowed us to apply `Proxy` even to primitives. Otherwise we just use whatever `o` is.

Here too, we return a function yielding a `Maybe` object. This function takes the user-provided mapping function `f`, applies it to `val` and wraps the result in a `Maybe` object. Even though our value to map over is always valid at this point, we must wrap it in a `Maybe` to stay predictable in our results and not suddenly exit the context. Only this way the user can rest assured there will be a reliable result no matter what the outcome is.

##### fromSafe

```js
function fromSafe(o, lastProxyable) {
    return val => o === invalid
        ? val
        : o === lastProxyable
            ? lastProxyable.value
            : o;
}
```
`fromSafe` works in a very similar fashion to `map`. Same inputs. Returns a function.

It has the purpose to exit the _Safe_ context directly. As mentioned earlier, the user has to provide a default value. This is an important feature in order to avoid inconvenient `undefined` values being returned. Something the proposed optional chaining operator lacks entirely.

The returned function's input `val` is the default value specified by the user. Whenever the object `fromSafe` was called on is invalid, that default value is returned. If the object is valid, just like in `map`, we return the relevant value considering it might be wrapped in `lastProxyable`.

## Closing words

That's about it. We saw that JavaScript already has all it needs to make a smooth optional chaining experience and at the same time eliminate repetitive null-checks. No need to bloat the language standard with a lukewarm feature.

What's worth considering, though, is having one [standard library](https://github.com/tc39/proposal-javascript-standard-library) that encompasses these kind of things without messing with the syntax. `SafeAccess` and `Maybe` would be excellent candidates. Let's hope it will make it one day.

You can find the source code for `SafeAccess` and `Maybe` on GitHub in a repository that collects various _utilities_ that follow a _functional programming_ approach. Appropriately named....

::: tip fUtility
[https://github.com/ZabutonNage/fUtility](https://github.com/ZabutonNage/fUtility)
:::

::: tip Note
This article refers to the initial version of `SafeAccess` [c1240080](https://github.com/ZabutonNage/fUtility/commit/c1240080f740fdc612c9de0e36c1890eb55efec1)  
Meanwhile, it has been updated. Make sure to check out the latest version!
:::

So before hopping on the train for another lame feature proposal, do some experimenting. Try to find the JavaScript way! It's indeed fun and sometimes eye-opening! Many proposals, it seems, try to avoid exactly that and aim for allowing developers to stick with their non-JS habits.

So don't waste your time on redundant proposals. Use it to discover the **brilliance of JavaScript!**

Peace! :v:
