# Inheritance

Inheritance is a very complex topic in JavaScript, much more so than in any other object-oriented language. Unlike most other OO languages, where a simple keyword will allow you to inherit from a class, JavaScript requires a series of steps in order to pass on public members in the same way. To further complicate the issue, JavaScript is one of the few languages that uses prototypal inheritance (we will show you how this is actually a huge benefit). Because of the flexibility of the language, you can choose to use standard class-based inheritance, or the slightly trickier (but also potentially more efficient) prototypal inheritance.
In this chapter, we look at the techniques that can be used to create subclasses in JavaScript, and the situations where it would be appropriate to use each.

## Why Do You Need Inheritance?

Before we even get into any code, we need to figure out what’s to gain by using inheritance. Generally speaking, you want to design your classes in such a way as to reduce the amount of duplicate code and make your objects as loosely coupled as possible. Inheritance helps with the first of those two design principles, and allows you to build upon existing classes and leverage the methods they already have. It also allows you to make changes more easily. If you require several classes to each have a toString method that outputs the structure of the class in a certain way, you could copy and paste a toString method declaration into each class, but then each time you need to change how the method works, you would have to make the change to each class. If instead you create a ToStringProvider class and make each of the other classes inherit from it, this method would be declared in only one place.
There is the possibility that by making one class inherit from another, you are making them strongly coupled. That is, one class depends on the internal implementation of another. We will look at different ways to prevent that, including using `mixin` classes to provide methods to other classes.

## 1. Classical Inheritance

JavaScript can be made to look like a classically inherited language. By using functions to declare classes, and the new keyword to create instances, objects can behave very similarly to objects in Java or C++. This is a basic class declaration in JavaScript:

```javascript
/* Class Person. */
function Person(name) {
  this.name = name;
}
Person.prototype.getName = function() {
  return this.name;
};
```

First create the constructor. By convention, this should be the name of the class, starting with a capital letter. Within the constructor, use the this keyword to create instance attributes. To create methods, add them to the class’s prototype object, as in Person.prototype.getName. To create an instance of this class, you need only invoke the constructor with the new keyword:

```javascript
var reader = new Person('John Smith');
reader.getName();
```

You can then access all instance attributes and call all instance methods. This is a very simple example of a class in JavaScript.

## The Prototype Chain

To create a class that inherits from Person, it gets a little more complex:

```javascript
/* Class Author. */
function Author(name, books) {
  Person.call(this, name); // Call the superclass's constructor in the scope of this. this.books = books; // Add an attribute to Author.
}
Author.prototype = new Person(); // Set up the prototype chain.
Author.prototype.constructor = Author; // Set the constructor attribute to Author.
Author.prototype.getBooks = function() {
  // Add a method to Author.
  return this.books;
};
```

Setting up one class to inherit from another takes multiple lines of code (unlike the simple extend keyword in most object-oriented languages). First, create a constructor function, as in the previous example. Within that constructor, call the superclass’s constructor, and pass in the name argument. This line deserves a little more explanation. When you use the new operator, several things are done for you. The first is that an empty object is created. The constructor function is then called with this empty object at the front of the scope chain; the this in each constructor function refers to this empty object. So to call the superclass’s constructor within Author, you must do the same thing manually. Person.call(this, name) will invoke the Person constructor with that empty object (which we refer to as this) at the front of the scope chain, while passing in name as an argument.
The next step is to set up the prototype chain. Despite the fact that the code used to do this is fairly simple, it is actually a very complex topic. As mentioned before, JavaScript has no extends keyword; instead, every object has an attribute named prototype. This attribute points to either another object or to null. When a member of an object is accessed (as in reader.getName), JavaScript looks for this member in the prototype object if it does not exist in the current object. If it is not found there, it will continue up the chain, accessing each objects’ prototype until the member is found (or until the prototype is null). This means that in order to make one class inherit from another, you simply need to set the subclasses’s prototype to point to an instance of the superclass. This is completely different from how inheritance works in other languages and can be very confusing and counterintuitive.
In order to have instances of Author inherit from Person, you must manually set Author’s prototype to be an instance of Person. The final step is to set the constructor attribute back to Author (**when you set the prototype attribute to an instance of Person, the constructor attribute is wiped out**).
Despite the fact that setting up this inheritance takes three extra lines, creating an instance of this new subclass is the same as with Person:

```javascript
var author = [];
author[0] = new Author('Dustin Diaz', ['JavaScript Design Patterns']);
author[1] = new Author('Ross Harmes', ['JavaScript Design Patterns']);
author[1].getName();
author[1].getBooks();
```

All of the complexity of classical inheritance lies within the class declaration. Creating new instances is still simple.

## The extend Function

In order to make the class declaration more simple, you can wrap the whole subclassing process in a function, called extend. It will do what the extend keyword does in other languages—create a new object from a given class structure:

```javascript
/* Extend function. */
function extend(subClass, superClass) {
  // to prevent a new and possible large instance of the superclass from having to be instantiated
  var F = function() {};
  F.prototype = superClass.prototype;
  subClass.prototype = new F();
  subClass.prototype.constructor = subClass;
}
```

