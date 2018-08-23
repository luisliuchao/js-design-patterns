# The Factory Pattern

Often a class or object will contain other objects within it. When these member objects need to be created, it is tempting to just instantiate them normally, using the new keyword and the class constructor. The problem is that this creates a dependency between the two classes. In this chapter, we look at a pattern that will help decouple these two classes, and instead use
a method to decide which specific class to instantiate. We discuss the simple factory pattern, which uses a separate class (often a singleton) to create instances, and the more complex factory pattern, which uses subclasses to decide what concrete class to instantiate as a member object.

## The Simple Factory

The simple factory pattern is best illustrated with an example. Let’s say you want to create
a few bicycle shops, each of which offers several models of bikes for sale. This could be represented with a class like this:

```javascript
/* BicycleShop class. */
var BicycleShop = function() {};
BicycleShop.prototype = {
  sellBicycle: function(model) {
    var bicycle;
    switch (model) {
      case 'The Speedster':
        bicycle = new Speedster();
        break;
      case 'The Lowrider':
        bicycle = new Lowrider();
        break;
      case 'The Comfort Cruiser':
      default:
        bicycle = new ComfortCruiser();
    }
    Interface.ensureImplements(bicycle, Bicycle);
    bicycle.assemble();
    bicycle.wash();
    return bicycle;
  }
};
```

You check to see which model of bicycle is requested and then create a new instance of it using a switch statement. You can treat these instances interchangeably because they all respond to the Bicycle interface:

> The factory pattern depends heavily on interfaces. Without some way of checking an object’s type and ensuring that it implements the needed methods, you will lose a lot of the benefits that you would gain from the factory. In all of these examples, you can create objects and treat them identically because you can ensure that they all respond to the same set of methods.

```javascript
/* The Bicycle interface. */
var Bicycle = new Interface('Bicycle', ['assemble', 'wash', 'ride', 'repair']);
/* Speedster class. */
var Speedster = function() { // implements Bicycle
  ...
};
Speedster.prototype = {
  assemble: function() {
    ...
  },
  wash: function() {
    ...
  },
  ride: function() {
    ...
  },
  repair: function() {
    ...
  }
};
```

To sell a certain model of bike, call the sellBicycle method:

```javascript
var californiaCruisers = new BicycleShop();
var yourNewBike = californiaCruisers.sellBicycle('The Speedster');
```

This all works very well until you want to make a change. What if you want to add a new model of bicycle to your lineup? This would require you to modify the code in BicycleShop, even though the actual functionality of BicycleShop hasn’t really changed: you still create a new instance of a bike, assemble it, wash it, and give it to the customer. A better solution would be to pass off the “create a new instance” part of the method to a simple factory object:

```javascript
/* BicycleFactory namespace. */
var BicycleFactory = {
  createBicycle: function(model) {
    var bicycle;
    switch (model) {
      case 'The Speedster':
        bicycle = new Speedster();
        break;
      case 'The Lowrider':
        bicycle = new Lowrider();
        break;
      case 'The Comfort Cruiser':
      default:
        bicycle = new ComfortCruiser();
    }
    Interface.ensureImplements(bicycle, Bicycle);
    return bicycle;
  }
};
```

BicycleFactory is a singleton that is used as a namespace to contain the method createBicycle. This method returns an object that responds to the Bicycle interface, which can then be assembled and washed, just as before:

```javascript
/* BicycleShop class, improved. */
var BicycleShop = function() {};
BicycleShop.prototype = {
  sellBicycle: function(model) {
    var bicycle = BicycleFactory.createBicycle(model);
    bicycle.assemble();
    bicycle.wash();
    return bicycle;
  }
};
```

Any number of classes can use this BicycleFactory object to create new instances. All of the information about which models are available is centralized in one location. This means that it is easy to add more models of bikes:

```javascript
/* BicycleFactory namespace, with more models. */
var BicycleFactory = {
  createBicycle: function(model) {
    var bicycle;
    switch (model) {
      case 'The Speedster':
        bicycle = new Speedster();
        break;
      case 'The Lowrider':
        bicycle = new Lowrider();
        break;
      case 'The Flatlander':
        bicycle = new Flatlander();
        break;
      case 'The Comfort Cruiser':
      default:
        bicycle = new ComfortCruiser();
    }
    Interface.ensureImplements(bicycle, Bicycle);
    return bicycle;
  }
};
```

