The ```browser``` field is provided by a module author as a hint to javascript bundlers or component tools when packaging modules for client side use. The field is found in a ```package.json``` file (described [here](http://browsenpm.org/package.json)) usually located at the root of a project source tree.

## terms

Below are common terms used in the rest of the document.

### server
> This is a non-dom based javascript execution environment. It usually only contains the base javascript language spec libraries and objects along with modules to communicate with OS features (available through commonjs require).

### client
> This is a browser execution environment. It may provide additional built in objects exposed in the global namespace. It is a specialized execution environment which provides builtin capabilities beyond the base javascript language spec.

### commonjs
> A require system for specifying which modules or files a particular file uses.

### bundler
> A tool which takes a plain javascript package and creates client usable files. It may include, but is not limited to: replacing modules or files with client versions (since the client may already provide the functionality), merging all the dependencies into a single file, etc.

### package.json
> Metadata information about a module. Usually found at the root of the project source tree.

### packaging
> The use of a bundler to create a file(s) suitable for running on a client.

## Overview

When a javascript module is prepared for use on a client there are two major concerns: certain features are already provided by the client, and certain features are not available. Features provided by a client can include http requests, websockets, dom manipulation. Features not available would include tcp sockets, system disk IO.

The ```browser``` field is where the module author can hint to the bundler which elements (other modules or source files) need to be replaced when packaging.

## Spec

### alternate main - basic

When you specify a single string for the ```browser``` field, it will replace ```main``` and be the module entry point. The ```main``` field specifies the entry point to the module so by replacing it, you replace the entry point when the module is packaged by a bundler for browser use.

```javascript
"browser": "./browser/specific/main.js"
```

Whenever another module ```requires``` your module by name, the bundler will load javascript from ```./browser/specific/main.js``` instead of the typical ```main``` field entry point (or index.js by default).

All paths for browser fields are relative to the ```package.json``` file location (and usually project root as a result).

### replace specific files - advanced

In many cases, there is a large amount of code which is applicable to both client and server. If is easier to just replace some files instead of creating a completely new entry point. To do this, just specify an object versus a single string.

When using an object. The left hand side (key) is the name of a module or file you wish to replace and the right side is the replacement.

```javascript
"browser": {
    "module-a": "./shims/module-a.js",
    "./server/only.js": "./shims/client-only.js"
}
```

Now when you package your code, uses of ```module-a``` will be replaced with code from ```./shims/module-a.js``` versus the module code itself and anytime ```./server/only.js``` is used, it will be replaced with ```./shims/client-only.js```

If a module you depend on already includes a ```browser``` field, then you don't have to do anything special. Whenever you require that module, the bundler SHOULD use the hint provided by the module.

### ignore a module
You can simply prevent a module or file from being loaded into a bundle by specifying a value of ```false``` for any of the keys. This is useful if you know certain codepaths will not be executed client side but find it awkward to split up or change the code structure.

```javascript
"browser": {
    "module-a": false,
    "./server/only.js": "./shims/server-only.js"
}
```

The above will cause the following to return an empty object into `a`.
```javascript
var a = require('module-a');
```

Note: The use of `false` should be restricted to only the most essential places in your app code where other browser field approaches do not work. It is discouraged but sometimes a necessary approach to blacklist a module from client packaging.

## Advantages

Using the ```browser``` field in package.json allows a module author to clearly articulate which files are innapropriate for client use and provide alternatives. It allows the module code (and subsequently dependants on the module) to not use preprocessor hacks, source code changes, or runtime detection hacks to identify which code is appropriate when creating a client bound package.

## Notes

* If your module is pure javascript and can run in both client and server environments, then you do not need a browser field.
* The ```browser``` field is located in the ```package.json``` file as it provides metadata in the form of a hint to bundlers about what files you have indicated are targeted for the client. It allows your source code to remain clean and free of hacks.
* Consider that the client environment as the special case as it exposes objects into the global space to provide certain features and limits others.
