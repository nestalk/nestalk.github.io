---
layout: post
date: 2025-01-29
categories: tips, c#, braindump
title: Setting up a AspNet Website with custom user class and all user management pages
---

Step through on how to get a site started with individual authentication setup and ready to be extended.

1. Create site using template
    - Use individual authentication
    - Use localdb

    ```
    dotnet new webapp --auth Individual -uld -o {SiteName}
    ```

2. Create custom identity user
   - Create new user class
    ```c#
    public class ApplicationUser : IdentityUser<Guid>
    {
    }
    ```
   - Update context to use class
    ```c#
    public class ApplicationDbContext : IdentityDbContext<ApplicationUser, IdentityRole<Guid>, Guid>
    ```
   - Update ```Views/Shared/_LoginPartial.cshtml``` to use new class
    ```razor
    @inject SignInManager<ApplicationUser> SignInManager
    @inject UserManager<ApplicationUser> UserManager
    ```
   - Update service setup
    ```c#
    builder.Services.AddDefaultIdentity<ApplicationUser>(options => options.SignIn.RequireConfirmedAccount = false)
    ```
   - Remove initial migration: ```dotnet ef migrations remove```
   - Add initial migration: ```dotnet ef migrations add InitialCreate```
3. Scaffold Identity pages
    - Install/Update scaffold template: ```dotnet tool install -g dotnet-aspnet-codegenerator```
    - Install/Update ef tools: ```dotnet tool install --global dotnet-ef```
    - Add nuget packages:
    ```
    dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
    dotnet add package Microsoft.EntityFrameworkCore.Design
    dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
    dotnet add package Microsoft.AspNetCore.Identity.UI
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    dotnet add package Microsoft.EntityFrameworkCore.Tools
    ```
    - Scaffold all the identity pages 
    ```
    dotnet aspnet-codegenerator identity -dc WebAuth.Data.ApplicationDbContext  --files "Account._StatusMessage;Account.AccessDenied;Account.ConfirmEmail;Account.ConfirmEmailChange;Account.ExternalLogin;Account.ForgotPassword;Account.ForgotPasswordConfirmation;Account.Lockout;Account.Login;Account.LoginWith2fa;Account.LoginWithRecoveryCode;Account.Logout;Account.Manage._Layout;Account.Manage._ManageNav;Account.Manage._StatusMessage;Account.Manage.ChangePassword;Account.Manage.DeletePersonalData;Account.Manage.Disable2fa;Account.Manage.DownloadPersonalData;Account.Manage.Email;Account.Manage.EnableAuthenticator;Account.Manage.ExternalLogins;Account.Manage.GenerateRecoveryCodes;Account.Manage.Index;Account.Manage.PersonalData;Account.Manage.ResetAuthenticator;Account.Manage.SetPassword;Account.Manage.ShowRecoveryCodes;Account.Manage.TwoFactorAuthentication;Account.Register;Account.RegisterConfirmation;Account.ResendEmailConfirmation;Account.ResetPassword;Account.ResetPasswordConfirmation;"
    ```
4. Apply migrations
    - Apply Migrations: ```dotnet ef database update```