BicycleFactory is a good example of a simple factory. It takes the creation of member objects and moves it to an external object. This object could either be a simple namespace, as in this example, or an instance of a class. It often makes sense to use singletons or static class methods to create these member instances when the creation methods don’t vary. If, for instance, you have to offer several different lineups of bicycles, it might make more sense to implement the creation method in a class that can then be subclassed.

## The Factory Pattern

The true factory pattern differs from the simple factory in that instead of using another class or object to create the bicycles (as in the previous example), you will use a subclass. The formal definition of the factory is a class that defers instantiation of its member objects to a subclass. Let’s use the BicycleShop example to illustrate the differences between the simple factory and the factory pattern.
You want to allow each bicycle shop to get its inventory from any manufacturer it chooses. Because of this, a single BicycleFactory object isn’t going to be able to provide all of the bicycle instances necessary. Instead, you can create BicycleShop as an abstract class and let the subclasses implement their own createBicycle method, using whatever manufacturer they choose:

```javascript
/* BicycleShop class (abstract). */
var BicycleShop = function() {};
BicycleShop.prototype = {
  sellBicycle: function(model) {
    var bicycle = this.createBicycle(model);
    bicycle.assemble();
    bicycle.wash();
    return bicycle;
  },
  createBicycle: function(model) {
    throw new Error('Unsupported operation on an abstract class.');
  }
};
```

You define a createBicycle method, but it will throw an error if it is actually invoked. BicycleShop is now abstract; it should not be instantiated, only subclassed. To create
a subclass that uses a specific bicycle manufacturer, extend BicycleShop and override the createBicycle method. Here there are two subclasses, one that receives its bikes from the Acme company, the other from the General Products company:

```javascript
/* AcmeBicycleShop class. */
var AcmeBicycleShop = function() {};
extend(AcmeBicycleShop, BicycleShop);
AcmeBicycleShop.prototype.createBicycle = function(model) {
  var bicycle;
  switch (model) {
    case 'The Speedster':
      bicycle = new AcmeSpeedster();
      break;
    case 'The Lowrider':
      bicycle = new AcmeLowrider();
      break;
    case 'The Flatlander':
      bicycle = new AcmeFlatlander();
      break;
    case 'The Comfort Cruiser':
    default:
      bicycle = new AcmeComfortCruiser();
  }
  Interface.ensureImplements(bicycle, Bicycle);
  return bicycle;
};
/* GeneralProductsBicycleShop class. */
var GeneralProductsBicycleShop = function() {};
extend(GeneralProductsBicycleShop, BicycleShop);
GeneralProductsBicycleShop.prototype.createBicycle = function(model) {
  var bicycle;
  switch (model) {
    case 'The Speedster':
      bicycle = new GeneralProductsSpeedster();
      break;
    case 'The Lowrider':
      bicycle = new GeneralProductsLowrider();
      break;
    case 'The Flatlander':
      bicycle = new GeneralProductsFlatlander();
      break;
    case 'The Comfort Cruiser':
    default:
      bicycle = new GeneralProductsComfortCruiser();
  }
  Interface.ensureImplements(bicycle, Bicycle);
  return bicycle;
};
```

All of the objects created from these factory methods respond to the Bicycle interface, so any code written can treat them as being completely interchangeable. Selling bicycles is done in the same way as before, only now you can create shops that sell either Acme or General Products bikes:

