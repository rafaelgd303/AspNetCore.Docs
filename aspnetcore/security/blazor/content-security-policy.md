---
title: Enforce a Content Security Policy for ASP.NET Core Blazor
author: guardrex
description: Learn how to use a Content Security Policy (CSP) in ASP.NET Core Blazor apps to help protect against Cross-Site Scripting (XSS) attacks.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/28/2020
no-loc: [Blazor, SignalR]
uid: security/blazor/content-security-policy
---
# Enforce a Content Security Policy for ASP.NET Core Blazor

By [Javier Calvarro Nelson](https://github.com/javiercn) and [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[Cross-Site Scripting (XSS)](xref:security/cross-site-scripting) is a security vulnerability where an attacker places malicious client-side scripts into rendered content. A Content Security Policy (CSP) helps protect against XSS attacks by informing the browser of valid:

* Sources for loaded content, including scripts, stylesheets, and images.
* Actions taken by a page, including URL targets for form submissions.

To use CSP, the developer specifies several CSP *directives* to browsers in one or more `Content-Security-Policy` headers or `<meta>` tags.

Policies are evaluated while a document is loading. The browser inspects candidate sources and determines if the sources meet the requirements of the document's policies. When policy directives aren't met for a source, the browser blocks loading the resource. For example, consider a document with a policy that doesn't allow third-party scripts. When a document contains a script with a third-party origin in the `src` attribute of a `<script>` tag, the browser prevents the script from loading.

CSP is supported in most modern desktop and mobile browsers, including Chrome, Edge, Firefox, Opera, and Safari. CSP is recommended for Blazor apps to help protect apps from XSS and leaking circuit IDs in Blazor Server apps. [Subresource Integrity (SRI)](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) works in conjunction with CSPs.

## Policy directives

Minimally, specify the following directives and sources for Blazor apps. Add additional directives and sources as needed. The following directives are used in the [Apply the policy](#apply-the-policy) section of this article, where example security policies for Blazor WebAssembly and Blazor Server are provided:

* [base-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri) &ndash; Restricts the URLs for a document's `<base>` tag.
  * Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.
  * Supported by all browsers except IE.
* [block-all-mixed-content](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content) &ndash; Prevents loading mixed HTTP and HTTPS content. Supported by all browsers except IE.
* [default-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/default-src) &ndash; Indicates a fallback for CSP directives.
  * Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.
  * Supported by all browsers except IE.
* [img-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/img-src) &ndash; Indicates valid sources for images.
  * Specify `data:` to permit loading images from `data:` URLs.
  * Specify `https:` to permit loading images from HTTPS endpoints.
  * Supported by all browsers except IE.
* [object-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/object-src) &ndash; Indicates valid sources for the `<object>`, `<embed>`, and `<applet>` tags.
  * Specify `none` to prevent all URL sources.
  * Supported by all browsers except IE.
* [script-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) &ndash; Indicates valid sources for scripts.
  * Specify the `https://stackpath.bootstrapcdn.com/` host source for Bootstrap scripts.
  * Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.
  * In a Blazor WebAssembly app:
    * Specify the following hashes to permit inline scripts to load:
      * `sha256-v8ZC9OgMhcnEQ/Me77/R9TlJfzOBqrMTW8e1KuqLaqc=`
      * `sha256-If//FtbPc03afjLezvWHnC3Nbu4fDM04IIzkPaf3pH0=`
      * `sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=`
    * Specify `unsafe-eval` to use `eval()` and methods for creating code from strings.
  * In a Blazor Server app, specify the `sha256-34WLX60Tw3aG6hylk0plKbZZFXCuepeQ6Hu7OqRf8PI=` hash for the inlined script that performs fallback detection for stylesheets.
  * Supported by all browsers except IE.
* [style-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/style-src) &ndash; Indicates valid sources for stylesheets.
  * Specify the `https://stackpath.bootstrapcdn.com/` host source for Bootstrap stylesheets.
  * Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.
  * Specify `unsafe-inline` to allow the use of inline styles. The inline declaration is required for the UI in Blazor Server apps for reconnecting the client and server after the initial request. In a future release, inline styling might be removed so that `unsafe-inline` is no longer required.
  * Supported by all browsers except IE.
* [upgrade-insecure-requests](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests) &ndash; Indicates that content URLs from insecure (HTTP) sources should be acquired securely over HTTPS.

For a Content Security Policy Level 2 browser support matrix, see [Can I use: Content Security Policy Level 2](https://www.caniuse.com/#feat=contentsecuritypolicy2).

## Apply the policy

Use a `<meta>` tag to apply the policy:

* Set the value of the `http-equiv` attribute to `Content-Security-Policy`.
* Place the directives in the `content` attribute value. Separate directives with a semicolon (`;`).
* Always place the `meta` tag in the `<head>` content.

To test a policy over a period of a few days or weeks without enforcing the policy directives, set the `http-equiv` attribute (or header name) to `Content-Security-Policy-Report-Only`. Failure reports are sent as JSON documents to a specified URL. Testing helps confirm that third-party scripts aren't inadvertently blocked when building an initial policy. For more information, see [MDN web docs: Content-Security-Policy-Report-Only](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only).

For reporting on violations while a policy is active, see the following articles:

* [report-to](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [report-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)

Although `report-uri` is no longer recommended for use, both directives should be used until `report-to` is supported by all of the major browsers. Don't exclusively use `report-uri` because support for the directive in any given browser version could be dropped *at any time*. Remove support for `report-uri` in your apps when `report-to` is fully supported. To track adoption of `report-to`, see [Can I use: report-to](https://caniuse.com/#feat=mdn-http_headers_csp_content-security-policy_report-to).

For information on integrating [Subresource Integrity (SRI)](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) with CSPs, see [MDN web docs: require-sri-for](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/require-sri-for).

Test and update an app's policy every release. The following sections show example policies for Blazor WebAssembly and Blazor Server. These examples are versioned with this article for each release of Blazor. To use a version appropriate for your release, select the document version with the **Version** drop down selector on this webpage.

### Blazor WebAssembly

In the `<head>` content of the *wwwroot/index.html* host page, apply the directives described in the [Policy directives](#policy-directives) section:

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-v8ZC9OgMhcnEQ/Me77/R9TlJfzOBqrMTW8e1KuqLaqc=' 
                          'sha256-If//FtbPc03afjLezvWHnC3Nbu4fDM04IIzkPaf3pH0=' 
                          'sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=' 
                          'unsafe-eval';
               style-src https://stackpath.bootstrapcdn.com/
                         'self'
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

### Blazor Server

In the `<head>` content of the *Pages/_Host.cshtml* host page, apply the directives described in the [Policy directives](#policy-directives) section:

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-34WLX60Tw3aG6hylk0plKbZZFXCuepeQ6Hu7OqRf8PI=';
               style-src https://stackpath.bootstrapcdn.com/
                         'self' 
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

## Meta tag limitations

A `<meta>` tag policy doesn't support the following directives:

* [frame-ancestors](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
* [report-to](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [report-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)
* [sandbox](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox)

To support the preceding directives, use a header named `Content-Security-Policy`. The directive string is the header's value.

## Troubleshoot

* Errors appear in the browser's developer tools console. Chrome and Firefox provide information about:
  * Elements that don't comply with the policy.
  * How to modify the policy to allow for a blocked item.
* A policy is only completely effective when the client's browser supports all of the included directives. For a current browser support matrix, see [Can I use: Content-Security-Policy](https://caniuse.com/#search=Content-Security-Policy).

## Additional resources

* [MDN web docs: Content-Security-Policy](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy)
* [Content Security Policy Level 2](https://www.w3.org/TR/CSP2/)
* [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
