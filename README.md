This repo holds the API specifications for AMD and some APIs that are strongly related to AMD.

* [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md): The Asynchronous Module Definition. The primary building block for referencing and defining modular JS code.
* [require](https://github.com/amdjs/amdjs-api/blob/master/require.md): An API for the require() function that allows dynamic, asynchronous loading of modules, and for resolving some module ID-based strings to file paths.
* [Loader Plugins](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md): Loader plugins extend an AMD implementation by allowing loading of resources that are not traditional JavaScript dependencies.
* [Common-Config](https://github.com/amdjs/amdjs-api/blob/master/CommonConfig.md): Optional common configuration. If a loader supports functionality that matches capabilities specified in these configuration values, these structures should be used to allow easier interop with other loaders.

Some documents on the wiki that are not actual APIs but information related to AMD use:

* [jQuery-and-AMD](https://github.com/amdjs/amdjs-api/wiki/jQuery-and-AMD): Describes jQuery 1.7+ support for registering as an AMD module, but only if define.amd.jQuery is set to true.

Documents related to AMD and its use in other libraries:

* [AMD Forks](https://github.com/amdjs/amdjs-api/wiki/AMD-Forks): A listing of forks for libraries that have AMD registration built in.

The [wiki](https://github.com/amdjs/amdjs-api/wiki) for this project also contains copies of the information in this repo. Those pages are kept for historical reasons/links, but the contents of this repo are considered the definitive source.

---

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:vcard="http://www.w3.org/2001/vcard-rdf/3.0#">
  <a rel="license"
     href="http://creativecommons.org/publicdomain/zero/1.0/">
    <img src="http://i.creativecommons.org/p/zero/1.0/88x31.png" style="border-style: none;" alt="CC0" />
  </a>
  <br />
  To the extent possible under law,
  <a rel="dct:publisher"
     href="https://github.com/amdjs">
    <span property="dct:title">the amdjs organization</span></a>
  has waived all copyright and related or neighboring rights to
  <span property="dct:title">AMD specifications</span>.
This work is published from:
<span property="vcard:Country" datatype="dct:ISO3166"
      content="US" about="https://github.com/amdjs">
  United States</span>.
</p>