This function does the same things that you have done manually up to this point. It sets the prototype and then resets the correct constructor. As a bonus, it adds the empty class F into the prototype chain in order **to prevent a new (and possible large) instance of the superclass from having to be instantiated**. This is also beneficial in situations where the superclass’s constructor has side effects or does something that is computationally intensive. Since the object that gets instantiated for the prototype is usually just a throwaway instance, you don’t want to create it unnecessarily.

The previous Person/Author example now looks like this:

```javascript
/* Class Person. */
function Person(name) {
  this.name = name;
}
Person.prototype.getName = function() {
  return this.name;
};
/* Class Author. */
function Author(name, books) {
  Person.call(this, name);
  this.books = books;
}
extend(Author, Person);
Author.prototype.getBooks = function() {
  return this.books;
};
```

Instead of setting the prototype and constructor attributes manually, simply call the extend function immediately after the class declaration (and before you add any methods to the prototype). The only problem with this is that the name of the superclass (Person) is hardcoded within the Author declaration. It would be better to refer to it in a more general way:

```javascript
/* Extend function, improved. */
function extend(subClass, superClass) {
  var F = function() {};
  F.prototype = superClass.prototype;
  subClass.prototype = new F();
  subClass.prototype.constructor = subClass;
  // get access to the superclass
  subClass.superclass = superClass.prototype;
  if (superClass.prototype.constructor == Object.prototype.constructor) {
    superClass.prototype.constructor = superClass;
  }
}
```

This version is a little longer but provides the superclass attribute, which you can now use to make Author less tightly coupled to Person. The first four lines of the function are the same as before. The last three lines simply ensure that the constructor attribute is set correctly on the superclass (even if the superclass is the Object class itself). This will become important when you use this new superclass attribute to call the superclass’s constructor:

```javascript
/* Class Author. */
function Author(name, books) {
  Author.superclass.constructor.call(this, name);
  this.books = books;
}
extend(Author, Person);
Author.prototype.getBooks = function() {
  return this.books;
};
```

Adding the superclass attribute also allows you to call methods directly from the superclass. This is useful if you want to override a method while still having access to the superclass’s implementation of it. For instance, to override Person’s implementation of getName with a new version, you could use Author.superclass.getName to first get the original name and then add to it:

```javascript
Author.prototype.getName = function() {
  var name = Author.superclass.getName.call(this);
  return name + ', Author of ' + this.getBooks().join(', ');
};
```

## 2. Prototypal Inheritance

Prototypal inheritance is a very different beast. We’ve found the best way to think about it is to forget everything you know about classes and instances, and think only in terms of objects. The classical approach to creating an object is to (a) define the structure of the object, using a class declaration, and (b) instantiate that class to create a new object. Objects created in this manner have their own copies of all instance attributes, plus a link to the single copy of each of the instance methods.
In prototypal inheritance, instead of defining the structure through a class, you simply create an object. This object then gets reused by new objects, thanks to the way that prototype chain lookups work. It is called the `prototype object` because it provides a prototype for what the other objects should look like (in order to prevent confusion with the other prototype object, it will appear in italics). It is where prototypal inheritance gets its name.
We will now recreate Person and Author using prototypal inheritance:

```javascript
/* Person Prototype Object. */
var Person = {
  name: 'default name',
  getName: function() {
    return this.name;
  }
};
```

Instead of using a constructor function named Person to define the class structure, Person is now an object literal. It is the prototype object for any other Person-like objects that you want to create. Define all attributes and methods you want these objects to have, and give them default values. For the methods, those default values will probably not be changed; for attributes, they almost certainly will be:

```javascript
var reader = clone(Person);
alert(reader.getName()); // This will output 'default name'.
reader.name = 'John Smith';
alert(reader.getName()); // This will now output 'John Smith'.
```

To create a new Person-like object, use the clone function (we go into more detail about this function later in the section “The clone Function”). This provides an empty object with the prototype attribute set to the prototype object. This means that if any method or attribute lookup on this object fails, that lookup will instead look to the prototype object.
To create Author, you don’t make a subclass of Person. Instead you make a clone:

```javascript
/* Author Prototype Object. */
var Author = clone(Person);
Author.books = []; // Default value.
Author.getBooks = function() {
  return this.books;
};
```

The methods and attributes of this clone can then be overridden. You can change the default values given by Person, or you can add new attributes and methods. That creates a new prototype object, which you can then clone to create new Author-like objects:

```javascript
var author = [];
author[0] = clone(Author);
author[0].name = 'Dustin Diaz';
author[0].books = ['JavaScript Design Patterns'];
author[1] = clone(Author);
author[1].name = 'Ross Harmes';
author[1].books = ['JavaScript Design Patterns'];
author[1].getName();
author[1].getBooks();
```

## Asymmetrical Reading and Writing of Inherited Members

