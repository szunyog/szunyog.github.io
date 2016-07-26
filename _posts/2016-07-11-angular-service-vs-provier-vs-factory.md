---
layout: post
title:  "Angular service vs provider vs factory"
date:   2016-07-11 08:31:49
categories: ["js angular"]
description: The difference between angular service, factory and provider when declaring as an injectable argument.
---

The difference between angular service, factory and provider when declaring as an
injectable argument.

Services
--------

{% highlight javascript %}
module.service( 'serviceName', function );
{% endhighlight %}

When declaring serviceName as an injectable argument you will get a reference
to the actual function passed to module.service(...). 

Factories
---------

{% highlight javascript %}
module.factory( 'factoryName', function );
{% endhighlight %}

When declaring factoryName as an injectable argument you will get the value
that is returned by invoking the function reference passed to
module.factory(...).

Providers
---------

{% highlight javascript %}
module.provider( 'providerName', function );
{% endhighlight %}

When declaring providerName as an injectable argument you will get the value
that is returned by invoking the $get method of the function
reference passed to module.provider(...).

Sample code
-----------
HTML:

{% highlight javascript %}
<div ng-controller="MyCtrl">
  <p>{{ "{{service" }}}}</p>
  <p>{{ "{{provider" }}}}</p>
  <p>{{ "{{factory"}}}}</p>
</div>
{% endhighlight %}

Javascript:

{% highlight javascript %}
var myApp = angular.module('myApp',[]);

var func = function() {
    this.name = "func name";
  this.$get = function() {
    this.name = "func $get name";
    return "func.$get(): " + this.name;
  }
  return "func return value: " + this.name;
}

myApp.service('testService', func);
myApp.provider('testProvider', func);
myApp.factory('testFactory', func);

function MyCtrl($scope, testService, testProvider, testFactory) {
    $scope.service = "the Service is actual function value = " + testService;
    $scope.provider = "the Provider is actual function value = " + testProvider;
    $scope.factory = "the factory is actual function value = " + testFactory;
}
{% endhighlight %}

The sample code also can be found on [JSFiddle][jsfiddle].

[jsfiddle]: http://jsfiddle.net/5tbmtaz0/4/

