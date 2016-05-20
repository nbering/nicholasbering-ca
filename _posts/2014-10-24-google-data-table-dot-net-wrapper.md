---
layout: post
title: Google DataTable .Net Wrapper
description: A Google Visualization DataTable wrapper for building google chart objects from the server.  A great time saver for preparing data to be displayed client-side with the Google Charts API, from an Asp.Net MVC or Web API back-end.
category: dot-net
tags: [google-visualization, asp.net]
date: 2014-10-24 23:31:00
author: Nicholas Bering
---

So far my blog has focused on using the <a class="tracked" href="https://developers.google.com/chart/">Google Charts API</a> in <a class="tracked" href="https://angularjs.org/">AngularJS</a> with <a class="tracked" href="https://github.com/angular-google-chart/angular-google-chart/">Angular-Google-Chart</a>.  While this a fantastic combination for a client-side solution, so far it doesn't help us at all on the server side.  In fact, if you're trying to take full advantage of the Google Charts API, your job just got a little more complicated.

Getting data into a Google Chart is pretty easy for small amounts of static data, but if you're working with large amounts of dynamic data the JSON-formatted <a class="tracked" href="https://developers.google.com/chart/interactive/docs/datatables_dataviews">DataTables</a> needed are a lot of work to set up.  For starters, the structure is kind of a collection of arrays that are logically linked.  Setting something like this up on either the server or client side takes a bit of work.  Also, the date format used by the API is not standard (at least, its not any kind of standard I've ever seen).

In my search for a simpler solution, I came across the <a class="tracked href="http://googledatatablelib.codeplex.com/">Google DataTable .Net Wrapper</a>.  This helper/conversion library, available on <a class="tracked" href="https://www.nuget.org/packages/Google.DataTable.Net.Wrapper/">Nuget</a>, makes it a lot easier to build a valid Google DataTable on the server, to send off to the client.  It will even convert the standard <a class="tracked" href="http://msdn.microsoft.com/en-us/library/system.data.datatable(v=vs.110).aspx">.Net DataTable</a> class, so there's a good chance you'll be able to adapt some of your existing data access code to new purpose.

## How-To

#### Get the Library

The easiest way to get the library to to install it with Nuget in Visual Studio.  From the Package Manager Console:

<div class="nuget-console-command">PM> Install-Package Google.DataTable.Net.Wrapper -Version 3.1.0</div>

Or if you prefer, you can download the library from the project site on <a class="tracked" href="http://googledatatablelib.codeplex.com/">Codeplex</a>, and add the references manually.
#### Use It

You can work with the Google DataTable Wrapper three different ways:

* Create an object of type `Google.DataTable.Net.Wrapper.DataTable` and add your columns and rows directly to this object.  This may work well if you are building your project from the group up to work with Google Charts.

```csharp
public string GetStatisticsForChart(string messageCode)
{
    //some repository that returns data....
    var data = _statisticsRepository.GetPerMessage(messageCode);

    //It simply returns a list of objects with Year and Count properties.
    var query = (from t in data
                    group t by new {t.TimeStamp.Year}
                    into grp
                    select new
                        {
                            grp.Key.Year,
                            Count = grp.Count()
                        }).ToList();

    //let's instantiate the DataTable.
    var dt = new Google.DataTable.Net.Wrapper.DataTable();
    dt.AddColumn(new Column(ColumnType.String, "Year", "Year"));
    dt.AddColumn(new Column(ColumnType.Number, "Count", "Count"));

    foreach (var item in query)
    {
        Row r = dt.NewRow();
        r.AddCellRange(new Cell[]
            {
                new Cell(item.Year),
                new Cell(item.Count)
            });
        dt.AddRow(r);
    }

    //Let's create a Json string as expected by the Google Charts API.
    return dt.GetJson();
}
```

* Convert an `IEnumerable<T>` into a `Google.DataTable.Net.Wrapper.DataTable` using the extension method provided by the Wrapper Library.

```csharp
var list = new[]
                {
                    new {Name = "Dogs", Count = 5},
                    new {Name = "Cats", Count = 2}
                };

var json = list.ToGoogleDataTable()
               .NewColumn(new Column(ColumnType.String, "Name"), x => x.Name)
               .NewColumn(new Column(ColumnType.Number, "Count"), x => x.Count)
               .Build()
               .GetJson();
```

* Or, Convert a `System.Data.DataTable` directly into a `Google.DataTable.Net.Wrapper.DataTable` instance.  This worked very well for me as I was already working with data from an SQL Server using ADO.Net, so I could just fill a DataTable from an adapter and convert it to a Google DataTable.

```csharp
using (var sysDt = new System.Data.DataTable())
{
    sysDt.Columns.Add("firstcolumn", typeof(string));
    sysDt.Columns.Add("secondcolumn", typeof(int));
    sysDt.Columns.Add("thirdcolumn", typeof(decimal));
    sysDt.Locale = CultureInfo.InvariantCulture;

    var row1 = sysDt.NewRow();
    row1[0] = "Ciao";
    row1[1] = 10;
    row1[2] = 2.2;
    sysDt.Rows.Add(row1);

    var dataTable = sysDt.ToGoogleDataTable();

    var json = dataTable.GetJson();
}
```

The past three code samples are from the Google DataTable .Net Wrapper Project's <a class="tracked" href="http://googledatatablelib.codeplex.com/documentation">Documentation Page</a>, used with permission.

#### Using with ASP.Net Web API

If you've gotten this far and have decided this solves some problems for you, I'll give you a free solution to one gotcha here.  If you're using this library to return JSON from ASP.Net Web API, you'll need a little more overheard code.  The problem is that Web API wants to be helpful and automatically attempts to format the output for you.  Unfortunately to return a string as JSON we have to override this functionality and set the response headers manually.

```csharp
public class ChartDataController : ApiController
{

  [Route("api/google-chart")]
  [HttpGet]
  public HttpResponseMessage getChart()
  {
    HttpResponseMessage response;

    //Get DataTable instance from first example above.
    Google.DataTable.Net.Wrapper.DataTable dt = GetStatisticsForChart(string messageCode);

    //Check if the request accepts a Json Result
    if (AcceptJson(Request))
    {
      response = new HttpResponseMessage();
      response.Content = new StringContent(dt.GetJson());
      response.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("application/json");
    }
    //If Json is not accepted, let Web API's media formatters
    //handle the result (the request should accept json, but you never know...)
    else
      response=Request.CreateResponse(HttpStatusCode.OK, dt);
    return response;
  }

  protected bool AcceptsJson(HttpRequestMessage request)
  {
    var acceptHeader = request.Headers.Accept;
    bool result = false;
    foreach (var mime in acceptHeader)
    {
      if (mime.MediaType == "application/json")
      {
        result = true;
      }
    }
    return result;
  }
}
```

## Conclusion

The Google Charts API is a great starting point if you need a chart on a web page.  If you're using angular the Angular-Google-Charts project provides a simple directive to handle loading the library and drawing the chart.  These client-side tools make setting up the data on the server side a little more complicated though, so using the Google DataTable .Net Wrapper library completes the solution nicely if you're using ASP.Net Web API.