We mentioned before that in order to use prototypal inheritance effectively, you must forget everything you know about classical inheritance. Here is one example of that. In classical inheritance, each instance of Author has its own copy of the books array. You could add to it by writing author[1].books.push('New Book Title'). That is not initially possible with the object you created using prototypal inheritance because of the way prototype chaining works. A clone is not a fully independent copy of its prototype object; it is a new empty object with its prototype attribute set to the prototype object. When it is just created, author[1].name is actually a link back to the primitive Person.name. This is because of the asymmetry inherent in reading and writing objects linked from the prototype. When you read the value of author[1].name, you are getting the value linked from the prototype, provided you haven’t defined the name attribute directly on the author[1] instance yet. When you write to author[1].name, you are defining a new attribute directly on the author[1] object.
This example illustrates that asymmetry:

```javascript
var authorClone = clone(Author);
alert(authorClone.name); // Linked to the primative Person.name, which is the string 'default name'.
authorClone.name = 'new name'; // A new primative is created and added to the authorClone object itself.
alert(authorClone.name); // Now linked to the primative authorClone.name, which is the string 'new name'.
authorClone.books.push('new book'); // authorClone.books is linked to the array Author.books. We just modified the prototype object's default value, and all other objects that link to it will now have a new default value there.
authorClone.books = []; // A new array is created and added to the authorClone object itself.
authorClone.books.push('new book'); // We are now modifying that new array.
```

This also illustrates why you must create new copies of data types that are passed by reference. In the previous example, pushing a new value onto the authorClone.books array is actually pushing it to Author.books. This is bad because you just modified the value not only for Author but for any object inheriting from Author that has not yet overwritten the default. You must create new copies of all arrays and objects before you start changing their members. It is very easy to forget this and modify the value of the prototype object. This should be avoided at all costs; debugging these types of errors can be very time-consuming. In these situations, you can use the hasOwnProperty method to distinguish between inherited members and the object’s actual members.

Sometimes prototype objects will have child objects within them. If you want to override a single value within that child object, you have to recreate the entire thing. This can be done by setting the child object to be an empty object literal and then recreating it, but that would mean that the cloned object would have to know the exact structure and defaults for each child object. In order to keep all objects as loosely coupled as possible, any complex child objects should be created using methods:

```javascript
var CompoundObject = {
  string1: 'default value',
  childObject: {
    bool: true,
    num: 10
  }
};
var compoundObjectClone = clone(CompoundObject);
// Bad! Changes the value of CompoundObject.childObject.num.
compoundObjectClone.childObject.num = 5;
// Better. Creates a new object, but compoundObject must know the structure
// of that object, and the defaults. This makes CompoundObject and
// compoundObjectClone tightly coupled.
compoundObjectClone.childObject = {
  bool: true,
  num: 5
};
```

In this example, childObject is recreated and compoundObjectClone.childObject.num is modified. The problem is that compoundObjectClone must know that childObject has two attributes, with values true and 10. A better approach is to have a factory method that creates the childObject:

```javascript
// Best approach. Uses a method to create a new object, with the same structure and
// defaults as the original.
var CompoundObject = {};
CompoundObject.string1 = 'default value';
CompoundObject.createChildObject = function() {
  return {
    bool: true,
    num: 10
  };
};
CompoundObject.childObject = CompoundObject.createChildObject();
var compoundObjectClone = clone(CompoundObject);
compoundObjectClone.childObject = CompoundObject.createChildObject();
compoundObjectClone.childObject.num = 5;
```

> liuchao: i will use deep clone instead

## The clone Function

So what is the amazing function that creates these cloned objects?

```javascript
/* Clone function. */
function clone(object) {
  function F() {}
  F.prototype = object;
  return new F();
}
```

First the clone function creates a new and empty function, F. It then sets the prototype attribute of F to the prototype object. You can see here the intent of the original JavaScript creators. The prototype attribute is meant to point to the prototype object, and through prototype chaining it provides links to all the inherited members. Lastly, the function creates a new object by calling the new operator on F. The cloned object that is returned is completely empty, except for the prototype attribute, which is (indirectly) pointing to the prototype object, by way of the F object.

## Comparing Classical and Prototypal Inheritance

The classical and prototypal paradigms for creating new objects are very different from each other, and the objects that each one produces behave differently. Each paradigm has its own pros and cons, which should help you determine which one to use in a given situation.

Classical inheritance is well understood, both in JavaScript and the programmer community in general. Almost all object-oriented code written in JavaScript uses this paradigm. If you are creating an API for widespread use, or if there is the possibility that other programmers not familiar with prototypal inheritance will be working on your code, `it is best to go with classical`. JavaScript is the only popular, widely used language that uses prototypal inheritance, so odds are most people will never have used it before. It can also be confusing to have an object with links back to its prototype object. Programmers who don’t fully understand prototypal inheritance will think of this as some sort of reverse inheritance, where the parent inherits from its children. Even though this isn’t the case, it can still be a very confusing topic. But since this form of classical inheritance is only imitating true class-based inheritance, advanced JavaScript programmers need to understand how prototypal inheritance truly works at some point anyway. Some would argue that hiding this fact does more harm than good.

