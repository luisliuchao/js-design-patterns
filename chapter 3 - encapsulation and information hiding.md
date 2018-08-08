# Chapter 3 Encapsulation and Information Hiding

Creating objects with private members is one of the most basic and useful features of any object-oriented language. Declaring a method or attribute as private allows you to shield your implementation details from other objects and promotes a loose coupling between them. It allows you to maintain the integrity of your data and impose constraints on how it can be modified. It also makes your code more reliable and easier to debug in environments where many people are working on the same code base. In short, encapsulation is a cornerstone of object-oriented design.
Despite the fact that JavaScript is an object-oriented language, it does not have any built- in mechanisms for declaring members to be public or private. As in the previous chapter on interfaces, we will create our own way to implement this feature. There are several established patterns for creating objects with public, private, and privileged methods, each with its own strengths and weaknesses. We also take a look at the situations where using complex encapsu- lated objects can benefit the JavaScript programmer.

## The Information Hiding Principle

Let’s use an example to illustrate the information hiding principle. Every evening, you receive a report from a coworker outlining the day’s revenues. This is a well-defined interface; you request the information, and your coworker finds the raw data, calculates the revenue, and reports back to you. If either you or your coworker moves to another company, that interface will remain, ensuring that it is easy for your replacement to request information the same way.
One day you decide that you want to receive this information more frequently than your coworker is willing to give it to you. You find out where the raw data is stored, retrieve it your- self, and perform the calculations. Everything works fine until the format of the data changes. Instead of a file of comma-separated values, it is now formatted in XML. Also, the calculations can change depending on accounting and tax laws, which you have no expertise in. If you quit, you must first train your replacement to perform these same tasks, which are much more complex than just requesting the end calculation from your coworker.
You have become dependent on the internal implementation; when that implementation changes, you must relearn the entire system and start again. In object-oriented design terms, you have become tightly coupled to the raw data. The information hiding principle serves to reduce the interdependency of two actors in a system. It states that all information between two actors should be obtained through well-defined channels. In this case, these channels are the interfaces of your objects.

## Encapsulation vs. Information Hiding

How is encapsulation related to information hiding? You can think of it as two ways of refer- ring to the same idea. Information hiding is the goal, and encapsulation is the technique you use to accomplish that goal. This chapter deals mainly with concrete examples of encapsula- tion in JavaScript.
Encapsulation can be defined as the hiding of internal data representation and imple- mentation details in an object. The only way to access the data within an encapsulated object is to use defined operations. By using encapsulation, you are enforcing information hiding. Many object-oriented languages use keywords to specify that methods and attributes should be hidden. In Java, for instance, adding the private keyword to a method will ensure that only code within the object can execute it. There is no such keyword in JavaScript; we will instead use the concept of the `closure` to create methods and attributes that can only be accessed from within the object. It is more complicated (and confusing) than just using keywords, but the same end result can be achieved.

## The Role of the Interface

How does the interface help you hide information from other objects? It provides a contract that documents the publicly accessible methods. It defines the relationship that two objects can have; either object in this relationship can be replaced as long as the interface is main- tained. It isn’t always necessary to use a strict interface, like the one we defined in Chapter 2, but most of the time you will find it very helpful to have the available methods documented. Even with a known interface in place, it is important to not expose methods that are not defined in that interface. Conversely, it can be dangerous for other objects to rely on methods that are not part of the interface. They may change or be removed at any point, causing the whole system to fail.
The ideal software system will define interfaces for all classes. Those classes will provide only the methods defined in their interfaces; any other method will be kept private. All attributes will be private and only accessible through accessor and mutator operations defined in the interface. Rarely in the real world does a system have all of these characteristics. Good code should aim toward them whenever possible, but not at the cost of complicating a simple project that doesn’t really need them.

## Basic Patterns

In this section we look at examples of the various ways an object can be created and the features available in each. There are three basic patterns that can be used to create objects. The `fully exposed object` is the simplest but provides only public members. The next pattern improves upon this by `using underscores` to denote methods and attributes that are intended to be private. The third basic pattern `uses closures` to create true private members, which can only be accessed through the use of privileged methods.

We will use the Book class as our example. You are given this assignment: create a class to store data about a book, and implement a method for displaying the book’s data in HTML. You will only be creating the class; other programmers will be instantiating it. Here is an example of how it will be used:

