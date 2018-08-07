# Chapter 2 Interface

The interface is one of the most useful tools in the object-oriented JavaScript programmer’s toolbox. The first principle of reusable object-oriented design mentioned in the Gang of Four’s Design Patterns says “Program to an interface, not an implementation,” telling you how funda- mental this concept is.
The problem is that JavaScript has no built-in way of creating or implementing interfaces. It also lacks built-in methods for determining whether an object implements the same set of methods as another object, making it difficult to use objects interchangeably. Luckily, JavaScript is extremely flexible, making it easy to add these features.
In this chapter, we look at how other object-oriented languages implement interfaces, and try to emulate the best features of each. We look at several ways of doing this in JavaScript, and eventually come up with a reusable class that can be used to check objects for needed methods.

## What Is an Interface?

An interface provides a way of specifying what methods an object should have. It does not specify how those methods should be implemented, though it may indicate (or at least hint at) the semantics of the methods. For example, if an interface contains a method called setName, you can be reasonably sure that the implementation of that method is expected to take a string argument and assign it to a name variable.
This allows you to group objects based on what features they provide. For example, a group of extremely dissimilar objects can all be used interchangeably in object.compare(anotherObject) if they all implement the Comparable interface. It allows you to exploit the commonality between different classes. Functions that would normally expect an argument to be of a specific class can instead be changed to expect an argument of a specific interface, allowing you to pass in objects of any concrete implementation. It allows unrelated objects to be treated identically.

## Benefits of Using Interfaces

What does an interface do in object-oriented JavaScript? Established interfaces are self- documenting and promote reusability. An interface tells programmers what methods a given class implements, which makes it easier to use. If you are familiar with a certain interface, you already know how to use any class that implements it, increasing the odds that you will reuse existing classes.
Interfaces also stabilize the ways in which different classes can communicate. By knowing the interface ahead of time, you can reduce the problems of integrating two objects. It also allows you to specify in advance what features and operations you want a class to have. One program- mer can create an interface for a class he requires and then pass it to another programmer. The second programmer can implement the code in any way she wants, and as long as the class implements the interface, it should work. This is especially helpful in large projects.
Testing and debugging become much easier. In a loosely typed language such as JavaScript, tracking down type-mismatch errors is very difficult. Using interfaces makes these easier to find because explicit errors with useful messages are given if an object does not seem to be of the expected type or does not implement the required methods. Logic errors are then limited to the methods themselves, instead of the object’s composition. It also makes your code more stable by ensuring that any changes made to an interface must also be made to all classes that implement it. If you add an operation to an interface, you can rely on the fact that you will see an error immediately if one of your classes does not have that operation added to it.

## Drawbacks of Using Interfaces

Using interfaces is not entirely without drawbacks. JavaScript is an extremely expressive lan- guage, in large part because it is loosely typed. Using interfaces is a way of partially enforcing strict typing. This reduces the flexibility of the language.
JavaScript does not come with built-in support for interfaces, and there is always a danger in trying to emulate some other language’s native functionality. There is no Interface keyword, so any method you use to implement this will be very different from what languages such as C++ and Java use, making the transition to JavaScript a little more difficult.
Using any interface implementation in JavaScript will create a small performance hit, due in part to the overhead of having another method invocation. Our implementation uses two for loops to iterate through each of the methods in each of the required interfaces; for large interfaces and for objects that are expected to implement many different interfaces, this check could take a while and negatively affect performance. If this is a concern, you could always strip this code out after development or tie it to a debugging flag so it is not executed in production environments. But be sure to avoid premature optimization. The use of a profiler, such as Fire- bug, can help you determine whether stripping out the interface code is truly necessary.
The biggest drawback is that there is no way to force other programmers to respect the interfaces you have created. In other languages, the concept of the interface is built-in, and if someone is creating a class that implements an interface, the compiler will ensure that the class really does implement that interface. In JavaScript, you must manually ensure that a given class implements an interface. You can mitigate this problem by using coding conventions and helper classes, but it will never entirely go away. If other programmers working on a project with you choose to ignore interfaces, there is no way to force them to be used. Everyone on your project must agree to use them and check for them; otherwise much of their value is lost.