Prototypal inheritance is very memory-efficient. Because of the way prototype chain reads members, all cloned objects share a single copy of each attribute and method, until those attributes and methods are written to directly on the cloned object. Contrast this with the objects created using classical inheritance, where each object has a copy of every attribute (and private method) in memory. The savings here are enormous. It also seems to be a much more elegant approach, needing only a single clone function, rather than several lines of incomprehensible syntax such as SuperClass.call(this, arg)andSubClass.prototype = new SuperClass for each class you want to extend (it is true, however, that some of these lines can, in turn, be condensed into the extend function). Don’t think that just because prototypal inheritance is simple that it isn’t also complex. Its power lies in its simplicity.

The decision to use classical or prototypal inheritance probably depends most on how well you like each paradigm. Some people seem naturally drawn to the simplicity of prototypal inheritance, while others are much more comfortable in the more familiar classical. Both paradigms can be used for each pattern described in this book. We tend toward classical inheritance for the later patterns, to make them easier to understand, but both can be used interchangeably throughout this book.

## Inheritance and Encapsulation

Up to this point in the chapter there has been little mention of how encapsulation affects inheritance. When you create a subclass from an existing class, only the public and privileged members are passed on. This is similar to other object-oriented languages. In Java, for instance, no private methods are accessible in subclasses; you have to explicitly define a method to be protected in order to pass it on to the subclasses.

It is because of this that fully exposed classes are the best candidates for subclassing. All of the members are public and will be passed on to the subclasses. If a member needs to be shielded a bit, the underscore convention can always be used.

If a class with true private members is subclassed, the privileged methods will be passed on, since they are publicly accessible. These will allow access to the private attributes indirectly, but none of the subclass’s instance methods will have direct access to these private attributes. Private members can only be accessed through these already established privileged methods; new ones cannot be added in the subclass.

## Mixin Classes

There is a way to reuse code without using strict inheritance. If you have a function that you wish to use in more than one class, you can share it among multiple classes through augmentation. In practice, it goes something like this: you create a class that contains your general-purpose methods, and then use it to augment other classes. These classes with the general-purpose methods are called `mixin classes`. They are usually not instantiated or called directly. They exist only to pass on their methods to other classes. This is best illustrated with an example:

```javascript
/* Mixin class. */
var Mixin = function() {};
Mixin.prototype = {
  serialize: function() {
    var output = [];
    for (key in this) {
      output.push(key + ': ' + this[key]);
    }
    return output.join(', ');
  }
};
```

