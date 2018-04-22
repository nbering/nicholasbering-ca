---
layout: post
title: How To Build Dual-Y Charts with Angular-Google-Chart
description: Sometimes you want to display data in the same chart, but the values for the y-axis are too different to plot on the same scale. Here's how to set up a chart that renders the y-values on different scales.
category: AngularJS
tags: [AngularJS, Google-Charts-API, Google-Visualization, Angular-Google-Chart]
author: Nicholas Bering
---

This post is a follow-up for a help request for Angular Google Chart. The documentation from Google seems to have moved on from their older charts, and shifted toward the newer Material Charts. The problem being that people are still using the old chart types, and by showing Dual-Y demos with the Material Charts, and removing the old charts examples, they are leaving a gap in the documentation. I had used the Dual-Y feature in the past, so here's a little how-to for anyone who might need to use Dual-Y without Material.

## Dual-Why?

Dual-Y charts solve a problem with graphing to display relationships between data where the scale or units of measure are not a good match. Examples might be things like rainfall (in inches or mm) vs. flow rate of a river (in liters per second). There would likely be a correlation between these two measurements, but they could not be plotted on the same axis.

So Dual-Y provides us with a second axis, usually with the scale marked on the opposite side of the chart, to plot our mis-matched data. I do not recommend Dual-Y unless the value gained by comparing the data in the same chart area outweighs the complexity of the chart itself. Reading Dual-Y charts can be a little complicated and are probably best presented to experts who understand exactly what it is they are looking at. Otherwise, they may require a little bit more explanation.

## Supported Charts

Note that Classic and Material chart variants have a different API for Dual-Y support. Also, BarCharts have the axis purposes reversed, so they use a Dual-X model in place of Dual-Y and the property names reflect that.

### Classic

- AreaChart
- BarChart - Dual-X
- CandlestickChart
- ColumnChart
- ComboChart
- Histogram
- LineChart
- SteppedAreaChart

### Material

- Bar - Dual-X
- Line

## Chart Options

### Classic

```js
$scope.myChartObject.type = "ColumnChart";
$scope.myChartObject.options = {
  vAxes: {
    0: { title: "Rainfall (mm)" },
    1: { title: "Flow Rate (L/s)" }
  },
  series: {
    0: { targetAxisIndex: 0 },
    1: { targetAxisIndex: 1 },
    2: { targetAxisIndex: 0 },
    3: { targetAxisIndex: 1 }
  }
};
```

### Material

```js
$scope.myChartObject.type = "google.charts.Bar";
$scope.myChartObject.options = {
  axes: {
    y: {
      rainfall: { label: "Rainfall (mm)" },
      flow: { label: "Flow (L/s)"}
    }
  },
  series: {
    0: { axis: 'rainfall' },
    1: { axis: 'flow' },
    2: { axis: 'rainfall' },
    3: { axis: 'flow'}
  }
};
```

## Examples

### Classic

`Insert CodePen Embed Here`

### Material

`Insert CodePen Embed Here`

## Conclusion

Dual-Y charts are great for representing relationships in data that is recorded in on different scales, but they can be a bit confusing, so they should not be over-used.

## Links
