---
layout: post
title: Google Charts API Promise
description: The Google Charts API must be loaded asynchronously, so how can you tell when it's loaded?  In this post I answer that, and also give some related tips and tricks.
category: AngularJS
tags: [AngularJS, Google-Charts-API, Google-Visualization, Angular-Google-Chart]
date: 2015-04-02
last_modified_at: 2016-05-20 00:14:22 2016 -0400
author: Nicholas Bering
---

The Google Charts API must be loaded Asynchronously. So, when using the
[angular-google-chart] directive, how can you tell when then API has become
available to the controller?

### The Problem

The [Angular-Google-Chart][angular-google-chart] directive is designed in such
a way that you may never actually need direct access to the [Charts API]. But
this simple case is pretty narrow. If you need to use some of the advanced
visualization features, or have need to manipulate or filter data, then you will
need access to the Charts API directly.

### The Promise

The module provided with the angular-google-chart project provides a service
factory that returns a promise. That promise will resolve when the API has
loaded.

Now, to be more specific, the module is providing a promise that resolves when
the API is loaded. It does not return an error if the load fails. It does not
return any object on success. The resolve on success is just a signal that the
API is now available for use, at the namespaces defined by the Google Charts
team.

### Simple Case

This example simply instantiates a controller, which receives the
[googleChartApiPromise][code reference] through dependency injection.  When the
api has successfully loaded, the controller creates a new blank [DataTable]
instance.

```js
(function (){
  //app module already declared elsewhere
  angular.module('myApp')
    .controller('MainController', MainController);

  MainController.$inject = ['$scope', 'googleChartApiPromise'];

  function MainController($scope, googleChartApiPromise){
    // Setup bind-able objects
    $scope.chart = {};

    // Run function to initialize controller
    init();

    function init(){
      googleChartApiPromise.then(chartApiSuccess);
    }

    function chartApiSuccess(){
      $scope.chart.type = 'LineChart';
      $scope.chart.data = new google.visualization.DataTable();
    }
  }
})();
```

### $q.all()

So the first example works very well for simple scenarios. The next scenario is
a solution for when you need to use the api to process some data, or just to run
after some data has loaded.

This example uses the [$q][q] service that comes built into [AngularJS] to wrap two
promises together into a new promise that resolves when both promises have
resolved. This allows us to wait until both the Charts API and a request made
with the [http] service have resolved, and then do something with both.

```js
(function(){
  angular.module('myApp')
    .controller('myApp', MainController);

  MainController.$inject = ['$scope', '$q', '$http', 'googleChartApiPromise'];

  function MainController($scope, $q, googleChartApiPromise){
    $scope.chart = {};

    init();

    function init(){
      var dataPromise = $http({
        method: 'GET',
        url: 'https://api.example.com/some-data/'
      });

      $q.all({data: dataPromise, api: googleChartApiPromise})
        .then(apiLoadSuccess);
    }

    function apiLoadSuccess(result){
      $scope.chart.type = 'LineChart';
      //create a new DataTable loaded with data from the HTTP response
      $scope.chart.data = new google.visualization.DataTable(result.data.data);
    }


  }
})();
```

### The Not-So-Good Idea

I was going to write about passing the googleChartsApiPromise as a resolve to
ngRoute, but then while researching for this post I realized the the promise
never fails. And with that detail, I decided that it would theoretically be a
very bad idea to pass this promise to a route resolve, because if it fails the
route change will stall and never return an error. Not such a great idea after
all.

### Links

* [Angular Google Chart on GitHub][angular-google-chart]
* [Google Charts API Documentation][Charts API]
* [AngularJS]
  * [$q Service][q]
  * [$http Service][http]

[angular-google-chart]: <https://github.com/angular-google-chart/angular-google-chart/>
[Charts API]: <https://developers.google.com/chart/>
[code reference]: <https://github.com/angular-google-chart/angular-google-chart/blob/96a9b1d37c9d30d8666e45fe9d1255290d84951b/ng-google-chart.js#L37-78>
[DataTable]: <https://developers.google.com/chart/interactive/docs/reference#DataTable>
[q]: <https://docs.angularjs.org/api/ng/service/$q>
[AngularJS]: <https://angularjs.org/>
[http]: <https://docs.angularjs.org/api/ng/service/$http>
