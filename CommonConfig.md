# Common Config

## DRAFT, Not completed yet

An AMD loader is not **required** to implement all of these configuration values, but if it does provide a capability that is satisfied by these configuration values, it should use these configuration names, structures and behavior.

Some common terms:

* **module ID prefix**: means part of a module ID that starts from the beginning of a string, and stop on any of the slash delimiters, up to and including the full name. So, for the complete module ID `some/very/long/name`, these are all module ID prefixes:
    * `some`
    * `some/very`
    * `some/very/long`
    * `some/very/long/name`
* **first segment of a module ID prefix**: the part of a module ID up to the first slash, or the whole name if there are no slashes in the module ID. For the example above, `some` is the first segment.

## baseUrl <a name="baseUrl"></a>

String: Indicates the root used for ID-to-path resolutions. Relative paths are
relative to the current working directory. In web browsers, the current working
directory is the directory containing the web page running the script.

Example:

```javascript
{
    baseUrl: './foo/bar'
}
```

## paths <a name="paths"></a>

Object. For specifying a path for a the given module ID prefix.

A property in the `paths` object is an absolute module ID prefix, and the value
can either be:

* String: a path value to use for the module ID prefix.  If it is a relative path, it it relative to baseUrl. It can be an absolute path, like `/top/level/dir` or `//top/level/dir` or `http://some.domain.com/top/level/dir`.
* Array: Optional. If the module loader provides a failover capability, the loader can allow an Array of String path values. If the loader cannot load the module at the first path in the Array, it can try the next path in the Array, and so on.

## packages <a name="packages"></a>

Array of package configuration (packageConfig) objects. Package configuration
is for traditional CommonJS packages, which has different path lookup rules
than the default ID-to-path lookup rules used by an AMD loader.

**Default lookup rules** are ,`baseUrl +
'module/id' + .js`, where **paths** config can be used to map part of
`'module/id'` to another path.

**Package lookup rules** use a special lookup rule when 'packageName' is used.
For the "main module" of a package, instead of it being
`baseUrl + 'packageName' + '.js'`, it is:

    baseUrl + (packageConfig.location || 'packageName') + '/' + (packageConfig.main || 'main') + '.js'

For 'packageName/otherId', the rule is similar to paths, but packageConfig is
used instead:

    baseUrl + (packageConfig.location || 'packageName') + '/' + 'otherId' + '.js'

The package configuration object can either be:

* String: The first segment of an absolute module ID prefix, which indicates the package name. So, for a module ID of `some/thing/here`, the first segment of the module ID is `some`. In that example `some` is the package name, and the value used for this config value.
* Object: a configuration object that can have the following properties:
    * **name**: String. first segment of an absolute module ID (see String section immediately above).
    * **location**: String. Optional.
    * **main**: String. Optional. Default value is "main". The module ID to use inside the package for the "main module". The value may have a ".js" at the end of it. In that case, the ".js" should be ignored by the loader. This is done to accommodate a package config style in use by some node packages.

```javascript
    packages: [
        {
            name: 'dojo',
            location: 'dojo/1.7.1',
            main:'main'
        }
    ]
```

## map <a name="map"></a>

Object. Specifies for a given module ID prefix, what module ID prefix to use in place of another module ID prefix. For example, how to express "when 'bar' asks for module ID 'foo', actually use module ID 'foo1.2'".

This sort of capability is important for larger projects which may have two sets of modules that need to use two different versions of 'foo', but they still need to cooperate with each other.

This is different from `paths` config. `paths` is only for setting up root paths for module IDs, not for mapping one module ID to another one.

```javascript
{
    map: {
        'some/newmodule': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
}
```

If the modules are laid out on disk like this:

* foo1.0.js
* foo1.2.js
* some/
    * newmodule.js
    * oldmodule.js

When 'some/newmodule' asks for 'foo' it will get the 'foo1.2' module from foo1.2.js, and when 'some/oldmodule' asks for 'foo' it will get the 'foo1.0' module from foo1.0.js file.

This feature only works well for scripts that are real AMD modules that call define() and register as anonymous modules. If named modules are being used, it will not work.

Any module ID prefix can be used for the map properties, and the mappings can map to any other module ID prefix. The more specific module ID prefixes are chosen when resolving which map value to use.