````javascript
var alecsCruisers = new AcmeBicycleShop();
var yourNewBike = alecsCruisers.sellBicycle('The Lowrider');
var bobsCruisers = new GeneralProductsBicycleShop();
var yourSecondNewBike = bobsCruisers.sellBicycle('The Lowrider');
```ateBicycle method, but it will throw an error if it is actually invoked. BicycleShop is now abstract; it should not be instantiated, only subclassed. To create
a subclass that uses a specific bicycle manufacturer, extend BicycleShop and override the createBicycle method. Here there are two subclasses, one that receives its bikes from the Acme company, the other from the General Products company:

```javascript
/* AcmeBicycleShop class. */
var AcmeBicycleShop = function() {};
extend(AcmeBicycleShop, BicycleShop);
AcmeBicycleShop.prototype.createBicycle = function(model) {
  var bicycle;
  switch (model) {
    case 'The Speedster':
      bicycle = new AcmeSpeedster();
      break;
    case 'The Lowrider':
      bicycle = new AcmeLowrider();
      break;
    case 'The Flatlander':
      bicycle = new AcmeFlatlander();
      break;
    case 'The Comfort Cruiser':
    default:
      bicycle = new AcmeComfortCruiser();
  }
  Interface.ensureImplements(bicycle, Bicycle);
  return bicycle;
};
/* GeneralProductsBicycleShop class. */
var GeneralProductsBicycleShop = function() {};
extend(GeneralProductsBicycleShop, BicycleShop);
GeneralProductsBicycleShop.prototype.createBicycle = function(model) {
  var bicycle;
  switch (model) {
    case 'The Speedster':
      bicycle = new GeneralProductsSpeedster();
      break;
    case 'The Lowrider':
      bicycle = new GeneralProductsLowrider();
      break;
    case 'The Flatlander':
      bicycle = new GeneralProductsFlatlander();
      break;
    case 'The Comfort Cruiser':
    default:
      bicycle = new GeneralProductsComfortCruiser();
  }
  Interface.ensureImplements(bicycle, Bicycle);
  return bicycle;
};
````

All of the objects created from these factory methods respond to the Bicycle interface, so any code written can treat them as being completely interchangeable. Selling bicycles is done in the same way as before, only now you can create shops that sell either Acme or General Products bikes:

```javascript
var alecsCruisers = new AcmeBicycleShop();
var yourNewBike = alecsCruisers.sellBicycle('The Lowrider');
var bobsCruisers = new GeneralProductsBicycleShop();
var yourSecondNewBike = bobsCruisers.sellBicycle('The Lowrider');
```

Since both manufacturers make bikes in the same styles, customers can go into a shop and order a certain style without caring who originally made it. Or if they only want an Acme bike, they can go to the shops that only sell Acme bikes.

Adding additional manufacturers is easy; simply create another subclass of BicycleShop and override the createBicycle factory method. You can also modify each subclass to allow for additional models specific to a certain manufacturer. This is the most important feature of the factory pattern. You can write all of your general Bicycle code in the parent class, BicycleShop, and then defer the actual instantiation of specific Bicycle objects to the subclasses. The general code is all in one place, and the code that varies is encapsulated in the subclasses.

## When Should the Factory Pattern Be Used?

The simplest way to create new objects is to use the new keyword and a concrete class. The extra complexity of creating and maintaining a factory only makes sense in certain situations, which are outlined in this section.

### Dynamic Implementations

If you need to create objects **with the same interface but different implementations**, as in the previous bicycle example, a factory method or a simple factory object can simplify the process of choosing which implementation is used. This can happen explicitly, as in the bicycle example, when a customer chooses one model of bicycle over another, or implicitly, as in the XHR factory example in the next section, where the type of connection object returned is based on factors such as perceived bandwidth and network latency. In these situations, you usually have a number of classes that implement the same interface and can be treated identically. In JavaScript, this is the most common reason for using the factory pattern.

### Combining Setup Costs

If objects have complex but related setup costs, using a factory can reduce the amount of code needed for each. This is especially true if the setup needs to be done only once for all instances of a certain type of object. Putting the code for this setup in the class constructor is inefficient because it will be called even if the setup is complete and because it decentralizes it among the different classes. A factory method would be ideal in this situation. It can perform the setup once and then instantiate all of the needed objects afterward. It also keeps the setup code in one place, regardless of how many different classes are instantiated.
This is especially helpful if you are using classes that require external libraries to be loaded. The factory method can test for the presence of these libraries and dynamically load any that aren’t found. This setup code will then exist in only one place, which makes it much easier to change later on.

### Abstracting Many Small Objects into One Large Object

