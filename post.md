If you feel like "no matter what you do, there is always something funky in your JavaScript code", I would bet that your Module strategy is not working out so well.

The importance of adopting a proper JavaScript Module strategy is often underestimated as a preference contest, so it is indeed important to really understand your needs. This article exposes the foundations of different JavaScript Module strategies such as ad hoc, CommonJS, AMD and ES6 modules, and how to get started with ES6 modules right now.

## JavaScript Module 101

In brief terms, JavaScript Modules were created in order to apply some classic Object Orientation ideas when building components, once the current JavaScript language support for those ideas isn't as explicit as in other languages as C++, Java and Ruby.

In order to build a module as a special type of object, which strongly needs to leverage encapsulation, we need to add support for declaring private/public attributes and methods inside a single object. Such encapsulation is achieved through **function closures**, taking advantage of the **function scope** to publicly disclose only what is necessary through the return of the function. Look at the following example for an ad hoc module implementation:

```javascript
// zoo.js
var Zoo = (function() { 
  var getBarkStyle = function(isHowler) {
    return isHowler? 'woooooow!': 'woof, woof!';
  }; 
  var Dog = function(name, breed) {
    this.bark = function() {
      return name + ': ' + getBarkStyle(breed === 'husky');
    };
  };
  var Wolf = function(name) {
    this.bark = function() {
      return name + ': ' + getBarkStyle(true);
    };
  };
  return {
    Dog: Dog,
    Wolf: Wolf
  };
})();
```

```javascript
// main.js
var myDog = new Zoo.Dog('Sherlock', 'beagle');
console.log(myDog.bark()); // Sherlock: woof, woof!

var myWolf = new Zoo.Wolf('Werewolf');
console.log(myWolf.bark()); // Werewolf: woooooow!
```

In the above example, we have built the module ```Zoo``` which only publicly exposes the functions ```Dog``` and ```Wolf```, keeping the function ```getBarkStyle``` as private to the module.

