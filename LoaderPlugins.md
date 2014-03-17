# Loader Plugins

## DRAFT, Not completed yet

Loader plugins extend an [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) implementation by allowing loading of resources that are not traditional JavaScript dependencies.

The API is designed to allow running a loader plugin as part of an optimization tool so that some kinds of plugin resources to be inlined if it makes sense for that type of loader plugin to do so. Because of this capability, a loader plugin should be coded to be able to run in many kinds of JavaScript environments, like the browser, Node and/or Rhino.

* [Tests](https://github.com/amdjs/amdjs-tests)
* [Discussion](https://groups.google.com/group/amd-implement)

## Terms <a name="terms"></a>

A **plugin dependency** is a dependency expressed in AMD that is intended to be loaded by a loader plugin. It has the form:

    [Plugin Module ID]![resource ID]

where **plugin module ID** is a normal AMD module ID that indicates the JavaScript module that implements the loader plugin API, and **resource ID** is a plugin-specific string that the loader plugins knows how to parse to load the resource.

## Examples of a plugin dependencies <a name="dependencyExamples"></a>

Here is one that uses a **text** loader plugin to load an HTML template.

```javascript
    define(['text!../templates/start.html'], function (template) {
        //do something with the template text string.
    });
```

Here is another one that has a more complex resource ID structure. It is a bit of a contrived example. It just chooses the module name that is part of the resource ID that matches to the array index given by the first number in the resource ID. So the impl below would be the module represented by './b':

```javascript
    define(function (require) {
        var impl = require('index!1:./a:./b:./c');
    });
```

# API Specification

## load: function (resourceId, require, load, config) <a name="load"></a>

A function that is called to load a resource. This is the only mandatory API method that needs to be implemented for the plugin to be useful, assuming the resource IDs do not need special ID normalization (see the [normalize() method](#wiki-normalize) below for more details.

* **resourceId**: String. The resource ID that the plugin should load. This ID MUST be [normalized](#wiki-normalize).
* **require**: Function. A local **require** function to use to load other modules. This require function has some utilities on it:
    * **require.toUrl("moduleId+extension")**. See the [require.toUrl API notes](https://github.com/amdjs/amdjs-api/wiki/require#wiki-toUrl) for more information.
* **load**: Function. A function to call once the value of the resource ID has been determined. This tells the loader that the plugin is done loading the resource.
* **config**: Object, optional. A configuration object. This is a way for the optimizer and the web app to pass configuration information to the plugin. An optimization tool may set an isBuild property in the config to true if this plugin (or pluginBuilder (TODOC)) is being called as part of an optimizer build.

An example plugin that does not do anything interesting, just does a normal require to load a JS module:

```javascript
    define({
        load: function (name, req, load, config) {
            //req has the same API as require().
            req([name], function (value) {
                load(value);
            });
        }
    });
```

## normalize: function (resourceId, normalize) <a name="normalize"></a>

A function to normalize the passed-in resource ID. Normalization of an module ID normally means converting relative paths, like './some/path' or '../another/path' to be non-relative, absolute IDs. This is useful in providing optimal caching and optimization, but it only needs to be implemented if:

* the resource IDs have complex normalization
* only needed if the resource name is not a module name.

If the plugin does not implement **normalize** then the loader will assume it is something like a regular module ID and try to normalize it.

The arguments passed to normalize:

* **resourceId**: String. The resource ID to normalize.
* **normalize**: Function. A normalization function that accepts a string ID to normalize using the standard relative module normalization rules using the loader's current configuration.

An example: suppose there is an index! plugin that will load a module name given an index. This is a contrived example, just to illustrate the concept. A module may reference an index! dependency like so:

```javascript
    define(['index!2?./a:./b:./c'], function (indexResource) {
        //indexResource will be the module that corresponds to './c'.
    });
```
In this case, the normalized IDs for './a', './b', and './c' will be determined relative to the module asking for this resource. Since the loader does not know how to inspect 'index!2?./a:./b:./c' to normalize the IDs for './a', './b', and './c', it needs to ask the plugin. This is the purpose of the normalize call.

By properly normalizing the resource name, it allows the loader to cache the value effectively, and to properly build an optimized build layer in the optimizer. It is also required so that the loader can pass always pass a normalized ID to the plugin's [load](#wiki-load) method.

The index! plugin could be written like so:

```javascript
    (function () {

        //Helper function to parse the 'N?value:value:value'
        //format used in the resource name.
        function parse(name) {
            var parts = name.split('?'),
                index = parseInt(parts[0], 10),
                choices = parts[1].split(':'),
                choice = choices[index];

            return {
                index: index,
                choices: choices,
                choice: choice
            };
        }

        //Main module definition.
        define({
            normalize: function (name, normalize) {
                var parsed = parse(name),
                    choices = parsed.choices;

                //Normalize each path choice.
                for (i = 0; i < choices.length; i++) {
                    //Call the normalize() method passed in
                    //to this function to normalize each
                    //module ID.
                    choices[i] = normalize(choices[i]);
                }

                return parsed.index + '?' + choices.join(':');
            },

            load: function (name, require, load, config) {
                require([parse(name).choice], function (value) {
                    load(value);
                });
            }
        });

    }());
```

## dynamic: Boolean <a name="dynamic"></a>

If the plugin has a **dynamic** property set to true, then it means the loader MUST NOT cache the value of a normalized plugin dependency, but instead call the plugin's [load](#wiki-load) method for each instance of a plugin dependency.

This allows some plugins to provide time or per-call dependent values.