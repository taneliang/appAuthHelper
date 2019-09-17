# Transparently Using Origin Isolation to Protect Access Tokens

OAuth 2 Resource Server URLs are a specific type of network service that follow a standard means of authorization. In particular, they are designed to require an "access token" as the credential included in the requests that are sent to them. An access token is a sensitive value that represents an assertion from a trusted authority (called an authorization server). The client making the request to a resource server URL is assumed to be legitimate based purely on the validity of the access token that it includes. For this reason, access token-bearing clients must take every possible measure available to them to ensure that their access token is not exposed to an illegitimate audience.

Browser-based applications that are designed to make direct requests to a resource server URL must necessarily have the access token value available within the browser in order for those requests to be authorized. However, browser-based applications are subject to exploitation in ways that other types of access token-bearing clients are not. The primary risk that is specific to browser-based applications is called "Cross-Site Scripting Attacks".

A cross-site scripting attack occurs when malicious software is injected into a browser-based application, usually due to a mistake made by the application's developers. Once injected, the malicious software has control over the user's browser equivalent to the application that it was injected within. For example, if the exploited application has the option to directly read an access token value, then the malicious software injected from a cross-site scripting attack will also be able to read it (potentially transmitting it to the author of the malicious software). This would allow the author of the malicious software to make requests to a resource server URL, performing some form of illegitimate action.

To mitigate the risk of cross-site scripting attacks, browser-based application developers should design their application in such a way so as to not require the ability to directly read an access token value. By designing the application this way, malicious software injected into their application will also not have the ability to directly read the access token. Applications designed this way must only indirectly rely on the presence of access tokens in the browser. However, for this design to be adopted and practical it must be one done in such a way that does not place an extreme burden on the application developer in order to be implemented properly. Otherwise, developers are unlikely to use it.

Web application servers are identified by the unique combination of protocol (e.g. HTTPS), host name and port used to serve the application; this is called the server "origin". A fundamental aspect of browser application security is to ensure that values saved by applications hosted within one origin are only readable by applications hosted within that same origin. This is called "Origin Isolation". This is the fundamental feature that can be used to enable indirect usage of access token values.

Access tokens values can be saved and read by an application that is served from a different origin than the one used to host the main browser application. This separate application should be designed exclusively for the purpose of token-management. As a result of this exclusive purpose, it would not be susceptible to cross-site script attacks in the way that the wider-purpose main application would be.

Since the access token is stored by an application in a separate origin, the main browser application cannot read the access token value (and likewise, neither can any malicious software that might have been injected into the main application). However, the main application still needs to make requests to resource servers that include the access token. It can do this by making use of another feature that browsers support - the "postMessage" function.

If the main application embeds a hidden frame within itself, this hidden frame can load the alternative origin application. One frame can pass a message to another frame using the postMessage function, even if the frames are not hosted by the same origin. This is the fundamental feature that allows the main application to make access token-bearing requests to resource servers without having the ability to directly read the access token; instead, requests are initiated by sending a message using postMessage to the alternative-origin application frame. The alternative-origin application makes the token-bearing request to the resource server; the response is then passed back to the main application as another message.

For this technique to be practical to implement, there should be a mechanism for transparently intercepting network requests to resource servers made by the main application. These intercepted requests would then be forwarded to the alternative origin frame, and the responses from the frame would then be transparently presented back to the application. By having this interception mechanism, application developers could continue using the normal network request techniques that they normally use; there would be no need to alter the application substantially in order to directly make postMessage requests. This mechanism for transparent request interception and response can be thought of as an "Identity Proxy".

An Identity Proxy can be built using different techniques. For more modern browsers, there is a feature called a "service worker" which is particularly well-suited for this purpose. A service workers is a separate JavaScript thread which exists in the browser apart from the primary application thread. The browser will pass any network request issued by the primary application to the service worker, giving the service worker an opportunity to alter the request or the response that the primary application receives. By programming the service worker to use postMessage to the alternate-origin frame, it can provide the sort of secure, transparent access token usage that is the goal of an Identity Proxy.

For browsers which do not support service workers, there is another way to build an Identity Proxy. The mechanism used by older browsers to make network requests is called the "XMLHttpRequest" object. The behavior of this object can be customized by using JavaScript to override its default methods. Customizing it so that network requests to resource servers result in postMessage calls to the alternate-origin frame achieves the same result as the service worker method.

Since the acquisition and usage of access tokens are well-defined parts of the OAuth 2 standard, this is a design pattern that can be packaged into a reusable library and shared amongst any browser-based OAuth 2 client application. Doing so allows the application developer to quickly and easily meet their obligation to keep access token values secure.