A factory method can be useful for creating an object that encapsulates a lot of smaller objects. As an example, imagine the constructors for the bicycle objects. A bicycle is comprised of many smaller subsystems: wheels, a frame, a drive train, brakes. If you don’t want to tightly couple one of those subsystems to the larger object, but instead want to be able to choose one out of many subsystems at run-time, a factory method is ideal. Using this technique, you could create all of the bicycles with a certain type of chain on one day, and change that type the next day if you find one that is better suited to your needs. Making this change is easy because the bicycles don’t depend on a specific type of chain in their constructor. The RSS reader example later in the chapter illustrates this further.

## Example: XHR Factory

A common task in any web page these days is to make an asynchronous request using Ajax. Depending on the user’s browser, you will have to instantiate one of several different classes in order to get an object that can be used to make a request. If you are making more than one Ajax request in your code, it makes sense to abstract this object creation code into a class and to create a wrapper for the different steps it takes to actually make the request. A simple factory works very well here to create an instance of either XMLHttpRequest or ActiveXObject, depending on the browser’s capabilities:

```javascript
/* AjaxHandler interface. */
var AjaxHandler = new Interface('AjaxHandler', ['request', 'createXhrObject']);
/* SimpleHandler class. */
var SimpleHandler = function() {}; // implements AjaxHandler
SimpleHandler.prototype = {
  request: function(method, url, callback, postVars) {
    var xhr = this.createXhrObject();
    xhr.onreadystatechange = function() {
      if (xhr.readyState !== 4) return;
      xhr.status === 200
        ? callback.success(xhr.responseText, xhr.responseXML)
        : callback.failure(xhr.status);
    };
    xhr.open(method, url, true);
    if (method !== 'POST') postVars = null;
    xhr.send(postVars);
  },
  createXhrObject: function() {
    // Factory method.
    var methods = [
      function() {
        return new XMLHttpRequest();
      },
      function() {
        return new ActiveXObject('Msxml2.XMLHTTP');
      },
      function() {
        return new ActiveXObject('Microsoft.XMLHTTP');
      }
    ];
    for (var i = 0, len = methods.length; i < len; i++) {
      try {
        methods[i]();
      } catch (e) {
        continue;
      }
      // If we reach this point, method[i] worked.
      this.createXhrObject = methods[i]; // Memoize the method.
      return methods[i];
    }
    // If we reach this point, none of the methods worked.
    throw new Error('SimpleHandler: Could not create an XHR object.');
  }
};
```

The convenience method request performs the steps needed to send off a request and process the response. It creates an XHR object, configures it, and sends the request. The interesting part is the creation of the XHR object.

The factory method createXhrObject returns an XHR object based on what is available in the current environment. The first time it is run, it will test three different ways of creating an XHR object, and when it finds one that works, it will return the object created and overwrite itself with the function used to create the object. **This new function becomes the createXhrObject method.** This technique, called `memoizing`, can be used to create functions and methods that store complex calculations so that they don’t have to be repeated. All of the complex setup code is only called once, the first time the method is executed, and after that only the browser-specific code is executed. For instance, if the previous code is run on a browser that implements the XMLHttpRequest class, createXhrObject would effectively look like this the second time it is run:

```javascript
createXhrObject: function() { return new XMLHttpRequest(); }
```

Memoizing can make your code much more efficient because all of the setup and test code is only executed once. Factory methods are ideal for encapsulating this kind of code because you can call them knowing that the correct object will be returned regardless of what platform the code is running on. All of the complexity surrounding this task is centralized in one place.
Making a request with the SimpleHandler class is fairly straightforward. After instantiating it, you can use the request method to perform the asynchronous request:

```javascript
var myHandler = new SimpleHandler();
var callback = {
  success: function(responseText) {
    alert('Success: ' + responseText);
  },
  failure: function(statusCode) {
    alert('Failure: ' + statusCode);
  }
};
myHandler.request('GET', 'script.php', callback);
```

## Specialized Connection Objects

You can take this example one step further and use the factory pattern in two places to create specialized request objects based on network conditions. You are already using the simple factory pattern to create the XHR object. You can use another factory to return different handler classes, all inheriting from SimpleHandler.
First you will create two new handlers. QueuedHandler will ensure all requests have succeeded before allowing any new requests, and OfflineHandler will store requests if the user is not online:

