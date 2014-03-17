# require

The **require** function in an [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) implementation differs from the traditional [CommonJS require](http://wiki.commonjs.org/wiki/Modules/1.1.1#Require). There is a strong need for a callback-based require, since dynamically computed dependencies may load asynchronously.

* [Tests](https://github.com/amdjs/amdjs-tests)
* [Discussion](https://groups.google.com/group/amd-implement)

# API Specification

## Local vs Global require <a name="localGlobal"></a>

The **local** require is the require function passed to an [AMD define factory function](https://github.com/amdjs/amdjs-api/wiki/AMD#wiki-define). Examples:

```javascript
    define(['require'], function (require) {
        //the require in here is a local require.
    });

    define(function (require, exports, module) {
        //the require in here is a local require.
    });
```
The local require MAY support other APIs specific to the implementation.

A global require() function is one that is available in the global scope, like define(). An implementation is not required to implement a global require, but if it does, the behavior of global require is similar to the behavior of the local require() function, with the following qualifications:

* module IDs are treated as absolute instead of being resolved relatively to another module's ID.
* only the async, callback-style of require is expected to work for interoperation, since it may not be possible to synchronously load a module via require(String) from the top level.

There is often an implementation-dependent API that will kick off module loading; if interoperability with several loaders is needed, the global require() should be used to load the top level modules instead.

## require(String) <a name="requireString"></a>

Synchronously returns the module export for the module ID represented by the String argument. Based on the [CommonJS Modules 1.1.1 require](http://wiki.commonjs.org/wiki/Modules/1.1.1#Require).

It MUST throw an error if the module has not already been loaded and evaluated. In particular, the synchronous call to require MUST NOT try to dynamically fetch the module if it is not already loaded, but throw an error instead.

Dependencies can be found in an AMD module when this form of define() is used:

```javascript
    define(function (require) {
        var a = require('a');
    });
```

The define factory function can be parsed for require('') calls (for instance by using a language parser or using Function.prototype.toString() and regexps) to find dependencies, load and execute the dependencies, then run the code above. In that manner, the require('a') call can return the module value.

## require(Array, Function) <a name="requireArray"></a>

The Array is an array of String module IDs. The modules that are represented by the module IDs should be retrieved and once all the modules for those module IDs are available, the Function callback is called, passing the modules in the same order as the their IDs in the Array argument.

Example:

```javascript
    define(function (require) {
        require(['a', 'b'], function (a, b) {
            //modules a and b are now available for use.
        });
    });
```

## require.toUrl(String) <a name="toUrl"></a>

Converts a String that is of the form **[module ID] + '.extension'** to an URL path. require.toUrl resolves the module ID part of the string using its normal module ID-to-path resolution rules, except it does not resolve to a path with a ".js" extension. Then, the '.extension' part is then added to that resolved path. Example:

```javascript
    //cart.js contents:
    define (function(require) {
        //module ID part is './templates/a'
        //'.extension is '.html'
        //templatePath may end up to be something like
        //'modules/cart/templates/a.html'
        var templatePath = require.toUrl('./templates/a.html');

    });
```
