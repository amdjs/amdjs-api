# AMD

The Asynchronous Module Definition (**AMD**) API specifies a mechanism for defining modules such that the module and its dependencies can be asynchronously loaded. This is particularly well suited for the browser environment where synchronous loading of modules incurs performance, usability, debugging, and cross-domain access problems.

It is unrelated to the technology company [AMD](http://en.wikipedia.org/wiki/Advanced_Micro_Devices) and the processors it makes.

* [Tests](https://github.com/amdjs/amdjs-tests)
* [Discussion](https://groups.google.com/group/amd-implement)

# API Specification

## define() function <a name="define"></a>

The specification defines a single function "define" that is available as a free variable or a global variable. The signature of the function:

```javascript
    define(id?, dependencies?, factory);
```

### id <a name="define-id"></a>

The first argument, id, is a string literal. It specifies the id of the module being defined. This argument is optional, and if it is not present, the module id should default to the id of the module that the loader was requesting for the given response script. When present, the module id MUST be a "top-level" or absolute id (relative ids are not allowed).

#### module id format <a name="define-id-notes"></a>

Module ids can be used to identify the module being defined, and they are also used in the dependency array argument. Module ids in AMD are a superset of what is allowed in [CommonJS Module Identifiers](http://wiki.commonjs.org/wiki/Modules/1.1.1#Module_Identifiers). Quoting from that page:

* A module identifier is a String of "terms" delimited by forward slashes.
* A term must be a camelCase identifier, ".", or "..".
* Module identifiers may not have file-name extensions like ".js".
* Module identifiers may be "relative" or "top-level". A module identifier is "relative" if the first term is "." or "..".
* Top-level identifiers are resolved off the conceptual module name space root.
* Relative identifiers are resolved relative to the identifier of the module in which "require" is written and called.

The CommonJS module id properties quoted above are normally used for JavaScript modules.

Relative module ID resolution examples:

* if module `"a/b/c"` asks for `"../d"`, that resolves to `"a/d"`
* if module `"a/b/c"` asks for `"./e"`, that resoles to `"a/b/e"`

If [Loader Plugins](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md) are supported in the AMD implementation, then "!" is used to separate the loader plugin's module id from the plugin's resource id. Since plugin resource ids can be extremely free-form, most characters should be allowed for plugin resource ids.

### dependencies <a name="define-dependencies"></a>

The second argument, dependencies, is an array literal of the module ids that are dependencies required by the module that is being defined. The dependencies must be resolved prior to the execution of the module factory function, and the resolved values should be passed as arguments to the factory function with argument positions corresponding to indexes in the dependencies array.

The dependencies ids may be relative ids, and should be resolved relative to the module being defined. In other words, relative ids are resolved relative to the module's id, and not the path used to find the module's id.

This specification defines three special dependency names that have a distinct resolution. If the value of "require", "exports", or "module" appear in the dependency list, the argument should be resolved to the corresponding free variable as defined by the CommonJS modules specification.

The dependencies argument is optional. If omitted, it should default to ["require", "exports", "module"]. However, if the factory function's arity (length property) is less than 3, then the loader may choose to only call the factory with the number of arguments corresponding to the function's arity or length.

### factory <a name="define-factory"></a>

The third argument, factory, is a function that should be executed to instantiate the module or an object. If the factory is a function it should only be executed once. If the factory argument is an object, that object should be assigned as the exported value of the module.

If the factory function returns a value (an object, function, or any value that coerces to true), then that value should be assigned as the exported value for the module.

#### Simplified CommonJS wrapping <a name="commonjs-wrap"></a>

If the dependencies argument is omitted, the module loader MAY choose to scan the factory function for dependencies in the form of require statements (literally in the form of require("module-id")). The first argument must literally be named require for this to work.

In some situations module loaders may choose not to scan for dependencies due to code size limitations or lack of toString support on functions (Opera Mobile is known to lack toString support for functions).

If the dependencies argument is present, the module loader SHOULD NOT scan for dependencies within the factory function.

## define.amd property <a name="defineAmd"></a>

To allow a clear indicator that a global define function (as needed for script src browser loading) conforms to the AMD API, any global define function SHOULD have a property called "amd" whose value is an object. This helps avoid conflict with any other existing JavaScript code that could have defined a define() function that does not conform to the AMD API.

The properties inside the define.amd object are not specified at this time. It can be used by implementers who want to inform of other capabilities beyond the basic API that the implementation supports.

Existence of the define.amd property with an object value indicates conformance with this API. If there is another version of the API, it will likely define another property, like define.amd2, to indicate implementations that conform to that version of the API.

An example of how it may be defined for an implementation that allows loading more than one version of a module in an environment:

```javascript
    define.amd = {
      multiversion: true
    };
```
The minimum definition:

```javascript
    define.amd = {};
```

## Transporting more than one module at a time <a name="transporting"></a>

Multiple define calls can be made within a single script. The order of the define calls SHOULD NOT be significant. Earlier module definitions may specify dependencies that are defined later in the same script. It is the responsibility of the module loader to defer loading unresolved dependencies until the entire script is loaded to prevent unnecessary requests.

# Examples <a name="examples"></a>

## Using require and exports

Sets up the module with ID of "alpha", that uses require, exports and the module with ID of "beta":

```javascript
   define("alpha", ["require", "exports", "beta"], function (require, exports, beta) {
       exports.verb = function() {
           return beta.verb();
           //Or:
           return require("beta").verb();
       }
   });
```

An anonymous module that returns an object literal:

```javascript
   define(["alpha"], function (alpha) {
       return {
         verb: function(){
           return alpha.verb() + 2;
         }
       };
   });
```

A dependency-free module can define a direct object literal:

```javascript
   define({
     add: function(x, y){
       return x + y;
     }
   });
```

A module defined using the simplified CommonJS wrapping:

```javascript
   define(function (require, exports, module) {
     var a = require('a'),
         b = require('b');

     exports.action = function () {};
   });
```

# Global Variables <a name="global"></a>

This specification reserves the global variable "define" for use in implementing this specification, the package metadata asynchronous definition API and is reserved for other future CommonJS APIs. Module loaders SHOULD not add additional methods or properties to this function.

This specification reserves the global variable "require" for use by module loaders. Module loaders are free to use this global variable as they see fit. They may use the variable and add any properties or functions to it as desired for module loader specific functionality. They can also choose not to use "require" as well.

# Usage notes <a name="usage"></a>

It is recommended that define calls be in the literal form of 'define(...)' in order to work properly with static analysis tools (like build tools).

# Relation to CommonJS <a name="commonjs-relation"></a>

A version of this API started on the CommonJS wiki as a transport format, as Modules Transport/C, but it changed over time to also include a module definition API. Consensus was not reached on the CommonJS list about recommending this API as a module definition API. The API was transferred over to its own wiki and discussion group.

AMD can be used as a transport format for CommonJS modules as long as the CommonJS module does not use computed, synchronous require('') calls. CommonJS code that use computed synchronous require('') code can be converted to use the callback-style [require](https://github.com/amdjs/amdjs-api/blob/master/require.md) supported in most AMD loaders.
