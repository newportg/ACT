# Security Component

![image](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/newportg/NorthStarContractIntegration/master/plantuml/SecurityComponent.puml)

## Managed Identity

The Header of any incoming message will contain 2 header keys **IDENTITY\_ENDPOINT** and **IDENTITY\_HEADER** they sometimes appear under there alias names **MSI\_ENDPOINT** and **MSI\_HEADER**.

### Retrieve the MI token

Use the DefaultAzureCredential to obtain the token for the managed identity.

```c#
var credential = new DefaultAzureCredential();
var token = await credential.GetTokenAsync(new TokenRequestContext(new[] { "https://management.azure.com/.default" }));

```

### Validate the token

Use the Microsoft.IdentityModel.Tokens library to validate the token. This involves checking the token’s signature, issuer, audience, and expiration.

```c#
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;

public bool ValidateToken(string token)
{
    var tokenHandler = new JwtSecurityTokenHandler();
    var validationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidIssuer = "https://sts.windows.net/{tenant-id}/",
        ValidateAudience = true,
        ValidAudience = "https://management.azure.com/",
        ValidateLifetime = true,
        IssuerSigningKeyResolver = (token, securityToken, kid, parameters) =>
        {
            // Retrieve signing keys from Azure AD
            var discoveryDocument = "https://login.microsoftonline.com/{tenant-id}/v2.0/.well-known/openid-configuration";
            var keys = new JsonWebKeySet(discoveryDocument).GetSigningKeys();
            return keys;
        }
    };

    try
    {
        tokenHandler.ValidateToken(token, validationParameters, out SecurityToken validatedToken);
        return true;
    }
    catch
    {
        return false;
    }
}
```

### Authorise the caller

Ensure the caller’s managed identity has the necessary permissions to access the function. This can be done by checking the claims in the token.

```c#
var jwtToken = tokenHandler.ReadJwtToken(token);
var claims = jwtToken.Claims;

// Check for specific roles or permissions
var hasAccess = claims.Any(c => c.Type == "roles" && c.Value == "YourRequiredRole");
```

### Function Code

By combining the above snippets, you would be able to check and authorise the calling application.

```c#
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    var token = req.Headers["Authorization"].ToString().Replace("Bearer ", "");
    if (!ValidateToken(token))
    {
        return new UnauthorizedResult();
    }

    // Proceed with your function logic
    return new OkResult();
}
```

## Service Bus Queues
Due to the nature of the information we are dealing with, messages on the service bus should be encrypted with AES 256.
