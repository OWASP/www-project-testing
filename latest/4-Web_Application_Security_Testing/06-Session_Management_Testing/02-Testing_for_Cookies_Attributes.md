---

layout: col-document
title: WSTG - Latest
tags: WSTG

---
# Testing for Cookies Attributes

|ID           |
|-------------|
|WSTG-SESS-002|

## Summary

Cookies are often a key attack vector for malicious users (typically targeting other users) and the application should always take due diligence to protect cookies. This section looks at how an application can take the necessary precautions when assigning cookies, and how to test that these attributes have been correctly configured.

The importance of secure use of cookies cannot be understated, especially within dynamic web applications, which need to maintain state across a stateless protocol such as HTTP. To understand the importance of cookies it is imperative to understand what they are primarily used for. These primary functions usually consist of being used as a session authorization and authentication tokens, or as a temporary data container. Thus, if an attacker were able to acquire a session token (for example, by exploiting a cross site scripting vulnerability or by sniffing an unencrypted session), then they could use this cookie to hijack a valid session.

Additionally, cookies are set to maintain state across multiple requests. Since HTTP is stateless, the server cannot determine if a request it receives is part of a current session or the start of a new session without some type of identifier. This identifier is very commonly a cookie although other methods are also possible. There are many different types of applications that need to keep track of session state across multiple requests. A common one that comes to mind would be an online store. As a user adds multiple items to a shopping cart, this data needs to be retained in subsequent requests to the application. Cookies are very commonly used for this task and are set by the application using the `Set-Cookie` directive in the application's HTTP response, and is usually in a `name=value` format (if cookies are enabled and if they are supported, as is the case for all modern web browsers). Once an application has told the browser to use a particular cookie, the browser will send this cookie in each subsequent request. A cookie can contain data such as items from an online shopping cart, the price of these items, the quantity of these items, personal information, user IDs, etc.

Due to the sensitive nature of information in cookies, they are typically encoded or encrypted in an attempt to protect the information they contain. Often, multiple cookies will be set (separated by a semicolon) upon subsequent requests. For example, in the case of an online store, a new cookie could be set as the user adds multiple items to the shopping cart. Additionally, there will typically be a cookie for authentication (session token as indicated above) once the user logs in, and multiple other cookies used to identify the items the user wishes to purchase and their auxiliary information (i.e., price and quantity) in the online store type of application.

Once the tester has an understanding of how cookies are set, when they are set, what they are used for, why they are used, and their importance, they should take a look at what attributes can be set for a cookie and how to test if they are secure. The following is a list of the attributes that can be set for each cookie and what they mean. The next section will focus on how to test for each attribute.

- `Secure` - This attribute tells the browser to only send the cookie if the request is being sent over a secure channel such as HTTPS. This will help protect the cookie from being passed over unencrypted requests. If the application can be accessed over both HTTP and HTTPS, then there is the potential that the cookie can be sent in clear text.
- `HttpOnly` - This attribute is used to help prevent attacks such as cross-site scripting, since it does not allow the cookie to be accessed via a client side script such as JavaScript. Note that not all browsers support this functionality.
- `Domain` - This attribute is used to compare against the domain of the server in which the URL is being requested. If the domain matches or if it is a subdomain, then the `path` attribute will be checked next.

Note that only hosts within the specified domain can set a cookie for that domain. Additionally, the `domain` attribute cannot be a top level domain (such as `.gov` or `.com`) to prevent servers from setting arbitrary cookies for another domain. If the domain attribute is not set, then the host name of the server that generated the cookie is used as the default value of the `domain`.

For example, if a cookie is set by an application at `app.mydomain.com` with no domain attribute set, then the cookie would be resubmitted for all subsequent requests for `app.mydomain.com` and its subdomains (such as `hacker.app.mydomain.com`), but not to `otherapp.mydomain.com`. If a developer wanted to loosen this restriction, then he could set the `domain` attribute to `mydomain.com`. In this case the cookie would be sent to all requests for `app.mydomain.com` and its subdomains, such as `hacker.app.mydomain.com`, and even `bank.mydomain.com`. If there was a vulnerable server on a subdomain (for example, `otherapp.mydomain.com`) and the `domain` attribute has been set too loosely (for example, `mydomain.com`), then the vulnerable server could be used to harvest cookies (such as session tokens).

