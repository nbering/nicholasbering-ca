---
layout: post
title: Google DataTable .Net Wrapper
description: A Google Visualization DataTable wrapper for building google chart objects from the server.
category: dot-net
tags: [google-visualization, asp.net]
---

So far my blog has focused on using the [Google Charts API][1] in [AngularJS][2] with [Angular-Google-Chart][3].  While this a fantastic combination for a client-side solution, so far it doesn't help us at all on the server side.  In fact, if you're trying to take full advantage of the Google Charts API, your job just got a little more complicated.

Getting data into a Google Chart is pretty easy for simple or static data. But if you're working with complex, dynamic data the JSON-formatted [DataTables][3] needed are a lot of work to set up.  For starters, the structure is kind of a collection of arrays that are logically linked.  Setting something like this up on either the server or client side takes a fair bit of code.  Also, the date format used by the API is not standard (at least, its not any kind of standard I've ever seen).

In my search for a simpler solution, I came across the [Google DataTable .Net Wrapper][4].  This helper/conversion library, available on [Nuget][6], makes it a lot
easier to get a valid Google DataTable to send off to the client.  It will even convert the standard [.Net DataTable][5] class, so there's a good chance you'll be able to adapt some of your existing data access code to new purpose.

## How-To

#### Get the Library

The easiest way to get the library to to install it with Nuget in Visual Studio.  From the Package Manager Console:

{% highlight ps1 %}
PM> Install-Package Google.DataTable.Net.Wrapper -Version 3.1.0
{% endhighlight %}

Or if you prefer, you can download the library from the project site on [Codeplex][4], and add the references manually.

#### Use It

You can work with the Google DataTable Wrapper three different ways:

- Create an object of type `Google.DataTable.Net.Wrapper.DataTable` and add your columns and rows directly to this object.  This may work well if you are building your project from the group up to work with Google Charts.
- Convert an `IEnumerable<T>` into a `Google.DataTable.Net.Wrapper.DataTable` using the extension method provided by the Wrapper Library.
- Or, Convert a `System.Data.DataTable` directly into a `Google.DataTable.Net.Wrapper.DataTable` instance.  This worked very well for me as I was already working with data from an SQL Server using ADO.Net, so I could just fill a DataTable from an adapter and convert it to a Google DataTable.

Examples of all three can be found on the Google DataTable .Net Wrapper's [Documentation Page][7].

[1]: https://developers.google.com/chart/
[2]: https://angularjs.org/
[3]: https://developers.google.com/chart/interactive/docs/datatables_dataviews
[4]: http://googledatatablelib.codeplex.com/
[5]: http://msdn.microsoft.com/en-us/library/system.data.datatable(v=vs.110).aspx
[6]: https://www.nuget.org/packages/Google.DataTable.Net.Wrapper/
[7]: http://googledatatablelib.codeplex.com/documentation
