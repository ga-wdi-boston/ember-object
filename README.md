![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

# Ember.Object and Computed Properties

## Lesson Details
### Foundations
At this point, students have already learned how to:

- Download and install `ember-cli` and its dependencies.
- Name and describe the different parts of an Ember application.

### Objectives
By the end of this lesson, students should be able to:

- Explain the purpose of `Ember.Object`
- Create a new Ember Object using `.create`
- Read and write property values from an Ember Object using `.get` and `.set`
- Create a new Ember Class using `.extend`
- Define a custom computed property on an Ember Class.
- Create a new default computed property on an Ember Class.

## Ember Objects and Classes
One of the neat features of Ember that we mentioned earlier is **value binding** - linking property A to property B so that if A is changed, B gets updated too. As you know, this is not what normally happens; if you define a variable `xCount`, set another variable `yCount` equal to `xCount`, and then change the value of `xCount`, the value of `yCount` does not change. How does Ember allow this to happen?

Suppose for a minute that instead of two variables, we had two objects, with methods `get` and `set`, like this:
```javascript
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
As written, calling `get` and `set` on either object will get/set that own object's `count` property, but not the other object's. What if we wanted to ensure that both properties were always in sync?

One way we might accomplish that would be for each object to update the other any time that its own value is redefined.
```javascript
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
**By wrapping the `count` value in an object, and hiding the actual reading and writing of the `count` variable behind functions, we can define additional functionality to run any time a property might be updated.**

Ember utilizes this idea to great effect using a construct called `Ember.Object`. Here's an example of it in practice.

```javascript
import Ember from 'ember';

var objX = Ember.Object.create({
  count: 5
});

objX.set('count', 10);
objX.get('count');      // 10
```

Essentially, `Ember.Object` is an 'Ember Class' - it defines a bunch of methods (such as `get` and `set`), and provides a sort of constructor (`.create`) for making new objects that can access all of the methods that `Ember.Object` defines. These new objects can also access any other properties that get passed when `.create` is invoked (in this case, `count`).

Let's put aside the topic of value binding for now - we'll touch on it again later. The important thing to understand right now is that value binding is one of the main reasons why `Ember.Object` exists and why so much of how Ember works is built on top of it.

We can define our own custom 'sub-classes' from `Ember.Object` by using another method, called `.extend`. Suppose that we wanted to create a 'Person' Ember Class, with a method called `sayHello` - we might write the following.
```javascript
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

To create a new sub-Class based on the Person (Ember) Class we've just defined, we can simply call `.extend` again. A sub-class can add new methods or even overwrite old ones; when doing the latter, you can reference the method on the parent Class by calling `this._super`.

```javascript
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

The way that Ember uses Ember classes is very similar to how Rails uses Ruby classes. Essentially, Ember provides a number of in-built Ember Classes (e.g. `Ember.Application`, `Ember.Route`, `Ember.Controller`, `Ember.View`, `Ember.Component`), and each time we want to create a new Route/Controller/etc, we sub-class one of those existing classes using the `.extend` method.

For instance, suppose that we want to create a new Route. Ember has a generator, much like Rails and Express, and we can use it to create a new Route by running `ember g route about`. As you can see, Ember has created a couple of new files for us; here's what `app/routes/about.js` looks like:

```javascript
import Ember from 'ember';    // This line makes all predefined Ember code accessible in this file.

export default Ember.Route.extend({
});
```

> `export default` is new ES6 syntax for exporting data from a module - whatever comes after `default` is what gets exported.

As you can see, in this file/module, we will be creating a new Ember Class that sub-classes (i.e. inherits) from `Ember.Route`; if we were to import this file from another file, this new Ember Class is what would get loaded.

### YOUR TURN :: Ember Objects and Classes
If you haven't already, fork and clone this repository. Run `npm install && bower install` to load all of this app's dependencies. Next, start your app with `ember serve`, load up `localhost:4200` in your browser, and open up the inspector to the Console. In the console window, create a new Ember Class called 'Pet' with properties `name` and `age` and methods `eat` and `sleep` (which return, respectively, 'nom nom nom' and 'zzz'). Instantiate that Class by creating a new Pet object with name 'Bruce' and age '9'. From the console, change Bruce's age to 10, and print out Bruce's new age using `console.log`.

Next, create a new Ember Class called 'Dog'; it should have a new method, `bark`, which prints out the text 'arf'. Create a new Dog object with name 'Jellybeans' and age '7'. Call Jellybeans's `bark` method.

When you finish, tilt your laptop screen down.

## Computed Properties
As mentioned earlier, binding properties together is one of Ember's neat features, and `Ember.Object`'s `get` and `set` methods are a big part of making that possible. One common way that binding is implemented is through the use of **computed properties**, another construct that Ember provides.

Consider the following Ember Class:
```javascript
var Person = Ember.Object.extend({
});

var bob = Person.create({
  givenName: 'Bob',
  surname: 'Belcher'
})
```

Suppose we wanted `bob` to have a property called `fullName` which was equal to his given name plus his surname. We could obviously define a function in the Class definition to return that value.
```javascript
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
The nice thing about using a function is that it will automatically recalculate its result based on the latest values of `givenName` and `surname`. However, we must invoke the `fullName` function in order to get that updated value. As it turns out, there are times in Ember when we will not be able to invoke a function directly - one extremely common case is referencing a value from within a template. In those cases, Computed Properties give us a convenient loophole. Here's how `fullName` might look when set up as a computed property.
```javascript
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
As you can see, `fullName` is now accessible as if it were a normal property. The arguments in Ember.computed that come before the function tell Ember to watch the properties `givenName` and `surname` for changes; thus, the Computed Property `fullName` is updated when any of its watched properties is updated, _rather than when the property is accessed with `.get`_.

There are a variety of standard computed properties that Ember provides, all accessible by calling `Ember.computed.___`; here are a few common ones.
* `empty` : return true if the dependent property is `null` or an empty string, array, or function
* `equal` : tests equality
* `gt`/`gte`/`lt`/`lte` : tests inequality (as specified)

If the value of a property is an array, there are even more computed properties available:
* `map`
* `filter`
* `sort`


Speaking of arrays: suppose that you have an array of Ember Objects and want to access a property across all of those object. Ember provides a special key called `@each` that it can use to unpack those properties.

```javascript
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

### YOUR TURN : Computed Properties
Define a new Ember Class and instantiate it. Then, create at least three new properties on your Ember Object. Finally, pick three computed properties from the API docs (or write three of your own) and add them to the new Ember Class you've defined.

## Additional Resources
- [Ember 2.2.0 Guide : The Ember Object Model](http://guides.emberjs.com/v2.2.0/object-model/)
