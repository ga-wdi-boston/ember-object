![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

# Ember.Object and Computed Properties

One common reason to use a front-end framework is its interactivity - your
 application can provide functionality to the user without needing to hit your
 back-end.
Often, you will want what the user sees to be updated in real time, or some
 other kind of **value binding** (linking property A to property B so that if A
 is changed, B gets updated too).
Ember provides a way to implement value binding in applications through a
 construction called an Ember Object.

## Prerequisites

By now, you have already learned how to:

-   Download and install `ember-cli` and its dependencies.
-   Name and describe the different parts of an Ember application.

## Objectives

By the end of this session, you should be able to:

-   Explain the purpose of Ember Objects.
-   Instantiate a new Ember Object using `.create`.
-   Read and write property values from an Ember Object using `.get` and `.set`.
-   Create a new Ember Class using `.extend`.
-   Define a custom computed property on an Ember Class.
-   Create a new default computed property on an Ember Class.

## Preparation

1.  [Fork and clone](https://github.com/ga-wdi-boston/meta/wiki/ForkAndClone)
    this repository.
1.  Install dependencies with `npm install` and `bower install`.

## Ember Objects and Classes

Value binding, at its face, flies in the face of everything we know about how
 our programs work.
If you were to define a variable `xCount`, set another variable `yCount` equal
 to `xCount`, and then change the value of `xCount`, the value of `yCount`
 would not change.

So how does Ember make such a thing possible?

Suppose for a minute that instead of two variables, we had two objects, with
 `get` and `set` methods like so:

```js
var objX = {
  count: 5
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    return this.count;
  }
}, objY = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    return this.count;
  }
};
```

As written, calling `get` and `set` on either object will get/set that own
 object's `count` property, but not the other object's.

What if we wanted to ensure that both properties were always in sync?
One way we might accomplish that would be for each object to update the other
 any time that its own value is redefined.

```js
var objX = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    if (objY.get() !== this.count) {objY.set(newVal);}
    return this.count;
  }
}, objY = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    if (objY.get() !== this.count) {objY.set(newVal);}
    return this.count;
  }
};
```

By wrapping the `count` value in an object, and hiding the actual reading
 and writing of the `count` variable behind functions, we can set code to run
 any time a property might be updated.
This idea is why Ember decided to create a new object model, the Ember Object.
Almost all of the important piece of an Ember application are Ember Objects,
 so as a result they have the machinery for value binding built in.

Here's an example of how a new Ember Object can be instantiated.

```js
import Ember from 'ember';

var objX = Ember.Object.create({
  count: 5
});

objX.set('count', 10);
objX.get('count');      // 10
```

`Ember.Object` is an **Ember Class**.
Much like a Ruby class, it can define a bunch of instance methods (e.g. `get`
 and `set`) and provides a constructor (`.create`) for instantiating new
 objects, each of which can call those instance methods.
These new objects can also access any other properties that get passed when
 `.create` is invoked (in this case, `count`).

We can define our own custom 'sub-classes' from `Ember.Object` by using another
 method, `.extend`.
Suppose that we wanted to create a 'Person' Ember Class, with a method called
 `sayHello` that returns "Hi, my name is ... ".
We might write:

```js
var Person = Ember.Object.extend({
  sayHello: function(){
    return "Hi, my name is " + this.get('name');
  }
});

var frank = Person.create({
  name: "Frank"
})

frank.sayHello(); // "Hi, my name is Frank"
```

To create a new sub-Class based on the Person (Ember) Class we've just defined,
 we can simply call `.extend` again.
A sub-class can add new methods or even overwrite old ones;
 when doing the latter, you can reference the method on the parent Class
 by calling `this._super`.

```js
var Developer = Person.extend({
  code: function(){
    return "type type type"
  },
  sayHello: function(){
    return this._super() + ", and I'm a developer"
  }
});

var joe = Developer.create({
  name: "Joe"
});

joe.get('name'); // "Joe"
joe.sayHello(); // "Hi, my name is Joe, and I'm a developer"
```

Ember uses Ember Classes very similarly to how Rails uses Ruby Classes.
A number of Ember Classes are built in (e.g. `Ember.Application`, `Ember.Route`,
 `Ember.Component`), and each time we want to create a new Route/Component/etc,
 we sub-class one of those existing classes, using the `.extend` method.

For instance, suppose that we want to create a new Route.
Ember has a generator, much like Rails and Express, and we can use it to create
 a new Route by running `ember g route about`.
As you can see, Ember has created a couple of new files for us; here's what
 `app/about/routes.js` looks like:

```js
import Ember from 'ember';    // This line makes all predefined Ember code accessible in this file.

export default Ember.Route.extend({
});
```

In the code above, we will be creating a new Ember Class that
 sub-classes (i.e. inherits) from `Ember.Route`; if we were to import this file
 from another file, this new Ember Class is what would get loaded.

> `import` and `export` are new ES2015 syntax for importing and exporting data
> from a module.
> Unlike Node modules, ES2015 module can export multiple things.
> However, it only has _one_ default export value, which you can specify
> by writing `export default`.

### Lab : Ember Objects and Classes

Inside this repo, run `ember serve --proxy` to launch your app; then, load up
 `localhost:4200` in your browser, and open up the inspector to the Console.
In the console window, create a new Ember Class called 'Pet' with properties
 `name` and `age` and methods `eat` and `sleep` (which return, respectively,
 'nom nom nom' and 'zzz').
Instantiate that Ember Class by creating a new Pet object with name 'Bruce'
 and age '9'.
From the console, change Bruce's age to 10, and print out Bruce's new age using
 `console.log`.

Next, create a new Ember Class called 'Dog'; it should have a new method,
 `bark`, which prints out the text 'arf'.
Create a new Dog object with name 'Jellybeans' and age '7'.
Call Jellybeans's `bark` method.

When you finish, tilt your laptop screen down.

## Computed Properties

As mentioned earlier, binding properties together is one of Ember's neatest
 features, and `Ember.Object`'s `get` and `set` methods are a big part of making
 that possible.
One common way that binding is implemented is through the use of
 **computed properties**.

Consider the following Ember Class:

```js
var Person = Ember.Object.extend({
});

var bob = Person.create({
  givenName: 'Bob',
  surname: 'Belcher'
})
```

Suppose we wanted `bob` to have a property called `fullName` which was equal
 to his given name plus his surname.
We could obviously define a function in the Class definition to return that
 value.

```js
var Person = Ember.Object.extend({
  fullName: function(){
    return this.get('givenName') + ' ' + this.get('surname');
  }
});

var bob = Person.create({
  givenName: 'Bob',
  surname: 'Belcher'
})

bob.fullName(); // 'Bob Belcher'
```

The nice thing about using a function is that it will automatically recalculate
 its result based on the latest values of `givenName` and `surname`.
However, we must invoke the `fullName` function in order to get that updated
 value.
As it turns out, there are times in Ember when we will not be able to invoke a
 function directly - one extremely common case is referencing a value from
 within a template.
In those cases, Computed Properties give us a convenient loophole.
Here's how `fullName` might look when set up as a computed property.

```js
var Person = Ember.Object.extend({
  fullName: Ember.computed('givenName', 'surname', function(){
    return this.get('givenName') + ' ' + this.get('surname');
  })
});

var bob = Person.create({
  givenName: 'Bob',
  surname: 'Belcher'
})

bob.get
```

As you can see, `fullName` is now accessible as if it were a normal property.
The arguments in Ember.computed that come before the function tell Ember to
 watch the properties `givenName` and `surname` for changes; thus, the
 Computed Property `fullName` is updated when any of its watched properties are
 updated, _rather than when the property is accessed with `.get`_.

There are a variety of standard computed properties that Ember provides, all
 accessible by calling `Ember.computed.___`; here are a few common ones.

-   `empty` : return true if the dependent property is `null` or an empty
     string, array, or function
-   `equal` : tests equality
-   `gt`/`gte`/`lt`/`lte` : tests inequality (as specified)

If the value of a property is an array, there are even more computed properties
 available:

-   `map`
-   `filter`
-   `sort`

Speaking of arrays: suppose that you have an array of Ember Objects and want to
 access a property across all of those object.
Ember provides a special key called `@each` that it can use to unpack those
 properties.

```js
export default Ember.Object.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  remaining: Ember.computed('todos.@each.isDone', function() {
    var todos = this.get('todos');
    return todos.filterBy('isDone', false).get('length');
  })
});
```

For a full list of computed properties, you can check the [API docs](http://emberjs.com/api/classes/Ember.computed.html)

### Lab : Computed Properties

Define a new Ember Class and instantiate it.
Then, create at least three new properties on your Ember Object.
Finally, pick three computed properties from the API docs (or write three of
 your own) and add them to the new Ember Class you've defined.

## Additional Resources

-   [Ember 2.2.0 Guide : The Ember Object Model](http://guides.emberjs.com/v2.2.0/object-model/)

## [License](LICENSE)

Source code distributed under the MIT license. Text and other assets copyright
General Assembly, Inc., all rights reserved.