```javascript
// Book(isbn, title, author)
var theHobbit = new Book('0-395-07122-4', 'The Hobbit', 'J. R. R. Tolkien');

theHobbit.display(); // Outputs the data by creating and populating an HTML element.
```

**1. Fully Exposed Object**

The easiest way to implement Book is to create a class in the conventional way, using a function as a constructor. We call this the fully exposed object because all of the class’s attributes and methods are public and accessible. The public attributes are created using the this keyword:

```javascript
var Book = function(isbn, title, author) {
  if (isbn == undefined) throw new Error('Book constructor requires an isbn.');
  this.isbn = isbn;
  this.title = title || 'No title specified';
  this.author = author || 'No author specified';
};

Book.prototype.display = function() {
  ...
};
```

The display method depends entirely on having an accurate ISBN. Without this, you can’t fetch the image or provide a link to buy the book. Because of this, an error is thrown in the constructor if an ISBN is not given. The title and author attributes are both optional, so you provide defaults if they are not given. The Boolean OR operator, ||, can be used here to provide fallback values. If a title or author is given, the left side will evaluate to true and will be returned. If a title or author is not given, the left side of the operator will evaluate to false, and the right side will be returned instead.
At first glance, this class seems to meet every need. The biggest outstanding problem is that you can’t verify the integrity of the ISBN data, which may cause your display method to fail. This breaks the contract you have with the other programmers. If the Book object doesn’t throw any errors, the display method should work, but without integrity checks, it won’t. To fix this problem, you implement stronger checks on the ISBN:

```javascript
var Book = function(isbn, title, author) {
  if (!this.checkIsbn(isbn)) throw new Error('Book: Invalid ISBN.');
  this.isbn = isbn;
  this.title = title || 'No title specified';
  this.author = author || 'No author specified';
};

Book.prototype = {
  checkIsbn: function(isbn) {
    if(isbn == undefined || typeof isbn != 'string') {
      return false;
    }
    isbn = isbn.replace(/-/. ''); // Remove dashes.
    if(isbn.length != 10 && isbn.length != 13) {
      return false;
    }
    var sum = 0;
    if(isbn.length === 10) { // 10 digit ISBN.
      if(!isbn.match(\^\d{9}\)) { // Ensure characters 1 through 9 are digits.
        return false;
      }
      for(var i = 0; i < 9; i++) {
        sum += isbn.charAt(i) * (10 - i);
      }
      var checksum = sum % 11;
      if(checksum === 10) checksum = 'X';
      if(isbn.charAt(9) != checksum) {
        return false;
      }
    } else { // 13 digit ISBN.
      if(!isbn.match(\^\d{12}\)) { // Ensure characters 1 through 12 are digits.
        return false;
      }
      for(var i = 0; i < 12; i++) {
        sum += isbn.charAt(i) * ((i % 2 === 0) ? 1 : 3);
      }
      var checksum = sum % 10;
      if(isbn.charAt(12) != checksum) {
        return false;
      }
    }
    return true; // All tests passed.
  },
  display: function() {
    ...
  }
};
```

Here we add a checkIsbn method that ensures the ISBN is a string with the correct number of digits and the correct checksum. Since there are now two methods for this class, Book.prototype is set to an object literal, for defining multiple methods without having to start each one with Book.prototype. Both ways of defining methods are identical, and we use both interchangeably throughout the chapter.
This seems to be an improvement. You are now able to verify that the ISBN is valid when the object is created, thus ensuring that the display method will succeed. However, a problem comes up. Another programmer notices that a book may have multiple editions, each with its own ISBN. He creates an algorithm for selecting among these different editions, and is using it to change the isbn attribute directly after instantiating the object:

```javascript
theHobbit.isbn = '978-0261103283';
theHobbit.display();
```

Even though you can verify the integrity of the data in the constructor, you don’t have any control over what another programmer will assign to the attribute directly. In order to protect the internal data, you create accessor and mutator methods for each attribute. An accessor method (usually named in the form getAttributeName) will get the value of any of the attributes. A mutator method (usually named in the form setAttributeName) will set the value of the attribute. Using mutators, you can implement any kind of verification you like before you actually assign a new value to any of your attributes. Here is a new version of the Book object with accessors and mutators added:

```javascript
var Publication = new Interface('Publication', ['getIsbn', 'setIsbn', 'getTitle',
  'setTitle', 'getAuthor', 'setAuthor', 'display']);

var Book = function(isbn, title, author) { // implements Publication
  this.setIsbn(isbn);
  this.setTitle(title);
  this.setAuthor(author);
}

Book.prototype = {
  checkIsbn: function(isbn) {
    ...
  },
  getIsbn: function() {
    return this.isbn;
  },
  setIsbn: function(isbn) {
    if(!this.checkIsbn(isbn)) throw new Error('Book: Invalid ISBN.');
    this.isbn = isbn;
  },
  getTitle: function() {
    return this.title;
  },
  setTitle: function(title) {
    this.title = title || 'No title specified';
  },
  getAuthor: function() {
    return this.author;
  },
  setAuthor: function(author) {
    this.author = author || 'No author specified';
  },
  display: function() {
    ...
} };
```

Notice that an interface is also defined. From now on, other programmers should only interact with the object using those methods defined in the interface. Also, the mutator methods are used in the constructor; there is no point implementing the same verifications twice, so you rely on those methods internally.
This is as good as it gets with the fully exposed object pattern. You have a well-defined interface, accessor and mutator methods protecting the data, and validation methods. Despite having all of these features, there is still a hole in the design. Even though we provide mutator methods for setting attributes, **the attributes are still public, and can still be set directly**. With this pattern, there is no way of preventing that. It is possible to set an invalid ISBN, either acci- dentally (by a programmer who doesn’t know he’s not supposed to set it directly) or intentionally (by a programmer who knows the interface but ignores it).
Despite that single flaw, this pattern still holds a lot of benefits. It’s easy to use and easy for new JavaScript programmers to pick up quickly. It isn’t necessary to have a deep understanding of scope or the call chain in order to create a class like this. Subclassing is very easy, as is unit testing, since all methods and attributes are publicly available. The only drawbacks are the fact that you cannot protect the internal data, and accessor and mutator methods add extra code that isn’t strictly needed. This could be a concern in situations where JavaScript file size is important.

**2. Private Methods Using a Naming Convention**

Next we will take a look at a pattern that emulates private members by using a naming convention. This pattern addresses one of the problems encountered in the previous section: the inability to prevent another programmer from accidentally bypassing all of your validations. It is essentially the same as the fully exposed object but with **underscores** in front of methods and attributes you want to keep private.

It is still possible for programmers to use this function to game the system, but it is less likely they will do it unintentionally.
Using an underscore is a well-known naming convention; it says that the attribute (or method) is used internally, and that accessing it or setting it directly may have unintended consequences. It should prevent programmers from setting it in ignorance, but it still won’t prevent those that use it knowingly. For that, you need real private methods.
This pattern has all of the benefits of a fully exposed object, and one less drawback. It is, however, a convention that must be agreed upon to have any real use. No enforcement is possible, and as such, it is not a real solution for hiding the internal data of an object. It is instead used mostly for methods and attributes that are internal but not sensitive—methods and attrib- utes that most programmers using the class won’t care about since they aren’t in the public interface.

**3. Scope, Nested Functions, and Closures**

Before we get into real private methods and attributes, we should take a moment to explain the theory behind the technique we will use. In JavaScript, **only functions have scope**; that is to say, a variable declared within a function is not accessible outside of that function. Private attributes are essentially variables that you would like to be inaccessible from outside of the object, so it makes sense to look to this concept of scope to achieve that inaccessibility. A variable defined within a function is accessible to its nested functions. Here is an example demonstrating scope in JavaScript:

```javascript
function foo() {
  var a = 10;
  function bar() {
    a *= 2;
  }
  bar();
  return a;
}
```

In this example, a is defined in the function foo, but the function bar can access it because bar is also defined within foo. When bar is executed, it sets a to a times 2. It makes sense that bar can access a when it is executed within foo, but what if you could execute bar outside of foo?

```javascript
function foo() {
  var a = 10;
  function bar() {
    a *= 2;
    return a;
  }
  return bar;
}

var baz = foo(); // baz is now a reference to function bar.
baz(); // returns 20.
baz(); // returns 40.
baz(); // returns 80.

var blat = foo(); // blat is another reference to bar.
blat(); // returns 20, because a new copy of a is being used.
```