```javascript
/* QueuedHandler class. */
var QueuedHandler = function() {
  // implements AjaxHandler
  this.queue = [];
  this.requestInProgress = false;
  this.retryDelay = 5; // In seconds.
};
extend(QueuedHandler, SimpleHandler);
QueuedHandler.prototype.request = function(
  method,
  url,
  callback,
  postVars,
  override
) {
  if (this.requestInProgress && !override) {
    this.queue.push({
      method: method,
      url: url,
      callback: callback,
      postVars: postVars
    });
  } else {
    this.requestInProgress = true;
    var xhr = this.createXhrObject();
    var that = this;
    xhr.onreadystatechange = function() {
      if (xhr.readyState !== 4) return;
      if (xhr.status === 200) {
        callback.success(xhr.responseText, xhr.responseXML);
        that.advanceQueue();
      } else {
        callback.failure(xhr.status);
        setTimeout(function() {
          that.request(method, url, callback, postVars);
        }, that.retryDelay * 1000);
      }
    };
    xhr.open(method, url, true);
    if (method !== 'POST') postVars = null;
    xhr.send(postVars);
  }
};
QueuedHandler.prototype.advanceQueue = function() {
  if (this.queue.length === 0) {
    this.requestInProgress = false;
    return;
  }
  var req = this.queue.shift();
  this.request(req.method, req.url, req.callback, req.postVars, true);
};
```

QueuedHandler’s request method looks similar to SimpleHandlers’s, but it first checks to make sure that there are no other requests in progress before allowing a new one to be made. It also retries any request that doesn’t succeed, at a set interval, until it does:

```javascript
/* OfflineHandler class. */
var OfflineHandler = function() {
  // implements AjaxHandler
  this.storedRequests = [];
};
extend(OfflineHandler, SimpleHandler);
OfflineHandler.prototype.request = function(method, url, callback, postVars) {
  if (XhrManager.isOffline()) {
    // Store the requests until we are online.
    this.storedRequests.push({
      method: method,
      url: url,
      callback: callback,
      postVars: postVars
    });
  } else {
    // Call SimpleHandler's request method if we are online.
    this.flushStoredRequests();
    OfflineHandler.superclass.request(method, url, callback, postVars);
  }
};
OfflineHandler.prototype.flushStoredRequests = function() {
  for (var i = 0, len = storedRequests.length; i < len; i++) {
    var req = storedRequests[i];
    OfflineHandler.superclass.request(
      req.method,
      req.url,
      req.callback,
      req.postVars
    );
  }
};
```

OfflineHandler is a little simpler. Using the XhrMananger.isOffline method (which we will talk more about in a moment), it ensures that the user is online before allowing the request to be made, through SimpleHandler’s request method. It also executes all stored requests as soon as it detects that the user is online.

## Choosing Connection Objects at Run-Time

Here is where the factory pattern comes into play. Instead of requiring the programmer to choose among these different classes at development time, when they have absolutely no idea what the network conditions will be for any of the end users, you use a factory to choose the most appropriate class at run-time. The programmer simply calls the factory method and uses the object that gets returned. Since all of these handlers implement the AjaxHandler interface, you can treat them identically. The interface remains the same; only the implementation changes:

```javascript
/* XhrManager singleton. */
var XhrManager = {
  createXhrHandler: function() {
    var xhr;
    if(this.isOffline()) {
      xhr = new OfflineHandler();
    }
    else if(this.isHighLatency()) {
      xhr = new QueuedHandler();
    } else {
      xhr = new SimpleHandler()
    }
    Interface.ensureImplements(xhr, AjaxHandler);
    return xhr;
  },
  isOffline: function() { // Do a quick request with SimpleHandler and see if it succeeds.
    ...
  },
  isHighLatency: function() { // Do a series of requests with SimpleHandler and  time the responses. Best done once, as a branching function.
    ...
  }
};
```

The programmer now calls the factory method instead of instantiating a specific class:

```javascript
var myHandler = XhrManager.createXhrHandler();
var callback = {
  success: function(responseText) {
    alert('Success: ' + responseText);
  },
  failure: function(statusCode) {
    alert('Failure: ' + statusCode);
  }
};
myHandler.request('GET', 'script.php', callback);
```

