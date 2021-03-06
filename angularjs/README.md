# Angular Style Guide

*Opinionated Angular style guide for teams by [@valorkin](//twitter.com/valorkin)*

If you are looking for an opinionated style guide for syntax, conventions, and structuring Angular applications, then step right in. These styles are based on my and Valor Software developers development experience with [Angular](//angularjs.org), presentations, and working in teams.

The purpose of this style guide is to provide guidance on building Angular applications by showing the conventions I use and, more importantly, why I choose them.

## Community Awesomeness and Credit
Fill free to check guidelines used as a basis for this guideline:
- [Todd's guidelines](https://github.com/toddmotto/angularjs-styleguide)
- [John Papa's guidelines](https://github.com/johnpapa/angular-styleguide)

## See the Styles in a Sample App
Cooming soon
<!--
While this guide explains the *what*, *why* and *how*, I find it helpful to see them in practice. This guide is accompanied by a sample application that follows these styles and patterns. You can find the [sample application (named modular) here](https://github.com/johnpapa/ng-demos) in the `modular` folder. Feel free to grab it, clone it, or fork it. [Instructions on running it are in its readme](https://github.com/johnpapa/ng-demos/tree/master/modular).
-->

## Table of Contents

  1. [Single Responsibility](#single-responsibility)
  1. [IIFE](#iife)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Factories](#factories)
  1. [Data Services](#data-services)
  1. [Directives](#directives)
  1. [Resolving a Controller](#resolving-a-controller)
  1. [Manual Annotating for Dependency Injection](#manual-annotating-for-dependency-injection)
  1. [Minification and Annotation](#minification-and-annotation)
  1. [Exception Handling](#exception-handling)
  1. [Naming](#naming)
  1. [Application Structure LIFT Principle](#application-structure-lift-principle)
  1. [Application Structure](#application-structure)
  1. [Modularity](#modularity)
  1. [Startup Logic](#startup-logic)
  1. [Angular $ Wrapper Services](#angular--wrapper-services)
  1. [Testing](#testing)
  1. [Animations](#animations)
  1. [Comments](#comments)
  1. [ESLint](#eslint)
  1. [Constants](#constants)
  1. [Routing](#routing)
  1. [Task Automation](#task-automation)
  1. [Filters](#filters)
  1. [Angular Docs](#angular-docs)
  1. [Contributing](#contributing)
  1. [License](#license)

## Single Responsibility

### Rule of 1
<!--###### [Style [Y001](#style-y001)]-->

  - Define 1 component per file.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```javascript
  /* avoid */
  angular
    .module('app', ['ngRoute'])
    .controller('SomeController', SomeController)
    .factory('someFactory', someFactory);

  function SomeController() { }

  function someFactory() { }
  ```

  The same components are now separated into their own files.

  ```javascript
  /* recommended */

  // app.module.js
  angular
    .module('app', ['ngRoute']);
  ```

  ```javascript
  /* recommended */

  // someController.js
  angular
    .module('app')
    .controller('SomeController', [
      function SomeController() {}
    ]);
  ```

  ```javascript
  /* recommended */

  // someFactory.js
  angular
    .module('app')
    .factory('someFactory', [
      function someFactory() {}
    ]);
  ```

**[Back to top](#table-of-contents)**

## IIFE

#### Good idea:

  - To avoid polluting the global scope with our function declarations
    that get passed into Angular, ensure build tasks wrap the concatenated
    files inside an IIFE

#### But:

  - I see wrapping each component in IIFE hard and not necessary.
    Just keep all required code inside of function passed to Angular.

  ```javascript
  /* avoid */

  // someFactory.js
  angular
    .module('app')
    .factory('someFactory', [
      function someFactory() {
        this.onClick = sayHello;
      }
    ]);

  function sayHello() {}
  ```

  ```javascript
  /* recommended */

  // someFactory.js
  angular
    .module('app')
    .factory('someFactory', [
      function someFactory() {
        this.onClick = sayHello;

        function sayHello() {}
      }
    ]);
  ```

Original: [Style [Y010](https://github.com/johnpapa/angular-styleguide#style-y010)]

**[Back to top](#table-of-contents)**

## Modules

### Avoid Naming Collisions
<!--###### [Style [Y020](#style-y020)]-->

  - Use unique naming conventions with separators for sub-modules.

  *Why?*: Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.

### Definitions (aka Setters)
<!--###### [Style [Y021](#style-y021)]-->

  - Declare modules without a variable using the setter syntax.

  *Why?*: With 1 component per file, there is rarely a need to introduce a variable for the module.

  ```javascript
  /* avoid */
  var app = angular.module('app', [
    'ngAnimate',
    'ngRoute',
    'app.shared',
    'app.dashboard'
  ]);
  ```

  Instead use the simple setter syntax.

  ```javascript
  /* recommended */
  angular
    .module('app', [
      'ngAnimate',
      'ngRoute',
      'app.shared',
      'app.dashboard'
    ]);
  ```

### Getters
<!--###### [Style [Y022](#style-y022)]-->

  - When using a module, avoid using a variable and instead use chaining with the getter syntax.

  *Why?*: This produces more readable code and avoids variable collisions or leaks.

  ```javascript
  /* avoid */
  var app = angular.module('app');
  app.controller('SomeController', [
    function SomeController() { }
  ]);
  ```

  ```javascript
  /* recommended */
  angular
    .module('app')
    .controller('SomeController', [
      function SomeController() { }
    ]);
  ```

### Setting vs Getting
<!--###### [Style [Y023](#style-y023)]-->

  - Only set once and get for all other instances.

  *Why?*: A module should only be created once, then retrieved from that point and after.

    - Use `angular.module('app', []);` to set a module.
    - Use `angular.module('app');` to get a module.

### Named vs Anonymous Functions
<!--###### [Style [Y024](#style-y024)]-->

  - Use named functions instead of passing an anonymous function in as a callback.

  *Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

  ```javascript
  /* avoid */
  angular
    .module('app')
    .controller('Dashboard', function() { })
    .factory('logger', function() { });
  ```

  ```javascript
  /* recommended */

  // dashboard.js
  angular
    .module('app')
    .controller('Dashboard', [
      function Dashboard() { }
    ]);
  ```

  ```javascript
  // logger.js
  angular
    .module('app')
    .factory('logger', [
      function logger() { }
    ]);
  ```

**[Back to top](#table-of-contents)**

## Controllers

### controllerAs View Syntax
<!--###### [Style [Y030](#style-y030)]-->

#### *Important: never use `ngController` directive*

  - Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

  *Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  ```html
  <!-- avoid -->
  <div ng-controller="Customer">
    {{ name }}
  </div>
  ```

  ```html
  <!-- recommended - but avoid `ngController` directive usage -->
  <div ng-controller="Customer as customer">
    {{ customer.name }}
  </div>
  ```

### controllerAs Controller Syntax
<!--###### [Style [Y031](#style-y031)]-->

  - Use the `controllerAs` syntax over the `classic controller with $scope` syntax.

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

  *Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

  *Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move them to a factory. Consider using `$scope` in a factory, or if in a controller just when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) consider moving these uses to a factory and invoke from the controller.

  ```javascript
  /* avoid */
  function Customer($scope) {
    $scope.name = {};
    $scope.sendMessage = function() { };
  }
  ```

  ```javascript
  /* recommended - but see next section */
  function Customer() {
    this.name = {};
    this.sendMessage = function() { };
  }
  ```

### controllerAs with `self`
<!--###### [Style [Y032](#style-y032)]-->

  - Use a capture variable for `this` when using the `controllerAs` syntax.
    Choose a  variable name such as `self` to
    [keep `this` usage consistent](http://eslint.org/docs/rules/consistent-this).

    *or: such as `vm`, which stands for ViewModel.*

  *Why?*: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

  ```javascript
  /* avoid */
  function Customer() {
    this.name = {};
    this.sendMessage = function() { };
  }
  ```

  ```javascript
  /* recommended */
  function Customer() {
    var self = this;
    self.name = {};
    self.sendMessage = function() { };
  }
  ```

  Note: When creating watches in a controller using `controller as`, you can watch the `ctrl.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)

  ```html
  <input ng-model="someCtrl.title"/>
  ```

  ```javascript
  function SomeController($scope, $log) {
    var self = this;
    self.title = 'Some Title';

    $scope.$watch('someCtrl.title', function(current, original) {
      $log.info('someCtrl.title was %s', original);
      $log.info('someCtrl.title is now %s', current);
    });
  }
  ```

### Bindable Members Up Top
<!--###### [Style [Y033](#style-y033)]-->

  - Place bindable members at the top of the controller, and not spread through the controller code.
  - Proposed order:
    - bind default values
    - bind event handlers
    - initiate controller
    - event handlers declarations

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

    *Why?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read.

  ```javascript
  /* avoid */
  function Sessions() {
    var self = this;

    self.gotoSession = function() {
      /* ... */
    };
    self.refresh = function() {
      /* ... */
    };
    self.search = function() {
      /* ... */
    };
    self.sessions = [];
    self.title = 'Sessions';
  }
  ```

  ```javascript
  /* recommended */
  function SessionsController() {
    var self = this;

    // bind default values
    self.sessions = [];
    self.title = 'Sessions';

    // bind event handlers
    self.gotoSession = gotoSession;
    self.refresh = refresh;
    self.search = search;

    // initiate controller
    activate();

    // event handler declarations
    function gotoSession() {
      /* */
    }

    function refresh() {
      /* */
    }

    function search() {
      /* */
    }
  }
  ```

  Note: If the function is a 1 liner consider keeping it right up top, as long as readability is not affected.

  ```javascript
  /* avoid */
  function SessionsController(data) {
    var self = this;

    self.sessions = [];
    self.title = 'Sessions';

    self.gotoSession = gotoSession;
    self.refresh = function() {
      /**
       * lines
       * of
       * code
       * affects
       * readability
       */
    };
    self.search = search;
  }
  ```

  ```javascript
  /* recommended */
  function SessionsController(dataservice) {
    var self = this;

    self.sessions = [];
    self.title = 'Sessions';

    self.gotoSession = gotoSession;
    self.refresh = dataservice.refresh; // 1 liner is OK
    self.search = search;
  }
  ```

### Function Declarations to Hide Implementation Details
<!--###### [Style [Y034](#style-y034)]-->

  - Use function declarations to hide implementation details. Keep your bindable members up top. When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section Bindable Members Up Top. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code).

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. (Same as above.)

    *Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    *Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    *Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

    *Why?*: Order is critical with function expressions

  ```javascript
  /**
   * avoid
   * Using function expressions.
   */
  function Avengers(dataservice, logger) {
    var self = this;
    self.avengers = [];
    self.title = 'Avengers';

    var activate = function() {
      return getAvengers().then(function() {
        logger.info('Activated Avengers View');
      });
    }

    var getAvengers = function() {
      return dataservice.getAvengers().then(function(data) {
        self.avengers = data;
        return self.avengers;
      });
    }

    self.getAvengers = getAvengers;

    activate();
  }
  ```

  Notice that the important stuff is scattered in the preceding example. In the example below, notice that the important stuff is up top. For example, the members bound to the controller such as `self.avengers` and `self.title`. The implementation details are down below. This is just easier to read.

  ```javascript
  /*
   * recommend
   * Using function declarations
   * and bindable members up top.
   */
  function Avengers(dataservice, logger) {
    var self = this;
    self.avengers = [];
    self.title = 'Avengers';

    self.getAvengers = getAvengers;

    activate();

    function activate() {
      return getAvengers().then(function() {
        logger.info('Activated Avengers View');
      });
    }

    function getAvengers() {
      return dataservice.getAvengers().then(function(data) {
        self.avengers = data;
        return self.avengers;
      });
    }
  }
  ```

### Defer Controller Logic to Services
<!--###### [Style [Y035](#style-y035)]-->

  - Defer logic in a controller by delegating to services and factories.

    *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

    *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

    *Why?*: Removes dependencies and hides implementation details from the controller.

    *Why?*: Keeps the controller slim, trim, and focused.

  ```javascript

  /* avoid */
  function Order($http, $q, config, userInfo) {
    var self = this;
    self.checkCredit = checkCredit;
    self.isCreditOk;
    self.total = 0;

    function checkCredit() {
      var settings = {};
      // Get the credit service base URL from config
      // Set credit service required headers
      // Prepare URL query string or data object with request data
      // Add user-identifying info so service gets the right credit limit for this user.
      // Use JSONP for this browser if it doesn't support CORS
      return $http.get(settings)
        .then(function(data) {
          // Unpack JSON data in the response object
          // to find maxRemainingAmount
          self.isCreditOk = self.total <= maxRemainingAmount
        })
        .catch(function(error) {
          // Interpret error
          // Cope w/ timeout? retry? try alternate service?
          // Re-reject with appropriate error for a user to see
        });
    };
  }
  ```

  ```javascript
  /* recommended */
  function Order(creditService) {
    var self = this;
    self.isCreditOk;
    self.total = 0;

    self.checkCredit = checkCredit;

    function checkCredit() {
      return creditService.isOrderTotalOk(self.total)
        .then(function(isOk) { self.isCreditOk = isOk; })
        .catch(showServiceError);
    };
  }
  ```

### Keep Controllers Focused
<!--###### [Style [Y037](#style-y037)]-->

  - Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view.

    *Why?*: Reusing controllers with several views is brittle and good end to end (e2e) test coverage is required to ensure stability across large applications.

### Assigning Controllers
<!--###### [Style [Y038](#style-y038)]-->

  - When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes.

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. 

 ```javascript
  /* avoid - when using with a route and dynamic pairing is desired */

  // route-config.js
  angular
    .module('app')
    .config([
      '$routeProvider',
      function config($routeProvider) {
      $routeProvider
        .when('/avengers', {
          templateUrl: 'avengers.html'
        });
      }
    ]);
  ```

  ```html
  <!-- avengers.html - avoid -->
  <div ng-controller="AvengersController as avengersCtrl">
  </div>
  ```

  ```javascript
  /* recommended */

  // route-config.js
  angular
    .module('app')
    .config([
      '$routeProvider',
      function config($routeProvider) {
        $routeProvider
          .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'AvengersController',
            controllerAs: 'avengersCtrl'
          });
      }
    ]);
  ```

  ```html
  <!-- avengers.html -->
  <div></div>
  ```

**[Back to top](#table-of-contents)**

## Services

### Singletons
<!--###### [Style [Y040](#style-y040)]-->

  - Services are instantiated with the `new` keyword, use `this` for public methods and variables. Since these are so similar to factories, use a factory instead for consistency.

    Note: [All Angular services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

    Note: [Ng Source service vs factory](https://github.com/angular/angular.js/blob/master/src/auto/injector.js#L686-L696)

  ```javascript
  // service
  angular
    .module('app')
    .service('logger', [
      function logger() {
        this.logError = function(msg) {
        /* */
        };
      }
    ]);
  ```

  ```javascript
  // factory
  /* avoid - returning raw object, see later */
  angular
    .module('app')
    .factory('logger', [
      function logger() {
        return {
          logError: function(msg) {
            /* */
          }
        };
      }
    ]);

  ```

**[Back to top](#table-of-contents)**

## Factories

### Single Responsibility
<!--###### [Style [Y050](#style-y050)]-->

  - Factories should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

### Singletons
<!--###### [Style [Y051](#style-y051)]-->

  - Factories are singletons and return an object that contains the members of the service.

    Note: [All Angular services are singletons](https://docs.angularjs.org/guide/services).

### Accessible Members Up Top
<!--###### [Style [Y052](#style-y052)]-->

  - Expose the callable members of the service (its interface) at the top, using a technique derived from the [Revealing Module Pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript).

    *Why?*: Placing the callable members at the top makes it easy to read and helps you instantly identify which members of the service can be called and must be unit tested (and/or mocked).

    *Why?*: This is especially helpful when the file gets longer as it helps avoid the need to scroll to see what is exposed.

    *Why?*: Setting functions as you go can be easy, but when those functions are more than 1 line of code they can reduce the readability and cause more scrolling. Defining the callable interface via the returned service moves the implementation details down, keeps the callable interface up top, and makes it easier to read.

  ```javascript
  /* avoid */
  function dataService() {
    var someValue = '';
    function save() {
      /* */
    };
    function validate() {
      /* */
    };

    return {
      save: save,
      someValue: someValue,
      validate: validate
    };
  }
  ```

  ```javascript
  /* recommended */
  function dataService() {
    var someValue = '';
    var service = {
      save: save,
      someValue: someValue,
      validate: validate
    };
    return service;

    ////////////

    function save() {
      /* */
    };

    function validate() {
      /* */
    };
  }
  ```

  This way bindings are mirrored across the host object, primitive values cannot update alone using the revealing module pattern.

### Function Declarations to Hide Implementation Details
<!-- ###### [Style [Y053](#style-y053)] -->

  - Use function declarations to hide implementation details. Keep your accessible members of the factory up top. Point those to function declarations that appears later in the file. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code).

    *Why?*: Placing accessible members at the top makes it easy to read and helps you instantly identify which functions of the factory you can access externally.

    *Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    *Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    *Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

    *Why?*: Order is critical with function expressions

  ```javascript
  /**
   * avoid
   * Using function expressions
   */
   function dataservice($http, $location, $q, exception, logger) {
      var isPrimed = false;
      var primePromise;

      var getAvengers = function() {
          // implementation details go here
      };

      var getAvengerCount = function() {
          // implementation details go here
      };

      var getAvengersCast = function() {
         // implementation details go here
      };

      var prime = function() {
         // implementation details go here
      };

      var ready = function(nextPromises) {
          // implementation details go here
      };

      var service = {
          getAvengersCast: getAvengersCast,
          getAvengerCount: getAvengerCount,
          getAvengers: getAvengers,
          ready: ready
      };

      return service;
  }
  ```

  ```javascript
  /**
   * recommended
   * Using function declarations
   * and accessible members up top.
   */
  function dataservice($http, $location, $q, exception, logger) {
      var isPrimed = false;
      var primePromise;

      function DataService() {}

      DataService.prototype = {
          getAvengersCast: getAvengersCast,
          getAvengerCount: getAvengerCount,
          getAvengers: getAvengers,
          ready: ready
      };

      return new DataService();

      ////////////

      function getAvengers() {
          // implementation details go here
      }

      function getAvengerCount() {
          // implementation details go here
      }

      function getAvengersCast() {
          // implementation details go here
      }

      function prime() {
          // implementation details go here
      }

      function ready(nextPromises) {
          // implementation details go here
      }
  }
  ```

**[Back to top](#table-of-contents)**

## Data Services

### Separate Data Calls
<!-- ###### [Style [Y060](#style-y060)] -->

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

    *Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

    *Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

    *Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as $http. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

  ```javascript
  /* recommended */

  // dataservice factory
  angular
      .module('app.core')
      .factory('dataservice', [
        '$http', 'logger',
        function dataservice($http, logger) {
          function AvengeresRepository() {}
          AvengeresRepository.prototype = {
              getAvengers: getAvengers
          };

          return new AvengeresRepository() {}

          function getAvengers(cb) {
              return $http.get('/api/maa')
                  .then(getAvengersComplete)
                  .catch(getAvengersFailed);

              function getAvengersComplete(response) {
                  return cb(null, response.data.results);
              }

              function getAvengersFailed(error) {
                  logger.error('XHR Failed for getAvengers.' + error.data);
                  return cb(error.data);
              }
          }
        }
      ]);
  ```

    Note: The data service is called from consumers, such as a controller, hiding the implementation from the consumers, as shown below.

  ```javascript
  /* recommended */

  // controller calling the dataservice factory
  angular
      .module('app.avengers')
      .controller('Avengers', [
        'dataservice', 'logger',
        function Avengers(dataservice, logger) {
            var self = this;
            self.avengers = [];

            activate();

            function activate() {
                return getAvengers(function() {
                    logger.info('Activated Avengers View');
                });
            }

            function getAvengers(cb) {
                return dataservice
                  .getAvengers(function(err, data) {
                        self.avengers = data;
                        return cb(err, data);
                  });
            }
        }
      ]);
  ```

### **NEVER** Return a Promise from Data Calls
<!-- ###### [Style [Y061](#style-y061)] -->

  - When calling a data service that returns a promise such as `$http`, never return a promise in your calling function.
  - Always use callbacks with applying of `node.js` signature convention `function(err, data)`

    *Why?*: It will be really hard to debug promise chains

  - Use promises only to integrate with events outside of `angular` `$digest`

    **[Back to top](#table-of-contents)**

## Directives
### Limit 1 Per File
<!-- ###### [Style [Y070](#style-y070)] -->

  - Create one directive per file. Name the file for the directive.

    *Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module.

    *Why?*: One directive per file is easy to maintain.

    > Note: "**Best Practice**: Directives should clean up after themselves. You can use `element.on('$destroy', ...)` or `scope.$on('$destroy', ...)` to run a clean-up function when the directive is removed" ... from the Angular documentation

  ```javascript
  /* avoid */
  /* directives.js */

  angular
      .module('app.widgets')

      /* order directive that is specific to the order module */
      .directive('orderCalendarRange', orderCalendarRange)

      /* sales directive that can be used anywhere across the sales app */
      .directive('salesCustomerInfo', salesCustomerInfo)

  function orderCalendarRange() {
      /* implementation details */
  }

  function salesCustomerInfo() {
      /* implementation details */
  }
  ```

  ```javascript
  /* recommended */
  /* calendarRange.directive.js */

  /**
   * @desc order directive that is specific to the order module at a company named Acme
   * @example <div acme-order-calendar-range></div>
   */
  angular
      .module('sales.order')
      .directive('acmeOrderCalendarRange', [
        function orderCalendarRange() {
          /* implementation details */
        }
      ]);
  ```

  ```javascript
  /* recommended */
  /* customerInfo.directive.js */

  /**
   * @desc sales directive that can be used anywhere across the sales app at a company named Acme
   * @example <div acme-sales-customer-info></div>
   */
  angular
      .module('sales.widgets')
      .directive('acmeSalesCustomerInfo', [
        function salesCustomerInfo() {
          /* implementation details */
        }
      ]);
  ```

    Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one that makes the directive and its file name distinct and clear. Some examples are below, but see the [Naming](#naming) section for more recommendations.

### Manipulate DOM in a Directive
<!-- ###### [Style [Y072](#style-y072)] -->

  - When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow.

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

### Provide a Unique Directive Prefix
<!-- ###### [Style [Y073](#style-y073)] -->

  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

    *Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company.

    Note: Avoid `ng-` as these are reserved for Angular directives. Research widely used directives to avoid naming conflicts, such as `ion-` for the [Ionic Framework](http://ionicframework.com/).

### Restrict to Elements and Attributes
<!-- ###### [Style [Y074](#style-y074)] -->

  - When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.

    *Why?*: It makes sense.

    *Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

    Note: EA is the default for Angular 1.3 +

  ```html
  <!-- avoid -->
  <div class="my-calendar-range"></div>
  ```

  ```javascript
  /* avoid */
  angular
      .module('app.widgets')
      .directive('myCalendarRange', [
          function myCalendarRange() {
              var directive = {
                  link: link,
                  templateUrl: '/template/is/located/here.html',
                  restrict: 'C'
              };
              return directive;

              function link(scope, element, attrs) {
                /* */
              }
          } 
      ]);
  ```

  ```html
  <!-- recommended -->
  <my-calendar-range></my-calendar-range>
  <div my-calendar-range></div>
  ```

  ```javascript
  /* recommended */
  angular
      .module('app.widgets')
      .directive('myCalendarRange', [
          function myCalendarRange() {
              var directive = {
                  link: link,
                  templateUrl: '/template/is/located/here.html',
                  restrict: 'EA'
              };
              return directive;

              function link(scope, element, attrs) {
                /* */
              }
          }
      ]);
  ```

### Directives and ControllerAs
<!-- ###### [Style [Y075](#style-y075)] -->

  - Use `controller as` syntax with a directive to be consistent with using `controller as` with view and controller pairings.

    *Why?*: It makes sense and it's not difficult.

    Note: The directive below demonstrates some of the ways you can use scope inside of link and directive controllers, using controllerAs. I in-lined the template just to keep it all in one place.

    Note: Regarding dependency injection, see [Manually Identify Dependencies](#manual-annotating-for-dependency-injection).

    Note: Note that the directive's controller is outside the directive's closure. This style eliminates issues where the injection gets created as unreachable code after a `return`.

  ```html
  <div my-example max="77"></div>
  ```

  ```javascript
  // my-example-directive.js
  angular
      .module('app')
      .directive('myExample', [
         function myExample() {
            var directive = {
                restrict: 'EA',
                templateUrl: 'app/feature/example.directive.html',
                scope: {
                    max: '='
                },
                link: linkFunc,
                controller: ExampleController,
                controllerAs: 'ctrl',
                bindToController: true // because the scope is isolated
            };

            return directive;

            function linkFunc(scope, el, attr, ctrl) {
                console.log('LINK: scope.min = %s *** should be undefined', scope.min);
                console.log('LINK: scope.max = %s *** should be undefined', scope.max);
                console.log('LINK: scope.self.min = %s', scope.self.min);
                console.log('LINK: scope.self.max = %s', scope.self.max);
            }
          }
      ]);
```

```javascript
  // example-controller.js
  angular
      .module('app')
      .controller('ExampleController', [
          '$scope',
          function ExampleController($scope) {
              // Injecting $scope just for comparison
              var self = this;

              self.min = 3;

              console.log('CTRL: $scope.self.min = %s', $scope.self.min);
              console.log('CTRL: $scope.self.max = %s', $scope.self.max);
              console.log('CTRL: self.min = %s', self.min);
              console.log('CTRL: self.max = %s', self.max);
          }
      ])
  ```

  ```html
  <!-- example.directive.html -->
  <div>hello world</div>
  <div>max={{ctrl.max}}<input ng-model="ctrl.max"/></div>
  <div>min={{ctrl.min}}<input ng-model="ctrl.min"/></div>
  ```

    Note: You can also name the controller when you inject it into the link function and access directive attributes as properties of the controller.

  ```javascript
  // Alternative to above example
  function linkFunc(scope, el, attr, self) {
      console.log('LINK: scope.min = %s *** should be undefined', scope.min);
      console.log('LINK: scope.max = %s *** should be undefined', scope.max);
      console.log('LINK: self.min = %s', self.min);
      console.log('LINK: self.max = %s', self.max);
  }
  ```

<!-- ###### [Style [Y076](#style-y076)] -->
### Use `bindToController` in directive definitions

  - Use `bindToController = true` when using `controller as` syntax with a directive when you want to bind the outer scope to the directive's controller's scope.

    *Why?*: It makes it easy to bind outer scope to the directive's controller scope.

    Note: `bindToController` was introduced in Angular 1.3.0.

  ```html
  <div my-example max="77"></div>
  ```

  ```javascript
  // my-example-directive.js
  angular
      .module('app')
      .directive('myExample', [
          function myExample() {
              var directive = {
                  restrict: 'EA',
                  templateUrl: 'app/feature/example.directive.html',
                  scope: {
                      max: '='
                  },
                  controller: ExampleController,
                  controllerAs: 'ctrl',
                  bindToController: true
              };

              return directive;
          }
  ]);
```
  
```javascript
  // example-controller.js
  angular
      .module('app')
      .controller('ExampleController', [
          function ExampleController() {
              var self = this;
              self.min = 3;
              console.log('CTRL: self.min = %s', self.min);
              console.log('CTRL: self.max = %s', self.max);
          }
      ]);
  ```

  ```html
  <!-- example.directive.html -->
  <div>hello world</div>
  <div>max={{ctrl.max}}<input ng-model="ctrl.max"/></div>
  <div>min={{ctrl.min}}<input ng-model="ctrl.min"/></div>
  ```

**[Back to top](#table-of-contents)**

## Resolving a Controller

### Controller Activation
<!-- ###### [Style [Y080](#style-y080)] -->

  - Resolve start-up logic for a controller in an `activate` function.

    *Why?*: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

    *Why?*: The controller `activate` makes it convenient to re-use the logic for a refresh for the controller/View, keeps the logic together, gets the user to the View faster, makes animations easy on the `ng-view` or `ui-view`, and feels snappier to the user.

    Note: If you need to conditionally cancel the route before you start use the controller, use a `canActive` approach introduced in `angular 1.4`.

  ```javascript
  /* avoid */
  function Avengers(dataservice) {
      var self = this;
      self.avengers = [];
      self.title = 'Avengers';

      dataservice.getAvengers().then(function(data) {
          self.avengers = data;
          return self.avengers;
      });
  }
  ```

  ```javascript
  /* recommended */
  function Avengers(dataservice) {
      var self = this;
      self.avengers = [];
      self.title = 'Avengers';

      activate();

      ////////////

      function activate() {
          return dataservice.getAvengers(function(err, data) {
              self.avengers = data;
          });
      }
  }
  ```

### Route Resolve Promises
<!-- ###### [Style [Y081](#style-y081)] -->
  - Try to avoid usage of `$routeProvider` `resolve` promises, it leads to concepts mess and promis based logic on configurations step

**[Back to top](#table-of-contents)**

## Manual Annotating for Dependency Injection

### UnSafe from Minification
<!-- ###### [Style [Y090](#style-y090)] -->

  - Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.

    *Why?*: The parameters to the component (e.g. controller, factory, etc) will be converted to mangled variables. For example, `common` and `dataservice` may become `a` or `b` and not be found by Angular.

    ```javascript
    /* avoid - not minification-safe*/
    angular
        .module('app')
        .controller('Dashboard', Dashboard);

    function Dashboard(common, dataservice) {
    }
    ```

    This code may produce mangled variables when minified and thus cause runtime errors.

    ```javascript
    /* avoid - not minification-safe*/
    angular.module('app').controller('Dashboard', d);function d(a, b) { }
    ```

### Manually Identify Dependencies
<!-- ###### [Style [Y091](#style-y091)] -->

  - Use the inline array annotation ([preferred](https://docs.angularjs.org/guide/di))
  - **Always** use `ngStrictDi` with `ngApp` to avoid unexpected behaviour after minification

    *Why?*: This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by Angular.

    ```javascript
    /* avoid */
    angular
        .module('app')
        .controller('Dashboard', Dashboard);

    Dashboard.$inject = ['$location', '$routeParams', 'common', 'dataservice'];

    function Dashboard($location, $routeParams, common, dataservice) {
    }
    ```

    ```javascript
    /* recommended */
    angular
        .module('app')
        .controller('Dashboard', [
            '$location', '$routeParams', 'common', 'dataservice',
            function Dashboard($location, $routeParams, common, dataservice) {}
        ]);
    ```

**[Back to top](#table-of-contents)**

## Minification and Annotation
TODO
<!-- TODO : write concat\uglify gulp -->
**[Back to top](#table-of-contents)**

## Exception Handling

### decorators
<!-- ###### [Style [Y110](#style-y110)] -->

  - Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.

    *Why?*: Provides a consistent way to handle uncaught Angular exceptions for development-time or run-time.

    Note: Another option is to override the service instead of using a decorator. This is a fine option, but if you want to keep the default behavior and extend it a decorator is recommended.

    ```javascript
    /* recommended */
    angular
        .module('blocks.exception')
        .config([
            '$provide',
            function exceptionConfig($provide) {
                $provide.decorator('$exceptionHandler', [
                    '$delegate', 'toastr',
                    function extendExceptionHandler($delegate, toastr) {
                        return function(exception, cause) {
                            $delegate(exception, cause);
                            var errorData = {
                                exception: exception,
                                cause: cause
                            };
                            /**
                             * Could add the error
                             * to a service's collection,
                             * add errors to $rootScope,
                             * log errors to remote web server,
                             * or log locally. 
                             * Or throw hard. It is entirely up to you.
                             * throw exception;
                             */
                            toastr.error(exception.msg, errorData);
                        };
                    }
                ]);
            }
        ]);
    ```

### Exception Catchers
<!-- ###### [Style [Y111](#style-y111)] -->

  - Create a factory that exposes an interface to catch and gracefully handle exceptions.

    *Why?*: Provides a consistent way to catch exceptions that may be thrown in your code (e.g. during XHR calls or promise failures).

    Note: The exception catcher is good for catching and reacting to specific exceptions from calls that you know may throw one. For example, when making an XHR call to retrieve data from a remote web service and you want to catch any exceptions from that service and react uniquely.

    ```javascript
    /* recommended */
    angular
        .module('blocks.exception')
        .factory('exception', [
            'logger',
            function exception(logger) {
                function ExceptionLogger() {}
                ExceptionLogger.prototype = {
                    catcher: catcher
                };
                return new ExceptionLogger();

                function catcher(message) {
                    return function(reason) {
                        logger.error(message, reason);
                    };
                }
            }
        ]);
    ```

### Route Errors
<!-- ###### [Style [Y112](#style-y112)] -->

  - Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError).

    *Why?*: Provides a consistent way to handle all routing errors.

    *Why?*: Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

    ```javascript
    /* recommended */
    var handlingRouteChangeError = false;

    function handleRoutingErrors() {
        /**
         * Route cancellation:
         * On routing error, go to the dashboard.
         * Provide an exit clause if it tries to do it twice.
         */
        $rootScope.$on('$routeChangeError',
            function(event, current, previous, rejection) {
                if (handlingRouteChangeError) { return; }
                handlingRouteChangeError = true;
                var destination = (current && (current.title ||
                    current.name || current.loadedTemplateUrl)) ||
                    'unknown target';
                var msg = 'Error routing to ' + destination + '. ' +
                    (rejection.msg || '');

                /**
                 * Optionally log using a custom service or $log.
                 * (Don't forget to inject custom service)
                 */
                logger.warning(msg, [current]);

                /**
                 * On routing error, go to another route/state.
                 */
                $location.path('/');

            }
        );
    }
    ```

**[Back to top](#table-of-contents)**

## Naming

### Naming Guidelines
<!-- ###### [Style [Y120](#style-y120)] -->

  - Use consistent names for all components following a pattern that describes the component's feature then its type. My recommended pattern is `feature.type.js`. There are 2 names for most assets:
    * the file name (`avengers.controller.js`)
    * the registered component name with Angular (`AvengersController`)

    *Why?*: Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

    *Why?*: The naming conventions should simply help you find your code faster and make it easier to understand.

### Feature File Names
<!-- ###### [Style [Y121](#style-y121)] -->

  - Use consistent names for all components following a pattern that describes the component's feature then its type. My recommended pattern is `feature.type.js`.

    *Why?*: Provides a consistent way to quickly identify components.

    *Why?*: Provides pattern matching for any automated tasks.

    ```javascript
    /**
     * common options
     */

    // Controllers
    avengers.controller.js
    avengersController.js

    // Services/Factories
    logger.service.js
    loggerService.js
    ```

    ```javascript
    /**
     * recommended
     */

    // controllers
    avengers.controller.js
    avengers.controller.spec.js

    // services/factories
    logger.service.js
    logger.service.spec.js

    // constants
    constants.js

    // module definition
    avengers.module.js

    // routes
    avengers.routes.js
    avengers.routes.spec.js

    // configuration
    avengers.config.js

    // directives
    avenger-profile.directive.js
    avenger-profile.directive.spec.js
    ```
  
### Test File Names
<!-- ###### [Style [Y122](#style-y122)] -->

  - Name test specifications similar to the component they test with a suffix of `spec`.

    *Why?*: Provides a consistent way to quickly identify components.

    *Why?*: Provides pattern matching for [karma](http://karma-runner.github.io/) or other test runners.

    ```javascript
    /**
     * recommended
     */
    avengers.controller.spec.js
    logger.service.spec.js
    avengers.routes.spec.js
    avenger-profile.directive.spec.js
    ```

### Controller Names
<!-- ###### [Style [Y123](#style-y123)] -->

  - Use consistent names for all controllers named after their feature. Use UpperCamelCase for controllers, as they are constructors.

    *Why?*: Provides a consistent way to quickly identify and reference controllers.

    *Why?*: UpperCamelCase is conventional for identifying object that can be instantiated using a constructor.

    ```javascript
    /**
     * recommended
     */

    // hero-avengers.controller.js
    angular
        .module
        .controller('HeroAvengersController', [
            function HeroAvengersController() { }
        ]);
    ```

### Controller Name Suffix
<!-- ###### [Style [Y124](#style-y124)] -->

  - Append the controller name with the suffix `Controller`
  - Don't use `Ctrl`, it is good only for tutorials

    *Why?*: The `Controller` suffix is more commonly used and is more explicitly descriptive.

    ```javascript
    /**
     * recommended
     */

    // avengers.controller.js
    angular
        .module
        .controller('AvengersController', [
            function AvengersController() { }
        ]);
    ```

### Factory Names
<!-- ###### [Style [Y125](#style-y125)] -->

  - Use consistent names for all factories named after their feature. Use camel-casing for services and factories. Avoid prefixing factories and services with `$`.
  - Append the factory name with the suffix `Service`

    *Why?*: Provides a consistent way to quickly identify and reference factories.

    *Why?*: Avoids name collisions with built-in factories and services that use the `$` prefix.

    ```javascript
    /**
     * recommended
     */

    // logger.service.js
    angular
        .module
        .factory('loggerService', [
            function loggerService() { }
        ]);

    
    ```

### Directive Component Names
<!-- ###### [Style [Y126](#style-y126)] -->

  - Use consistent names for all directives using camel-case. Use a short prefix to describe the area that the directives belong (some example are company prefix or project prefix).

    *Why?*: Provides a consistent way to quickly identify and reference components.

    ```javascript
    /**
     * recommended
     */

    // avenger-profile.directive.js
    angular
        .module
        // usage is <xx-avenger-profile> </xx-avenger-profile>
        .directive('xxAvengerProfile', [
            function xxAvengerProfile() { }
        ]);
    ```

### Modules
<!-- ###### [Style [Y127](#style-y127)] -->

  - When there are multiple modules, the main module file is named `app.module.js` while other dependent modules are named after what they represent. For example, an admin module is named `admin.module.js`. The respective registered module names would be `app` and `admin`.

    *Why?*: Provides consistency for multiple module apps, and for expanding to large applications.

    *Why?*: Provides easy way to use task automation to load all module definitions first, then all other angular files (for bundling).

### Configuration
<!-- ###### [Style [Y128](#style-y128)] -->

  - Separate configuration for a module into its own file named after the module. A configuration file for the main `app` module is named `app.config.js` (or simply `config.js`). A configuration for a module named `admin.module.js` is named `admin.config.js`.

    *Why?*: Separates configuration from module definition, components, and active code.

    *Why?*: Provides an identifiable place to set configuration for a module.

### Routes
<!-- ###### [Style [Y129](#style-y129)] -->

  - Separate route configuration into its own file. Examples might be `app.route.js` for the main module and `admin.route.js` for the `admin` module. Even in smaller apps I prefer this separation from the rest of the configuration.

**[Back to top](#table-of-contents)**

## Application Structure LIFT Principle
### LIFT
<!-- ###### [Style [Y140](#style-y140)] -->

  - Structure your app such that you can `L`ocate your code quickly, `I`dentify the code at a glance, keep the `F`lattest structure you can, and `T`ry to stay DRY. The structure should follow these 4 basic guidelines.

    *Why LIFT?*: Provides a consistent structure that scales well, is modular, and makes it easier to increase developer efficiency by finding code quickly. Another way to check your app structure is to ask yourself: How quickly can you open and work in all of the related files for a feature?

    When I find my structure is not feeling comfortable, I go back and revisit these LIFT guidelines

    1. `L`ocating our code is easy
    2. `I`dentify code at a glance
    3. `F`lat structure as long as we can
    4. `T`ry to stay DRY (Don’t Repeat Yourself) or T-DRY

### Locate
<!-- ###### [Style [Y141](#style-y141)] -->

  - Make locating your code intuitive, simple and fast.

    *Why?*: I find this to be super important for a project. If the team cannot find the files they need to work on quickly, they will not be able to work as efficiently as possible, and the structure needs to change. You may not know the file name or where its related files are, so putting them in the most intuitive locations and near each other saves a ton of time. A descriptive folder structure can help with this.

    ```
    /public
      /assets
      /components
        /avengers
        /blocks
          /exception
          /logger
        /core
        /dashboard
        /data
        /layout
        /shared
        /widgets
      /content
      /libs <!-- bower_components -->
      index.html
    .bower.json
    ```

### Identify
<!-- ###### [Style [Y142](#style-y142)] -->

  - When you look at a file you should instantly know what it contains and represents.

    *Why?*: You spend less time hunting and pecking for code, and become more efficient. If this means you want longer file names, then so be it. Be descriptive with file names and keeping the contents of the file to exactly 1 component. Avoid files with multiple controllers, multiple services, or a mixture. There are deviations of the 1 per file rule when I have a set of very small features that are all related to each other, they are still easily identifiable.

### Flat
<!-- ###### [Style [Y143](#style-y143)] -->

  - Keep a flat folder structure as long as possible. When you get to 7+ files, begin considering separation.

    *Why?*: Nobody wants to search 7 levels of folders to find a file. Think about menus on web sites … anything deeper than 2 should take serious consideration. In a folder structure there is no hard and fast number rule, but when a folder has 7-10 files, that may be time to create subfolders. Base it on your comfort level. Use a flatter structure until there is an obvious value (to help the rest of LIFT) in creating a new folder.

### T-DRY (Try to Stick to DRY)
<!-- ###### [Style [Y144](#style-y144)] -->

  - Be DRY, but don't go nuts and sacrifice readability.

    *Why?*: Being DRY is important, but not crucial if it sacrifices the others in LIFT, which is why I call it T-DRY. I don’t want to type session-view.html for a view because, well, it’s obviously a view. If it is not obvious or by convention, then I name it.

**[Back to top](#table-of-contents)**

## Application Structure

### Overall Guidelines
<!-- ###### [Style [Y150](#style-y150)] -->

  - Have a near term view of implementation and a long term vision. In other words, start small and but keep in mind on where the app is heading down the road. All of the app's code goes in a root folder named `components`. All content is 1 feature per file. Each controller, service, module, view is in its own file. All 3rd party vendor scripts are stored in another root folder and not in the `components` folder. I didn't write them and I don't want them cluttering my app (`bower_components`, `scripts`, `libs`).

    Note: Find more details and reasoning behind the structure at [this original post on application structure](http://www.johnpapa.net/angular-app-structuring-guidelines/).

    Note: Small and good article about [components oriented structure](https://scotch.io/tutorials/angularjs-best-practices-directory-structure)

### Layout
<!-- ###### [Style [Y151](#style-y151)] -->

  - Place components that define the overall layout of the application in a folder named `layout`. These may include a shell view and controller may act as the container for the app, navigation, menus, content areas, and other regions.

    *Why?*: Organizes all layout in a single place re-used throughout the application.

### Folders-by-Feature Structure
<!-- ###### [Style [Y152](#style-y152)] -->

  - Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed.

    *Why?*: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names.

    *Why?*: The LIFT guidelines are all covered.

    *Why?*: Helps reduce the app from becoming cluttered through organizing the content and keeping them aligned with the LIFT guidelines.

    *Why?*: When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

    ```javascript
    /**
     * recommended
     */

    app/
        app.module.js
        app.config.js
        components/
            layouts/
                shell/
                    shell.html
                    shell.controller.js
                topnav/
                    topnav.html
                    topnav.controller.js
            people/
                people.routes.js
                attendees/
                    attendees.html
                    attendees.controller.js
                speakers/
                    speakers.html
                    speakers.controller.js
                speaker-detail/
                    speaker-detail.html
                    speaker-detail.controller.js
            shared/
                user-profile/
                    user-profile.directive.js
                    user-profile.directive.html
                calendar/
                    calendar.directive.js
                    calendar.directive.html
            sessions/
                sessions.html
                sessions.controller.js
                sessions.routes.js
                session-detail/
                    session-detail.html
                    session-detail.controller.js
    ```

      Note: **Do not use structuring using folders-by-type**. This requires moving to multiple folders when working on a feature and gets unwieldy quickly as the app grows to 5, 10 or 25+ views and controllers (and other features), which makes it more difficult than folder-by-feature to locate files.

    ```javascript
    /*
    * avoid
    * Alternative folders-by-type.
    * I recommend "folders-by-feature", instead.
    */

    app/
        app.module.js
        app.config.js
        app.routes.js
        controllers/
            attendees.js
            session-detail.js
            sessions.js
            shell.js
            speakers.js
            speaker-detail.js
            topnav.js
        directives/
            calendar.directive.js
            calendar.directive.html
            user-profile.directive.js
            user-profile.directive.html
        services/
            dataservice.js
            localstorage.js
            logger.js
            spinner.js
        views/
            attendees.html
            session-detail.html
            sessions.html
            shell.html
            speakers.html
            speaker-detail.html
            topnav.html
    ```

**[Back to top](#table-of-contents)**

## Modularity

### Many Small, Self Contained Modules
<!-- ###### [Style [Y160](#style-y160)] -->

  - Create small modules that encapsulate one responsibility.

    *Why?*: Modular applications make it easy to plug and go as they allow the development teams to build vertical slices of the applications and roll out incrementally. This means we can plug in new features as we develop them.

### Create an App Module
<!-- ###### [Style [Y161](#style-y161)] -->

  - Create an application root module whose role is pull together all of the modules and features of your application. Name this for your application.

    *Why?*: Angular encourages modularity and separation patterns. Creating an application root module whose role is to tie your other modules together provides a very straightforward way to add or remove modules from your application.

### Keep the App Module Thin
<!-- ###### [Style [Y162](#style-y162)] -->

  - Only put logic for pulling together the app in the application module. Leave features in their own modules.

    *Why?*: Adding additional roles to the application root to get remote data, display views, or other logic not related to pulling the app together muddies the app module and make both sets of features harder to reuse or turn off.

    *Why?*: The app module becomes a manifest that describes which modules help define the application.

### Feature Areas are Modules
<!-- ###### [Style [Y163](#style-y163)] -->

  - Create modules that represent feature areas, such as layout, reusable and shared services, dashboards, and app specific features (e.g. customers, admin, sales).

    *Why?*: Self contained modules can be added to the application with little or no friction.

    *Why?*: Sprints or iterations can focus on feature areas and turn them on at the end of the sprint or iteration.

    *Why?*: Separating feature areas into modules makes it easier to test the modules in isolation and reuse code.

### Reusable Blocks are Modules
<!-- ###### [Style [Y164](#style-y164)] -->

  - Create modules that represent reusable application blocks for common services such as exception handling, logging, diagnostics, security, and local data stashing.

    *Why?*: These types of features are needed in many applications, so by keeping them separated in their own modules they can be application generic and be reused across applications.

### Module Dependencies
<!-- ###### [Style [Y165](#style-y165)] -->

  - The application root module depends on the app specific feature modules and any shared or reusable modules.

    ![Modularity and Dependencies](https://raw.githubusercontent.com/johnpapa/angular-styleguide/master/assets/modularity-1.png)

    *Why?*: The main app module contains a quickly identifiable manifest of the application's features.

    *Why?*: Each feature area contains a manifest of what it depends on, so it can be pulled in as a dependency in other applications and still work.

    *Why?*: Intra-App features such as shared data services become easy to locate and share from within `app.core` (choose your favorite name for this module).

    Note: This is a strategy for consistency. There are many good options here. Choose one that is consistent, follows Angular's dependency rules, and is easy to maintain and scale.

    > My structures vary slightly between projects but they all follow these guidelines for structure and modularity. The implementation may vary depending on the features and the team. In other words, don't get hung up on an exact like-for-like structure but do justify your structure using consistency, maintainability, and efficiency in mind.

    > In a small app, you can also consider putting all the shared dependencies in the app module where the feature modules have no direct dependencies. This makes it easier to maintain the smaller application, but makes it harder to reuse modules outside of this application.

**[Back to top](#table-of-contents)**

## Startup Logic

### Configuration
<!-- ###### [Style [Y170](#style-y170)] -->

  - Inject code into [module configuration](https://docs.angularjs.org/guide/module#module-loading-dependencies) that must be configured before running the angular app. Ideal candidates include providers and constants.

    *Why?*: This makes it easier to have a less places for configuration.

  ```javascript
  angular
      .module('app')
      .config([
          'routerHelperProvider', 'exceptionHandlerProvider', 'toastr',
          function configure (routerHelperProvider, exceptionHandlerProvider, toastr) {
              exceptionHandlerProvider.configure(config.appErrorPrefix);
              configureStateHelper();

              toastr.options.timeOut = 4000;
              toastr.options.positionClass = 'toast-bottom-right';

              ////////////////

              function configureStateHelper() {
                  routerHelperProvider.configure({
                      docTitle: 'NG-Modular: '
                  });
              }
          } 
      ]);
  ```

### Run Blocks
<!-- ###### [Style [Y171](#style-y171)] -->

  - Any code that needs to run when an application starts should be declared in a factory, exposed via a function, and injected into the [run block](https://docs.angularjs.org/guide/module#module-loading-dependencies).

    *Why?*: Code directly in a run block can be difficult to test. Placing in a factory makes it easier to abstract and mock.

  ```javascript
  angular
      .module('app')
      .run([
          'authenticator', 'translator',
          function runBlock(authenticator, translator) {
              authenticator.initialize();
              translator.initialize();
          }
      ]);
  ```

**[Back to top](#table-of-contents)**

## Angular $ Wrapper Services

### $document and $window
<!-- ###### [Style [Y180](#style-y180)] -->

  - Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

    *Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

### $timeout and $interval
<!-- ###### [Style [Y181](#style-y181)] -->

  - Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

    *Why?*: These services are wrapped by Angular and more easily testable and handle Angular's digest cycle thus keeping data binding in sync.

**[Back to top](#table-of-contents)**

## Testing
Unit testing helps maintain clean code, as such I included some of my recommendations for unit testing foundations with links for more information.

### Write Tests with Stories
<!-- ###### [Style [Y190](#style-y190)] -->

  - Write a set of tests for every story. Start with an empty test and fill them in as you write the code for the story.

    *Why?*: Writing the test descriptions helps clearly define what your story will do, will not do, and how you can measure success.

    ```javascript
    it('should have Avengers controller', function() {
        // TODO
    });

    it('should find 1 Avenger when filtered by name', function() {
        // TODO
    });

    it('should have 10 Avengers', function() {
        // TODO (mock data?)
    });

    it('should return Avengers via XHR', function() {
        // TODO ($httpBackend?)
    });

    // and so on
    ```

### Testing Library
<!-- ###### [Style [Y191](#style-y191)] -->

  - Use [Mocha](http://mochajs.org) for unit testing.

    Note: When using Mocha, also consider choosing an assert library such as [Chai](http://chaijs.com).

### Test Runner
<!-- ###### [Style [Y192](#style-y192)] -->

  - Use [Karma](http://karma-runner.github.io) as a test runner.

    *Why?*: Karma is easy to configure to run once or automatically when you change your code.

    *Why?*: Karma hooks into your Continuous Integration process easily on its own or through Grunt or Gulp.

    *Why?*: Some IDE's are beginning to integrate with Karma, such as [WebStorm](http://www.jetbrains.com/webstorm/)

    *Why?*: Karma works well with task automation leaders such as [Gulp](http://www.gulpjs.com) (with [gulp-karma](https://github.com/lazd/gulp-karma)).

### Stubbing and Spying
<!-- ###### [Style [Y193](#style-y193)] -->

  - Use [Sinon](http://sinonjs.org/) for stubbing and spying.

    *Why?*: Sinon works well with both Jasmine and Mocha and extends the stubbing and spying features they offer.

    *Why?*: Sinon makes it easier to toggle between Jasmine and Mocha, if you want to try both.

    *Why?*: Sinon has descriptive messages when tests fail the assertions.

### Headless Browser
<!-- ###### [Style [Y194](#style-y194)] -->

  - Use [PhantomJS](http://phantomjs.org/) to run your tests on a server.

    *Why?*: PhantomJS is a headless browser that helps run your tests without needing a "visual" browser. So you do not have to install Chrome, Safari, IE, or other browsers on your server.

    Note: You should still test on all browsers in your environment, as appropriate for your target audience.

### Code Analysis
<!-- ###### [Style [Y195](#style-y195)] -->

  - Run ESLint on your tests.

    *Why?*: Tests are code. ESLint can help identify code quality issues that may cause the test to work improperly.

### Alleviate Globals for ESLint Rules on Tests
<!-- ###### [Style [Y196](#style-y196)] -->

  - Relax the rules on your test code to allow for common globals such as `describe` and `expect`. Relax the rules for expressions, as Mocha uses these.

    *Why?*: Your tests are code and require the same attention and code quality rules as all of your production code. However, global variables used by the testing framework.
    Add the following to your JSHint Options file.

    ```javascript
    "jasmine": true,
    "mocha": true
    ```

  ![Testing Tools](https://raw.githubusercontent.com/johnpapa/angular-styleguide/master/assets/testing-tools.png)

### Organizing Tests
<!-- ###### [Style [Y197](#style-y197)] -->

  - Place unit test files (specs) side-by-side with your client code. Place specs that cover server integration or test multiple components in a separate `tests` folder.

    *Why?*: Unit tests have a direct correlation to a specific component and file in source code.

    *Why?*: It is easier to keep them up to date since they are always in sight. When coding whether you do TDD or test during development or test after development, the specs are side-by-side and never out of sight nor mind, and thus more likely to be maintained which also helps maintain code coverage.

    *Why?*: When you update source code it is easier to go update the tests at the same time.

    *Why?*: Placing them side-by-side makes it easy to find them and easy to move them with the source code if you move the source.

    *Why?*: Having the spec nearby makes it easier for the source code reader to learn how the component is supposed to be used and to discover its known limitations.

    *Why?*: Separating specs so they are not in a distributed build is easy with grunt or gulp.

    ```
    /src/client/app/customers/customer-detail.controller.js
                             /customer-detail.controller.spec.js
                             /customers.controller.spec.js
                             /customers.controller-detail.spec.js
                             /customers.module.js
                             /customers.route.js
                             /customers.route.spec.js
    ```

**[Back to top](#table-of-contents)**

## Animations

### Usage
<!-- ###### [Style [Y210](#style-y210)] -->

  - Use subtle [animations with Angular](https://docs.angularjs.org/guide/animations) to transition between states for views and primary visual elements. Include the [ngAnimate module](https://docs.angularjs.org/api/ngAnimate). The 3 keys are subtle, smooth, seamless.

    *Why?*: Subtle animations can improve User Experience when used appropriately.

    *Why?*: Subtle animations can improve perceived performance as views transition.

### Sub Second
<!-- ###### [Style [Y211](#style-y211)] -->

  - Use short durations for animations. I generally start with 300ms and adjust until appropriate.

    *Why?*: Long animations can have the reverse affect on User Experience and perceived performance by giving the appearance of a slow application.

### animate.css
<!-- ###### [Style [Y212](#style-y212)] -->

  - Use [animate.css](http://daneden.github.io/animate.css/) for conventional animations.

    *Why?*: The animations that animate.css provides are fast, smooth, and easy to add to your application.

    *Why?*: Provides consistency in your animations.

    *Why?*: animate.css is widely used and tested.

    Note: See this [great post by Matias Niemelä on Angular animations](http://www.yearofmoo.com/2013/08/remastered-animation-in-angularjs-1-2.html)

**[Back to top](#table-of-contents)**

## Comments

### jsDoc
<!-- ###### [Style [Y220](#style-y220)] -->

  - If planning to produce documentation, use [`jsDoc`](http://usejsdoc.org/) syntax to document function names, description, params and returns. Use `@namespace` and `@memberOf` to match your app structure.

    *Why?*: You can generate (and regenerate) documentation from your code, instead of writing it from scratch.

    *Why?*: Provides consistency using a common industry tool.

    ```javascript
    /**
     * Logger Factory
     * @namespace Factories
     */
    (function() {
      angular
          .module('app')
          .factory('logger', logger);

      /**
       * @namespace Logger
       * @desc Application wide logger
       * @memberOf Factories
       */
      function logger($log) {
          var service = {
             logError: logError
          };
          return service;

          ////////////

          /**
           * @name logError
           * @desc Logs errors
           * @param {String} msg Message to log
           * @returns {String}
           * @memberOf Factories.Logger
           */
          function logError(msg) {
              var loggedMsg = 'Error: ' + msg;
              $log.error(loggedMsg);
              return loggedMsg;
          };
      }
    })();
    ```

**[Back to top](#table-of-contents)**

## ESLint

### Use an options file

  - Use ESLint for linting and checking your coding styles of your JavaScript and be sure to customize the ESLint options file and include in source control. See the [ESLint docs](http://eslint.org/docs/rules/) for details on the options.

    *Why?*: Provides a first alert prior to committing any code to source control.

    *Why?*: Provides consistency across your team.

    Latest eslint config you can find [here](https://github.com/valor-software/valor-style-guides/blob/master/.eslintrc)


**[Back to top](#table-of-contents)**

## Constants

### Vendor Globals
<!-- ###### [Style [Y240](#style-y240)] -->

  - Create an Angular Constant for vendor libraries' global variables.

    *Why?*: Provides a way to inject vendor libraries that otherwise are globals. This improves code testability by allowing you to more easily know what the dependencies of your components are (avoids leaky abstractions). It also allows you to mock these dependencies, where it makes sense.

    ```javascript
    // constants.js

    /* eslint-env toastr, moment */
    angular
        .module('app.core')
        .constant('toastr', toastr)
        .constant('moment', moment);
    ```

<!-- ###### [Style [Y241](#style-y241)] -->

  - Use constants for values that do not change and do not come from another service. When constants are used only for a module that may be reused in multiple applications, place constants in a file per module named after the module. Until this is required, keep constants in the main module in a `constants.js` file.

    *Why?*: A value that may change, even infrequently, should be retrieved from a service so you do not have to change the source code. For example, a url for a data service could be placed in a constants but a better place would be to load it from a web service.

    *Why?*: Constants can be injected into any angular component, including providers.

    *Why?*: When an application is separated into modules that may be reused in other applications, each stand-alone module should be able to operate on its own including any dependent constants.

    ```javascript
    // Constants used by the entire app
    angular
        .module('app.core')
        .constant('moment', moment);

    // Constants used only by the sales module
    angular
        .module('app.sales')
        .constant('events', {
            ORDER_CREATED: 'event_order_created',
            INVENTORY_DEPLETED: 'event_inventory_depleted'
        });
    ```

**[Back to top](#table-of-contents)**

<!-- ## File Templates and Snippets
Use file templates or snippets to help follow consistent styles and patterns. Here are templates and/or snippets for some of the web development editors and IDEs.
 -->

<!-- ### WebStorm -->
<!-- ###### [Style [Y252](#style-y252)] -->

<!--   - Angular snippets and file templates that follow these styles and guidelines. You can import them into your WebStorm settings:

    - Download the [WebStorm Angular file templates and snippets](assets/webstorm-angular-file-template.settings.jar?raw=true)
    - Open WebStorm and go to the `File` menu
    - Choose the `Import Settings` menu option
    - Select the file and click `OK`
    - In a JavaScript file type these commands followed by a `TAB`:

    ```javascript
    ng-c // creates an Angular controller
    ng-f // creates an Angular factory
    ng-m // creates an Angular module
    ```
 -->
<!-- **[Back to top](#table-of-contents)** -->

## Routing
Client-side routing is important for creating a navigation flow between views and composing views that are made of many smaller templates and directives.

<!-- ###### [Style [Y270](#style-y270)] -->

  - Use the [AngularUI Router](http://angular-ui.github.io/ui-router/) for client-side routing.

    *Why?*: UI Router offers all the features of the Angular router plus a few additional ones including nested routes and states.

    *Why?*: The syntax is quite similar to the Angular router and is easy to migrate to UI Router.

<!-- ###### [Style [Y271](#style-y271)] -->

  - Define routes for views in the module where they exist. Each module should contain the routes for the views in the module.

    *Why?*: Each module should be able to stand on its own.

    *Why?*: When removing a module or adding a module, the app will only contain routes that point to existing views.

    *Why?*: This makes it easy to enable or disable portions of an application without concern over orphaned routes.

**[Back to top](#table-of-contents)**

## Task Automation
Use [Gulp](http://gulpjs.com) for creating automated tasks.

<!-- ###### [Style [Y400](#style-y400)] -->

  - Use task automation to list module definition files `*.module.js` before all other application JavaScript files.

    *Why?*: Angular needs the module definitions to be registered before they are used.

    *Why?*: Naming modules with a specific pattern such as `*.module.js` makes it easy to grab them with a glob and list them first.

    ```javascript
    var clientApp = './src/client/app/';

    // Always grab module files first
    var files = [
      clientApp + '**/*.module.js',
      clientApp + '**/*.js'
    ];
    ```

**[Back to top](#table-of-contents)**

## Filters

<!-- ###### [Style [Y420](#style-y420)] -->

  - Avoid using filters for scanning all properties of a complex object graph. Use filters for select properties.

    *Why?*: Filters can easily be abused and negatively effect performance if not used wisely, for example when a filter hits a large and deep object graph.

**[Back to top](#table-of-contents)**

## Angular docs
For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions. If you have questions with the guide, feel free to leave them as issues in the repository. If you find a typo, create a pull request. The idea is to keep the content up to date and use github’s native feature to help tell the story with issues and PR’s, which are all searchable via google. Why? Because odds are if you have a question, someone else does too! You can learn more here at about how to contribute.

*By contributing to this repository you are agreeing to make your content available subject to the license of this repository.*

### Process
    1. Discuss the changes in a GitHub issue.
    2. Open a Pull Request, reference the issue, and explain the change and why it adds value.
    3. The Pull Request will be evaluated and either merged or declined.

## License

_tldr; Use this guide. Attributions are appreciated._

### Copyright

Copyright (c) 2015 [valorkin](https://github.com/valorkin)
Copyright (c) 2014-2015 [John Papa](http://johnpapa.net)

### (The MIT License)
Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**[Back to top](#table-of-contents)**