I have put together a full [live example of the ad-hoc module](http://tiagorg.com/js-modules/ad-hoc/index.html). You can also [check the source code](https://github.com/tiagorg/js-modules/tree/gh-pages/ad-hoc).

## Ok, modules are cool, but why should I use them?

Just to name a few reasons why every JavaScript developer should use modules as much as possible:

- Writing scattered global JavaScript code is bad for performance, terrible for reusability, awkward for readability, painful for side-effects and horrible for code organization.
- In JavaScript, literal objects' attributes and methods are all public, making it impossible  to conceal internal details of objects, which is a real demand for Components, Features, Subsystems and Façades.
- A module can be delivered as a dependency for other modules, leveraging a composite architecture of reusable components, when properly implemented.
- Modules can also be packaged and deployed separately from each other, allowing changes on a particular module to be properly isolated from everything else, therefore mitigating those dreaded side-effects known as the "butterfly effect" (yes, just like in the movie).
- Splitting your global code into modules is the first step on bringing cohesion up and coupling down.

## To be or not to be, CommonJS or AMD?

Have another look on the ad hoc module example above. Since we are defining 2 files, we are still writing and reading the variable ```Zoo``` into the global JavaScript context. This is definitely not recommended, once it is:
- fragile (because any next code can modify your module definition),
- not scalable (if you need to define 100 modules, all of them will be loaded on the global JavaScript context, even if you actually consume just 1 out of those 100 modules, making it really bad for performance),
- counter-productive (if you have dependencies on your modules you will have to manually port them all over if you intend to use your module in another app).

Since modules are definitely a good idea, but ad hoc modules are not that solid, many developers started to elaborate around them, striving for a module standard that would overcome the setbacks above. After some coming and going, two module standards have gained some momentum:

1) [**CommonJS**](http://www.commonjs.org/) is a standard for synchronous modules, adopted as the official module format for [Node.js](https://nodejs.org) and [NPM](http://npmjs.com) components. On this format, your module file will publicly expose whatever is assigned to ```module.exports``` while everything else is private. Check out the following example:

```javascript
// zoo.js
var getBarkStyle = function(isHowler) {
  return isHowler? 'woooooow!': 'woof, woof!';
}; 
var Dog = function(name, breed) {
  this.bark = function() {
    return name + ': ' + getBarkStyle(breed === 'husky');
  };
};
var Wolf = function(name) {
  this.bark = function() {
    return name + ': ' + getBarkStyle(true);
  };
};
 
module.exports = {
  Dog: Dog,
  Wolf: Wolf
};
```

Note that the public content is returned at once through ```module.exports```.

```javascript
// main.js
var Zoo = require('./zoo');

var myDog = new Zoo.Dog('Sherlock', 'beagle');
console.log(myDog.bark()); // Sherlock: woof, woof!

var myWolf = new Zoo.Wolf('Werewolf');
console.log(myWolf.bark()); // Werewolf: woooooow!
```

This code is also available as a [live example of a Common.js module](http://tiagorg.com/js-modules/commonjs/index.html). You can also [check the source code](https://github.com/tiagorg/js-modules/tree/gh-pages/commonjs). In this example I am using [Browserify](http://browserify.org) to transform the source code on a browser bundle.

2) [**AMD**](https://github.com/amdjs/amdjs-api) is a standard for asynchronous modules, which is specially interesting for client-side JavaScript. On this format, your module file will publicly expose whatever is being returned on the callback function, just like our first ad hoc example. The following example uses the quintessential AMD implementation, [Require.js](http://requirejs.org):

```javascript
// zoo.js
define('zoo', [], function() {
  var getBarkStyle = function (isHowler) {
    return isHowler? 'woooooow!': 'woof, woof!';
  }; 
  var Dog = function (name, breed) {
    this.bark = function() {
      return name + ': ' + getBarkStyle(breed === 'husky');
    };
  };
  var Wolf = function (name) {
    this.bark = function() {
      return name + ': ' + getBarkStyle(true);
    };
  };
  return {
    Dog: Dog,
    Wolf: Wolf
  };
});
```

Note that the public content is returned at once through the function return, just like the ad hoc implementation.

```javascript
// main.js
require(['zoo'], function(Zoo) {
  var myDog = new Zoo.Dog('Sherlock', 'beagle');
  console.log(myDog.bark()); // Sherlock: woof, woof!

  var myWolf = new Zoo.Wolf('Werewolf');
  console.log(myWolf.bark()); // Werewolf: woooooow!
});
```

This code is also available as a [live example of an AMD module](http://tiagorg.com/js-modules/amd/index.html). You can also [check the source code](https://github.com/tiagorg/js-modules/tree/gh-pages/amd).

PS: As you might suppose, you will run into both standards quite frequently, and there will be times you might want to use a CommonJS module on an AMD component and vice-versa. Please allow me to spoil your surprise: they are not naturally compatible!

Nevertheless, a number of approaches are there to provide such compatibility,  striving to come up with a way to write your module just once and have it working on both standards. Great examples are [UMD](https://github.com/umdjs/umd), [SystemJS](https://github.com/systemjs/systemjs) and [uRequire](http://urequire.org).

## Now, forget about that. ES6 is right around the corner!

Earlier I've affirmed that the JavaScript language support for Modules isn't much explicit on its current version (officially known as ECMAScript 5 or just ES5). However, it turns out that JavaScript Modules **just have become explicit**!

The upcoming version of JavaScript (ECMAScript 6 or ES6) offers native support for modules in a compact and effective way, quite a bit similar to CommonJS. See how it will look like:
    
```javascript
// zoo.js
var getBarkStyle = function(isHowler) {
  return isHowler? 'woooooow!': 'woof, woof!';
}; 
export function Dog(name, breed) {
  this.bark = function() {
    return name + ': ' + getBarkStyle(breed === 'husky');
  };
}
export function Wolf(name) {
  this.bark = function() {
    return name + ': ' + getBarkStyle(true);
  };
}
```

Note that now we can have more than one ```export``` per module. This way, the client code can choose which functions it wants to ```import``` from the module:

```javascript
// main.js
import { Dog, Wolf } from './zoo';

var myDog = new Dog('Sherlock', 'beagle');
console.log(myDog.bark()); // Sherlock: woof, woof!

var myWolf = new Wolf('Werewolf');
console.log(myWolf.bark()); // Werewolf: woooooow!
```    

This code is also available as a [live example of an ES6 module](http://tiagorg.com/js-modules/es6/index.html). You can also [check the source code](https://github.com/tiagorg/js-modules/tree/gh-pages/es6). 

Not only ES6 has brought modules, but it also brought a solution for the CommonJS vs AMD battle! According to [Dr. Axel Rauschmayer's article](http://www.2ality.com/2014/09/es6-modules-final.html), ES6 modules will support both synchronous and asynchronous loading within the same syntax.

One more good news: this syntax is confirmed to be finalized, i.e., syntax changes aren't expected for ES6 Modules. 

## Babel for the rescue

Using ES6 modules sound thrilling, but since they are targeted for June 2015, what can you do before it gets released and adopted by all the major browsers and devices?

Enter [Babel](https://babeljs.io/), my personal choice of ES6 to ES5 transpiler (a transpiler is a source-to-source compiler, i.e., it will transform ES6 source code into ES5 source code). Babel works for both client-side and server-side JavaScript.

To implement the example above, I am using the [Babelify plugin](https://github.com/babel/babelify) over Browserify to transform the ES6 source code directly into a ES5 browser bundle. However, it would be just as simple to transpile the files individually, if you are curious to know how the ES5 source would be. All you would need to do is:

```
npm install -g babel
babel --modules common zoo.js -o zoo-commonjs.es6
babel --modules amd zoo.js -o zoo-amd.es6
```

Since in a real project you are hopefully not transpiling files manually, but instead adopting some build system to do that for you, you will be very happy to know that Babel supports most of builders and assets pipelines, even Rails is on the list! Check out how Babel integrates with your [favorite asset pipeline](https://babeljs.io/docs/using-babel/#build-systems).

## Who you gonna call?

Modules are a big deal for sure. That is why the JavaScript community is so concerned on striving for everyone to use them as seamless as possible, as you can see with the ES6 modules initiative.

However, we still have to take into consideration the plethora of existing code which is too big to be rewritten into ES6 soon enough. Depending on the size, complexity and risk of the project, this might never happen. Still, even if all the major browsers start supporting ES6 this year, it would take a handful of years for most of the worldwide population (or at least your clients) to be effectively using browsers supporting ES6. Just as a sad-trombone example, IE 8 was released exactly 6 years ago.

Anyhow, the other way around is totally feasible and recommended. Writing new code on ES6 with a good transpiler for now is something worth taking a deep look at. The benefits are great: more concise syntax, great support for the current Module strategies, and the best thing: you are embracing the future of JavaScript!

When in doubt, remember that Fortune favors the brave. Or just remember Ghostbusters: I ain't afraid of no ghost.