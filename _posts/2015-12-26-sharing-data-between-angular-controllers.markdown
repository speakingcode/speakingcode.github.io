---
layout: post
status: publish
published: true
title: Sharing Data Between AngularJS Controllers 
date: '2015-12-26 2:01:20'
categories:
- Code
- JavaScript
- AngularJS
---
Ok so we have a **controller**, and we got another controller. We want to share data, i.e. models, between the two controllers. And we want the binding to be such that whenever the models change, all the views see the change.

Simple. We need a **service** to host the model, and then the controllers can reference the service.

Let's start with the service. In this example, we'll call the _angular.service()_ factory method to create our service, conveniently named _Service_, wrapped up in its very own _Services_ **module** for cleanliness...

{% highlight javascript %}
angular.module('Services', [])
.service('Service', function() {
  var val = 0;
  
  this.getValue = function() {
    return val;
  };
  
  this.setValue = function(newval) {
    val = newval;
  }
});
{% endhighlight %}

This guy is basic POJO (plain ole javascript object)... he sets initializes his _val_ field to 0 and exposes an **accessor** and **mutator** for it. _val_ is our simple model we want to share.

Next we need our controllers. Let's start simple. We'll define a couple of basic controllers, each in their own module for clean separations. The modules will inject the _Services_ module we just created, and then the controllers themselves within these modules will inject the aptly named _Service_ service defined in the _Services_ module, now wired up to their respective modules. The controllers will set a field named _val_ on their _$scope_ property and set it to the value in the service via the _Service.getValue()_ method.

{% highlight javascript %}
angular.module('AControllers', ['Services'])
.controller('ACtrl', ['$scope', 'Service', function($scope, Service) {
  $scope.val = Service.getValue();
}]);

angular.module('BControllers', ['Services'])
.controller('BCtrl', ['$scope', 'Service', function($scope, Service) {
  $scope.val = Service.getValue();
}]);
{% endhighlight %}

Dead simple. Last little bits, we wire it all together in a top-level _app_ module and throw it into the page:

{% highlight javascript %}
angular.module('app', ['AControllers', 'BControllers']);
{% endhighlight %}

{% highlight html %}
<div ng-app="app">
  Controller A:
  <div ng-controller="ACtrl">
    <input ng-model="val" />
  </div>
  
  Controller B:
  <div ng-controller="BCtrl">
    <input ng-model="val" />
  </div>
</div>
{% endhighlight %}

Looking good. We see the model value, _0_, all the way up to the view. But wait! #@%! $*&% When we type a new value into the input boxes, it only changes on the respective controller's local _$scope_, not in _Service_ and thus not in the other controller, meaning our two views get out of sync. No worries, I gotcha bro! We could address this several ways. We could define a method in the controllers, say _updateValue(newValue)_, which calls _Service.setValue()_ with the new value, and then call it via the **ng-change** directive, or add a button and call _updateValue()_ via **ng-click**... but that won't take care of telling the other controller to update. We can take care of it all using _$scope.$watch()_. We'll need to set two watches in each controller, one to tell the service to update when the local _$scope.val_ changes, and one to update _$scope.val_ when the value is changed in _Service_...

{% highlight javascript %}
angular.module('AControllers', ['Services'])
.controller('ACtrl', ['$scope', 'Service', function($scope, Service) {
  $scope.val = Service.getValue();
  
  //if value changes in Service, change here in scope
  $scope.$watch(
    function(){ return Service.getValue() },
    function(newVal) {
      $scope.val = newVal;
    }
  );
  
  //if value changes in scope, change it in service
  $scope.$watch(
    function(){ return $scope.val },
    function(newVal) {
      Service.setValue(newVal);
    }
  );
}]);

angular.module('BControllers', ['Services'])
.controller('BCtrl', ['$scope', 'Service', function($scope, Service) {
  $scope.val = Service.getValue();
  
  
  //if value changes in Service, change here in scope
  $scope.$watch(
    function(){ return Service.getValue() },
    function(newVal) {
      $scope.val = newVal;
    }
  );
  
  //if value changes in scope, change it in service
  $scope.$watch(
    function(){ return $scope.val },
    function(newVal) {
      Service.setValue(newVal);
    }
  );
}]);
{% endhighlight %}

Awesome. Now it all works. [See it working in a live plunker](http://plnkr.co/edit/UrANDC2D9zjN5gCTUac6?p=preview). Happy coding!

