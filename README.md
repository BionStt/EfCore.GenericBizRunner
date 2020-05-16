# EfCore.GenericBizRunner

**NOTE: Version 4.1.0 now requires you to install the NuGet package [GenericServices.StatusGeneric](https://www.nuget.org/packages/GenericServices.StatusGeneric/) in any of your assemblies that needs to interact with the GenericBizRunner's status.**

EfCore.GenericBizRunner (shortened to GenericBizRunner) is a framework to help build and run business logic when you are using [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/) for database accesses. Its aim is to totally isolate the business logic from other parts of the application, especially the user presentation/UI layers. It provides the following features:

* A standard pattern for writing business logic, including helper classes.
* An anti-corruption layer feature that act as a barrier between the business logic and the user presentation/UI layers. 
* The *BizRunner* handles the call to EF Core's `SaveChanges`, with optional validation.
* A service, known as a *BizRunner*, that runs your business logic.
* Very good use of Dependency Injection (DI), making calls to business logic very easy.

EfCore.GenericBizRunner is available as a [NuGet package](https://www.nuget.org/packages/EfCore.GenericBizRunner/), and on the [EfCore.GenericBizRunner](https://github.com/JonPSmith/EfCore.GenericBizRunner) GitHub repo. It is an open-source project under the MIT license.

**NOTE: Version 4 of GenericBizRunner supports both EF Core >=2.1 and EF Core >=3.0 via NetStandard2.0 and NetStandard2.1 versions.**

**NOTE: Version 3 of GenericBizRunner changed the way it handles DTOs to make the library more useful in Web APIs - see [upgrade guide](https://github.com/JonPSmith/EfCore.GenericBizRunner/blob/master/V3UpgradeGuide.md).**

## Example of using GenericBizRunner in ASP.NET Core MVC

Here is some code taken from the ExampleWebApp (which you can run), from the [OrdersController](https://github.com/JonPSmith/EfCore.GenericBizRunner/blob/master/ExampleWebApp/Controllers/OrdersController.cs)
to give you an idea of what it looks like. Note that every business logic call is very similar, just different interfaces and DTO/ViewModel classes.

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult ChangeDelivery(WebChangeDeliveryDto dto,
    [FromServices]IActionService<IChangeDeliverAction> service)
{
    if (!ModelState.IsValid)
    {
        service.ResetDto(dto); //resets any dropdown list etc.
        return View(dto);
    }

    service.RunBizAction(dto);

    if (!service.Status.HasErrors)
    {
        //We copy the message from the business logic to show 
        return RedirectToAction("ConfirmOrder", "Orders", 
            new { dto.OrderId, message = service.Status.Message });
    }

    //Otherwise errors, so I need to redisplay the page to the user
    service.Status.CopyErrorsToModelState(ModelState, dto);
    service.ResetDto(dto); //resets any dropdown list etc.
    return View(dto); //redisplay the page, with the errors
}
```

## Example of using GenericBizRunner in ASP.NET Core Web API

Here is a very simple example of creating a simple `TodoItem` taken from the [EfCore.GenericService.AspNet](https://github.com/JonPSmith/EfCore.GenericServices.AspNetCore/tree/master/ExampleWebApi) ExampleWebApi application. 

*Note: The [EfCore.GenericService.AspNetCore NuGet library](https://www.nuget.org/packages/EfCore.GenericServices.AspNetCore/) contains the `Response` methods to convert the output of the business logic called by GenericBizRunner into a useful json response.*

```csharp
[ProducesResponseType(typeof (TodoItem), 201)] //Tells Swagger that the success status is 201, not 200
[HttpPost]
public ActionResult<TodoItem> Post(CreateTodoDto item, 
    [FromServices]IActionService<ICreateTodoBizLogic> service)
{
    var result = service.RunBizAction<TodoItem>(item);
    return service.Status.Response(this, 
        "GetSingleTodo", new { id = result?.Id }, result);
}
```

## Why did I write this library?

I have built quite a few applications that contain business logic, some of it quite complex - things like optimisers and pricing engines to name but a few. 
Business logic can be a real challenge, and over the years I have perfected a architecture pattern that isolates the business logic so its easier to write and manage - see [this article](http://www.thereformedprogrammer.net/architecture-of-business-layer-working-with-entity-framework-core-and-v6-revisited/) and chapter 4 of my book, [Entity Framework in Action](http://bit.ly/2m8KRAZ).

Having perfected my pattern for handling business logic, then the next step was to automate the common parts of the pattern into a library. Which is where the
EfCore.GenericBizRunner library came from.

## More information on the business logic pattern and library

The following links start with general descriptions and get deeper towards the end.

* **[This article](http://www.thereformedprogrammer.net/architecture-of-business-layer-working-with-entity-framework-core-and-v6-revisited/)**, which describes my business pattern.
* **[Chapter 4 of my book](http://bit.ly/2m8KRAZ)**, which covers building of business logic using this pattern.
* **[This article](http://www.thereformedprogrammer.net/a-library-to-run-your-business-logic-when-using-entity-framework-core/)** that describes the EfCore.GenericBizAction library with examples.
* **[Project's Wiki](https://github.com/JonPSmith/EfCore.GenericBizRunner/wiki)**, which has a quick start guide and deeper documentation.
* **[Clone this repo](https://github.com/JonPSmith/EfCore.GenericBizRunner/)**, and run the ASP.NET Core application in it to see the business logic in action.
* **Read the example code**, in this repo.  
