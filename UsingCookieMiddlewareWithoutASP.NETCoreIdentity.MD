# 在ASP.NET Core 中使用Cookie中间件

ASP.NET Core 提供了Cookie中间件来序列化用户主题到一个加密的Cookie中并且在后来的请求中校验这个Cookie，再现用户并且分配到HttpContext对象的User属性中。如果你想提供自己的登录方式和用户数据你可以使用Cookie中间件来实现独立的功能。

## 添加和配置
第一步是增加Cookie中间件到你的应用中。首先使用nuget增加Microsoft.AspNetCore.Authentication.Cookies 程序包。然后添加下面的几行代码到Startup.cs文件的Configure方法中，且要在app.UseMvc()之前。
``` cs
app.UseCookieAuthentication(new CookieAuthenticationOptions()
   {
       AuthenticationScheme = "MyCookieMiddlewareInstance",
       LoginPath = new PathString("/Account/Unauthorized/"),
       AccessDeniedPath = new PathString("/Account/Forbidden/"),
       AutomaticAuthenticate = true,
       AutomaticChallenge = true
   });
```
上面的代码片段配置了一下几个选项；
1. 认证方案：这是一个已知中间件的值，当有多个实例的中间件如果你想限制授权到一个实例时这个选项将会起作用。
2. 登录路径：这是当用户试图访问资源但未经过身份验证时，程序将会将请求重定向到这个相对路径。
3. 禁止访问路径：当用户试图访问资源时，但未通过该资源的任何授权策略，请求将被重定向到这个相对路径。
4. 自动认证：这个标志表明中间件应该会在每个请求上进行验证和重建他创建的序列化主体。
5. 自动挑战：这个标志标明当中间件认证失败时应该重定向浏览器到登录路径或者禁止访问路径。

其他选项包括设置中间件所创建的声明的发行者，中间件存储的cookie名称，Cookie的域和cookie上的各种安全属性。默认情况下Cookie中间件将使用适当的安全选项，设置HTTPONLY避免cookie在客户端被JavaScript操作。当请求方式为HTTPS时限制Cookie的HTTPS操作。

## 创建Cookie
创建Cookie保存自己的信息，必须要初始化一个ClaimsPrincipal（类型）来序列化和保存你想保存的用户信息到Cookie中。每一次的方法调用都会在你的Controller（控制器）中有一个合适的ClaimsPrincipal对象。
``` cs
await HttpContext.Authentication.SignInAsync("MyCookieMiddlewareInstance", principal);
``` 
上面的代码将会创建一个加密的Cookie并且增加到当前的请求响应中。AuthenticationScheme明确规定在配置期间

## 退出
退出当前用户的登录，删除登录的cookie信息，可以在控制器中调用下面的方法。
``` cs
await HttpContext.Authentication.SignOutAsync("MyCookieMiddlewareInstance");
``` 
## 响应后端的变化

> **警告**  
> 一旦cookie创建就会成为身份单一认证的来源，即使在后台系统已经不可用，中间件也是不知道的，并且始终保持登录直到cookie失效。    

Cookie认证中间件在他的选项类中提供了一系列的事件，其中 **ValidateAsync()** 事件可以用来中断和重写cookie认证的验证方法。

考虑到后台用户的数据库中可能会有‘最后的修改时间’这一列，为了在数据库修改之后你可以废止当前的Cookie，第一当创建这个Cookie时添加一个最后修改的声明并包含当前的值，当数据库中的数据改变时，这个值也同时更新。

实现一个ValidateAsync()的事件重写你必须写一个具有如下签名的方法。

```cs
Task ValidateAsync(CookieValidatePrincipalContext context);
``` 

ASP.NET Core 认证在SecurityStampValidator中实现了这个验证。下面是一个类似的例子：
```cs
public static class LastChangedValidator
   {
       public static async Task ValidateAsync(CookieValidatePrincipalContext context)
       {
           // Pull database from registered DI services.
           var userRepository = context.HttpContext.RequestServices.GetRequiredService<IUserRepository>();
           var userPrincipal = context.Principal;

           // Look for the last changed claim.
           string lastChanged;
           lastChanged = (from c in userPrincipal.Claims
                          where c.Type == "LastUpdated"
                          select c.Value).FirstOrDefault();

           if (string.IsNullOrEmpty(lastChanged) ||
               !userRepository.ValidateLastChanged(userPrincipal, lastChanged))
           {
               context.RejectPrincipal();
               await context.HttpContext.Authentication.SignOutAsync("MyCookieMiddlewareInstance");
           }
       }
   }
``` 
这些要在Cookie中间件配置时进行注册
``` cs
app.UseCookieAuthentication(options =>
   {
       options.Events = new CookieAuthenticationEvents
       {
           // Set other options
           OnValidatePrincipal = LastChangedValidator.ValidateAsync
       };
   });
``` 

如果你想非破坏性的更新用户主体，例如，name更新了，要想以不影响安全的方式你可以调用 **context.ReplacePrincipal()** 并且设置 **context.ShouldRenew** 为 **true**。


## 控制Cookie选项

**CookieAuthenticationOptions** 配备了各种各样的配置选项是你能够很好的调节创建的Cookie。

- ClaimsIssuer - 被用来在任何中间件创建的属性之上。（看不懂）

