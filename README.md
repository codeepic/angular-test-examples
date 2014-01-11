# How to use this repo:

1. Use the search bar above to look for tests.
2. If we are missing a test please PR and be sure to add keywords to it.

### Controllers

```
describe('controllers', function(){
  beforeEach(module('myApp.controllers'));

  it('should ....', inject(function() {
    //spec body
  }));

  it('should ....', inject(function() {
    //spec body
  }));
});
```

### Directives

```
describe('directives', function() {
  beforeEach(module('myApp.directives'));

  describe('app-version', function() {
    it('should print current version', function() {
      module(function($provide) {
        $provide.value('version', 'TEST_VER');
      });
      inject(function($compile, $rootScope) {
        var element = $compile('<span app-version></span>')($rootScope);
        expect(element.text()).toEqual('TEST_VER');
      });
    });
  });
});
```

### Filters

```
describe('filter', function() {
  beforeEach(module('myApp.filters'));

  describe('interpolate', function() {
    beforeEach(module(function($provide) {
      $provide.value('version', 'TEST_VER');
    }));

    it('should replace VERSION', inject(function(interpolateFilter) {
      expect(interpolateFilter('before %VERSION% after')).toEqual('before TEST_VER after');
    }));
  });
});
```

### Services

```
describe('service', function() {
  beforeEach(module('myApp.services'));

  describe('version', function() {
    it('should return current version', inject(function(version) {
      expect(version).toEqual('0.1');
    }));
  });
});
```

http://stackoverflow.com/questions/20916228/angularjs-what-would-this-look-like-had-it-been-created-tdd-style

```
describe('Module, Service, and Controller', function() {
  beforeEach(module('system', function($provide) {
    api = {
      get: function(url, params) {},
      put: function(url, params) {},
      post: function(url, params) {}
    };
  
    spyOn(api, 'get');
    spyOn(api, 'put');
    spyOn(api, 'post');
  
    $provide.value('Api', api);
  }));
  
  beforeEach(module('system_centers'));

  beforeEach(inject(function(System) {
    system = System;
  }));
  
  it('should load the system', function() {
    system.loadSystem(1, 0);
    expect(api.get).toHaveBeenCalledWith('lmc/contact/system/1', {card_id : 0});
  });
  
  it('should be able to complete the system', function() {
    system.completeSystem(20);
    expect(api.put).toHaveBeenCalledWith('system/complete/20');
  });
  
  it('should create the system', function() {
    system.createSystem(1, 3);
    expect(api.post).toHaveBeenCalledWith('contact/system/1', { card_id: 3, type: 'systems', origin: 'lmc'});
  });
  
  it('should not create the system if contact_id is 0', function() {
    system.createSystem(0, 20);
    expect(api.post).not.toHaveBeenCalled();
  });
  
  it('should not create the system if card_id is 0', function() {
    system.createSystem(1, 0);
    expect(api.post).not.toHaveBeenCalled();
  });
});
```

http://stackoverflow.com/questions/15927919/using-ngmock-to-simulate-http-calls-in-service-unit-tests

```
angular.module('myApp.services', [])
    .factory("exampleService", function ($http) {
        return {
            getData: function () {
                return $http.get("/exampleUrl");
            }
        }
  });
```

```
describe('service', function() {
  var $httpBackend;

  beforeEach(module('myApp.services'));

  beforeEach(inject(function ($injector) {
    $httpBackend = $injector.get("$httpBackend");
    $httpBackend.when("GET", "/exampleUrl")
        .respond(200, {value:"goodValue"});
  }));

  afterEach(function () {
    $httpBackend.flush()
    $httpBackend.verifyNoOutstandingExpectation();
    $httpBackend.verifyNoOutstandingRequest();
  });

  describe('exampleService successful http request', function () {
    it('.value should be "goodValue"', inject(function (exampleService) {

        exampleService.getData().success(function(response) {
          expect(response.value).toEqual("goodValue");
        }).error( function(response) {
          //should not error with $httpBackend interceptor 200 status
          expect(false).toEqual(true);
        });

    }));
  });
});
```

## Service that uses $http
http://stackoverflow.com/questions/12084410/angularjs-testing-service-method-that-uses-http-service

```
beforeEach(module('MyApp'));
beforeEach(inject(function(MyService, _$httpBackend_) {
    service = MyService;
    $httpBackend = _$httpBackend_;
}));

it('should invoke service with right paramaeters', function() {
    $httpBackend.expectPOST('/foo/bar', {
        "user": "testUser",
        "action": "testAction",
        "object": {}
    }, function(headers){
        return headers.Authorization === 'Basic YWxhZGRpbjpvcGVuIHNlc2FtZQ==';
    }).respond({});
    service.addStatement('testUser', 'testAction', {});
    $httpBackend.flush();
});
```

## Module checking Controller and making $http request with a service dependent on a different module

```
describe("Module: accountability |", function() {
    describe("Controllers |", function() {
        var api, accountability, $httpBackend, scope, ctrl;

        beforeEach(angular.mock.module('gamify'));

        beforeEach(inject(function(Accountability, Api, _$httpBackend_, _$rootScope_, _$controller_) {
            api = Api;
            accountability = Accountability;
            $httpBackend = _$httpBackend_;
            scope = _$rootScope_.$new();
            ctrl = _$controller_("AccountabilityCtrl", { $scope: scope });
        }));

        it('should have a AccountabiltyCtrl', function() {
            // not really sure this actually does anything.
            expect(module.AccountabilityCtrl).not.to.equal(null);
        });

        it('gamify object is hydrated', function() {
            expect(scope.gamify).to.exist;
            expect(scope.gamify).to.have.property('card_id');
            expect(scope.gamify).to.have.property('date');
        });

        it ('should have a load progress function', function() {
            expect(scope.loadProgress).to.not.equal(undefined);
        });

        it ('should have a go to rmc function', function() {
            expect(scope.goToAccountability).to.not.equal(undefined);
        });

        it('should load progress', function() {
            var date = "2013-12-31 21:30:23";
            var userId = 1;
            var cardId = 1;

            api.setBase('/api/v2/');

            $httpBackend.expectGET('/api/v2/user/progress/1?date=' + date + '&card_id=' + cardId + '&').respond({
                completed: 10,
                caught_up_completed: 3,
                caught_up: 10,
                upcoming: 100
            });
            scope.loadProgress(userId, date, cardId);
            $httpBackend.flush();

            expect(scope.progress.completed).to.equal(10);
            expect(scope.progress.caught_up_completed).to.equal(3);
            expect(scope.progress.caught_up).to.equal(10);
            expect(scope.progress.upcoming).to.equal(100);
            expect(scope.progress.todaysProgress).to.equal(13);
        });
    });
});
```

### Directives with events

```
describe('directives', function(){
    var windowMock, scope, controller, element, form;
  
    var changeInputValue;
  
    beforeEach(function(){ module("myApp.directives"); });
  
    beforeEach(function () {
        inject(function ($compile, $rootScope, $window){
			scope = $rootScope;
			element = angular.element('<form name="editForm"><input type="text" name="email" data-ng-model="name" text-box-blur  /></form>');
			  
			scope.name="";
			form = $compile(element)(scope);
			scope.$digest();
          
			windowMock=$window;
			spyOn(windowMock, "alert");
          
			changeInputValue = function (elem, value) {
				elem.val(value);
				elem.triggerHandler('blur');
			};
        });
    });
  
    it('Should call alert on losing focus', function(){
		changeInputValue(form.find('input'), "Ravi");
		expect(windowMock.alert).toHaveBeenCalled();
	});
});
```