## Emulating an Interface in JavaScript

We will explore three ways of emulating interfaces in JavaScript: comments, attribute checking, and duck typing. No single technique is perfect, but a combination of all three will come close.

1.  Describing Interfaces with Comments

The easiest and least effective way of emulating an interface is with comments. Mimicking the style of other object-oriented languages, the interface and implements keywords are used but are commented out so they do not cause syntax errors. Here is an example of how these key- words can be added to code to document the available methods:

```javascript
/*
interface Composite {
    function add(child);
    function remove(child);
    function getChild(index);
}
interface FormItem {
    function save();
}
*/
var CompositeForm = function(id, method, action) {
    // implements Composite, FormItem
    ...
};

// Implement the Composite interface.
CompositeForm.prototype.add = function(child) {
    ...
};

CompositeForm.prototype.remove = function(child) {
    ...
};

CompositeForm.prototype.getChild = function(index) {
    ...
};

// Implement the FormItem interface.
CompositeForm.prototype.save = function() {
    ...
};
```

This doesn’t emulate the interface functionality very well. There is no checking to ensure that CompositeForm actually does implement the correct set of methods. No errors are thrown to inform the programmer that there is a problem. It is really more documentation than any- thing else. All compliance is completely voluntary.
That being said, there are some benefits to this approach. It’s easy to implement, requiring no extra classes or functions. It promotes reusability because classes now have documented interfaces and can be swapped out with other classes implementing the same ones. It doesn’t affect file size or execution speed; the comments used in this approach can be trivially stripped out when the code is deployed, eliminating any increase in file size caused by using interfaces. However, it doesn’t help in testing and debugging since no error messages are given.

2.  Emulating Interfaces with Attribute Checking

The second technique is a little stricter. All classes explicitly declare which interfaces they implement, and these declarations can be checked by objects wanting to interact with these classes. The interfaces themselves are still just comments, but you can now check an attribute to see what interfaces a class says it implements:

```javascript
/*
interface Composite {
    function add(child);
    function remove(child);
    function getChild(index);
}
interface FormItem {
    function save();
}
*/
var CompositeForm = function(id, method, action) {
    this.implementsInterfaces = ['Composite', 'FormItem'];
    ...
};

...

function addForm(formInstance) {
    if(!implements(formInstance, 'Composite', 'FormItem')) {
        throw new Error("Object does not implement a required interface.");
    }
    ...
}

// The implements function, which checks to see if an object declares that it
// implements the required interfaces.
function implements(object) {
    for(var i = 1; i < arguments.length; i++) {
        // Looping through all arguments
        // after the first one.
        var interfaceName = arguments[i];
        var interfaceFound = false;
        for(var j = 0; j < object.implementsInterfaces.length; j++) {
            if(object.implementsInterfaces[j] == interfaceName) {
                interfaceFound = true;
                break;
            }
        }
        if(!interfaceFound) {
            return false; // An interface was not found.
        }
    }
    return true; // All interfaces were found.
}
```

In this example, CompositeForm declares that it implements two interfaces, Composite and FormItem. It does this by adding their names to an array, labeled as implementsInterfaces. The class explicitly declares which interfaces it supports. Any function that requires an argument to be of a certain type can then check this property and throw an error if the needed interface is not declared.
There are several benefits to this approach. You are documenting what interfaces a class implements. You will see errors if a class does not declare that it supports a required interface. You can enforce that other programmers declare these interfaces through the use of these errors.
The main drawback to this approach is that you are not ensuring that the class really does implement this interface. You only know if it says it implements it. It is very easy to create a class that declares it implements an interface and then forget to add a required method. All checks will pass, but the method will not be there, potentially causing problems in your code. It is also added work to explicitly declare the interfaces a class supports.

3.  Interfaces with Duck Typing

In the end, it doesn’t matter whether a class declares the interfaces it supports, as long as the required methods are in place. That is where duck typing comes in. Duck typing was named after the saying, “If it walks like a duck and quacks like a duck, it’s a duck.” It is a technique to determine whether an object is an instance of a class based solely on what methods it imple- ments, but it also works great for checking whether a class implements an interface. The idea behind this approach is simple: if an object contains methods that are named the same as the methods defined in your interface, it implements that interface. Using a helper function, you can ensure that the required methods are there:

