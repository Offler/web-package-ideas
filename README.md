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

### Possible in-browser structure.

Given a package aware module bundling tool that brings together and processes code modules into a format suitable to serve to a web browser we may end up with JS code
bundles similar in structure to the one below.

```javascript
	define-module("test-package/DIV/another-test-package/DIV/more-package/dir/more-required-by-main-module", function( require, exports, module ){
		console.log("Hello from dir/more-required-by-main-module module. I have no dependencies.");
	});
```