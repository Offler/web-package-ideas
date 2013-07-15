# Web Packages.

This is a sketch for how web packages could be structured. It copies, mixes and takes inspiration from the CommonJS Packages specification and the Node.js modules approach.

## Stand alone JS only packages.

Firstly some simple web package examples that contain only JS code.

### Possible repository structure.

A package directory contains a *package.json* file that contains the definition of the package.
It contains a *lib* directory where its code modules reside.
The *lib* directory may have sub directories containing code modules.
There is a function e.g. *import* that when given a module or package identifier will return module's or package's exports.

	test-package/
		lib/
			main-module.js
			required-by-main-module.js			
		package.json
			{
			   "name" : "test-package",
			   "version" : "0.1.0",
			   "main" : "./lib/main-module",
			   "dependencies": {
				"another-test-package": "0.2.0"
			   }
			}

	another-test-package/
		lib/
			another-main-module.js
			another-required-by-main-module.js
		package.json
			{
			   "name" : "another-test-package",
			   "version" : "0.2.0",
			   "main" : "./lib/another-main-module",
			   "dependencies": {
				"more-package": "0.3.0"
			   }
			}

	more-package/
		lib/
			more-main-module.js
			dir/more-required-by-main-module.js
		package.json
			{
			   "name" : "more-package",
			   "version" : "0.3.0",
			   "main" : "./lib/more-main-module"
			}

#### Within the *test-package* package.

* The file *required-by-main-module.js* is only loadable by *main-module.js*, loaded with *import("./required-by-main-module");* in the *main-module.js* file.
* Any module that is part of *test-package* can import its dependencies by using *import*.
 + i.e. *import("another-test-package")*. Will return the module defined within *another-test-package*'s *package.json* "main" attribute.
 + In this case it will return the *another-main-module.js* code module's exports.
* The *import* function will **not** return any other modules as they are not defined as explicit dependencies of *test-package*, even if they are available.
 + i.e. *import("more-package")*. Will throw an error ( maybe ReferenceError? Or a new Error type? )

#### Within the *another-test-package* package.

* The file *another-required-by-main-module.js* is only loadable by *another-main-module.js*, loaded with *import("./another-required-by-main-module");* 
in the *another-main-module.js* file.

#### Within the *more-package* package.

* The file *more-required-by-main-module.js* is only loadable by *more-main-module.js*, loaded with *import("./dir/more-required-by-main-module");* 
in the *more-main-module.js* file.

### Possible development structure.

To work on a web-package a developer would pull the repository and using a web package management tool would populate a directory within the package directory 
with all the required package dependencies. Presented below is a possible directory layout upon completion, by the package management tool, 
of the package dependency fetching.


	test-package/
		lib/
			main-module.js
			required-by-main-module.js
		packages/
				another-test-package/
					lib/
						another-main-module.js
						another-required-by-main-module.js
					packages/
							more-package/
								lib/
									more-main-module.js
									dir/more-required-by-main-module.js
								package.json
									{
									   "name" : "more-package",
									   "version" : "0.3.0",
									   "main" : "./lib/more-main-module"
									}
					package.json
						{
						   "name" : "another-test-package",
						   "version" : "0.2.0",
						   "main" : "./lib/another-main-module",
						   "dependencies": {
							"more-package": "0.3.0"
						   }
						}
		package.json
			{
			   "name" : "test-package",
			   "version" : "0.1.0",
			   "main" : "./lib/main-module",
			   "dependencies": {
				"another-test-package": "0.2.0"
			   }
			}

The package management tool would populate a directory e.g. *packages* inside the package directory with all the direct dependencies of a package.
Within each dependency would reside their own dependencies within their own *packages* directory.

### Possible module code style.

The developer would be free to write in any style he/she chooses within a module as long as they respect two mechanisms.

1. The one to export dependencies. Either attaching to *module.exports* or *exports*, both variables will be injected into the module by the bundling system.
2. The one to import dependencies. Using the *import* function.

Some of the above classes are show below.

#### Inside more-package package

*more-required-by-main-module.js*

```javascript
	console.log( "Hello from dir/more-required-by-main-module module. I have no dependencies." );
	
	exports.hello = function(){ return "hello"; };
});
```
*more-main-module.js*

```javascript
	var hello = import( "./dir/more-required-by-main-module" ).hello;
	
	console.log( "Hello from more-main-module.js. I have a dependency. It says ", hello() );
```


### Possible in-browser structure.

Given a package aware module bundling tool that brings together and processes code modules into a format suitable to serve to a web browser we may end up with JS code
bundles similar in structure to the one below. Two functions are prerequisite for the bundle to work in a browser.

1. The module registration function *define-module* which is used to register the module.
2. The module dependency accessor function *import* which is used to access other required modules.

#### define-module's definition.

The *define-module* function is probably the least transparent concept encountered so far. It has several parameters. The first few enumerate the dependencies 
that pulled a module into the bundle, the dependency graph. The reason for this data is explained below but firstly the parameter list. 

1. An array of package IDs. These are the parent package all the way to the package the module is part of.
2. An array of directories that must be traversed to reach the module. Empty if module is inside the *lib* directory.
3. The module ID string.
4. A boolean indicating whether the module is the package's main module.
5. The module body wrapped within a function.

The reason for flattening modules out instead of nesting them is to more closely mirror ES6 modules which do not support nested modules. 
i.e. you cannot place a module definition within another module definition.

That means to make use of ES6 modules all module definitions must be placed at the top level of a script.

#### import's definition.

The *import* function follows the CommonJS *require* function closely. The points below are directly from the *require* 
[definition](http://wiki.commonjs.org/wiki/Modules/1.1).

1. The *import* function accepts a module identifier.
2. *import* returns the exported API of the foreign module.
3.  If there is a dependency cycle, the foreign module may not have finished executing at the time it is required by one of its transitive dependencies; in this case, 
the object returned by "require" must contain at least the exports that the foreign module has prepared before the call to require that led to the current module's execution.
4. If the requested module cannot be returned, *import* must throw an error.

#### Bundle tool example code.

Given all these prerequisites this code example should successfully load up and execute the *test-package* module code.

```javascript
define-module( [ "test-package", "another-test-package", "more-package" ], [ "dir" ], "more-required-by-main-module", false, function( import, exports, module ){
	console.log( "Hello from dir/more-required-by-main-module module. I have no dependencies." );
	
	exports.hello = function(){ return "hello"; };
});

define-module( ["test-package", "another-test-package", "more-package"], [], "more-main-module", true, function( import, exports, module ){
	var hello = import( "./dir/more-required-by-main-module" ).hello;
	
	console.log( "Hello from more-main-module.js. I have a dependency. It says ", hello() );
});
```