```javascript
// Interfaces.
var Composite = new Interface('Composite', ['add', 'remove', 'getChild']);
var FormItem = new Interface('FormItem', ['save']);

// CompositeForm class
var CompositeForm = function(id, method, action) {
   ...
};

...

function addForm(formInstance) {
    ensureImplements(formInstance, Composite, FormItem);
    // This function will throw an error if a required method is not implemented.
    ...
}
```

This differs from the other two approaches in that it uses no comments. All aspects of this are enforceable. The ensureImplements function takes at least two arguments. The first argument is the object you want to check. The other arguments are the interfaces that the first object will be compared against. The function checks that the object given as the first argument implements the methods declared in those interfaces. If any method is missing, an error will be thrown with a useful message, including both the name of the missing method and the name of the interface that is incorrectly implemented. This check can be added anywhere in your code that needs to ensure an interface. In this example, you only want the addForm function to add the form if it supports the needed methods.
While probably being the most useful of the three methods, it still has some drawbacks. A class never declares which interfaces it implements, reducing the reusability of the code and not self-documenting like the other approaches. It requires a helper class, Interface, and a helper function, ensureImplements. It does not check the names or numbers of arguments used in the methods or their types, only that the method has the correct name.

### The Interface Implementation for This Book

For this book, we are using a combination of the first and third approaches. We use comments to declare what interfaces a class supports, thus improving reusability and improving documen- tation. We use the Interface helper class and the class method Interface.ensureImplements to perform explicit checking of methods. We return useful error messages when an object does not pass the check.
Here is an example of our Interface class and comment combination:

```javascript
// Interfaces.
var Composite = new Interface('Composite', ['add', 'remove', 'getChild']);
var FormItem = new Interface('FormItem', ['save']);

// CompositeForm class
var CompositeForm = function(id, method, action) { // implements Composite, FormItem
   ...
};

...

function addForm(formInstance) {
    Interface.ensureImplements(formInstance, Composite, FormItem);
    // This function will throw an error if a required method is not implemented,
    // halting execution of the function.
    // All code beneath this line will be executed only if the checks pass.
    ...
}
```

Interface.ensureImplements provides a strict check. If a problem is found, an error will be thrown, which can either be caught and handled or allowed to halt execution. Either way, the programmer will know immediately that there is a problem and where to go to fix it.

### The Interface Class

The following is the Interface class that we use throughout the book:

```javascript
// Constructor.
var Interface = function(name, methods) {
  if (arguments.length != 2) {
    throw new Error(
      'Interface constructor called with ' +
        arguments.length +
        'arguments, but expected exactly 2.'
    );
  }
  this.name = name;
  this.methods = [];
  for (var i = 0, len = methods.length; i < len; i++) {
    if (typeof methods[i] !== 'string') {
      throw new Error(
        'Interface constructor expects method names to be ' +
          'passed in as a string.'
      );
    }
    this.methods.push(methods[i]);
  }
};

// Static class method.
Interface.ensureImplements = function(object) {
  if (arguments.length < 2) {
    throw new Error(
      'Function Interface.ensureImplements called with ' +
        arguments.length +
        'arguments, but expected at least 2.'
    );
  }
  for (var i = 1, len = arguments.length; i < len; i++) {
    var interface = arguments[i];
    if (interface.constructor !== Interface) {
      throw new Error(
        'Function Interface.ensureImplements expects arguments' +
          'two and above to be instances of Interface.'
      );
    }
    for (
      var j = 0, methodsLen = interface.methods.length;
      j < methodsLen;
      j++
    ) {
      var method = interface.methods[j];
      if (!object[method] || typeof object[method] !== 'function') {
        throw new Error(
          'Function Interface.ensureImplements: object ' +
            'does not implement the ' +
            interface.name +
            ' interface. Method ' +
            method +
            ' was not found.'
        );
      }
    }
  }
};
```

