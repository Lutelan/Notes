To bypass Same Site Cookie Restrictions one must first learn how a "site" or "origin" is defined in the context of Same Site cookies, consider a url as follows
```
https://app.example.com
```
The _site_ here consists of the schema i.e `https://` and the top level domain plus the and additional level of the domain name, denoted as _TLD+1_, which is `example.com`. These two things define a site, on the other hand a _origin_ is defined using the entire thing, schema, domain name, even port number, everything is included in the origin. Now let us move on to Same-Site Restrictions

##### Same-Site Cookie Restriction Types
All major browsers have three levels of Same Site restrictions, `strict, lax, none`. Developers can manually configure which level to use for which cookie using the `SameSite` attribute in the response headers, an example is as follows
```
Set-Cookie: session=<cookie>; SameSite=strict
```

__Strict__
If the cookie is set using `SameSite=strict`, browsers do not include the cookie in any cross site requests, this setting is used for cookies which might be involved in sensitive actions, this is the most secure option but can lead to loss of accessibility features.

__Lax__
A cookie with this setting will only be included in a cross site request if,
- The request uses the `GET` method
- The request resulted from a top-level navigation by the user, rather than by iframes, scripts or loading images/resources.
Thus these cookies are not included in cross site requests if the request method is `POST`.

__None__
If this attribute is set for a some cookie, SameSite restrictions are disabled completely for that particular cookie. The website must also include the `Secure` attribute in the response headers, to make sure the cookie is sent over `HTTPS` i.e. in a encrypted format.
```
Set-Cookie: cookie=<cookie>; SameSite=None; Secure
```

__Bypassing Same Site Lax restrictions using GET requests__
Most websites don't fuss to much about the method of the requests given to a certain endpoint, and as long as the the requests involves top-level navigation the cookie will be included, the simplest way to do so is using the `document.location` parameter.
```
<script>
	document.location = "https://sus-website.com/account/change-email?email=sus_email@gmail.com";
</script>
```
Also even if the `GET` request is not allowed, some frameworks have parameters which when set override the HTTP's method. For example in the _Symfony_ framework for PHP, if the `framework.http_method_override` option is set the the hidden parameter `_method` overrides the HTTP request's method, thus we can set the HTTP request method to GET but set `_method=POST`, which will let the request go through.

> Note in the Symfony framework, one can restrict which HTTP methods can be overridden using the `framework.allowed_http_method_override` option to prevent this vulnerability.

```
<form action="https://vulnerable-website.com/account/transfer-payment" method="GET"> 
	<input type="hidden" name="_method" value="POST"> 
	<input type="hidden" name="recipient" value="hacker"> 
	<input type="hidden" name="amount" value="1000000"> 	
</form>
```

__Bypassing SameSite restrictions using on-site gadgets__
If a cookie is set using the `SameSite=Strict` attribute then these cookies are not included in any cross site requests, one way to work around this using on site redirections. Suppose there is a part of the website which redirects to another part of the same website, and the redirect value is controllable by the user, one could use the attackers site to go to this part of the website, the url is designed in such a way that it redirects to the email change or whatever sensitive functionality.

And since the redirection occurs from the same site to the same site the cookies are included and thus the SameSite restriction has been bypassed.

__Bypassing SameSite Lax Restrictions using newly issued cookies__
On chrome if the SameSite type is not specified then the `lax` setting is used, but the thing is when it is not specified, chrome only applies the lax setting 120 seconds after the cookie has been assigned, this to avoid breaking OAuth systems.

Thus what can be done here is that the attacker's website opens a new tab using `window.open`, if the browser does not allow this without user interaction a `onclick`  event handler can be used, this opens a new tab which asks the user to login again, this results in the cookies being refreshed, then the previous tab requests the sensitive endpoint and since the lax restrictions have not yet been updated the request is carried out cross site along with the appropriate cookies.

