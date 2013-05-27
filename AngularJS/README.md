# Saltarelle.AngularJS

An attempt to port AngularJS to C# and Saltarelle compiler.

# How to use this library

Saltarelle.AngularJS is not a pure metadata import library, there are some 
helper functions that are written in C#, so it's necessary to link the
"Saltarelle.AngularJS.js" in your script bundles. Copy "Saltarelle.AngularJS.js" 
from this repo to the "Scripts/" directory and put the following reference: 

<script src="Scritps/Saltarelle.AngularJS.js"></script>

just after after the mscorlib.js reference. Also add a reference to
"Saltarelle.AngularJS.dll" in your project references.

Also, in your C# source files, add:

using AngularJS;

to make the AngularJS namespace available.

# Creating an AngularJS module

Create an angular module to be referenced with the ng-app directive:

```C#
Module app = new Module("myApp"); 
app.Register();
```

# Creating an AngularJS Controller

An AngularJs controller in C# is a static method defined in class-container. 
The class and the method need to be tagged with the `[Reflectable]` attribute.

The method name is the the name for the controller that can be later referenced in the
`ng-controller` directive.

Example:

```C#
[Reflectable]
public class ShoppingCartControllers 
{     
    [Reflectable]      
    public static void CartController(CartScope scope)
    {
	}
}
```

to be able to reference the `scope` in your controller you need to derive a class from `Scope`, for example:

```C#
public class CartScope : Scope
{
    public List<CartItem> items;
    public Action<int> remove;                 
    public Func<double> totalCart;
    public Func<double> subtotal;
    public double billDiscount;
}      
```

parameters in the controller method are injectables, e.g. you can write both:

```C#
public static void CartController(CartScope scope, Http http)
public static void CartController(Http http, CartScope scope)
```

without taking care of actual parameter ordering. The only requirement is that what you pass
as parameter is a registered angular service. Angular comes with a set of predefined services
like Scope, Http, Location and so on. For more details see Dependency Injection in AngularJS.

After the controller is defined you need to register it within the application-module:

```C#
app.RegisterControllers(new ShoppingCartControllers());                                  
```

all the static method contained in the `ShoppingCartControllers` class will be registered as
angular controller.

# Implementing a controller

Functions within a controller can be defined as `Action` or `Func` using the lamba expression syntax:

```C#
scope.remove = (index) => 
{
	scope.items.RemoveAt(index);
};
```

# Adding a watch 

A watch keeps on listening to a function, and when its value changes another function is called. First you define the monitoring function:

```C#
// defined inside the scope class
public Func<double> subtotal;

// defined within the controller method
scope.totalCart = () =>
{
	// ...
	return total;
};
```

then you define the callback method which is a function taking two parameters: newValue and oldValue.

```C#
WatchListener<double> calculateDiscount = (newValue,oldValue) =>
{
	scope.billDiscount = newValue > 100 ? 10 : 0;
};
```

finally you add the actual watch:

```C#
scope.Watch<double>(scope.totalCart, calculateDiscount);
```

# Defining a config function

Configs are functions that are called on module initialization. As with controllers, they need to be 
declared within a class container as static methods:

```C#
[Reflectable]
public class Configs
{            
    [Reflectable]
    public static void MyConfig()
    {
    }
}                 
```

and then registered with

```C#
app.RegisterConfig(new Configs());
```

as for controllers, config function parameters are injectable.


# Defining a Service (Factory)

A service (factory) is a function that returns an object resource that can be used in controllers or other parts of Angular and 
that is injected automatically using the service name. In C#, other than matching service name, the type of the service must match too.

In the following example the service called "Items" is defined

```C#
[Reflectable]
public class CartFactory 
{            
    [Reflectable]
    public static List<CartItem> Items()
    {
        var items = new List<CartItem>();
        items.Add( new CartItem() { title="AAAA", quantity= 1024, price= 44.95 } );
        items.Add( new CartItem() { title="BBBB", quantity= 2048, price= 55.95 } );
        return items;
    }
}    
```

and then registered with:

```C#
app.RegisterFactory(new CartFactory());
```

once the factory is registered, it can be used in any angular function by refering its name ("Items"). For example you can define a controller:

```C#
public static void CartController(CartScope scope, List<CartItem> Items)
```

and have the Items object passed automagically thanks to dependency injection.

# Defining a filter

A filter is a function that formats data that is used in html templates. For example the following filter adds the word "dollars" to a numeric value:

```C#
[Reflectable]
public class CartFilters
{            
    [Reflectable]
    public static string dollars(double input)
    {
        return input.ToString() + " dollars";
    }
}                 
```

and then registered with:

```C#
app.RegisterFilters(new CartFilters());  
```

in html template:

```HTML
<span>{{amount | dollars }}<span>
```

# Working with Http

Http is a pre-defined service ($http) that helps working with http requests, e.g. for RESTful services. There are methods for Get, Post, Put, Delete, Head and Jsonp. 

The result of Http requests are `promise`s that can be chained:

```C#
[Reflectable]
public static void PhoneListController(PhoneScope scope, Http http)
{
    http.Get("items.json").Success((data,status)=> {
		scope.items = data;
    }).Error((data,status)=>{ 
		Window.Alert("error!");
    });
}   
```

# Working with the $route service to define application routing a views

The RouteProvider service can be used to define the client-side application routings that are 
mapped to views that are rendered under the ng-view directive.

```C#
[Reflectable]
public static void ConfigRoute(RouteProvider routeProvider)
{
    routeProvider.when("/phones"          , new RouteMap() { TemplateUrl = "phonemain.html"   })
                 .when("/phones/:phoneId" , new RouteMap() { TemplateUrl = "phonedetail.html" })
                 .otherwise(                new RouteMap() { RedirectTo  = "/phones"          });
}
```C#

```HTML
<div ng-controller="PhoneListControllerDetail">   
   <p>phoneId parameter is {{ phoneId }}</p>
</div>
```HTML

# Working with directives

















 