---
layout: post
title:  "You Can't Force New CSRF Tokens in ColdFusion's CSRFGenerateToken() Without a Custom Key"
date:   2013-07-30 08:46:00 -0400
categories: ColdFusion
---

ColdFusion 10 includes a lot of security-related additions, including (finally!) built-in functions to generate and validate tokens to help prevent cross-site request forgery (CSRF) attacks. The two functions that work in parallel to deal with these kind of attacks are:

- CSRFGenerateToken([key] [,forceNew])  ([Docs](http://help.adobe.com/en_US/ColdFusion/10.0/CFMLRef/WS932f2e4c7c04df8f-1a0d37871353e31b968-7fff.html))
- CSRFVerifyToken(token [,key]) ([Docs](http://help.adobe.com/en_US/ColdFusion/10.0/CFMLRef/WS932f2e4c7c04df8f-1a0d37871353e31b968-7ffe.html))

The CSRFGenerateToken() function has an optional argument called "forceNew," which, according to the docs: "If set to true, a new token is generated every time the method is called. If false, in case a token exists for the key, the same key is returned." To prevent re-use of CSRF tokens during highly secure events like login, you may want to force the generation of a new CSRF token on each request.

Although the official ColdFusion docs state that both arguments to the CSRFGenerateToken() function are optional, they really are not. In order to utilize the forceNew option, you have to provide a custom key. 

You can create your own custom key just by typing in some random string into this function:

{% highlight javascript %}
<input type="hidden" name="csrfTokenToCheck" value="#CSRFGenerateToken("someLongishStringThatIsHopefullySecure", true)#" />
{% endhighlight %}

A better method would be to create a random string on application startup using ColdFusion's generateSecretKey() function.

{% highlight javascript %}
<cfset application.customCSRFTokenKey = generateSecretKey("AES") />
{% endhighlight %}

You'd then use that random string in every instance of the CSRFGenerateToken() function call in your application.

Note that every time the application restarts, a new custom CSRF token key is generated, and any existing token keys that may be out there (from customers sitting on your login page, for example), will be invalid and will not validate.

On your form processing page, you need to pass this same custom CSRF token key to the CSRFVerifyToken() function.

{% highlight html %}
<cfif NOT CSRFVerifyToken(form.csrfTokenToCheck, customCSRFTokenKey)>
  <h1>Bad!</h1>
  <cfabort>
</cfif>
{% endhighlight %}

So you have to go through a couple extra steps to use the forceNew option in CSRFGenerateToken, but I think it's worth it.