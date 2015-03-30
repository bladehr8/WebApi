---
layout: post
title: "String function parameter issue"
description: ""
category: Work around
---

Updated 2015-03-30: 
Since [Web API OData V5.5-beta](http://www.nuget.org/packages/Microsoft.AspNet.OData/5.5.0-beta), The [FromODataUri] for string parameter as below is resolved. If you doesn't use [FromODataUri] for string parameter and get some problem, please see the notes at bottom of this page. 

----------
Supposed you build an OData function with a string parameter, for example

{% highlight csharp %}

var builder = new ODataConventionModelBuilder();
builder.EntityType<Customer>().Function("MyFunction").Parameter<string>("p").Returns<int>();
return builder.GetEdmModel();

{% endhighlight %}

And you create the method in controller:

{% highlight csharp %}
public class CustomersController : ODataController
{
    [HttpGet]
    public int MyFunction(int key, [FromODataUri]string p)
    {
        ... 
    }
}
{% endhighlight %}

Then you invoke this function using the following parameters:

1. ~/Customers(1)/Default.MyFunction(p='113')
2. ~/Customers(1)/Default.MyFunction(p='114a')
3. ~/Customers(1)/Default.MyFunction(p='a115')

You will get the following parameter value in the MyFunction() method in the controller as:

{% highlight csharp %}
1. public int MyFunction(1, 113)
2. public int MyFunction(1, 114)  // wrong value
3. public int MyFunction(1, null) // wrong value

{% endhighlight %}


The workaround so far is to remove the [FromODataUri] attribute from MyFunction() in the controller. And, it's only for integer-starting string.

Thanks.


----
Notes:
For string parameter without using [FromODataUri] in the controller as:
{% highlight csharp %}
public class CustomersController : ODataController
{
    [HttpGet]
    public int MyFunction(int key, string p)
    {
        ... 
    }
}
{% endhighlight %}

Then you invoke this function using the following parameters and can get the parameter values as:

1. ~/Customers(1)/Default.MyFunction(p='') 
    
    key = 1, p = null and ModelState.IsValid = false
 
2. ~/Customers(1)/Default.MyFunction(p=null)

    key = 1, p = null and ModelState.IsValid = true

3. ~/Customers(1)/Default.MyFunction(p='a115')

    key = 1, p = "'a115'" and ModelState.IsValid = true