The class Mixin has a single method, serialize. This method walks through each member in this and outputs it as a string. (This is only a simple example; a more robust version of this sort of function can be found in the toJSONString method, part of Douglas Crockford’s JSON library at http://json.org/json.js.) This sort of method could potentially be useful in many different types of classes, but it doesn’t make sense to have each of these classes inherit from Mixin. Similarly, duplicating the code in each class doesn’t make much sense either. The best approach is to use the augment function to add this method to each class that needs it:

```javascript
augment(Author, Mixin);
var author = new Author('Ross Harmes', ['JavaScript Design Patterns']);
var serializedString = author.serialize();
```

Here we augment the Author class with all of the methods from the Mixin class. Instances of Author can now call serialize. This can be thought of as a way to implement multiple inheritance in JavaScript. Languages such as C++ and Python allow subclasses to inherit from more than one superclass; you cannot do that in JavaScript because the prototype attribute can only point to one object. But a class can be augmented by more than one mixin class, which effectively provides the same functionality.
The augment function is fairly simple. Using a for..in loop, walk through each of the members of the giving class’s prototype and add them to the receiving class’s prototype. If the member already exists, skip it. Nothing gets overwritten in the receiving class:

```javascript
* Augment function. */
function augment(receivingClass, givingClass) {
  for(methodName in givingClass.prototype) {
    if(!receivingClass.prototype[methodName]) {
      receivingClass.prototype[methodName] = givingClass.prototype[methodName];
    }
  }
}
```

We can improve on this slightly. Let’s say you have a mixin class containing several methods but only want to copy one or two of them over to another class. With the version of augment given previously, that would be impossible. This new version looks for optional arguments, and if they exist, only copies methods with names matching those arguments:

```javascript
/* Augment function, improved. */
function augment(receivingClass, givingClass) {
  if (arguments[2]) {
    // Only give certain methods.
    for (var i = 2, len = arguments.length; i < len; i++) {
      receivingClass.prototype[arguments[i]] =
        givingClass.prototype[arguments[i]];
    }
  } else {
    // Give all methods.
    for (methodName in givingClass.prototype) {
      if (!receivingClass.prototype[methodName]) {
        receivingClass.prototype[methodName] =
          givingClass.prototype[methodName];
      }
    }
  }
}
```

You can now write augment(Author, Mixin, 'serialize'); to only augment Author with the single serialize method. More method names can be added if you want to augment with more than one method.
Often it makes more sense to augment a class with a few methods than it does to make one class inherit from another. This is a lightweight way to prevent code duplication. Unfortunately, there aren’t many situations where it can be used. Only methods general enough to be used in very dissimilar classes make good candidates for sharing (if the classes aren’t that disimilar, normal inheritance is often a better choice).

# Example: Edit-in-Place

We will take you through this example three times, once each using classical inheritance, prototypal inheritance, and mixin classes. For this example, imagine that you have been given a task: write a modular, reusable API for creating and managing edit-in-place fields (edit-in-place refers to a normal block of text in a web page that when clicked turns into a form field and several buttons that allow that block of text to be edited). It should allow you to assign a unique ID to the object, give it a default value, and specify where in the page you want it to go. It should also let you access the current value of the field at any time and have a couple of different options for the type of editing field used (e.g., a text area or an input text field).

## Using Classical Inheritance

First we will create an API using classical inheritance:

```javascript
/* EditInPlaceField class. */
function EditInPlaceField(id, parent, value) {
  this.id = id;
  this.value = value || 'default value';
  this.parentElement = parent;
  this.createElements(this.id);
  this.attachEvents();
}
EditInPlaceField.prototype = {
  createElements: function(id) {
    this.containerElement = document.createElement('div');
    this.parentElement.appendChild(this.containerElement);
    this.staticElement = document.createElement('span');
    this.containerElement.appendChild(this.staticElement);
    this.staticElement.innerHTML = this.value;
    this.fieldElement = document.createElement('input');
    this.fieldElement.type = 'text';
    this.fieldElement.value = this.value;
    this.containerElement.appendChild(this.fieldElement);
    this.saveButton = document.createElement('input');
    this.saveButton.type = 'button';
    this.saveButton.value = 'Save';
    this.containerElement.appendChild(this.saveButton);
    this.cancelButton = document.createElement('input');
    this.cancelButton.type = 'button';
    this.cancelButton.value = 'Cancel';
    this.containerElement.appendChild(this.cancelButton);
    this.convertToText();
  },
  attachEvents: function() {
    var that = this;
    addEvent(this.staticElement, 'click', function() {
      that.convertToEditable();
    });
    addEvent(this.saveButton, 'click', function() {
      that.save();
    });
    addEvent(this.cancelButton, 'click', function() {
      that.cancel();
    });
  },
  convertToEditable: function() {
    this.staticElement.style.display = 'none';
    this.fieldElement.style.display = 'inline';
    this.saveButton.style.display = 'inline';
    this.cancelButton.style.display = 'inline';
    this.setValue(this.value);
  },
  save: function() {
    this.value = this.getValue();
    var that = this;
    var callback = {
      success: function() {
        that.convertToText();
      },
      failure: function() {
        alert('Error saving value.');
      }
    };
    ajaxRequest(
      'GET',
      'save.php?id=' + this.id + '&value=' + this.value,
      callback
    );
  },
  cancel: function() {
    this.convertToText();
  },
  convertToText: function() {
    this.fieldElement.style.display = 'none';
    this.saveButton.style.display = 'none';
    this.cancelButton.style.display = 'none';
    this.staticElement.style.display = 'inline';
    this.setValue(this.value);
  },
  setValue: function(value) {
    this.fieldElement.value = value;
    this.staticElement.innerHTML = value;
  },
  getValue: function() {
    return this.fieldElement.value;
  }
};
```

To create a field, instantiate the class:

```javascript
var titleClassical = new EditInPlaceField(
  'titleClassical',
  $('doc'),
  'Title Here'
);
var currentTitleText = titleClassical.getValue();
```

This gives an instance of the EditInPlaceField class (which will be subclassed later), with the text displayed in a span tag and a text input field used as the editing area. It has a couple of configuration methods (createElements, attachEvents), a few internal methods for converting and saving (convertToEditable, save, cancel, convertToText), and an accessor and mutator pair (getValue, setvalue). If this were to be used as production code, it would be a good idea to give each of the HTML elements specific class names so that they can be styled with CSS; for the sake of simplicity, we don’t include these lines of code.
Next, create a class that will use a text area instead of a text input. For the most part the EditInPlaceField and EditInPlaceArea classes are identical, so create one as a subclass of the other in order to prevent code duplication:

```javascript
/* EditInPlaceArea class. */
function EditInPlaceArea(id, parent, value) {
  EditInPlaceArea.superclass.constructor.call(this, id, parent, value);
}
extend(EditInPlaceArea, EditInPlaceField);
// Override certain methods.
EditInPlaceArea.prototype.createElements = function(id) {
  this.containerElement = document.createElement('div');
  this.parentElement.appendChild(this.containerElement);
  this.staticElement = document.createElement('p');
  this.containerElement.appendChild(this.staticElement);
  this.staticElement.innerHTML = this.value;
  this.fieldElement = document.createElement('textarea');
  this.fieldElement.value = this.value;
  this.containerElement.appendChild(this.fieldElement);
  this.saveButton = document.createElement('input');
  this.saveButton.type = 'button';
  this.saveButton.value = 'Save';
  this.containerElement.appendChild(this.saveButton);
  this.cancelButton = document.createElement('input');
  this.cancelButton.type = 'button';
  this.cancelButton.value = 'Cancel';
  this.containerElement.appendChild(this.cancelButton);
  this.convertToText();
};
EditInPlaceArea.prototype.convertToEditable = function() {
  this.staticElement.style.display = 'none';
  this.fieldElement.style.display = 'block';
  this.saveButton.style.display = 'inline';
  this.cancelButton.style.display = 'inline';
  this.setValue(this.value);
};
EditInPlaceArea.prototype.convertToText = function() {
  this.fieldElement.style.display = 'none';
  this.saveButton.style.display = 'none';
  this.cancelButton.style.display = 'none';
  this.staticElement.style.display = 'block';
  this.setValue(this.value);
};
```

You create the subclass using the extend function and then override a few methods to implement the changes. This new class uses a text area instead of a text input, and a paragraph tag instead of a span.
Classical inheritance seems like an ideal technique to use in this case. Subclassing the EditInPlaceField class is trivial, requiring only a few lines of code. Making changes to the class is as simple as overriding or adding methods on the prototype. We could link the field to another output by creating another subclass and overriding the save method. Since the changes between classes are small, strict inheritance like this is ideal.

## Using Prototypal Inheritance

Despite the fact that classical and prototypal inheritance are fundamentally different, repeating the exercise using prototypal inheritance really shows how similar the end code can be between the two:

```javascript
/* EditInPlaceField object. */
var EditInPlaceField = {
  configure: function(id, parent, value) {
    this.id = id;
    this.value = value || 'default value';
    this.parentElement = parent;
    this.createElements(this.id);
    this.attachEvents();
  },
  createElements: function(id) {
    this.containerElement = document.createElement('div');
    this.parentElement.appendChild(this.containerElement);
    this.staticElement = document.createElement('span');
    this.containerElement.appendChild(this.staticElement);
    this.staticElement.innerHTML = this.value;
    this.fieldElement = document.createElement('input');
    this.fieldElement.type = 'text';
    this.fieldElement.value = this.value;
    this.containerElement.appendChild(this.fieldElement);
    this.saveButton = document.createElement('input');
    this.saveButton.type = 'button';
    this.saveButton.value = 'Save';
    this.containerElement.appendChild(this.saveButton);
    this.cancelButton = document.createElement('input');
    this.cancelButton.type = 'button';
    this.cancelButton.value = 'Cancel';
    this.containerElement.appendChild(this.cancelButton);
    this.convertToText();
  },
  attachEvents: function() {
    var that = this;
    addEvent(this.staticElement, 'click', function() {
      that.convertToEditable();
    });
    addEvent(this.saveButton, 'click', function() {
      that.save();
    });
    addEvent(this.cancelButton, 'click', function() {
      that.cancel();
    });
  },
  convertToEditable: function() {
    this.staticElement.style.display = 'none';
    this.fieldElement.style.display = 'inline';
    this.saveButton.style.display = 'inline';
    this.cancelButton.style.display = 'inline';
    this.setValue(this.value);
  },
  save: function() {
    this.value = this.getValue();
    var that = this;
    var callback = {
      success: function() {
        that.convertToText();
      },
      failure: function() {
        alert('Error saving value.');
      }
    };
    ajaxRequest(
      'GET',
      'save.php?id=' + this.id + '&value=' + this.value,
      callback
    );
  },
  cancel: function() {
    this.convertToText();
  },
  convertToText: function() {
    this.fieldElement.style.display = 'none';
    this.saveButton.style.display = 'none';
    this.cancelButton.style.display = 'none';
    this.staticElement.style.display = 'inline';
    this.setValue(this.value);
  },
  setValue: function(value) {
    this.fieldElement.value = value;
    this.staticElement.innerHTML = value;
  },
  getValue: function() {
    return this.fieldElement.value;
  }
};
```

Instead of a class, there is now an object. Prototypal inheritance doesn’t use constructors, so you move that code into a configure method instead. Other than that, the code is almost identical to the first example. Creating new objects from this EditInPlaceField prototype object looks very different from instantiating a class:

```javascript
var titlePrototypal = clone(EditInPlaceField);
titlePrototypal.configure(' titlePrototypal ', $('doc'), 'Title Here');
var currentTitleText = titlePrototypal.getValue();
```

Instead of using the new operator, use the clone function to create a copy. Then configure that copy. At this point you can interact with the object titlePrototypal in the same way as you would with the previous titleClassical object. The two objects are almost indistinguishable and can be managed using the same API.
Creating a child object from this one also uses the clone function:

```javascript
/* EditInPlaceArea object. */
var EditInPlaceArea = clone(EditInPlaceField);
// Override certain methods.
EditInPlaceArea.createElements = function(id) {
  this.containerElement = document.createElement('div');
  this.parentElement.appendChild(this.containerElement);
  this.staticElement = document.createElement('p');
  this.containerElement.appendChild(this.staticElement);
  this.staticElement.innerHTML = this.value;
  this.fieldElement = document.createElement('textarea');
  this.fieldElement.value = this.value;
  this.containerElement.appendChild(this.fieldElement);
  this.saveButton = document.createElement('input');
  this.saveButton.type = 'button';
  this.saveButton.value = 'Save';
  this.containerElement.appendChild(this.saveButton);
  this.cancelButton = document.createElement('input');
  this.cancelButton.type = 'button';
  this.cancelButton.value = 'Cancel';
  this.containerElement.appendChild(this.cancelButton);
  this.convertToText();
};
EditInPlaceArea.convertToEditable = function() {
  this.staticElement.style.display = 'none';
  this.fieldElement.style.display = 'block';
  this.saveButton.style.display = 'inline';
  this.cancelButton.style.display = 'inline';
  this.setValue(this.value);
};
EditInPlaceArea.convertToText = function() {
  this.fieldElement.style.display = 'none';
  this.saveButton.style.display = 'none';
  this.cancelButton.style.display = 'none';
  this.staticElement.style.display = 'block';
  this.setValue(this.value);
};
```

You simply create a copy of the EditInPlaceField object, and then overwrite some of the methods. This prototype object can be used and cloned in the same way as the first one can. In fact, new prototype objects can be created in the same way, by cloning this one and making a few changes.
Prototypal inheritance also seems ideal for this example, for the same reasons that classical inheritance worked so well. The only differences between the two are the way the class/object is set up, and the way a new sub-object/instance is created. Most of the code (including all of the methods) is completely unchanged. This illustrates how easily you can convert from one paradigm to the other. It isn’t always this easy, especially with classes and objects that make extensive use of arrays or objects as members, but for the most part you need only modify
a bit of the syntax.
Using prototypal inheritance in this example doesn’t really provide anything over classical inheritance. The objects do not use many default values, so you aren’t really saving any memory. Personally, we would have a hard time picking one paradigm over the other in this example; both work equally well.

## Using Mixin Classes

We will repeat the example one more time using mixin classes. We will create one mixin class with all of the methods we want to share. Then we will create a new class and use augment to share those methods:

```javascript
/* Mixin class for the edit-in-place methods. */
var EditInPlaceMixin = function() {};
EditInPlaceMixin.prototype = {
  createElements: function(id) {
    this.containerElement = document.createElement('div');
    this.parentElement.appendChild(this.containerElement);
    this.staticElement = document.createElement('span');
    this.containerElement.appendChild(this.staticElement);
    this.staticElement.innerHTML = this.value;
    this.fieldElement = document.createElement('input');
    this.fieldElement.type = 'text';
    this.fieldElement.value = this.value;
    this.containerElement.appendChild(this.fieldElement);
    this.saveButton = document.createElement('input');
    this.saveButton.type = 'button';
    this.saveButton.value = 'Save';
    this.containerElement.appendChild(this.saveButton);
    this.cancelButton = document.createElement('input');
    this.cancelButton.type = 'button';
    this.cancelButton.value = 'Cancel';
    this.containerElement.appendChild(this.cancelButton);
    this.convertToText();
  },
  attachEvents: function() {
    var that = this;
    addEvent(this.staticElement, 'click', function() {
      that.convertToEditable();
    });
    addEvent(this.saveButton, 'click', function() {
      that.save();
    });
    addEvent(this.cancelButton, 'click', function() {
      that.cancel();
    });
  },
  convertToEditable: function() {
    this.staticElement.style.display = 'none';
    this.fieldElement.style.display = 'inline';
    this.saveButton.style.display = 'inline';
    this.cancelButton.style.display = 'inline';
    this.setValue(this.value);
  },
  save: function() {
    this.value = this.getValue();
    var that = this;
    var callback = {
      success: function() {
        that.convertToText();
      },
      failure: function() {
        alert('Error saving value.');
      }
    };
    ajaxRequest(
      'GET',
      'save.php?id=' + this.id + '&value=' + this.value,
      callback
    );
  },
  cancel: function() {
    this.convertToText();
  },
  convertToText: function() {
    this.fieldElement.style.display = 'none';
    this.saveButton.style.display = 'none';
    this.cancelButton.style.display = 'none';
    this.staticElement.style.display = 'inline';
    this.setValue(this.value);
  },
  setValue: function(value) {
    this.fieldElement.value = value;
    this.staticElement.innerHTML = value;
  },
  getValue: function() {
    return this.fieldElement.value;
  }
};
```

The mixin class holds nothing but the methods. To create a functional class, make a con- structor and then call augment:

```javascript
/* EditInPlaceField class. */
function EditInPlaceField(id, parent, value) {
  this.id = id;
  this.value = value || 'default value';
  this.parentElement = parent;
  this.createElements(this.id);
  this.attachEvents();
}
augment(EditInPlaceField, EditInPlaceMixin);
```

You can now instantiate the class in the exact same way as with classical inheritance. To create the class that uses a text area field, you will not subclass EditInPlaceField. Instead, simply create a new class (with a constructor) and augment it from the same mixin class. But before augmenting it, define a few methods. Since these are in place before augmenting it, they will not get overridden:

```javascript
/* EditInPlaceArea class. */
function EditInPlaceArea(id, parent, value) {
  this.id = id;
  this.value = value || 'default value';
  this.parentElement = parent;
  this.createElements(this.id);
  this.attachEvents();
}
// Add certain methods so that augment won't include them.
EditInPlaceArea.prototype.createElements = function(id) {
  this.containerElement = document.createElement('div');
  this.parentElement.appendChild(this.containerElement);
  this.staticElement = document.createElement('p');
  this.containerElement.appendChild(this.staticElement);
  this.staticElement.innerHTML = this.value;
  this.fieldElement = document.createElement('textarea');
  this.fieldElement.value = this.value;
  this.containerElement.appendChild(this.fieldElement);
  this.saveButton = document.createElement('input');
  this.saveButton.type = 'button';
  this.saveButton.value = 'Save';
  this.containerElement.appendChild(this.saveButton);
  this.cancelButton = document.createElement('input');
  this.cancelButton.type = 'button';
  this.cancelButton.value = 'Cancel';
  this.containerElement.appendChild(this.cancelButton);
  this.convertToText();
};
EditInPlaceArea.prototype.convertToEditable = function() {
  this.staticElement.style.display = 'none';
  this.fieldElement.style.display = 'block';
  this.saveButton.style.display = 'inline';
  this.cancelButton.style.display = 'inline';
  this.setValue(this.value);
};
EditInPlaceArea.prototype.convertToText = function() {
  this.fieldElement.style.display = 'none';
  this.saveButton.style.display = 'none';
  this.cancelButton.style.display = 'none';
  this.staticElement.style.display = 'block';
  this.setValue(this.value);
};
augment(EditInPlaceArea, EditInPlaceMixin);
```

The mixin technique works in this example, but not as well as the other two techniques. In the end, the objects created by each of the techniques are almost identical, but from an organizational standpoint, strict inheritance makes more sense than augmentation. Mixin classes work well for methods that are shared between several disparate classes, but in this example, the mixin class is used to provide all of the methods, for two very similar classes. Code maintenance would be easier with the first two examples because it is immediately obvi- ous where the methods came from and how the classes and objects were organized.
**Sharing general-purpose methods that can act on all types of objects is a much better use of mixin classes.** Some examples of this are methods that serialize an object to a string representation, or output its state for debugging. It is also possible to use mixin classes to emulate enumerations or iterators, as found in some other object-oriented languages.

# When Should Inheritance Be Used?

Inheritance adds some complexity to your code and makes it harder for JavaScript novices to understand what it does, so it should only be used in situations where its benefits outweigh these drawbacks. Most of the benefits have to do with code reuse. By having classes or objects inherit from each other, you only have to define a given method once. By the same token, if you ever have to make changes to this method or track down errors in it, the fact that it is defined in a single location can save you a great deal of time and effort.
Each paradigm also has its own pros and cons. Prototypal inheritance (with the clone func- tion) is best used in situations where memory efficiency is important. Classical inheritance (with the extend function) is best used when the programmers dealing with the objects are familiar with how inheritance works in other object-oriented languages. Both of these methods are well-suited to class hierarchies where the differences between each class are slight. If the classes are very different from each other, it usually makes more sense to augment them with methods from mixin classes.
You will find that simpler JavaScript programs rarely require this level of abstraction. It is only with large projects, with multiple programmers involved, that this sort of organization becomes necessary.

# Summary

In this chapter we discussed the pros and cons of inheritance, as well as three ways of making one class or object inherit from another. Classical inheritance tries to emulate the way that classes inherit from each other in other object-oriented languages such as C++ and Java. It is best suited to situations where memory efficiency isn’t an issue or the programmers are not familiar with the much less well-known prototypal inheritance. Using the extend function, you can eliminate most of the confusion surrounding subclassing.
Prototypal inheritance works by creating objects and then cloning them to create the equivalent of subclasses and instances. It is very easy to use once you understand the underly- ing principles, and the objects that it creates tend to be very memory efficient, due to the fact that attributes and methods are shared until they are overwritten. There can be some confusion surrounding cloned objects that contain arrays or objects as attributes, but using a method to set default values for these attributes can work around this problem. The clone function takes care of all of the steps involved in creating a cloned object.
Mixin classes provide a way to have objects and classes share methods without being in a parent-child relationship. It should be used where you have general-purpose methods that you want to share among several dissimilar classes. It is possible to share all of the methods in a mixin class, or just a few of them, using the augment function.
Using these three techniques, it is possible to create complex object hierarchies in a manner that would rival any other object-oriented language in its elegance. Inheritance in JavaScript is not obvious or intuitive to the novice programmer. It is an advanced technique that benefits from a low-level study of the language. But it can be made more simple and usable through several convenience functions, and it is ideal for creating APIs for other programmers to use.