Example:

```javascript
{
    map: {
        'some/newmodule': {
            'foo': 'foo2',
            'foo/bar': 'foo1.2/bar3'
        },
        'some/oldmodule': {
            'foo/bar/baz': 'foo1.0/bar/baz2'
        }
    }
}
```

If 'some/module/sub' asks for 'foo' it gets 'foo2'. If 'some/module/sub' asks for 'foo/bar' it gets 'foo1.2/bar3'.

There is a "*" map value which means "for all modules loaded, use this map config". If there is a more specific map config, that one will take precedence over the star config.

Example:

```javascript
{
    map: {
        '*': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
}
```

In this example if 'some/oldmodule' asks for 'foo', it will get 'foo1.0', where if any other module who asks for 'foo' will get 'foo1.2'.

## config <a name="config"></a>

A configuration object available to modules that match the absolute module ID listed in the config object. Modules with a matching module ID can access the configuration object via `module.config`, which is an Object value. The Object value is mutable, and an Object value is always returned. If there is no explicit config set, calling `module.config()` will return an empty object, and not undefined.

Example:

```javascript
{
    config: {
        'some/module/id': {
            limit: 40
        }
    }
}
```

For the module that resolves to the absolute module ID of 'some/module/id', `module.config.limit === 40`.

Modules can access this config by asking for `module.config()` when using the special `module` dependency:

```javascript
//sugared CommonJS form
define(function (require, exports, module) {
    module.config();
});

//or
define(['module'], function (module) {
    module.config();
});
```

## shim <a name="shim"></a>

shim config allows the use of non-AMD, browser-globals scripts by specifying the dependencies and the global value to use as the exports object. It also allows running an "init" function whose result will be used as the export value for that module ID.

Example for the module ID 'some/thing' that depends on an 'a' and 'b' and the global value at 'some.thing' with the string 'another' appended should be used as the module's export value:

```javascript
{
    shim: {
        'some/thing': {
            deps: ['a', 'b'],
            exports: 'some.thing',
            init: function (a, b) {
                return some.thing + 'another';
            }
        }
    }
}
```

* **deps**: Array of string module IDs. Optional. The dependencies that need to be executed before the script for 'some/thing' is executed.
* **exports**: String. Optional. Represents the global property to use for the exports value for the shimmed script. Dot values are supported. So, a value of `"some.thing"` means "use the value at global.some.thing".
* **init**: Function. Optional. Called after the shimmed script has executed. It is passed any module values for the dependencies specified in "deps". If the return value of this function is not `undefined`, then the value is used as the export, and takes precedence over any specified `"exports"` value.

If "exports" and "init" are not needed for the script (common for plugins for other libraries, like jQuery plugins), then the shim config can just be the array that corresponds to the "deps" setting:

```javascript
{
    shim: {
        'another/thing': ['c', 'd']
    }
}
```

On builds:

While not required to be conformant with this API, it is suggested that a define() wrapper is **not** placed around the shimmed script in a build/concatenation scenario. Many shimmed scripts declare their globals via a a simple `var globalValue = {}` in the script, and wrapping that code in a define() wrapper will result in that value not being global. If this shimmed script is used in combination with other shimmed scripts that depend on that global being there, there will likely be problems.

So if a define() wrapper is not used around the shimmed script, the "deps" for the shimmed script should be placed in the built file before the shimmed script.

Notes about the module ID used in the shim config ('some/thing' in the example):

* other config, like baseUrl and paths, can be used in resolving the value for 'some/thing'.
* It is a full, absolute module ID, not a module ID prefix.

Example of shim config for two commonly used libraries, Backbone and underscore, and a jQuery plugin in a file named 'jquery.rotator.js':

```javascript
{
    shim: {
        //This version just specifies the
        //dependency for the jQuery plugin.
        //Some implementations may be able to do
        //better error correction in older IE if
        //you specify the exports string value,
        //which in this case would be
        //exports: 'jQuery.fn.rotator'
        'jquery.rotator': ['jquery'],

        underscore: {
            exports: '_'
        },

        backbone: {
            deps: ['jquery', 'underscore'],
            exports: 'Backbone'
        }
    }
}
```