- CookieDomain - 如果cookie domain被设置为 ** .contoso.com** 那么 **contoso.com,www.contoso.com,staging.contoso.com** 等等类似这样的域名也会被允许。

- CookieHttpOnly  - 这个标志指示这个 **cookie** 只会被服务端访问。默认值是true，修改这个属性将会开放你的应用造成 **Cookie** 盗窃，造成跨站脚本的bug。

- CookiePath  - 这个可以用来隔离运行在同一个 **host** 下的应用。如果你有一个应用运行在 **/app1** 上，并且想限制 **cookie** 限制仅仅被发送给自己，那么你应该设置 **CookiePath** 属性为 **/app1** ；Cookie将会明白只适用于道 **/app1** 或者他下面的请求。

- ExpireTimeSpan   - 这个 **TimeSpan** 时间段之后 **Cookie** 将会过期。

- SlidingExpiration  - 这个标志标记了如果超过了过期时间的一半后被访问那么Cookie将会被重置。新的过期时间将会后移到当前时间加上ExpireTimespan之后。当调用 **SignInAsync** 时可以通过 ** AuthenticationProperties ** 设置绝对的过期时间。通过限制验证cookie有效的时间，绝对期满可以提高应用程序的安全性。

## 持续性Cookie和绝对过期时间

您可能希望通过浏览器会话使cookie过期。也许你也想通过绝对过期时间和认证来结束cookie，那么你可以在登录认证和创建Cookie时使用HttpContext.Authentication.SignInAsync方法中的AuthenticationProperties参数类实现。AuthenticationProperties类在Microsoft.AspNetCore.Http.Authentication命名空间中。


例如
``` cs
await HttpContext.Authentication.SignInAsync(
       "MyCookieMiddlewareInstance",
       principal,
       new AuthenticationProperties
       {
           IsPersistent = true
       });
```
这个代码片段将会实现创建一个认证和相应的Cookie来实现即时浏览器关闭Cookie也能继续保留。任何在cookie属性中的过期时间的设置都将会保存下来。如果浏览器关闭时Cookie也过期了那么在重新启动浏览器是Cookie将会别清理。

``` cs
await HttpContext.Authentication.SignInAsync(
       "MyCookieMiddlewareInstance",
       principal,
       new AuthenticationProperties
       {
           ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
       });
``` 
这段代码将创建一个身份认证和相应的cookie且将持续20分钟。
任何在Cookie options中配置的动态选项都会被忽略。
ExpiresUtc 和 IsPersistent 这两个属性是相互独立的。



---


## 其实上面bb了那么多，都没用不如来个demo

``` cs

// 1. 在Startup.cs的Configure方法中加上
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    AuthenticationScheme = "UserAuth",　   // Cookie 验证方案名称，在写cookie时会用到。
    AutomaticAuthenticate = true,　　　 　// 是否自动启用验证，如果不启用，则即便客服端传输了Cookie信息，服务端也不会主动解析。除了明确配置了 [Authorize(ActiveAuthenticationSchemes = "上面的方案名")] 属性的地方，才会解析，此功能一般用在需要在同一应用中启用多种验证方案的时候。比如分Area.
    LoginPath = "/User/Index"　　　　　// 登录页
});

// 2. 新建UserController
// 3. 创建一个测试登录的方法（这里为了方便测是我用的是get方法，方便传参请求）
public IActionResult Login(int userId, string userName)
{
    WriteUser(userId, userName);
    return Content("Write");
}

private async void WriteUser(int userId, string userName)
{
    var identity = new ClaimsIdentity("Forms");     // 指定身份认证类型
    identity.AddClaim(new Claim(ClaimTypes.Sid, userId.ToString()));　　// 用户Id
    identity.AddClaim(new Claim(ClaimTypes.Name, userName));　　　　　　 // 用户名称
    var principal = new ClaimsPrincipal(identity);
    await HttpContext.Authentication.SignInAsync("UserAuth", principal, new AuthenticationProperties { IsPersistent = true , ExpiresUtc = DateTime.UtcNow.AddMinutes(20) });  //过期时间20分钟
}

// 4. 创建一个退出登录的方法
public async Task<ActionResult> Logout()
{
    await HttpContext.Authentication.SignOutAsync("UserAuth");   // Startup.cs中配置的验证方案名
    return RedirectToAction("User", "Index");
}

// 5. 创建一个获取cookie用户信息的方法方便调用
private int GetUserId()
{ 
    //var userName = User.Identity.Name;  //获取登录时存储的用户名称
    var userId = User.FindFirst(ClaimTypes.Sid).Value; // 获取登录时存储的Id
    if (string.IsNullOrEmpty(userId))
    {
        return 0;
    }
    else
    {
        return int.Parse(userId);
    }
}
// 或者写一个测试Action
public JsonResult CheckLogin()
{
    var userName = User.Identity.Name;  //获取登录时存储的用户名称
    var userId = User.FindFirst(ClaimTypes.Sid).Value; // 获取登录时存储的Id
    return Json({UserId:userId,UserName:userName});
}

// 6. 以上是加密的方式如果直接写好像也是可以的
HttpContext.Response.Cookies.Append("Key", "Value");

```

 
[原文](https://docs.microsoft.com/zh-cn/aspnet/core/security/authentication/cookie)
[参考](http://www.cnblogs.com/Joes/p/6023820.html)