- `Path` - In addition to the domain, the URL path that the cookie is valid for can be specified. If the domain and path match, then the cookie will be sent in the request. Just as with the domain attribute, if the path attribute is set too loosely, then it could leave the application vulnerable to attacks by other applications on the same server. For example, if the path attribute was set to the web server root “/”, then the application cookies will be sent to every application within the same domain.
- `Expires` - This attribute is used to set persistent cookies, since the cookie does not expire until the set date is exceeded. This persistent cookie will be used by this browser session and subsequent sessions until the cookie expires. Once the expiration date has exceeded, the browser will delete the cookie. Alternatively, if this attribute is not set, then the cookie is only valid in the current browser session and the cookie will be deleted when the session ends.
- `SameSite` - This attribute is used to assert that a cookie ought not to be sent along with cross-site requests. These features allows the server to mitigate the risk of cross-orgin information leakage. In some cases, is used too as a risk reduction strategy in front of cross-site request forgery attacks. This attribute can be configured in three different modes:
  - `Strict`: this value is the most restrictive usage of `SameSite`, allowing the browser to send the cookie only to first-party context without top-level navigation. In other words, the data associated with the cookie will only be sent on the requests matching the current site shown on the browser URL menu bar. The cookie will not be sent on requests generated by third-party websites. This value is especially recommended for actions performed at the same domain. However, it can have some limitations with some session management systems negatively affecting the user navigation experience. Since the browser would not send the cookie on any requests generated from a third-party domain or email, the user would be required to sign in again in even if they already have an authenticated session.
  - `Lax`: this less restrictive value enables the top-level navigation of the cookie. This means that the cookie will be sent by the browser if the domain in the URL equals the cookie’s domain (first-party) even if the link is coming from a third-party domain. This value is considered by most browsers the default behavior since it provides a better user experience than the `Strict` value.
  - `None`: using this value the browser will send the cookie on cross-site requests (the normal behavior before the implementation of `SamseSite`) only if the `Secure` attribute is also used, _e.g._ `SameSite=None; Secure`.

By design cookies do not have the capabilities to guarantee the integrity and confidentiality of the information stored in them. Those limitations make it impossible for a server to have confidence about how a given cookie's attributes were set at creation. In order to give the servers such features in a backwards-compatible way, the industry has introduced the concept of [`Cookie Name Prefixes`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-prefixes-00) to facilitate passing such details embedded as part of the cookie name.

- `__Secure-` - This cookie prefix is less restrictive and can be introduced by adding the case-sensitive string `__Secure-` to the cookie name. While using this prefix both the server and browser would know that the cookie is expected to be created with the `Secure` attribute such as: `Set-Cookie: __Secure-SID=12345; path=/; Secure`. Otherwise, the cookie would not be accepted due to the lack of `Secure` flag or delivery by a secure channel. Any cookie that matches the prefix `__Secure-` would be expected to fulfill the following conditions:

  1. The cookie must be set with the `Secure` attribute.
  2. The cookie must be set from an URI considered secure by the user agent.

- `__Host-` - This much more restrictive cookie prefix provides additional requirements added to the ones requested by the `__Secure-` prefix. Any cookie that matches the prefix `__Host-` would be expected to fulfill the following conditions:

  1. The cookie must be set with the `Secure` attribute.
  2. The cookie must be set from an URI considered secure by the user agent.
  3. Sent only to the host who set the cookie and MUST NOT include any `Domain` attribute.
  4. The cookie must be set with the attribute `Path` with a value of "/" so it would be sent to every request to the host.

