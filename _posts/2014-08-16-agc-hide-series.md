---
layout: post
title: Hide a Series with Angular-Google-Chart
category: AngularJS
tags: [AngularJS, Google-Charts-API, Google-Visualization, Angular-Google-Chart]
description: In my previous post - Google Charts Directive for AngularJS - I introduced the Angular-Google-Chart Project.  In this post I'm going to skip some of the basics and go straight to something advanced that I found to be in demand for both this directive and the Google Charts API in general.
date: 2014-08-16 15:53:00
last_modified_at: 2016-05-20 00:14:22 2016 -0400
author: Nicholas Bering
---
In my previous post - <a class="tracked" href="{% post_url 2014-08-14-angular-google-chart %}">Google Charts Directive for AngularJS</a> - I introduced
the <a class="tracked" href="https://github.com/angular-google-chart/angular-google-chart/">Angular-Google-Chart Project</a>.  In this post I'm going to skip some of
the basics and go straight to something advanced that I found to be in demand
for both this directive and the <a class="tracked" href="https://developers.google.com/chart/">Google Charts API</a> in general.

Sometimes when you're displaying data in charts, you just have too much data to
show at once.  The chart gets very cluttered and difficult to read (especially
in line charts, because the lines will usually overlap), but you just *have* to
show the weekly, four-week, and twelve-week rolling averages.  How can your user
possibly make a decision without considering all these figures, right?  Well,
they probably don't need them all at once so it might be nice to hide one or
more lines so that they can focus on what is important to them right now.

Fortunately, the directive we're using exposes a way to work with the On Select
event provided by the Google Charts API.  We pass a function call to the
directive's on-select property and it will give us the row and column id's of
the selection.  If there is no row id, then the user selected the legend.

In this application, all we care about is when the user selects the legend.
When they do, we will replace the view of the column with a calculated column,
and assign it's colour a grey tone to make it clear that the column is hidden.
If the legend item is clicked again, we reverse the change and display the data
again.

Here's the code:

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
    <div google-chart chart="myChart" on-select="seriesSelected(selectedItem)"></div>
  </div>
</body>

</html>
```

```js
angular.module('myApp', ['googlechart'])
  .controller('myController', function($scope) {
    var chart1 = {};
    chart1.type = "LineChart";
    chart1.displayed = false;
    chart1.data = {
      "cols": [
        { id: "month",      label: "Month",    type: "string"},
        { id: "laptop-id",  label: "Laptop",   type: "number"},
        { id: "desktop-id", label: "Desktop",  type: "number"},
        { id: "server-id",  label: "Server",   type: "number"},
        { id: "cost-id",    label: "Shipping",  type: "number"}
        ],
      "rows": [
        { c: [
          { v: "January"},
          { v: 19, f: "42 items"},
          { v: 12, f: "Ony 12 items"},
          { v: 7,  f: "7 servers"},
          { v: 4}
          ]
        },
        { c: [
          { v: "February"},
          { v: 13},
          { v: 1, f: "1 unit (Out of stock this month)"},
          { v: 12},
          { v: 2}
          ]

        },
        { c: [
          { v: "March"},
          { v: 24},
          { v: 5},
          { v: 11},
          { v: 6}
        ]
      }]
    };
    chart1.options = {
      "title": "Sales per month",
      "colors": ['#0000FF', '#009900', '#CC0000', '#DD9900'],
      //defaultColors property is not part of the strandard options
      //object.  I've added it here to keep a backup of the original
      //colors to call on when re-enabling a series.
      "defaultColors": ['#0000FF', '#009900', '#CC0000', '#DD9900'],
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

    //we initilize a basic view here, which the series
    //hiding code will interact with.
    chart1.view = {
      columns: [0, 1, 2, 3, 4]
    };
    $scope.myChart = chart1;

    $scope.seriesSelected = function(selectedItem) {
      var col = selectedItem.column;
      //If there's no row value, user clicked the legend.
      if (selectedItem.row === null) {
        //If true, the chart series is currently displayed normally.  Hide it.
        if ($scope.myChart.view.columns[col] == col) {
          //Replace the integer value with this object initializer.
          $scope.myChart.view.columns[col] = {
            //Take the label value and type from the existing column.
            label: $scope.myChart.data.cols[col].label,
            type: $scope.myChart.data.cols[col].type,
            //makes the new column a calculated column based on a function that returns null,
            //effectively hiding the series.
            calc: function() {
              return null;
            }
          };
          //Change the series color to grey to indicate that it is hidden.
          //Uses color[col-1] instead of colors[col] because the domain column (in my case the date values)
          //does not need a color value.
          $scope.myChart.options.colors[col - 1] = '#CCCCCC';
        }
        //series is currently hidden, bring it back.
        else {
          //Simply reassigning the integer column index value removes the calculated column.
          $scope.myChart.view.columns[col] = col;
          //I had the original colors already backed up in another array.  If you want to do this in a more
          //dynamic way (say if the user could change colors for example), then you'd need to have them backed
          //up when you switch to grey.
          $scope.myChart.options.colors[col - 1] = $scope.myChart.options.defaultColors[col - 1];
        }
      }
    };
  });
```

<a class="tracked" href="http://embed.plnkr.co/lOXTg5XRggwdctUedvfl/preview">View the example on Plunker.</a>