All objects returned from the createXhrHandler method respond to the needed methods. And since they all inherit from SimpleHandler, you can implement the complicated createXhrObject method only once and have all of the classes use it. You are also able to reuse SimpleHandler’s request method from several places within OffineHandler, further reusing existing code.
The isOffline and isHighLatency methods are omitted here for simplicity. To actually implement them, you would need to first create a method that executes scheduled asynchronous requests with setTimeout and logs their round-trip time. The isOffline method would return false if any of these requests return successfully, and true otherwise. The isHighLatency method would check the times of the returned requests and return true or false based on how long they take. The implementation of these methods is nontrivial and isn’t covered here.

## Example: RSS Reader

Now you will create a widget that displays the latest entries from an RSS feed on a web page. Instead of writing the entire thing from scratch, you decide to reuse some modules that have already been created, such as the XHR handler from the previous example. The end result is an RSS reader object comprised of several member objects: an XHR handler object, a display object, and a configuration object.
You only want to interact with the RSS container object, so you use a factory to instantiate each of these external objects and link them together into a single RSS reader object. The benefit of using a factory method to do this is that you can create the RSS reader class without tightly coupling any of the member objects to it. You are able to use any display module that implements the needed methods, so there is no point in making the class dependant on a single type of display class.

The factory method allows you to swap out any of the modules whenever you like, at either development time or run-time. The programmers using the API are still given a complete RSS reader object, with all of the member objects instantiated and configured, but all of the classes involved are loosely coupled and can therefore be swapped at will.
Let’s first take a look at the classes that will be instantiated within the factory method. You have already seen the XHR handler classes; this example uses the XhrManager.createXhrHandler method to create the handler object. Next is the display class. It needs to implement several methods in order to be used in the RSS reader class. Here is one that responds to those needed methods and uses an unordered list to wrap the output:

```javascript
/* DisplayModule interface. */
var DisplayModule = new Interface('DisplayModule', [
  'append',
  'remove',
  'clear'
]);
/* ListDisplay class. */
var ListDisplay = function(id, parent) {
  // implements DisplayModule
  this.list = document.createElement('ul');
  this.list.id = id;
  parent.appendChild(this.list);
};
ListDisplay.prototype = {
  append: function(text) {
    var newEl = document.createElement('li');
    this.list.appendChild(newEl);
    newEl.innerHTML = text;
    return newEl;
  },
  remove: function(el) {
    this.list.removeChild(el);
  },
  clear: function() {
    this.list.innerHTML = '';
  }
};
```

Next you need the configuration object. This is simply an object literal with some settings that are used by the reader class and its member objects:

```javascript
/* Configuration object. */
var conf = {
  id: 'cnn-top-stories',
  feedUrl: 'http://rss.cnn.com/rss/cnn_topstories.rss',
  updateInterval: 60, // In seconds.
  parent: $('feed-readers')
};
```

The class that leverages each of these other classes is called FeedReader. It uses the XHR handler to get the XML from the RSS feed, an internal method to parse it, and then the display module to output it to the page:

```javascript
/* FeedReader class. */
var FeedReader = function(display, xhrHandler, conf) {
  this.display = display;
  this.xhrHandler = xhrHandler;
  this.conf = conf;
  this.startUpdates();
};
FeedReader.prototype = {
  fetchFeed: function() {
    var that = this;
    var callback = {
      success: function(text, xml) {
        that.parseFeed(text, xml);
      },
      failure: function(status) {
        that.showError(status);
      }
    };
    this.xhrHandler.request(
      'GET',
      'feedProxy.php?feed=' + this.conf.feedUrl,
      callback
    );
  },
  parseFeed: function(responseText, responseXML) {
    this.display.clear();
    var items = responseXML.getElementsByTagName('item');
    for (var i = 0, len = items.length; i < len; i++) {
      var title = items[i].getElementsByTagName('title')[0];
      var link = items[i].getElementsByTagName('link')[0];
      this.display.append(
        '<a href="' +
          link.firstChild.data +
          '">' +
          title.firstChild.data +
          '</a>'
      );
    }
  },
  showError: function(statusCode) {
    this.display.clear();
    this.display.append('Error fetching feed.');
  },
  stopUpdates: function() {
    clearInterval(this.interval);
  },
  startUpdates: function() {
    this.fetchFeed();
    var that = this;
    this.interval = setInterval(function() {
      that.fetchFeed();
    }, this.conf.updateInterval * 1000);
  }
};
```

