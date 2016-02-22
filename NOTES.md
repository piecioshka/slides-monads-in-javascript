# Notes

## https://www.youtube.com/watch?v=b0EF0VTs9Dc

- monads - strange, mysterious, weird, but powerful, expressive pattern that we can use for doing programming
- two senses of the term function programing:
    - one of them simply means to program with functions
        - but what we called functional programming only becomes as first class objects
        - functions which operate on other functions as arguments invention of Scheme
        - functions has access to the variables and it turns out that was on enormous breakthrough
        - first mainstream language which have this feature was JavaScript
        - JavaScript is leading the way
    - other sens of functional programming is sometimes called pure functional programming
        - it means functions are mathematical functions, which is quite different than subroutines that return arguments,
          because a subroutine is allowed to do mutation, and in a purely mathematical function, that doesn't happen
        - a function with some input is always going to have the same output
        - and the argument in favor of that style of programming is it makes programs a lot easier to reason about,
          because you don't have bugs that will occur as a result of side effects of something else happening
        - if something works, it's guaranteed to work that way correctly until the end of tim
- we can take any ordinary programming language and turn it into a purely functional programming language by deleting stuff
- that once minted the cannot be modified
- in JavaScript it means removing Date, because every time you call Date, it returns different value - that's mathematically absurd
- every time you call a function it should return the same thing
- some remove all those things from out languages, then we'll have purely function languages, and we'll also find
  we're not going to be able to get anything done anymore
- immutability makes it hard to work in the real world
- now there are pure functional programming languages like ML and Haskell
- there was a loophole in the function contract: and that loophole is that a function can take a function as a parameter
- the Haskell community had reason to adopt something called the I/O monad, because it gave them the ability to act 
  as though the language had I/O, even though it didn't
- there is some who will say, in order to understand monads, you first have to learn Haskell, and you have to learn
  Category Theory. If you don't know these things, there's just the no way to start. You can't learn monads.
- EXAMPLE: with buritos and Spanish
- some people will say, well you at least have to start with the types, that you need a really strongly typed language,
  or a super strongly typed languge in order to manage monads
 
Let's do it!
 
- it turns out, the language you need to learn first is JavaScript, because it has the higher order function stuff that we need

    ```javascript
    function unit(x) // returns `monad`
    function bind(monad, function (x)) // return `monad`
    ```

- and it doesn't have any the type stuff to get in your way. So you can just think about what's going on
- so all three of these functions return monads, and that's it, that monads. Thank you very much!
- a monad is an object in this case, it could be something else, but generally it's and object.
- so if you know anything about JavaScript, you're looking at the unit function, you're going, wait `unit` is a function
  that takes a value and returns an object, so must be a constructor.
- so unit is constructor, so nothing magical there
- so all the magic must be in the `bind` function
- so there are three axioms that you have to hold in order to be a monad

    ```javascript
    bind(unit(x), f) === f(x)
    bind(monad, unit) === monad
    bind(bind(monad, f), g) === bind(monad, (x) => bind(f(x), g))
    ```

  - the first to describe the relation between `bind` and `unit`, which is basically that unit creates a monad that
    represents some value, and the bind function allows another function to have acess to that value.
  - the interesting axiom is the third one, which tells us how we can do composition on the bind method, that we can
    have nested bind methods, and that does the same thing as calling bind with a function that calls bind
  
That's monad.

```javasript
bind(monad, func)
monad.bind(func)
```

- now we can make it easier to deal with these monads by converting it from functional notation into methodical notation
- you need to understand the mapping between functions and methods and once you can do that, 
  then we can very easily transform the way we invoke the `bind` method

Famous monads:

 * Identity Monad
 * AJAX Monad
 * Maybe Monad
 * Promise Monad

## https://curiosity-driven.org/monads-in-javascript

### Identity monad

```javascript
function Identity(value) {
    this.value = value;
}
Identity.prototype.bind = function(transform) {
    return transform(this.value);
};
Identity.prototype.toString = function() {
    return 'Identity(' + this.value + ')';
};
```

```javascript
var result = new Identity(5).bind(function (value) {
     return new Identity(6).bind(function (value2) {
        return new Identity(value + value2);
     });
});

console.log(result);
```

### Maybe monad

```javascript
function Just(value) {
    this.value = value;
}
​
Just.prototype.bind = function(transform) {
    return transform(this.value);
};
​
Just.prototype.toString = function() {
    return 'Just(' +  this.value + ')';
};
```

```javascript
var Nothing = {
    bind: function() {
        return this;
    },
    toString: function() {
        return 'Nothing';
    }
};
```

```javascript
var result = new Just(5).bind(function (value) {
    return Nothing.bind(function (value2) {
        return new Just(value + alert(value2));
    });
});

console.log(result)
```

Great example of another way to works with `null` values.

```javascript
function getUser() {
    return new Just({
        getAvatar: function() {
            return Nothing; // no avatar
        }
    });
}

var url = getUser()
    .bind(user => user.getAvatar())
    .bind(avatar => avatar.url);

if (url instanceof Just) {
    print('URL has value: ' + url.value);
} else {
    print('URL is empty.');
}
```

## List monad

```javascript
function* unit(value) {
    yield value;
}
​
function* bind(list, transform) {
    for (var item of list) {
        yield* transform(item);
    }
}
```

```javascript
var result = bind([0, 1, 2], function (element) {
    return bind([0, 1, 2], function* (element2) {
        yield element + element2;
    });
});
​
for (var item of result) {
    print(item);
}
```

## Continuation monad

* `Promise.resolve(value)` will serve as the Unit function
* `Promise.prototype.then` will serve as the Bind function

```javascript
var result = Promise.resolve(5).then(function(value) {
    return Promise.resolve(6).then(function(value2) {
        return value + value2;
    });
});

result.then(function(value) {
    print(value);
});
```
