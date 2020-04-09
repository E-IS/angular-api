Allow to expose and use API between AngularJS (v1.5+) components.

Project based on [Schweigi/angular-api-demo](https://github.com/Schweigi/angular-api-demo)

## Features

- Allow to raise/catch events (from/to service, directive or controller);
- Allow async event, with Promise (e.g. to wait the end of event processing, cacth error, etc.).


## Usage
Add AngularJS and the angular-expose-api.js to your main file (index.html)
	
```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.11/angular.js"></script>
<script src="angular-expose-api.js"></script>
```

Set `ngApi` as a dependency in your module:

```javascript
var app = angular.module('YourApp', ['ngApi'])
```

You can also use the `Api` service into any AngularJS component:

Expose API in a Service:
```javascript
angular.module('my-service', ['ngApi'])
      .factory('myService', function(Api) {
  
  function doSomething() {    
    api.data.raise.changed(data); // Raise event
  }

  // Declare 
  var api = new Api(this, $scope.$id);
  api.registerEvent('data', 'changed');
  api.registerMethod('data', 'doSomething', doSomething);

  return {api}; // Exports
})
```

Controller:
```javascript
function MainCtrl($scope, myService) {

  // Catch the event
  myService.api.data.on.changed($scope, function(data) {
    alert("Service found data: " + data);
  });

}
```

### Example

- Declaring an event API, in a service, then catch emitted events in a controller: 
    ```js
    angular.module('login-service', ['ngApi'])
      .factory('loginService', function(Api) {
        var api = new Api(this, 'loginService' /* api ID */);
        
        // Declare extension points
        api.registerEvent('events', 'login');
        api.registerEvent('events', 'logout');
    
        function login(username, pwd) {
          // Do login stuff 
          /// (...)
          
          // Raise the event
          api.events.raise.login(username);
        }
    
        // exports
        return {
          api: api,
          login: login
        }    
      });
    
    // Any other components (service, controller) can be notified 
    var app = angular.module('myapp', ['ngApi', 'login-service']);
    app.controller('MainCtrl', ['$scope', 'loginService', function ($scope, loginService) {
      
      // Catch the event
      loginService.api.events.on.login($scope, function(user) {
        alert("User auth!");
      });
    
    }]);
    ```
  * Example, based on previous use case: 
 
    ```js    
    angular.module('login-service', ['ngApi'])
        .factory('loginService', function(Api) {
        // (...)
    
        function login(username, pwd) {
          /// (...)
          
          // Raise the event, as promise
          api.events.raisePromise.login(username)
            .then(function() {
              // event has been processed
            })
            .catch(function() {
              // Error during event processing
            });
        }
        // (...)
      });
    
    app.controller('MainCtrl', ['$scope', 'loginService', function ($scope, loginService) {
      
      // Catch the event
      loginService.api.events.on.login($scope, function(user, deferred) {
        if (username !== 'admin'/* Do any check*/) {
          return deferred.reject("User not allowed !");
        }

        alert("User auth!");
        // WARN: to NOT forget to resolve or reject
        return deferred.resolve();
      });
    
    }]);
    ```
