---
layout: post
title: "Web Security Academy: Cross Site Scripting (XSS)"
date:   2022-4-1
categories: XSS
---
###### **All solved labs can be found here at `Web Security Academy`**

[https://portswigger.net/web-security/all-labs](https://portswigger.net/web-security/all-labs)
____________
## 0. Index
### 1. Reflected XSS into HTML context with nothing encoded
### 2. Stored XSS into HTML context with nothing encoded
### 3. DOM XSS in document.write sink using source location.search
### 4. DOM XSS in innerHTML sink using source location.search
### 5. DOM XSS in jQuery anchor href attribute sink using location.search source
### 6. DOM XSS in jQuery selector sink using a hashchange event
### 7. Reflected XSS into attribute with angle brackets HTML-encoded
### 8. Stored XSS into anchor href attribute with double quotes HTML-encoded
### 9. Reflected XSS into a JavaScript string with angle brackets HTML encoded
### 10.DOM XSS in document.write sink using source location.search inside a select element
### 11.DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
### 12.Reflected DOM XSS
### 13.Stored DOM XSS
### 14.Exploiting cross-site scripting to steal cookies
### 15.Exploiting cross-site scripting to capture passwords
### 16.Exploiting XSS to perform CSRF
### 17.Reflected XSS into HTML context with most tags and attributes blocked
### 18.Reflected XSS into HTML context with all tags blocked except custom ones
### 19.Reflected XSS with some SVG markup allowed
### 20.Reflected XSS in canonical link tag
### 21.Reflected XSS into a JavaScript string with single quote and backslash escaped
### 22.Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
### 23.Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped
### 24.Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
### 25.Reflected XSS with event handlers and href attributes blocked
### 26.Reflected XSS in a JavaScript URL with some characters blocked
### 27.Reflected XSS with AngularJS sandbox escape without strings
### 28.Reflected XSS with AngularJS sandbox escape and CSP
### 29.Reflected XSS protected by very strict CSP, with dangling markup attack
### 30.Reflected XSS protected by CSP, with CSP bypass
____________
## 1. Reflected XSS into HTML context with nothing encoded
