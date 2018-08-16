# Chaining

In this chapter we look at JavaScript’s ability to chain methods together. By using a few simple techniques, application developers can streamline their code authoring. As well as writing time-saving functions that reduce the burden of common tasks, you can improve how code is implemented. In the end, you can write an entire JavaScript library that embraces the tech- nique of chaining, and chains together all your favorite methods.
Chaining is really just a syntax hack. It allows you to express complex operations in a small amount of code by reusing an initial operation. Chaining requires two parts: a factory that creates an object around an HTML element (we cover factories in depth in Chapter 7), and methods that perform some action using that HTML element. Each of these methods can be added to the chain by appending the method name with a dot. Chaining can be thought of as the process of selecting an element on the page and performing one or more operations on it.
Let’s take a look at a small example. Using some predefined utility functions, you can get a basic idea of the concept of chaining by seeing a “before and after” contrast. How these utility functions work isn’t important for this illustration. This first example gets a reference to an element with the ID of example and assigns an event listener to it. When clicked, it sets the text color style to green, then shows the element:

```javascript
// Without chaining:
addEvent($('example'), 'click', function() {
  setStyle(this, 'color', 'green');
  show(this);
});
// With chaining:
$('example').addEvent('click', function() {
  $(this)
    .setStyle('color', 'green')
    .show();
});
```

## The Structure of a Chain

You are already familiar with the dollar function, which usually returns an HTML element or a collection of HTML elements as shown here:

```javascript
function $() {
  var elements = [];
  for (var i = 0, len = arguments.length; i < len; ++i) {
    var element = arguments[i];
    if (typeof element === 'string') {
      element = document.getElementById(element);
    }
    if (arguments.length === 1) {
      return element;
    }
    elements.push(element);
  }
  return elements;
}
```

However, if you modify the function to act as a constructor, store the elements as an array in an instance property, then return a reference to the instance in all prototype methods, you can give it the ability to chain. But let’s not get ahead of ourselves. You need to modify the dollar function so it becomes a factory method, creating an object that will support chaining. You also want the dollar function to be able to take in an array of elements, so you can use the same public interface as before. The modified code would look like this:

```javascript
(function() {
  // Use a private class.
  function _$(els) {
    this.elements = [];
    for (var i = 0, len = els.length; i < len; ++i) {
      var element = els[i];
      if (typeof element === 'string') {
        element = document.getElementById(element);
      }
      this.elements.push(element);
    }
  }
  // The public interface remains the same.
  window.$ = function() {
    return new _$(arguments);
  };
})();
```

Since all objects inherit from their prototype, you can take advantage of the reference to the instance object being returned and run each of the methods attached to the prototype as a chain. With that in mind, let’s go ahead and add methods to the private dollar constructor prototype. This will make chaining possible:

```javascript
(function() {
  function _$(els) {
    ...
  }
  _$.prototype = {
    each: function(fn) {
      for ( var i = 0, len = this.elements.length; i < len; ++i ) {
        fn.call(this, this.elements[i]);
      }
      return this;
    },
    setStyle: function(prop, val) {
      this.each(function(el) {
        el.style[prop] = val;
      });
      return this;
    },
    show: function() {
      var that = this;
      this.each(function(el) {
        that.setStyle('display', 'block');
      });
      return this;
    },
    addEvent: function(type, fn) {
      var add = function(el) {
        if (window.addEventListener) {
          el.addEventListener(type, fn, false);
        }
        else if (window.attachEvent) {
          el.attachEvent('on'+type, fn);
        }
      };
      this.each(function(el) {
add(el); });
      return this;
    }
  };
  window.$ = function() {
    return new _$(arguments);
  };
})();
```

If you examine the last line in each method of the class, you’ll notice that they all end in return this. This passes on the object to the next method in the chain. With a chainable interface, the possibilities are limitless. You can now start writing code like this:

```javascript
$(window).addEvent('load', function() {
  $('test-1', 'test-2')
    .show()
    .setStyle('color', 'red')
    .addEvent('click', function(e) {
      $(this).setStyle('color', 'green');
    });
});
```

This will attach an event to the window object’s load event. Upon firing, the elements with the IDs of test-1 and test-2 will instantly be shown, and the text within them will be set to the color red. They will then have click event listeners attached to them, which on firing will set the text color to green. That’s quite a bit packed into such a small amount of application code.
For those familiar with the jQuery JavaScript library, this interface is very similar. The anchor of the chain is the window object or an HTML element, and every operation is attached to that anchor. In the previous example, there are two chains: one that attaches the load event to the window object, and one that sets styles and attaches events to the elements with the IDs test-1 and test-2. Almost any set of existing utilities can be adapted to chaining using this style. We cover this more in the next section.

## Building a Chainable JavaScript Library