Here a reference to the function bar is returned and assigned to the variable baz. This function is now executed outside of foo, and it still has access to a. This is possible because JavaScript is lexically scoped. **Functions run in the scope they are defined in (in this case, the scope within foo), rather than the scope they are executed in.** As long as bar is defined within foo, it has access to all of foo’s variables, even if foo is finished executing.
This is an example of a closure. After foo returns, its scope is saved, and only the function that it returns has access to it. In the previous example, baz and blat each have a copy of this scope and a copy of a that only they can modify. The most common way of creating a closure is by returning a nested function.

## Private Members Through Closures

Back to the problem at hand: you need to create a variable that can only be accessed internally. A closure seems to be a perfect fit because it allows you to create variables that are accessible only to certain functions and are preserved in between those function calls. To create private attributes, you define variables in the scope of your constructor function. These attributes will be accessible to all functions defined within this scope, including privileged methods:

```javascript
var Book = function(newIsbn, newTitle, newAuthor) { // implements Publication
  // Private attributes.
  var isbn, title, author;
  // Private method.
  function checkIsbn(isbn) {
    ...
  }
  // Privileged methods.
  this.getIsbn = function() {
    return isbn;
  };
  this.setIsbn = function(newIsbn) {
    if(!checkIsbn(newIsbn)) throw new Error('Book: Invalid ISBN.');
    isbn = newIsbn;
  };
  this.getTitle = function() {
    return title;
  };
  this.setTitle = function(newTitle) {
    title = newTitle || 'No title specified';
  };
  this.getAuthor = function() {
    return author;
  };
  this.setAuthor = function(newAuthor) {
    author = newAuthor || 'No author specified';
  };
  // Constructor code.
  this.setIsbn(newIsbn);
  this.setTitle(newTitle);
  this.setAuthor(newAuthor);
};

// Public, non-privileged methods.
Book.prototype = {
  display: function() {
    ...
} };
```

So how is this different from the other patterns we’ve covered so far? In the other Book examples, we always created and referred to the attributes using the this keyword. In this example, we declared these variables using var. That means they will only exist within the Book constructor. We also declare the checkIsbn function in the same way, making it a private method.
Any method that needs to access these variables and functions need only be declared within Book. These are called `privileged methods` because they are public but have access to private attributes and methods. The this keyword is used in front of these privileged functions to make them publicly accessible. Because these methods are defined within the Book constructor’s scope, they can access the private attributes. They are not referred to using this because they aren’t public. All of the accessor and mutator methods have been changed to refer to the attributes directly, without this.
Any `public method` that does not need direct access to private attributes can be declared normally in the Book.prototype. An example of one of these methods is display; it doesn’t need direct access to any of the private attributes because it can just call getIsbn or getTitle. It’s a good idea to make a method privileged only if it needs direct access to the private mem- bers. Having too many privileged methods can cause memory problems because new copies of all privileged methods are created for each instance.
With this pattern, you can create objects that have true private attributes. It is impossible for other programmers to create an instance of Book and directly access any of the data. You can tightly control what gets set because they are forced to go through the mutator methods.
This pattern solves all of the problems with the other patterns, but it introduces a few draw- backs of its own. In the fully exposed object pattern, all methods are created off of the prototype, which means there is only one copy of each in memory, no matter how many instances you create. In this pattern, you create a new copy of every private and privileged method each time a new object is instantiated. This has the potential to use more memory than the other patterns, so it should only be used when you require true private members. This pattern is also hard to subclass. The new inherited class will not have access to any of the superclass’s private attributes or methods. It is said that “`inheritance breaks encapsulation`” because in most languages, the subclass has access to all of the private attributes and methods of the superclass. In JavaScript, this is not the case. If you are creating a class that might be subclassed later, it is best to stick to one of the fully exposed patterns.

## More Advanced Patterns

Now that you have three basic patterns at your disposal, we’ll show you a few advanced patterns. Part 2 of this book goes into much more detail about specific patterns, but we will take an introductory look at a few of them here.

**1. Static Methods and Attributes**

Applying the lesson of scope and closures from earlier in the chapter can lead to a way to create static members, which can be both private and publicly accessible. Most methods and attributes interact with an instance of a class; static members interact with the class itself. Another way of putting it is to say that static members operate on the class-level instead of the instance-level; there is only one copy of each static member. As you will see later in this section, static members are called directly off of the class object.
Here is the Book class with static attributes and methods:

```javascript
var Book = (function() {
  // Private static attributes.
  var numOfBooks = 0;
  // Private static method.
  function checkIsbn(isbn) {
    ...
  }
  // Return the constructor.
  return function(newIsbn, newTitle, newAuthor) { // implements Publication
    // Private attributes.
    var isbn, title, author;
    // Privileged methods.
    this.getIsbn = function() {
      return isbn;
    };
    this.setIsbn = function(newIsbn) {
      if(!checkIsbn(newIsbn)) throw new Error('Book: Invalid ISBN.');
      isbn = newIsbn;
    };
    this.getTitle = function() {
      return title;
    };
    this.setTitle = function(newTitle) {
      title = newTitle || 'No title specified';
    };
    this.getAuthor = function() {
      return author;
    };
    this.setAuthor = function(newAuthor) {
      author = newAuthor || 'No author specified';
    };
    // Constructor code.
    numOfBooks++; // Keep track of how many Books have been instantiated
                  // with the private static attribute.
    if(numOfBooks > 50) throw new Error('Book: Only 50 instances of Book can be '
        + 'created.');
    this.setIsbn(newIsbn);
    this.setTitle(newTitle);
    this.setAuthor(newAuthor);
  }
})();
// Public static method.
Book.convertToTitleCase = function(inputString) {
  ...
};
// Public, non-privileged methods.
Book.prototype = {
  display: function() {
    ...
} };
```

This is similar to the class created earlier in the chapter in the “Private Members Through Closures” section, with a couple of key differences. Private and privileged members are still declared within the constructor, using var and this respectively, but the constructor is changed from a normal function to a nested function that gets returned to the variable Book. This makes it possible to create a closure where you can declare private static members. The empty parentheses after the function declaration are extremely important. They serve to execute that function immediately, as soon as the code is loaded (not when the Book constructor is called). The result of that execution is another function, which is returned and set to be the Book constructor. When Book is instantiated, this inner function is what gets called; the outer function is used only to create a closure, within which you can put private static members.

In this example, the checkIsbn method is static because there is no point in creating a new copy of it for each instance of Book. There is also a static attribute called numOfBooks, which allows you to keep track of how many times the Book constructor has been called. In this example, we use that attribute to limit the constructor to creating only 50 instances.
These private static members can be accessed from within the constructor, which means that any private or privileged function has access to them. They have a distinct advantage over these other methods in that they are only stored in memory once. Since they are declared outside of the constructor, they do not have access to any of the private attributes, and as such, are not privileged; private methods can call private static methods, but not the other way around. A rule of thumb for deciding whether a private method should be static is to see whether it needs to access any of the instance data. If it does not need access, making the method static is more efficient (in terms of memory use) because only a copy is ever created.
Public static members are much easier to create. They are simply created directly off of the constructor, as with the previous method convertToTitleCase. This means you are essentially using the constructor as a namespace.

All public static methods could just as easily be declared as separate functions, but it is useful to bundle related behaviors together in one place. They are useful for tasks that are related to the class as a whole and not to any particular instance of it. They don’t directly depend on any of the data contained within the instances.

**2. Constants**

Constants are nothing more than variables that can’t be changed. In JavaScript, you can emulate constants by creating a private variable with an accessor but no mutator. Since constants are usually set at development time and don’t change with each instance that is created, it makes sense to create them as private static attributes. Here is how a call to get the constant UPPER_BOUND from Class would look:

```javascript
Class.getUPPER_BOUND();
```

To implement this accessor, you would need a privileged static method, which we haven’t covered yet. It is created just like a privileged instance method, with the this keyword:

```javascript
var Class = (function() {
  // Constants (created as private static attributes).
  var UPPER_BOUND = 100;
  // Privileged static method.
  this.getUPPER_BOUND() {
    return UPPER_BOUND;
  }
  ...
  // Return the constructor.
  return function(constructorArgument) {
    ...
  }
})();
```

If you have a lot of constants and don’t want to create an accessor method for each, you can create a single generic accessor method:

```javascript
var Class = (function() {
  // Private static attributes.
  var constants = {
    UPPER_BOUND: 100,
    LOWER_BOUND: -100
  }
  // Privileged static method.
  this.getConstant(name) {
    return constants[name];
  }
...
  // Return the constructor.
  return function(constructorArgument) {
    ...
  }
})();

Class.getConstant('UPPER_BOUND');
```

## Singletons and Object Factories

