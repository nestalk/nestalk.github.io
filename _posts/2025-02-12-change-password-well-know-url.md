---
layout: post
date: 2025-02-12
categories: tips, c#, security 
title: Well known change password url
---

Many modern password managers offer features to check if a password is secure and tell you when a password should be changed. If your site has a well known url for your password update page these managers can direct users to go straight to this page to make updating the password simpler.

The url for the change password page has been standardized as `/.well-known/change-password`. To implement this into your site you should redirect from that url to your own internal url for changing your password. For AspNet core this can be done using the rewriting middleware.

``` C#
var rewriterOptions = new RewriteOptions()
    .AddRedirect(@"^\.well-known/change-password", "/Identity/Account/Manage/ChangePassword");
app.UseRewriter(rewriterOptions);
```

One issue you may hit is if your site redirects not found urls to a not found page it may trip up the password managers into thinking the url does not work. A way to work around this is to use another standard to have a well known url that will always return the correct status code, this can be done by adding another redirect.

``` C#
.AddRedirect(@"^\.well-known/resource-that-should-not-exist-whose-status-code-should-not-be-200", "/", 404);
```