So far you’ve chained just a few of the most commonly used utility functions, but you can expand this to your heart’s desire. Building a JavaScript library takes much care and thought. It need not be thousands or even hundreds of lines of code; the length depends on what you need out of a library. You can look at some of the most common features that JavaScript libraries offer, and take it from there. The fundamentals that you will find in nearly all JavaScript libraries are shown in Table 6-1.

**Table 6-1** The Common Features Found in Most JavaScript Libraries
| Feature | Description                                                 |
| ------- | :---------------------------------------------------------- |
| Events  | Adding and removing listeners; normalizing the event object |
| DOM     | Class name management; style management                     |
| Ajax    | Normalizing XMLHttpRequest                                  |

If you were to build this out into pseudocode on top of the private dollar constructor, it might look something like this:

```javascript
// Include syntactic sugar to help the development of our interface.
Function.prototype.method = function(name, fn) {
  this.prototype[name] = fn;
  return this;
};

(function() {
  function _$(els) {
    ... 
  }
  /* Events
   * addEvent
   * getEvent */
  _$.method('addEvent', function(type, fn) {
    ...
  }).method('getEvent', function(e) {
    ...
  }). /*
    DOM
      * addClass
      * removeClass
      * replaceClass
      * hasClass
      * getStyle
      * setStyle
  */
  method('addClass', function(className) {
    ...
  }).method('removeClass', function(className) {
    ...
  }).method('replaceClass', function(oldClass, newClass) {
    ...
  }).method('hasClass', function(className) {
    ...
  }).method('getStyle', function(prop) {
    ...
  }).method('setStyle', function(prop, val) {
    ... 
  }).
  /* AJAX load. Fetches an HTML fragment from a URL and inserts it into an element.
  */
  method('load', function(uri, method) {
    // ...
  });
  window.$ = function() {
    return new _$(arguments);
  });
})();
```

Now that the API is stubbed out, it’s important to consider who might be using it and in what context. If there is an existing API that already uses the dollar function, this library will overwrite it. A simple solution is to change the name of the dollar function within the source. However, this isn’t ideal if you’re retrieving the code from an existing source-code repository; you would have to change it again each time you update the source from that repository. In that case, a better solution is to add an installer, as demonstrated here:

```javascript
Function.prototype.method = function(name, fn) {
  // ...
};
(function() {
  function _$(els) {
    // ...
  }
  _$.method('addEvent', function(type, fn) {
    // ...
  });
  window.installHelper = function(scope, interface) {
    scope[interface] = function() {
      return new _$(arguments);
    }
  };
})();
```

One possible implementation could look like this:

```javascript
installHelper(window, '$');
$('example').show();
```

Here is a more complex example that allows you to attach the functionality to a predefined namespaced object:

```javascript
// Define a namespace without overwriting it if it already exists.
window.com = window.com || {};
com.example = com.example || {};
com.example.util = com.example.util || {};
installHelper(com.example.util, 'get');
(function() {
  var get = com.example.util.get;
  get('example').addEvent('click', function(e) {
    get(this).addClass('hello');
  });
})();
```

## Using Callbacks to Retrieve Data from Chained Methods

In some cases, it isn’t a good idea to chain your methods together. For mutator methods, they’re just fine, but with accessor methods, you may wish to return the data that you are requesting, instead of returning this. Nevertheless, if chaining is your ultimate goal and you wish to keep methods consistent, you can work around this problem by using function callbacks to return your accessed data. This next example shows both of these techniques. The API class uses normal accessors (which break the chain), while the API2 class uses callback methods:

```javascript
// Accessor without function callbacks: returning requested data in accessors.
window.API = window.API || {};
API.prototype = (function() {
  var name = 'Hello world';
  // Privileged mutator method.
  setName: function(newName) {
    name = newName;
    return this;
  },
  // Privileged accessor method.
  getName: function() {
    return name;
  }
})();
// Implementation code.
var o = new API;
console.log(o.getName()); // Displays 'Hello world'.
console.log(o.setName('Meow').getName()); // Displays 'Meow'.
// Accessor with function callbacks.
window.API2 = window.API2 || {};
API2.prototype = (function() {
  var name = 'Hello world';
  // Privileged mutator method.
  setName: function(newName) {
    name = newName;
    return this;
  },
  // Privileged accessor method.
  getName: function(callback) {
    callback.call(this, name);
    return this;
  }
})();
// Implementation code.
var o2 = new API2;
o2.getName(console.log).setName('Meow').getName(console.log);
// Displays 'Hello world' and then 'Meow'.
```

## Summary

JavaScript passes all objects by reference, so you can pass these references back in every method. By returning this at the end of each method, you can create a class that is chainable. This style helps to streamline code authoring and, to a certain degree, make your code more elegant and easier to read. Often you can avoid situations where objects are redeclared several times and instead use a chain, which produces less code. If you want consistent interfaces for your classes, and you want both mutator and accessor methods to be chainable, you can use function callbacks for your accessors.
