# **Policy-Based Authorization in ASP.NET Core**

## **What is Policy-Based Authorization?**

Policy-based authorization is a flexible and powerful authorization approach in ASP.NET Core. Unlike role-based authorization (which checks only user roles), **policy-based authorization** allows you to define custom authorization logic based on multiple factors, such as claims, roles, or complex business rules.

A **policy** consists of:

- **Requirements**: Conditions that must be met for authorization to succeed.
- **Handlers**: Logic that evaluates whether the requirement is met.
- **Registration**: Policies are registered in `Program.cs` (or `Startup.cs` in older versions).

---

## **How to Implement Policy-Based Authorization in ASP.NET Core**

### **Step 1: Configure Authorization Policies**

Define policies in `Program.cs`.

### **Step 2: Create a Custom Authorization Requirement and Handler**

For complex authorization rules, create a custom requirement and handler.

### **Step 3: Apply Policies to Controllers/Actions**

Use `[Authorize(Policy = "PolicyName")]` to protect specific actions.

---

## **Step-by-Step Implementation**

### **1. Configure Authorization Policies in `Program.cs`**

Define policies based on claims or roles:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add Authentication (If using JWT)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

// Add Authorization Policies
builder.Services.AddAuthorization(options =>
{
    // Policy requiring "Admin" role
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));

    // Policy requiring a specific claim
    options.AddPolicy("MustHaveEmployeeId", policy =>
        policy.RequireClaim("EmployeeId"));

    // Policy requiring multiple claims
    options.AddPolicy("ManagersOnly", policy =>
        policy.RequireClaim("Department", "HR", "IT").RequireRole("Manager"));
});

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

---

### **2. Apply Policy to Controllers or Actions**

Now, use the defined policies in controllers.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[Authorize(Policy = "AdminOnly")]
public class AdminController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

[Authorize(Policy = "MustHaveEmployeeId")]
public class EmployeeController : Controller
{
    public IActionResult Dashboard()
    {
        return View();
    }
}
```

---

### **3. Implement a Custom Authorization Requirement**

For more advanced authorization, create a custom requirement.

#### **Custom Requirement**

Create a class that represents a custom authorization rule.

```csharp
using Microsoft.AspNetCore.Authorization;

public class MinimumExperienceRequirement : IAuthorizationRequirement
{
    public int Years { get; }
    public MinimumExperienceRequirement(int years)
    {
        Years = years;
    }
}
```

#### **Custom Requirement Handler**

Create a handler that evaluates the requirement.

```csharp
using Microsoft.AspNetCore.Authorization;
using System.Security.Claims;
using System.Threading.Tasks;

public class MinimumExperienceHandler : AuthorizationHandler<MinimumExperienceRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, MinimumExperienceRequirement requirement)
    {
        var experienceClaim = context.User.FindFirst("ExperienceYears");

        if (experienceClaim != null && int.TryParse(experienceClaim.Value, out int years))
        {
            if (years >= requirement.Years)
            {
                context.Succeed(requirement);
            }
        }

        return Task.CompletedTask;
    }
}
```

#### **Register Custom Requirement and Handler**

Modify `Program.cs` to register the handler.

```csharp
builder.Services.AddSingleton<IAuthorizationHandler, MinimumExperienceHandler>();

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SeniorEmployees", policy =>
        policy.Requirements.Add(new MinimumExperienceRequirement(5)));
});
```

#### **Apply the Custom Policy**

Use the policy in a controller.

```csharp
[Authorize(Policy = "SeniorEmployees")]
public class ReportsController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

---

## **4. Use Policy-Based Authorization in Views**

You can also use policy-based authorization inside Razor views.

```html
@inject IAuthorizationService AuthorizationService

@if ((await AuthorizationService.AuthorizeAsync(User, "AdminOnly")).Succeeded)
{
    <p>Welcome, Admin!</p>
}
```

---

## **Summary**

1. **Define policies** in `Program.cs` using roles or claims.
2. **Apply policies** using `[Authorize(Policy = "PolicyName")]`.
3. **Create custom requirements and handlers** for advanced rules.
4. **Register handlers** and use them in policies.
5. **Use policies in views** for UI-based authorization.

This approach allows **fine-grained access control** without hardcoding role checks in the code.

