---
layout: post
title:  "Symbols, Values, and Javascript"
date:   2017-03-16
categories: javascript symbols values es6
---

It feels like the Symbols API is aimed at your feet, and you're going to shoot
yourself with it.

All languages that support symbols or atoms as a language level construct,
support it in such a way that the atom evaluates to itself and thus is in
practical terms a _constant, unique name_.

```erlang
Eshell V7.3  (abort with ^G)
1> what.
what
```

```ruby
irb(main):001:0> :what
=> :what
```

```racket
Welcome to Racket v6.6.
> 'What
'What
```

And this symbols are _global_, because they are _values_.

Program wide, wherever you are, `what == what` will always be true in erlang,
`:what == :what` true in ruby, and `(eq? 'what 'what)` will always be `#t` in
racket.

This is mighty useful, as it works as both a cheap way to define constant
references to things (such as keys in maps, or module names), and also work
great for data structure annotations (such as tagged tuples).

I.e, it is a common erlang idiom for a function to return `{ok, Result}` or `{error,
Message}`.

## How Symbol works in Javascript

In Javascript, to create a Symbol, we rely on a new API, the `Symbol(a: string):
Symbol` constructor. Let's use it:

```javascript
let A = Symbol('my_symbol')
```

Great, now `A`, is a variable storing a unique value. This sort of reference would work very handy for plenty of things (such as we've seen before).

But look what happens when we try to compare the symbol:

```javascript
// while
A == A
// yields a very expectable true,
Symbol('my_symbol') == Symbol('my_symbol')
// is false!
```

God forbid we don't have access to the _original reference_ pointing to the
Symbol, because otherwise we won't be able to use it _at all_.

Implicit in this behavior is the fact that Symbols _are local by default_, since
we assign them to variables.

This means we need to `export` and `import` the specific Symbols we want to use
in our application.

Which leads to code that looks like this:

```javascript
export const MY_SYMBOL = Symbol('MY_SYMBOL')

// and it's counterpart

import MY_SYMBOL from 'some/module'

// or worse

import MY_SYMBOL as stuff from 'some/module'
```

The last case ilustrating how we can import a reference to a symbol with a
completely different name.

Without forgetting that these are actually only references to the Symbol, if
they are not made `const`, then the referencer can be made to point elsewhere,
and thus the symbol is utterly lost.

```javascript
> let A = B = Symbol('A')
undefined
> A = 1234
1234
> A = Symbol('A')
Symbol(A)
> A == B
false
// thank god we had B there to compare!
```

It'd be even possible to *swap* the names of two references. And good luck
debugging that!

```javascript
> let A = Symbol('A')
undefined
> let B = Symbol('B')
undefined
> let C = B
undefined
> B = A
Symbol(A)
> A = C
Symbol(B)
> A.toString()
"Symbol(B)"
> B.toString()
"Symbol(A)"
```

Summing up, remember to:

1. Keep your references `const`ant
2. Export the symbols you want
3. Import the symbols you want
4. Not rename your references, not make references to references

## How Symbol _should_ work in Javascript

Interestingly enough, the Symbol API includes a constructor of the form
`Symbol.for(a: string): Symbol` that regardless of when or where you call it,
_will always return the same symbol for the same identifier_.

What?

It means that, unlike this:

```javascript
> Symbol('A') == Symbol('A')
false
```

`Symbol.for` works like this:

```javascript
> Symbol.for('A') == Symbol.for('A')
true
```

Yes! A true, constant, unique name that can be used and reused across the
application without importing, exporting, shadowing, renaming, or having to
remember to make `const`.

A _value_. And yes it _Just Works_(tm).

Now, ideally we'd have syntax level support for this such as ruby's `:name` or a
Lisp-like `#'name` or `'name`, but for the time being I'm content with aliasing
a method:

```javascript
const atom(r: string): Symbol => Symbol.for(r)
```

and using that all over my Javascript, knowing that the symbol itself will
_always_ evaluate to itself.

Summing up, remember to:

1. Safely use `atom('yay')` anywhere you want to use that specific atom.
2. Profit