As you can see, it is very strict about the arguments given to each method and will throw an error if any check doesn’t pass. This is done intentionally, so that if you receive no errors, you can be certain the interface is correctly declared and implemented.

## When to Use the Interface Class

It doesn’t always make sense to use strict type checking. Most JavaScript programmers have worked for years without ever needing an interface or the kind of checks that it provides. It becomes most beneficial when you start implementing complex systems using design patterns. It might seem like interfaces reduce JavaScript’s flexibility, but they actually improve it by allow- ing your objects to be more loosely coupled. Your functions can be more flexible because you can pass in arguments of any type and still ensure that only objects with the needed method will be used. There are a few situations where interfaces can be useful.
In a large project, with many different programmers writing code, interfaces are essential. Often programmers are asked to use an API that hasn’t been written yet, or are asked to provide stubs so the development won’t be delayed. Interfaces can be very valuable in this situation for several reasons. They document the API and can be used as formal communication between two programmers. When the stubs are replaced with the production API, you will know imme- diately whether the methods you need are implemented. If the API changes in mid-development, another can be seamlessly put in its place as long as it implements the same interface.
It is becoming increasingly common to include code from Internet domains that you do not have direct control over. Externally hosted libraries are one example of this, as are APIs to services such as search, email, and maps. Even when these come from trusted sources, use caution to ensure their changes don’t cause errors in your code. One way to do this is to create Interface objects for each API that you rely on, and then test each object you receive to ensure it implements those interfaces correctly:

```javascript
var DynamicMap = new Interface('DynamicMap', ['centerOnPoint', 'zoom', 'draw']);
function displayRoute(mapInstance) {
    Interface.ensureImplements(mapInstace, DynamicMap);
    mapInstance.centerOnPoint(12, 34);
    mapInstance.zoom(5);
    mapInstance.draw();
    ...
}
```

In this example, the displayRoute function needs the passed-in argument to have three specific methods. By using an Interface object and calling Interface.ensureImplements, you will know for sure that these methods are implemented and will see an error if they are not. This error can be caught in a try/catch block and potentially used to send an Ajax request alerting you to the problem with the external API. This makes your mash-ups more stable and secure.

## How to Use the Interface Class

The most important step (and the one that is the most difficult to perform) is to determine whether it is worth using interfaces in your code. Small and less difficult projects may not benefit from the added complexity that interfaces bring. It is up to you to determine whether the benefits outweigh the drawbacks. Assuming that they do, here is how to use interfaces:

1.  Include the Interface class in your HTML file. The Interface.js file is available at the book’s website: http://jsdesignpatterns.com/.
2.  Go through the methods in your code that take in objects as arguments.Determine what methods these object arguments are required to have in order for your code to work.
3.  Create Interface objects for each discreet set of methods you require.
4.  Remove all explicit constructor checking. Since we are using duck typing, the type of
    the objects no longer matters.
5.  Replace constructor checking with Interface.ensureImplements.

What did you gain from this? Your code is now more loosely coupled because you aren’t relying on instances of any particular class. Instead, you are ensuring that the features you require are in place; any concrete implementation can be used, giving you more freedom to optimize and refactor your code.

## Example: Using the Interface Class

Imagine that you have created a class to take some automated test results and format them for viewing on a web page. This class’s constructor takes an instance of the TestResult class as an argument. It then formats the data encapsulated in the TestResult object and outputs it on request. Here is what the ResultFormatter class looks like initially:

```javascript
// ResultFormatter class, before we implement interface checking.
var ResultFormatter = function(resultsObject) {
  if(!(resultsObject instanceOf TestResult)) {
    throw new Error("ResultsFormatter: constructor requires an instance "
      + "of TestResult as an argument.");
  }
  this.resultsObject = resultsObject;
};

ResultFormatter.prototype.renderResults = function() {
  var dateOfTest = this.resultsObject.getDate();
  var resultsArray = this.resultsObject.getResults();
  var resultsContainer = document.createElement('div');
  var resultsHeader = document.createElement('h3');
  resultsHeader.innerHTML = 'Test Results from ' + dateOfTest.toUTCString();
  resultsContainer.appendChild(resultsHeader);
  var resultsList = document.createElement('ul');
  resultsContainer.appendChild(resultsList);
  for(var i = 0, len = resultsArray.length; i < len; i++) {
    var listItem = document.createElement('li');
    listItem.innerHTML = resultsArray[i];
    resultsList.appendChild(listItem);
  }
  return resultsContainer;
};
```

