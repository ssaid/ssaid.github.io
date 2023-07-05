# What is XSS?

XSS (Cross-site scripting) is a type of web security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. This can happen when a website does not properly validate user input, such as search queries or form submissions, and fails to sanitize the input before displaying it back to the user.

When an attacker injects malicious scripts into a web page, those scripts can execute in the context of the victim's browser, allowing the attacker to steal sensitive data, such as cookies, login credentials, or other personal information. The attacker can also use the malicious scripts to modify the content of the web page or redirect the user to a different website, potentially leading to further attacks.

XSS attacks can be prevented by properly validating and sanitizing user input, and by implementing Content Security Policy (CSP) headers, which can limit the sources of content that can be loaded on a webpage and prevent the execution of malicious scripts.

## Mitigation: implement CSP Headers

Content Security Policy (CSP) is a security feature that can be implemented by web developers to prevent certain types of attacks, such as cross-site scripting (XSS) and data injection attacks, by specifying which sources of content are allowed to be loaded on a webpage.

A CSP header is a piece of HTTP response header that is sent from the server to the browser along with the web page. It specifies a set of directives that tell the browser which types of resources, such as scripts, stylesheets, images, and frames, are allowed to be loaded on the page, and from which sources.

By implementing a CSP header, web developers can help to protect their websites and their users from various types of attacks, including XSS and clickjacking, by limiting the sources of content that can be loaded on the webpage and preventing the execution of malicious scripts or other types of code. CSP headers can also provide useful feedback to developers by reporting violations in the browser console, making it easier to identify and fix security issues.

### CSPs Directives

#### Fetch directives

These directives specify the locations for loading certain resource types. Fetch directives include:

* child-src: defines the whitelisted script sources for browsing context loaded in frames and web workers.
* connect-src: specifies the URLs to be loaded using scripts
* default-src: the fallback directive for all fetch directives. This policy directive defines the default source list for other fetch directives.
* object-src: defines allowed sources for applet, embed and object elements
* style-src: provides a list of valid sources for cascading style sheets

#### Document directives

Document directives help control the properties of the working environment or document where a policy will be effective. These include:

* sandbox: provides a sandbox for a specified resource similar to inline script elements.
* base-uri: specifies the URLs allowed in the document’s base element

#### Navigation directives

These directives govern the locations of a form submission or where the document initiates any navigations. They include:

* form-action: specifies the URLs that act as targets for the submission of form elements
* frame-ancestors: restricts the parents to be embedded on the web page using the "frame, object, iframe, applet and embed" elements

#### Reporting directives

These directives govern how CSP violations are documented and reported. They include:

* report-to: initiates a security policy violation event
* report-uri: a policy directive instructing the client environment to report any attempt to violate CSP specifications.

#### Other directives...

* require-sri-for: enforces the use of Subresource Integrity (SRI) for the style attribute and page script sources
* trusted-types: used to specify a list of accepted, non-spoofable typed values, thus mitigating DOM-based XSS attacks
* require-trusted-types-for: this policy directive enforces the trusted-types policy on scripts that act as sinks for DOM-based XSS
* upgrade-insecure-requests: in websites with numerous insecure legacy URLs, this policy instructs the browser to treat every insecure URL as if it has been replaced with an HTTPS secure URL

### Example

Making a request to website that has enabled CSP Header with curl.

```bash
(Omitted..)
< Content-Security-Policy: default-src 'self' *.googleapis.com *.klarna.com masterpass.com *.masterpass.com mastercard.com *.mastercard.com *.npci.org.in firstpay.co.kr *.firstpay.co.kr pay.google.com *.fdconnect.com commerce-connect.com *.paypal.com 'unsafe-eval' 'unsafe-inline'; frame-ancestors 'self' img-src 'self' data:
(Omitted)
```

1. default-src 'self' *.googleapis.com *.klarna.com ...: Allow the page to  the fallback directive for all fetch directives. This policy directive defines the default source list for other fetch directives.
2. frame-ancestors 'self' img-src 'self' data: The 'frame-ancestors' directive in the CSP header is used to control which websites are allowed to embed the current website in a frame or iframe. The 'self' value for the 'frame-ancestors' directive indicates that only the current website is allowed to embed itself in a frame or iframe.

### Resources

* https://es.wikipedia.org/wiki/Cross-site_scripting
* https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
* https://crashtest-security.com/content-security-policy-header/
* https://youtu.be/GiQYpU4zoz8Jasper
