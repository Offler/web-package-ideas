# Web Packages.

This is a sketch for how web packages could be structured. It leans heavily on the CommonJS Packages specification and the Node.js modules approach.

### Stand alone JS only packages.
This demonstrates how they could be laid out in their repository.

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
			*another-main-module.js*
			*another-required-by-main-module.js*			
				* Only loadable by another-main-module.js, loaded with " import("./another-required-by-main-module"); " in the another-main-module.js file.
		*package.json*
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
			*more-main-module.js*
			dir/*more-required-by-main-module.js*			
				* Only loadable by more-main-module.js, loaded with " import("./dir/more-required-by-main-module"); " in the more-main-module.js file.
		*package.json*
			{
			   "name" : "more-package",
			   "version" : "0.3.0",
			   "main" : "./lib/more-main-module"
			}

#### Within the *test-package* package.

* The file *required-by-main-module.js* is only loadable by *main-module.js*, loaded with *import("./required-by-main-module");* in the *main-module.js* file.
* Any module that is part of *test-package* can import its dependencies by using *import*.
 + i.e. *import("another-test-package")*. Will return the module defined within *another-test-package*'s *package.json* "main" attribute.
* In this case it will return the "another-main-module.js" code module's exports.
* The "import" function will **not** return any other modules as they are not defined as explicit dependencies of "test-package", even if they are available.
* i.e. " import("more-package") ". Will throw an error ( maybe ReferenceError? Or a new Error type? )
			