For this reason, the cookie `Set-Cookie: __Host-SID=12345; Secure; Path=/` would be accepted while any of the following ones would always be rejected:
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.example`
`Set-Cookie: __Host-SID=12345; Domain=site.example; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.example; Path=/`

Putting all this together, we can define the perfect cookie attribute configuration as: `Set-Cookie: __Host-SID=12345; path=/; Secure; HttpOnly; SameSite`.

## How to Test

### Testing for Cookie Attribute Vulnerabilities

By using an intercepting proxy or traffic-intercepting browser plug-in, trap all responses where a cookie is set by the application (using the `Set-Cookie` directive) and inspect the cookie for the following:

- `Secure`: Whenever a cookie contains sensitive information or is a session token, then it should always be passed using an encrypted tunnel. For example, after logging into an application and a session token is set using a cookie, then verify it is tagged using the `Secure` flag. If it is not, then the browser would agree to pass it via an unencrypted channel such as using HTTP, and this could lead to an attacker leading users into submitting their cookie over an insecure channel.

`Set-Cookie: id=<session_id>; Secure;`

- `HttpOnly`: This attribute should always be set even though not every browser supports it. This attribute aids in securing the cookie from being accessed by a client side script. It does not eliminate cross site scripting risks but does eliminate some exploitation vectors. Check to see if the `HttpOnly` tag has been set.

`Set-Cookie: id=<session_id>; Secure; HttpOnly;`

- `Domain` - Verify that the domain has not been set too loosely. As noted above, it should only be set for the server that needs to receive the cookie. For example if the application resides on server `app.mysite.com`, then it should be set to `Domain=app.mysite.com` and NOT `Domain=.mysite.com` as this would allow other potentially vulnerable servers to receive the cookie.

`Set-Cookie: id=<session_id>; Domain=app.mysite.com;`

- `Path`: Verify that the `Path` attribute, just as the `Domain` attribute, has not been set too loosely. Even if the `Domain` attribute has been configured as tight as possible, if the path is set to the root directory (`/`), it can be vulnerable to less secure applications on the same server. For example, if the application resides at `/myapp/`, then verify that the cookies path is set to `Path=/myapp/` and NOT `Path=/`.

`Set-Cookie: id=<session_id>; Path=/myapp/;`

- `Expires`: If this attribute is set to a time in the future, verify that the cookie does not contain any sensitive information. For example, if a cookie is set to `Expires=Sun, 31-Jul-2016 13:45:29 GMT` and it is currently July 31st 2014, then the tester should inspect the cookie. If the cookie is a session token that is stored on the user's hard drive, then an attacker or local user (such as an admin) who has access to this cookie can access the application by resubmitting this token until the expiration date passes.

`Set-Cookie: id=<session_id>; Expires=Wed, 6 May 2020 07:28:00 GMT;`

- `SameSite`: Verify that the cookie has the `SameSite` attribute defined with a value that aligns with business logic. As a general recommendation, the cookies used to store data used inside the scope of the application that creates it should be set to `SameSite=Strict`. Cookies used for session management should be `SameSite=Lax`. Any cookie that needs to be sent cross-origin should be created with the attribute `SameSite=None`.

  - `Set-Cookie: id=<session_id>; SameSite=Strict;`
  - `Set-Cookie: id=<session_id>; SameSite=Lax;`
  - `Set-Cookie: id=<session_id>; SameSite=None; Secure;`

### Testing for Cookie Authentication Replay

Some vulnerable sites are only using cookie sessions as a single authentication token. On those cases, an attacker able to steal the valid authentication cookies would be able to impersonate the user session until the cookie expiration date. The most common attack vectors in order to steal the authentication Cookies from the users are malware, Javascript injection and Man-in-the-middle attacks.

Therefore, the following testing steps can help to identify if the website simply use cookie for authentication token without other checking on web site.

1. Use Chrome Extension Cookie Editor (i.e. Chrome EditThisCookie) to view existing cookie key-value pair.
2. Use Chrome Login the target testing website.
3. Use Chrome Extension Cookie Editor to view the cookie again. Identify those newly added or updated cookies. These can be potentialy vulnerable authentication cookies.
4. Use Firefox to visit the target testing website and manually add all previous new identified cookies by FireFox Cookie Editor (i.e. Firefox Extension Advanced Cookie Manager)
5. Check Firefox browser exisitng website login status to see if current web page can get authenticated and logged in without username and password. If that's true, it means that the website is vulnerable too `Cookie Authentication Replay`.

## Remediation

- Securely encrypt all the communications between the server and the browser to reduce cookie interception risk by Man-in-the-middle attacks.
- Implement multiple session identifiers apart from the cookie session ID. (For example: last IP address, the browser's User-Agent, etc.)
- Once creating the cookie, set up the shortest expiration time as possible.
- Invalidate the cookie after some time of inactivity or when the user logs out.

## Tools

### Intercepting Proxy

- [OWASP Zed Attack Proxy Project](https://www.zaproxy.org)
- [Web Proxy Burp Suite](https://portswigger.net)

### Browser Plug-in

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
- [“FireSheep” for FireFox](https://github.com/codebutler/firesheep)
- [“EditThisCookie” for Chrome](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)
- [“Cookiebro - Cookie Manager” for FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## References

### Whitepapers

- [RFC 2965 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Same-Site Cookies - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [The important “expires” attribute of Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [HttpOnly Session ID in URL and Page Body](https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)