There are other patterns that utilize closures to create a protected variable space. The two that rely on it the most are the singleton pattern and the factory pattern. Both are covered in more detail later in the book, but we mention them here because they use these same concepts to hide information.
The `singleton` pattern uses a returned object literal to expose privileged members, while keeping private members protected in the enclosing function’s scope. It uses the same technique that we covered earlier, where an outer function is executed immediately and the result is assigned to a variable. In the examples so far in this chapter, a function has always been returned; **a singleton returns an object literal instead**. It is a very easy and straightforward way to create a sheltered namespace. We talk more about singletons in Chapter 5.
Object factories can also use closures to create objects with private members. In its simplest form, an object factory is the same as a class constructor, and all of the patterns we discuss here can be applied to it. The factory pattern is covered in detail in Chapter 7.

## Benefits of Using Encapsulation

It’s true that it would be much simpler to not have to worry about things such as closures and privileged methods when creating an object. In a perfect world, all methods could be public, and other programmers would only use the ones specified in the interface. So what do you gain by going through the trouble of hiding your implementation details?
Encapsulation protects the integrity of the internal data. By allowing access to the data only through accessor and mutator methods, you have complete control over what gets saved and returned. This allows you to reduce the amount of error-checking code you need in your other functions, and ensures that the data can never be in a bad state. It also has the added benefit of allowing easier refactoring of your objects. Since the internal details are shielded from the users of the object, you are free to change data structures and algorithms in midstream without anyone knowing or caring.
By making only the methods specified in the interface public, you are promoting loosely coupled modules. This is one of the most important principles of object-oriented design. Keeping your objects as independent as possible has many benefits. It improves reusability and allows objects to be swapped out if needed. Using private variables also protects you from having to worry about namespace collisions. By making a variable inaccessible to the rest of the code, you don’t have to constantly ask yourself if the variable name you are using might interfere with other objects or functions elsewhere in the program. It allows internal object details to change dramatically without affecting other pieces of code; in general, you can make changes more easily because you already know exactly what it will affect. If you expose internal data directly, it would be impossible to know what consequences code changes could have.

## Drawbacks to Using Encapsulation

It can be very hard to unit test private methods. Because of the very fact that they are hidden, and their internal variables are shielded, it is impossible to access them outside of the object. The workarounds for this aren’t very appealing. You must either provide access through public methods, removing most of the benefit of using private methods in the first place, or somehow define and execute all unit tests within the object. The best solution to this problem is to only unit test the public methods. This should provide complete coverage of the private methods, though only indirectly. This problem is not specific to JavaScript, and it is generally accepted that you should only unit test your public methods.
Having to deal with complicated scope chains can make debugging errors more difficult. This usually isn’t a big problem, but there are situations where it can be hard to distinguish between many identically named variables in different scopes. This problem is not unique to encapsulated objects, but it can be made more complicated by the closures needed to produce private methods and attributes.

It is possible to be too successful with encapsulation. If you don’t have a clear understanding of how your classes may be used by other programmers, actively preventing them from modifying the internal details may be too restrictive. It’s hard to predict how people will use your code. Encapsulation could make your classes so inflexible that it is impossible to reuse them to achieve a purpose you hadn’t anticipated.
The biggest drawback is that it is hard to implement encapsulation in JavaScript. It requires complicated object patterns, most of which are very unintuitive for novice programmers. Having to understand concepts such as the call chain and immediately executed anonymous functions adds a steep learning curve to a language that is already very different from most other object-oriented languages. Furthermore, it can make existing code hard to decipher for someone not well-versed in a particular pattern. Descriptive comments and documentation can reduce this problem a bit, but not eliminate it completely. If you are going to be using these patterns, it is important that the other programmers you work with also understand them.

## Summary

In this chapter we looked at the concept of information hiding and how to enforce it with encapsulation. Since JavaScript has no built-in way to do this, you must rely on other techniques. Fully exposed objects are useful when it isn’t crucial to maintain the integrity of internal data, or when other programmers can be trusted to use only the methods described in the interface. Naming conventions can also help to steer other programmers away from internal methods that shouldn’t be accessed directly. If true private members are needed, the only way to create them is through closures. By creating a protected variable space, you can implement public, private, and privileged members, along with static class members and constants. Most of the later chapters in this book rely on these basic techniques, so it is worth going over them carefully. Once you understand how scope can be manipulated in JavaScript, any object-oriented technique can be emulated.
