[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

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
let objX = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    return this.count;
  }
};
let objY = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    return this.count;
  }
};
console.log(`X: ${objX.get()}`);
objX.set(10);
console.log(`X: ${objX.get()}`);
console.log(`Y: ${objY.get()}`);
objY.set(100);
console.log(`Y: ${objY.get()}`);
console.log(`X: ${objX.get()}`);
```

As written, calling `get` and `set` on either object will get/set that own
 object's `count` property, but not the other object's.

What if we wanted to ensure that both properties were always in sync?
One way we might accomplish that would be for each object to update the other
 any time that its own value is redefined.

```js
let objX = {
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
let objY = {
  count: 5,
  get: function(){
    return this.count;
  },
  set: function(newVal){
    this.count = newVal;
    if (objX.get() !== this.count) {objX.set(newVal);}
    return this.count;
  }
};
console.log(`X: ${objX.get()}`);
objX.set(10);
console.log(`X: ${objX.get()}`);
console.log(`Y: ${objY.get()}`);
objY.set(100);
console.log(`Y: ${objY.get()}`);
console.log(`X: ${objX.get()}`);
```

By wrapping the `count` value in an object, and hiding the actual reading
 and writing of the `count` variable behind functions, we can set code to run
 any time a property might be updated.
This idea is why Ember decided to create a new object model, the Ember Object.
Almost all of the important piece of an Ember application are Ember Objects,
 so as a result they have the machinery for value binding built in.

Here's an example of how a new Ember Object can be instantiated.

```js
let objX = Ember.Object.create({
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
const Person = Ember.Object.extend({
  sayHello: function(){
    return `Hi, my name is ${this.get('name')}`
  }
});

let frank = Person.create({
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
const Developer = Person.extend({
  code: function(){
    return "type type type"
  },
  sayHello: function(){
    return `${this._super()}, and I am a developer`;
  }
});

let jane = Developer.create({
  name: "Jane"
});

jane.get('name'); // "Jane"
jane.sayHello(); // "Hi, my name is Jane, and I'm a developer"
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

> `import` and `export` are new ES2015 syntax for importing and exporting data
> from a module.
> Unlike Node modules, ES2015 module can export multiple things.
> However, it only has _one_ default export value, which you can specify
> by writing `export default`.

### Lab : Ember Objects and Classes

Inside this repo, run `ember serve --proxy` to launch your app; then, load up
 `localhost:4200` in your browser, and open up the inspector to the Console.
Write a script that will create a new Ember Class called 'Pet', with properties
 `name` and `age` and methods `eat` and `sleep` (which return, respectively,
 'nom nom nom' and 'zzz').
Instantiate that Ember Class by creating a new Pet object with name 'Bruce'
 and age '9'.
Change Bruce's age to 10, and print out Bruce's new age using `console.log`.
Copy your script into the console to run it.
Do you get the expected results?

Next, add a new Ember Class called 'Dog' to your script; it should have a new
 method, `bark`, which prints out the text 'arf'.
Create a new Dog object with name 'Jellybeans' and age '7', and call
 Jellybeans's `bark` method.
Finally, rerun your script in the console.
Did it work correctly?

## Computed Properties

As mentioned earlier, binding properties together is one of Ember's neatest
 features, and `Ember.Object`'s `get` and `set` methods are a big part of making
 that possible.
One common way that binding is implemented is through the use of
 **computed properties**.

Consider the following Ember Class:

```js
const Person = Ember.Object.extend({
});

let tina = Person.create({
  givenName: 'Tina',
  surname: 'Belcher'
})
```

Suppose we wanted `tina` to have a property called `fullName` which was equal
 to his given name plus his surname.
We could obviously define a function in the Class definition to return that
 value.

```js
const Person = Ember.Object.extend({
  fullName: function(){
    return `${this.get('givenName')} ${this.get('surname')}`;
  }
});

let tina = Person.create({
  givenName: 'Tina',
  surname: 'Belcher'
})

tina.get('fullName')(); // 'Tina Belcher'
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
const Person = Ember.Object.extend({
  fullName: Ember.computed('givenName', 'surname', function(){
    return `${this.get('givenName')} ${this.get('surname')}`;
  })
});

let tina = Person.create({
  givenName: 'Tina',
  surname: 'Belcher'
})

tina.get('fullName')();
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
 track changes to a property on all of those Ember Objects.
Ember provides a special key called `@each` that it can use to unpack those
 properties and track them.

```js
const Person = Ember.Object.extend({
  fullName: Ember.computed('givenName', 'surname', function(){
    return `${this.get('givenName')} ${this.get('surname')}`;
  }),
  kids: [],
  numKidsUnder18: Ember.computed('kids.@each.age', function(){
    let kids = this.get('kids');
    return kids.filter((kid) => kid.age < 18).get('length');
  })
});

let tina = Person.create({
  givenName: 'Tina',
  surname: 'Belcher',
  kids: [
   Ember.Object.create({ name: 'Tina', age: 13 }),
   Ember.Object.create({ name: 'Gina', age: 11 }),
   Ember.Object.create({ name: 'Louise', age: 9 })
  ]
})

tina.get('fullName');
console.log(`Kids under 18: ${tina.get('numKidsUnder18')}`);

tina.get('kids').forEach((kid) => {
  kid.set('age', kid.get('age') + 6);
})
console.log('6 years in the future...');
console.log(`Kids under 18: ${tina.get('numKidsUnder18')}`);

tina.get('kids').forEach((kid) => {
  kid.set('age', kid.get('age') + 2);
})
console.log('2 more years in the future...');
console.log(`Kids under 18: ${tina.get('numKidsUnder18')}`);

tina.get('kids').forEach((kid) => {
  kid.set('age', kid.get('age') + 2);
})
console.log('2 more years in the future...');
console.log(`Kids under 18: ${tina.get('numKidsUnder18')}`);
```

As the kids' ages increase, the value of the computed property changes, and
 because we used `@each`, a change to any of their ages would cause the
 computed property to recalculate.

For a full list of computed properties, you can check the [API docs](http://emberjs.com/api/classes/Ember.computed.html)

### Lab : Computed Properties

Define a new Ember Class and instantiate it.
Then, create at least three new properties on your Ember Object.
Finally, pick three computed properties from the API docs (or write three of
 your own) and add them to the new Ember Class you've defined.

## Additional Resources

-   [Ember 2.2.0 Guide : The Ember Object Model](http://guides.emberjs.com/v2.2.0/object-model/)

## [License](LICENSE)

1.  All content is licensed under a CC­BY­NC­SA 4.0 license.
1.  All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