This class performs a check in the constructor to ensure that the argument is really an instance of TestResult; if it isn’t, an error is thrown. This allows you to code the renderResults method knowing confidently that the getDate and getResults methods will be available to you. Or does it? In the constructor, you are only checking that the resultsObject is an instance of TestResult. That does not actually ensure that the methods you need are implemented. TestResult could be changed so that it no longer has a getDate method. The check in the con- structor would pass, but the renderResults method would fail.
The check in the constructor is also unnecessarily limiting. It prevents instances of other classes from being used as arguments, even if they would work perfectly fine. Say, for example, you have a class named WeatherData. It has a getDate and a getResults method and could be used in the ResultFormatter class without a problem. But using explicit type checking (with the instanceOf operator) would prevent any instances of WeatherData from being used.
The solution is to remove the instanceOf check and replace it with an interface. The first step is to create the interface itself:

```javascript
// ResultSet Interface.
var ResultSet = new Interface('ResultSet', ['getDate', 'getResults']);
```

This line of code creates a new instance of the Interface object. The first argument is the name of the interface, and the second is an array of strings, where each string is the name of a required method. Now that you have the interface, you can replace the instanceOf check with an interface check:

```javascript
// ResultFormatter class, after adding Interface checking.
var ResultFormatter = function(resultsObject) {
  Interface.ensureImplements(resultsObject, ResultSet);
  this.resultsObject = resultsObject;
};
ResultFormatter.prototype.renderResults = function() {
  ...
};
```

The renderResults method remains unchanged. The constructor, on the other hand, has been modified to use ensureImplements instead of instanceOf. You could now use an instance of WeatherData in this constructor, or any other class that implements the needed methods. By changing a few lines of code within the ResultFormatter class, you have made the check more accurate (by ensuring the required methods have been implemented) and more permissive (by allowing any object to be used that matches the interface).

## Patterns That Rely on the Interface

The following is a list of a few of the patterns, which we discuss in later chapters, that espe- cially rely on an interface implementation to work:

- The factory pattern: The specific objects that are created by a factory can change depending on the situation. In order to ensure that the objects created can be used interchangeably, interfaces are used. This means that a factory is guaranteed to pro- duce an object that will implement the needed methods.
- The composite pattern: You really can’t use this pattern without an interface. The most important idea behind the composite is that groups of objects can be treated the same as the constituent objects. This is accomplished by implementing the same interface. Without some form of duck typing or type checking, the composite loses much of its power.
- Thedecoratorpattern:Adecoratorworksbytransparentlywrappinganotherobject.This is accomplished by implementing the exact same interface as the other object; from the outside, the decorator and the object it wraps look identical. We use the Interface class to ensure that any decorator objects created implement the needed methods.
- The command pattern: All command objects within your code will implement the same methods (which are usually named execute, run, or undo). By using interfaces, you can create classes that can execute these commands without needing to know anything about them, other than the fact that they implement the correct interface. This allows you to create extremely modular and loosely coupled user interfaces and APIs.

The interface is an important concept that we use throughout this book. It’s worth playing around with interfaces to see if your specific situation warrants their use.

## Summary

In this chapter, we explored the way that interfaces are used and implemented in popular object-oriented languages. We showed that all different implementations of the concept of the interface share a couple features: a way of specifying what methods to expect, and a way to check that those methods are indeed implemented, with helpful error messages if they are not. We are able to emulate these features with a combination of documentation (in comments), a helper class, and duck typing. The challenge is in knowing when to use this helper class. Interfaces are not always needed. One of JavaScript’s greatest strengths is its flexibility, and enforcing strict type checking where it is not needed reduces this flexibility. But careful use of the Interface class can create more robust classes and more stable code.
