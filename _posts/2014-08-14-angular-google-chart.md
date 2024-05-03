---
layout: post
title: Google Charts Directive for AngularJS
description: An introduction to my favourite pair of tools for adding dynamic JavaScript charts to web sites.  Uses the Google Charts API and AngularJS.
category: AngularJS
tags: [AngularJS, Google-Charts-API, Google-Visualization, Angular-Google-Chart]
date: 2014-08-14 13:49:00
last_modified_at: 2016-05-20 00:14:22 2016 -0400
author: Nicholas Bering
---

### Background

There's no doubt about it, if you're working on a project that involves
the display of data, you're probably going to need or want a chart.  It's no
coincidence that just about every spreadsheet package ever has had some sort of
charting feature.  Unfortunately, if your display medium is the web browser,
there's nothing built in to solve this problem.

I did a lot of digging around to find a simple solution to this problem, and
despite the extremely high demand for such capabilities, there were not many
options available in my price range (about $0).  The packages I did find were
either very complicated to set up, or lacking features most of the features I'd
come to expect from using commercial charts on other sites.

Then I found the <a href="https://developers.google.com/chart/">Google Charts API</a>.  It seemed to be the solution to all my
problems.  It was a full-featured JavaScript chart library, available for free,
with some caveats.  Specifically, it is free but not open source.  So the source
code is not available in any easily human-readable format.  Also, the code
itself is included not with a script tag directly.  You add a loader script
which you then call, providing this loader function with a callback function to
be run on completion.

This last part added some considerable complexity for a new AngularJS developer.
I did some hunting around and found a couple of workarounds, where you manually
bootstrap your Angular app in the loader's callback function.  This worked but
seemed unnecessary when the chart would not be displayed in every page of the
app.  Making my own directive wasn't exactly a piece of cake either.

### Enter the Directive

Then I stumbled upon the <a href="https://github.com/angular-google-chart/angular-google-chart/">Angular-Google-Chart</a> directive by
<a href="https://github.com/bouil/">Nicolas Bouil</a>.  This simple directive takes much of the work out of the
initial setup of a basic Google Chart.  The directive comes bundled with a
service that handles the API Loader call asynchronously using the AngularJS
Promise API, so you don't have to worry about fetching the library and passing
it a callback function, the included code handles it for you.

For a very simple application you can just setup an object with your chart data
and options, and give that object to the directive from the markup side of
things.

```html
<html ng-app="myApp">

<head>
  <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.0-beta.18/angular.js"></script>
  <script type="text/javascript" src="app.js"></script>
  <script type="text/javascript" src="ng-google-chart.js"></script>
  <title>Angular-Google-Charts Example</title>
</head>

<body>
  <div ng-controller="myController">
    <div google-chart chart="myChart"></div>
  </div>
</body>

</html>
```

```js
angular.module('myApp', ['googlechart'])
  .controller('myController', function($scope) {
    var chart1 = {};
    chart1.type = "AreaChart";
    chart1.displayed = false;
    chart1.data = {
      "cols": [{
        id: "month",
        label: "Month",
        type: "string"
      }, {
        id: "laptop-id",
        label: "Laptop",
        type: "number"
      }, {
        id: "desktop-id",
        label: "Desktop",
        type: "number"
      }, {
        id: "server-id",
        label: "Server",
        type: "number"
      }, {
        id: "cost-id",
        label: "Shipping",
        type: "number"
      }],
      "rows": [{
        c: [{
          v: "January"
        }, {
          v: 19,
          f: "42 items"
        }, {
          v: 12,
          f: "Ony 12 items"
        }, {
          v: 7,
          f: "7 servers"
        }, {
          v: 4
        }]
      }, {
        c: [{
          v: "February"
        }, {
          v: 13
        }, {
          v: 1,
          f: "1 unit (Out of stock this month)"
        }, {
          v: 12
        }, {
          v: 2
        }]
      }, {
        c: [{
            v: "March"
          }, {
            v: 24
          }, {
            v: 5
          }, {
            v: 11
          }, {
            v: 6
          }

        ]
      }]
    };

    chart1.options = {
      "title": "Sales per month",
      "isStacked": "true",
      "fill": 20,
      "displayExactValues": true,
      "vAxis": {
        "title": "Sales unit",
        "gridlines": {
          "count": 10
        }
      },
      "hAxis": {
        "title": "Date"
      }
    };
    $scope.myChart = chart1;
  });
```

This directive works pretty well for basic scenarios, but does not necessarily
support 100% of the features provided by the <a href="https://developers.google.com/chart/">Google Charts API</a>.  To be more
precise, it does not support <a href="https://developers.google.com/chart/interactive/docs/gallery/controls">Controls and Dashboards</a>, as these
features require binding the Chart to a Dashboard and passes responsibility for
drawing the chart to the Dashboard.  It's a little complicated to explain but
basically the workflow is a little different and the directive isn't setup to
support it.

<a href="https://embed.plnkr.co/x9ttq50KYzuFSULNIX2L/preview">View the example in Plunker</a>
