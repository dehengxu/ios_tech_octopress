---
layout: post
title: "App Transport Security 技术"
date: 2015-11-05 10:18:25 +0800
comments: true
published: true
sharing: true
footer: true
categories: iOS
---

App Transport Security is a feature that improves the security of connections between an app and web services. The feature consists of default connection requirements that conform to best practices for secure connections. Apps can override this default behavior and turn off transport security.

Transport security is available in iOS 9.0 or later, and in OS X v10.11 and later.

## Default behavior

All connections using the NSURLConnection, CFURL, or NSURLSession APIs use App Transport Security default behavior in apps built for iOS 9.0 or later, and OS X v10.11 or later. Connections that do not follow the requirements will fail. For more information on various the connection methods, see NSURLConnection Class Reference, CFURL Reference, or NSURLSession Class Reference.


Info.plist can use `NSAppTransportSecurity` key to configurate ATS

*  NSAllowsArbitraryLoads   `Yes` to disable ATS checking, `No` to enable ATS checking.


The property keys are defined next.

NSAppTransportSecurity
A dictionary containing the settings for overriding default App Transport Security behaviors. The top level key for the app’s Info.plist file.

NSAllowsArbitraryLoads
A Boolean value used to disable App Transport Security for any domains not listed in the NSExceptionDomains dictionary. Listed domains use the settings specified for that domain.

The default value of NO requires the default App Transport Security behavior for all connections.

NSExceptionDomains
A dictionary of App Transport Security exceptions for specific domains. Each key is a string containing the domain name for the exceptions.

<domain-name-for-exception-as-string>
A dictionary of exceptions for the named domain. The name of the key is the name of the domain–for example, www.apple.com.

#### NSExceptionMinimumTLSVersion

A string that specifies a the minimum TLS version for connections. Valid values are:

    TLSv1.0
    TLSv1.1
    TLSv1.2

`TLSV1.2` is the default value.

#### NSExceptionRequiresForwardSecrecy

A Boolean value for overriding the requirement that the domain support forward secrecy using ciphers.

YES is the default value and limits the ciphers to those shown in Default Behavior.

Setting the value to NO adds the following the list of accepted ciphers:

* TLS_RSA_WITH_AES_256_GCM_SHA384
* TLS_RSA_WITH_AES_128_GCM_SHA256
* TLS_RSA_WITH_AES_256_CBC_SHA256
* TLS_RSA_WITH_AES_256_CBC_SHA
* TLS_RSA_WITH_AES_128_CBC_SHA256
* TLS_RSA_WITH_AES_128_CBC_SHA

#### NSExceptionAllowsInsecureHTTPLoads

A Boolean value for overriding the requirement that all connections use HTTPS. Use this key to access domains with no certificate, or with an error for a self-signed, expired, or hostname-mismatch certificate.

NO is the default value.

#### NSIncludesSubdomains

A Boolean value for applying the overrides to all subdomains of the top-level domain.

NO is the default value.

#### NSThirdPartyExceptionMinimumTLSVersion

A version of NSExceptionMinimumTLSVersion used when the domain is an app service that is not controlled by the developer.

#### NSThirdPartyExceptionRequiresForwardSecrecy

A version of NSExceptionRequiresForwardSecrecy used when the domain is an app service that is not controlled by the developer.

#### NSThirdPartyExceptionAllowsInsecureHTTPLoads

A version of NSExceptionAllowsInsecureHTTPLoads used when the domain is an app service that is not controlled by the developer.


[Infomation Property List Key Reference](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009248-SW1)