The feedProxy.php script used in the XHR request is a proxy that allows fetching data from external domains without running up against JavaScript’s same-domain restriction. An open proxy, which will fetch data from any URL given to it, leaves you open to abuse and should be avoided. When using proxies like this, be sure to hard-code a whitelist of URLs that should be allowed, and reject all others.
That only leaves one remaining part: the factory method that pieces all of these classes and objects together. It is implemented here as a simple factory:

```javascript
/* FeedManager namespace. */
var FeedManager = {
  createFeedReader: function(conf) {
    var displayModule = new ListDisplay(conf.id + '-display', conf.parent);
    Interface.ensureImplements(displayModule, DisplayModule);
    var xhrHandler = XhrManager.createXhrHandler();
    Interface.ensureImplements(xhrHandler, AjaxHandler);
    return new FeedReader(displayModule, xhrHandler, conf);
  }
};
```

It instantiates the needed modules, ensures that they implement the correct methods, and then passes them to the FeedReader constructor.
What is the gain from the factory method in this example? It is possible for a programmer using this API to create a FeedReader object manually, without the FeedManager.createFeedReader method. But using the factory method encapsulates the complex setup required for this class, as well as ensures that the member objects implement the needed interface. It also centralizes the places where you hard-code the particular modules you are using: ListDisplay and XhrManager. createXhrHandler. You could just as easily use ParagraphDisplay and QueuedHandler tomorrow, and you would only have to change the code within the factory method. You could also add code to select from the available modules at run-time, as with the XHR handler example. That being said, this example best illustrates the “abstract many small objects into one large object” principle. It uses the factory pattern to perform the setups for all of the needed objects and then returns the large container object, FeedReader. A working version of this code, embedded in a web page, is in the Chapter 7 code examples on the book’s website, http://jsdesignpatterns.com/.

## Benefits of the Factory Pattern

The main benefit to using the factory pattern is that you can decouple objects. Using a factory method instead of the new keyword and a concrete class allows you to centralize all of the instantiation code in one location. This makes it much easier to swap classes, or to assign classes dynamically at run-time. It also allows for greater flexibility when subclassing. **The factory pattern allows you to create an abstract parent class, and then implement the factory method in the subclasses.** Because of this, you can defer instantiation of the member objects to the more specialized subclasses.
All of these benefits are related to two of the object-oriented design principles: `make your objects loosely coupled, and prevent code duplication`. By instantiating classes within a method, you are removing code duplication. You are taking out a concrete implementation and replacing it with a call to an interface. These are all positive steps toward creating modular code.

## Drawbacks of the Factory Pattern

It’s tempting to try to use factory methods everywhere, instead of normal constructors, but this is rarely useful. **Factory methods shouldn’t be used when there isn’t any chance of swapping out a different class or when you don’t need to select interchangeable classes at run-time.** Most class instantiation is better done in the open, with the new keyword and a constructor. This makes your code simpler and easier to follow; instead of having to track down a factory method to see what class was instantiated, you can see immediately what constructor was called. Factory methods can be incredibly useful when they are needed, but be sure you don’t overuse them. If in doubt, don’t use them; you can always refactor your code later to use the factory pattern.

## Summary

In this chapter, we discussed the simple factory and the factory pattern. Using a bicycle shop as an example, we illustrated the differences between the two; the simple factory encapsulates instantiation, typically in a separate class or object, while the true factory pattern implements an abstract factory method and defers instantiation to subclasses. There are several well-defined situations where this pattern can be used. Chief among those is when the type of class being instantiated is known only at run-time, not at development time. It is also useful when you have many related objects with complex setup costs, or when you want to create a class with member objects but still keep them relatively decoupled. The factory pattern shouldn’t be blindly used for every instantiation, but when properly applied, it can be a very powerful tool for the JavaScript programmer.
