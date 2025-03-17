---
layout: post
date: 2025-03-17
categories: tips, c#, razor 
title: Using razor to template Html to a string
---

To generate html emails you need to use some sort of templating, for the backend it is best to use Razor templates as 
that is already there. You just need to use a custom renderer as it is not available out of the box. 

This renderer is based on [](https://stackoverflow.com/questions/63297920/programmatically-render-razor-page-as-html-string)

1. Create an interface `IRazorPartialToStringRenderer.cs`
    ```C#
    public interface IRazorPartialToStringRenderer
    {
        Task<string> RenderPartialToStringAsync<TModel>(string partialName, TModel model);
    }
    ```
2. Create the service `RazorPartialToStringRenderer.cs`
    ```C#
    public class RazorPartialToStringRenderer(
        IRazorViewEngine viewEngine,
        ITempDataProvider tempDataProvider,
        IServiceProvider serviceProvider)
        : IRazorPartialToStringRenderer
    {
        public async Task<string> RenderPartialToStringAsync<TModel>(string partialName, TModel model)
        {
            var actionContext = GetActionContext();
            var partial = FindView(actionContext, partialName);
            using (var output = new StringWriter())
            {
                var viewContext = new ViewContext(
                    actionContext,
                    partial,
                    new ViewDataDictionary<TModel>(
                        metadataProvider: new EmptyModelMetadataProvider(),
                        modelState: new ModelStateDictionary()) { Model = model },
                    new TempDataDictionary(
                        actionContext.HttpContext,
                        tempDataProvider),
                    output,
                    new HtmlHelperOptions()
                );
                await partial.RenderAsync(viewContext);
                return output.ToString();
            }
        }

        private IView FindView(ActionContext actionContext, string partialName)
        {
            var getPartialResult = viewEngine.GetView(null, partialName, false);
            if (getPartialResult.Success)
            {
                return getPartialResult.View;
            }

            var findPartialResult = viewEngine.FindView(actionContext, partialName, false);
            if (findPartialResult.Success)
            {
                return findPartialResult.View;
            }

            var searchedLocations = getPartialResult.SearchedLocations.Concat(findPartialResult.SearchedLocations);
            var errorMessage = string.Join(
                Environment.NewLine,
                new[] { $"Unable to find partial '{partialName}'. The following locations were searched:" }
                    .Concat(searchedLocations));

            throw new InvalidOperationException(errorMessage);
        }

        private ActionContext GetActionContext()
        {
            var httpContext = new DefaultHttpContext { RequestServices = serviceProvider };
            return new ActionContext(httpContext, new RouteData(), new ActionDescriptor());
        }
    }
    ```
3. Register the service in `Program.cs`, before the `WebApplication app = builder.Build();` line.
    ```C#
    builder.Services.AddScoped<IRazorPartialToStringRenderer, RazorPartialToStringRenderer>();
    ```


Then in you code where you are sending the email call the following to generate the html body for the email
```C#
// Inject IRazorPartialToStringRenderer renderer
var body = await renderer.RenderPartialToStringAsync("Emails/_ForgottenPasswordEmail",
            new ForgottenPasswordEmailModel() { Code = result });
```

The razor view can be any standard razor view, you can pass in a model and have layouts just as you do for any other razor view.