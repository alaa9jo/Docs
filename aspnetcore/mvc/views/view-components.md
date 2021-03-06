---
title: View Components
author: rick-anderson
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: ab4705b7-59d7-4f31-bc97-ea7f292fe926
ms.prod: aspnet-core
uid: mvc/views/view-components
---
# View Components

>[!WARNING]
> This page documents version 1.0.0-rc1 and has not yet been updated for version 1.0.0

By [Rick Anderson](https://twitter.com/RickAndMSFT)

[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnet/mvc/views/view-components/sample)

## Introducing view components

New to ASP.NET Core MVC, view components are similar to partial views, but they are much more powerful. View components don’t use model binding, and only depend on the data you provide when calling into it. A view component:

* Renders a chunk rather than a whole response

* Includes the same separation-of-concerns and testability benefits found between a controller and view

* Can have parameters and business logic

* Is typically invoked from a layout page

View Components are intended anywhere you have reusable rendering logic that is too complex for a partial view, such as:

* Dynamic navigation menus

* Tag cloud (where it queries the database)

* Login panel

* Shopping cart

* Recently published articles

* Sidebar content on a typical blog

* A login panel that would be rendered on every page and show either the links to log out or log in, depending on the log in state of the user

A [`view component`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponent/index.html) consists of two parts, the class (typically derived from  `ViewComponent`) and the result it returns (typically a view). Like controllers, a view component can be a POCO, but most developers will want to take advantage of the methods and properties available by deriving from `ViewComponent`.

## Creating a view component

This section contains the high level requirements to create a view component. Later in the article we'll examine each step in detail and create a view component.

### The view component class

A view component class can be created by any of the following:

* Deriving from *ViewComponent*

* Decorating a class with the `[ViewComponent]` attribute, or deriving from a class with the `[ViewComponent]` attribute

* Creating a class where the name ends with the suffix *ViewComponent*

Like controllers, view components must be public, non-nested, and non-abstract classes. The view component name is the class name with the "ViewComponent" suffix removed. It can also be explicitly specified using the [`ViewComponentAttribute.Name`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponentAttribute/index.html#prop-Microsoft.AspNetCore.Mvc.ViewComponentAttribute.Name) property.

A view component class:

* Fully supports constructor [dependency injection](../../fundamentals/dependency-injection.md)

* Does not take part in the controller lifecycle, which means you can't use [filters](../controllers/filters.md) in a view component

### View component methods

A view component defines its logic in an `InvokeAsync` method that returns an [`IViewComponentResult`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/IViewComponentResult/index.html). Parameters come directly from invocation of the view component, not from model binding. A view component never directly handles a request. Typically a view component initializes a model and passes it to a view by calling the [`View`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponent/index.html) method. In summary, view component methods:

* Define an *InvokeAsync`* method that returns an `IViewComponentResult`

* Typically initializes a model and passes it to a view by calling the [`ViewComponent`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponent/index.html)  [`View`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewResult/index.html) method

* Parameters come from the calling method, not HTTP, there is no model binding

* Are not reachable directly as an HTTP endpoint, they are invoked from your code (usually in a view). A view component never handles a request

* Are overloaded on the signature rather than any details from the current HTTP request

### View search path

The runtime searches for the view in the following paths:

   * Views/<controller_name>/Components/<view_component_name>/<view_name>

   * Views/Shared/Components/<view_component_name>/<view_name>

The default view name for a view component is *Default*, which means your view file will typically be named *Default.cshtml*. You can specify a different view name when creating the view component result or when calling the `View` method.

We recommend you name the view file *Default.cshtml* and use the *Views/Shared/Components/<view_component_name>/<view_name>* path. The `PriorityList` view component used in this sample uses *Views/Shared/Components/PriorityList/Default.cshtml* for the view component view.

## Invoking a view component

To use the view component, call `@Component.InvokeAsync("Name of view component", <anonymous type containing parameters>)` from a view. The parameters will be passed to the `InvokeAsync` method.  The `PriorityList` view component developed in the article is invoked from the *Views/Todo/Index.cshtml* view file. In the following, the `InvokeAsync` method is called with two parameters:

````HTML
@await Component.InvokeAsync("PriorityList", new { maxPriority = 2, isDone = false })
   ````

### Invoking a view component directly from a controller

View components are typically invoked from a view, but you can invoke them directly from a controller method. While view components do not define endpoints like controllers, you can easily implement a controller action that returns the content of a [`ViewComponentResult`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponentResult/index.html).

In this example, the view component is called directly from the controller:

[!code-csharp[Main](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?range=23-26)]

## Walkthrough: Creating a simple view component

[Download](https://github.com/aspnet/Docs/tree/master/aspnet/mvc/views/view-components/sample), build and test the starter code. It's a simple project with a `Todo` controller that displays a list of *Todo* items.

![image](view-components/_static/2dos.png)

### Add a ViewComponent class

Create a *ViewComponents* folder and add the following `PriorityListViewComponent` class.

[!code-csharp[Main](view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponent1.cs?range=3-33)]

Notes on the code:

* View component classes can be contained in **any** folder in the project.

* Because the class name `PriorityListViewComponent` ends with the suffix **ViewComponent**, the runtime will use the string "PriorityList" when referencing the class component from a view. I'll explain that in more detail later.

* The `[ViewComponent]` attribute can change the name used to reference a view component. For example, we could have named the class `XYZ`,  and  applied the  `ViewComponent` attribute:

  <!-- literal_block {"ids": [], "linenos": false, "xml:space": "preserve", "language": "csharp"} -->

  ````csharp
  [ViewComponent(Name = "PriorityList")]
     public class XYZ : ViewComponent
     ````

* The `[ViewComponent]` attribute above tells the view component selector to use the name `PriorityList` when looking for the views associated with the component, and to use the string "PriorityList" when referencing the class component from a view. I'll explain that in more detail later.

* The component uses [dependency injection](../../fundamentals/dependency-injection.md) to make the data context available.

* `InvokeAsync` exposes a method which can be called from a view, and it can take an arbitrary number of arguments.

* The `InvokeAsync` method returns the set of `ToDo` items that are not completed and have priority lower than or equal to `maxPriority`.

### Create the view component Razor view

1.  Create the *Views/Shared/Components* folder. This folder **must** be named *Components*.

2.  Create the *Views/Shared/Components/PriorityList* folder. This folder name must match the name of the view component class, or the name of the class minus the suffix (if we followed convention and used the *ViewComponent* suffix in the class name). If you used the `ViewComponent` attribute, the class name would need to match the attribute designation.

3.  Create a *Views/Shared/Components/PriorityList/Default.cshtml* Razor view.

    [!code-html[Main](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]
    
    The Razor view takes a list of `TodoItem` and displays them. If the view component `InvokeAsync` method doesn't pass the name of the view (as in our sample), *Default* is used for the view name by convention. Later in the tutorial, I'll show you how to pass the name of the view. To override the default styling for a specific controller, add a view to the controller specific view folder (for example *Views/Todo/Components/PriorityList/Default.cshtml)*.
    
    If the view component was controller specific, you could add it to the controller specific folder (*Views/Todo/Components/PriorityList/Default.cshtml*)

4.  Add a `div` containing a call to the priority list component to the bottom of the *Views/Todo/index.cshtml* file:

    [!code-html[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFirst.cshtml?range=34-38)]

The markup `@Component.InvokeAsync` shows the syntax for calling view components. The first argument is the name of the component we want to invoke or call. Subsequent parameters are passed to the component. `InvokeAsync` can take an arbitrary number of arguments.

The following image shows the priority items:

![image](view-components/_static/pi.png)

You can also call the view component directly from the controller:

[!code-csharp[Main](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?range=23-26)]

### Specifying a view name

A complex view component might need to specify a non-default view under some conditions. The following code shows how to specify the "PVC" view  from the `InvokeAsync` method. Update the `InvokeAsync` method in the `PriorityListViewComponent` class.

[!code-csharp[Main](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponentFinal.cs?highlight=4,5,6,7,8,9&range=28-39)]

Copy the *Views/Shared/Components/PriorityList/Default.cshtml* file to a view named *Views/Shared/Components/PriorityList/PVC.cshtml*. Add a heading to indicate the PVC view is being used.

[!code-html[Main](../../mvc/views/view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/PVC.cshtml?highlight=3)]

Update *Views/TodoList/Index.cshtml*

[!code-html[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFinal.cshtml?range=31-)]

Run the app and verify PVC view.

![image](view-components/_static/pvc.png)

If the PVC view is not rendered, verify you are calling the view component with a priority of 4 or higher.

### Examine the view path

1. Change the priority parameter to three or less so the priority view is not returned.

2. Temporarily rename the *Views/Todo/Components/PriorityList/Default.cshtml* to *Temp.cshtml*.

3. Test the app, you'll get the following error:

   <!-- literal_block {"ids": [], "xml:space": "preserve"} -->

   ````
   An unhandled exception occurred while processing the request.

      InvalidOperationException: The view 'Components/PriorityList/Default'
         was not found. The following locations were searched:
         /Views/ToDo/Components/PriorityList/Default.cshtml
         /Views/Shared/Components/PriorityList/Default.cshtml.
      Microsoft.AspNetCore.Mvc.ViewEngines.ViewEngineResult.EnsureSuccessful()
      ````

4. Copy  *Views/Shared/Components/PriorityList/Default.cshtml to *Views/Todo/Components/PriorityList/Default.cshtml*.

5. Add some markup to the *Todo* view component view to indicate the view is from the *Todo* folder.

6. Test the **non-shared** component view.

![image](view-components/_static/shared.png)

### Avoiding magic strings

If you want compile time safety you can replace the hard coded view component name with the class name. Create the view component without the "ViewComponent" suffix:

[!code-csharp[Main](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityList.cs?highlight=10,14&range=4-34)]

Add a `using` statement to your Razor view file and use the `nameof` operator:

[!code-html[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexNameof.cshtml?range=1-6,33-)]

## Additional Resources

* [Dependency injection into views](dependency-injection.md)

* [`ViewComponent`](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/ViewComponent/index.html)
