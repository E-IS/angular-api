Project based on [Schweigi/angular-api-demo](https://github.com/Schweigi/angular-api-demo)

## Features

- Allow to raise/catch events (from/to service, directive or controller):

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

 - Allow async event, with Promise (e.g. to wait the end of event processing, cacth error, etc.):
 
```js    
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
