# Security

*From "Building APIs with C# & ASP.NET Core" by Spencer Schneidenbach (Frontend Masters)*

---

## Lesson — Security Considerations for Your API

---

### Core Concepts

| Term | Definition |
|------|-----------|
| **Authentication** | The process of determining a user's identity |
| **Authorization** | The process of determining if an authenticated user has access to a resource |

### JSON Web Tokens (JWTs)

JWTs serve as a compact, URL-safe means of representing claims to be transferred between two parties. They are transmitted via the `Authorization` header using the `Bearer` scheme:

```
Authorization: Bearer <token>
```

**Three components:**

1. **Header** — Token metadata including type and signing algorithm
2. **Payload** — User claims and additional data
3. **Signature** — Ensures the token hasn't been modified

> Helpful debugging tool: [jwt.io](https://jwt.io)

### Claims-Based Authorization

Claims are key-value pairs defining user attributes such as roles, permissions, and departmental information. Examples: `Name`, `Role`, `Department`.

### The [Authorize] Attribute

Restricts controller or action access based on authentication status:

```csharp
[Authorize]
public class EmployeesController : BaseController { ... }

[Authorize(Roles = "Administrator")]
public class AdminController : BaseController { ... }
```

### Policy-Based Authorization

Policies provide flexible, requirement-based access control beyond simple role checks. Policies combine requirements and handlers to implement nuanced authorization rules.

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("HROnly", policy =>
        policy.RequireRole("HR")
              .RequireClaim("Department", "Human Resources"));
});
```

Apply with:

```csharp
[Authorize(Policy = "HROnly")]
public IActionResult GetSalaryData() { ... }
```

### Recommended Patterns

| Scenario | Recommendation |
|----------|---------------|
| Modern single-page applications | Cookie-based authentication with ASP.NET Core Identity (not bearer tokens stored in local storage) |
| Integration / machine-to-machine APIs | API key schemes with scoped permissions (similar to Azure, Twilio) |

### Important Note

The demonstration implementation in the course is **intentionally simplified for educational purposes** — it basically allows anyone to authenticate to the API and is NOT production-ready.

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
