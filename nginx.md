[[Web server]] or Gateway between an internet and a backend infrastructure, when requesting from the server, nginx knows where to look for the resources, it also used for [[reverse proxy]], handles caching and authentication processes

It could be configured as *web server*, [[reverse proxy]], load balancer, cache, firewall, etc.

NGINX configuration consists of modules that are controlled by directives specified in the configuration file.
Directives could be simple and block directives.
- *Simple* consists of name and parameters separated by space and end with semicolon
```nginx
user nobody;
```
- *Block* directive ends with additional instructions surrounded by braces
if bloc directive consists of other directives inside braces, it is called *context*
```nginx
http {
	# context
}
```


# basic configuration structure
![[Pasted image 20240531182352.png]]

- Main - global configurations like worker processes, error logs and other that affect entire NGINX instance 
	- Events - Context for configuring how NGINX handles connections and connection processing
	- HTTP - Context for configuring HTTP server and processing HTTP requests. It contains *directives* related to HTTP traffic handling, such as server *blocks*, location, proxy settings, and more
		- Server - Block within HTTP context, that *defines virtual server or web server instance*. Multiple server blocks can be *defined in context to handle different domain names or IP addresses*
			- Location - Inside server block, this block specify how NGINX should process requests *for specific URI* locations or paths
		- Upstream - Defines group of servers that can be used as *load balancers*
	- Stream - Context used to configure TCP/UDP ([[internet-protocol]]) [[proxy]] and *load balancing for non-HTTP protocols such as TCP or UDP servers*

For example if we want to server ([[Web server]]) vue js, we can use [npm http-server](https://www.npmjs.com/package/http-server) with zero configurations, or we can utilize nginx
Similarly we can do so for python backend, lets take [[fastAPI]] for example, it can run with build-in [[ASGI]] during development, but in production it's recommended using NGINX