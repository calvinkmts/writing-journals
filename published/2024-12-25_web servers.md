# Web Server vs Web Server: Why We Use Both Nginx and Kestrel

## Background

Not long ago, I found myself observing an after-hours debugging session at the office. A colleague was troubleshooting a service that repeatedly failed to start, and I decided to stick around to see how others approach problem-solving.

The issue revolved around a service called `kestrel-cpp.service` that kept shutting down. Using `systemctl`, they inspected the service status but couldn't pinpoint the root cause. I suggested using `journalctl` to check the logs, which revealed that the configuration for the service was either incomplete or overwritten, causing the failure.

Later, I discovered that `kestrel-cpp` was part of an ASP.NET core application. My reaction was, "Oh, they're using Kestrel! I never knew about it." Typically, I associate .NET applications with being hosted on IIS, so this caught my attention.

That's when my colleague asked an intriguing question:

> "Why do we need another web server like Kestrel if we already use Nginx or Apache?"

I gave a quick answer: "Nginx or Apache is usually used as a reverse proxy or load balancer, while Kestrel or a web app that is compiled with go is used to actually serves the application."

But I realized later that my response didn't fully capture the nuance, nor did it provide a satisfactory explanation.

## The Question

This question stayed with me. Why do we need multiple web servers? Why can't one do everything? This article is my attempt to explore and clarify the roles of different web servers and why they are often used together.

## The Lifecycle of a HTTP(s) Request

To see how the Kestrel and Nginx fit into the bigger picture, let's revisit the lifecycle of an HTTP(s) request—one of the most common scenarios where these web servers come into play.

Here's simplified flow from a browser's perpective:

1. _Input URL:_ The user enters a URL (optionally with a port, e.g., `:8080`).
2. _Resolve IP Address:_ The browser then uses DNS to convert the hostname into an IP Address.
3. _Establish Connection:_
   - HTTP: A TCP handshake occurs. involving three steps (SYN, SYN-ACK, ACK) to establish a connection.
   - HTTPS: A TLS handshake occurs (built on TCP), ensuring secure connection.
4. _Send Request:_ Once the connection is established, the browser sends an HTTP request.
5. _Receive Response:_ The server then processes the request and sends back a response, such as HTML, JSON, or an image file.

For a deeper dive into the TLS handshake, check out this resource: [What happens in a TLS handshake? | SSL handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

## So What is Nginx, Why Is It Different From Web Servers We Develop Using Languages Like Go or Frameworks like .NET (Kestrel)?

Nginx is a web server with a wide range of capabilities. According to the [official Nginx official site](https://nginx.org/en/), its HTTP features include (but are not limited to):

- Accelerated _reverse proxying_ with caching; load balancing and fault tolerance.
- Modular architecture. Filters include gzipping, byte ranges, _chunked responses_, XSLT, SSI, and image transformation filter. Multiple SSI inclusions within a single page can be processed in parallel if they are handled by proxied or FastCGI/uwsgi/SCGI servers.'
- _SSL and TLS SNI support_.
- Support for _HTTP/2_ with weighted and dependency-based prioritization and _HTTP/3_.
- _Name-based and IP-based virtual servers_.
- _Access log formats_, buffered log writing, fast log rotation, and syslog logging.
- _3xx-5xx error codes redirection_;
- The rewrite module: URI changing using regular expressions;
- Validation of HTTP referer;
- Response rate limiting;
- Limiting the number of simultaneous connections or requests coming from one address;
- And more.

In brief, Nginx already packed up with a lot of features,

### Key Features Relevant to this Article

Let's focus on three key features of Nginx that are relevant this writing.

#### 1. Reverse Proxying

A reverse proxy is a server that forwards client requests (e.g., from a web browser) to backend servers where resources are hosted. According to [Cloudflare](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy):

> A reverse proxy is a server that sits in front of web servers and forwards client (e.g. web browser) requests to those web servers.

Meaning, Nginx can acts as a gateway, redirecting clients to the appropriate backend web servers (resources). this is especially useful in distributed systems or microservices architecture.

#### 2. Load Balancing

When multiple instances of a service are running, Nginx can distribute incoming requests among them. This ensures no single instance get overloaded. Nginx support various load-balancing methods, such as:

- Round Robin: Requests are distributed evenly across the servers
- Least Connections: A request is sent to the server with the least number of active connections
- IP Hash: The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.
- And more.

#### 3. SSL and TLS SNI Support

> SNI is an extension for the TLS protocol (formerly known as the SSL protocol), which is used in HTTPS. It's included in the TLS/SSL handshake process in order to ensure that client devices are able to see the correct SSL certificate for the website they are trying to reach. The extension makes it possible to specify the hostname, or domain name, of the website during the TLS handshake, instead of when the HTTP connection opens after the handshake.

Read more: [What is SNI? How TLS server name indication works](https://www.cloudflare.com/learning/ssl/what-is-sni/)

### The Role of These Features in the HTTP(s) Lifecycle

Nginx's features, like reverse proxying, load balancing, and SSL/TLS handling, work behind the scenes when a HTTP(s) lifecycle happens. These tasks are often invisible to developers when looking at application-specific web servers, but they are crucial for building scalable and secure systems.

## Now, Kestrel Is Itself a Web Server Too—Do We Still Need Nginx?

Let's take a look at kestrel and what features it offers:

> Kestrel is a cross-platform web server for ASP.NET Core. Kestrel is the recommended server for ASP.NET Core, and it's configured by default in ASP.NET Core project templates.

### Key Features of Kestrel

Kestrel is designed with specific goals in mind, particularly for hosting .NET applications. It's key features include:

- Cross-platform: Kestrel is a cross-platform web server that runs on Windows, Linux, and macOS.
- High performance: Kestrel is optimized to handle a large number of concurrent connections efficiently.
- Lightweight: Optimized for running in resource-constrained environments, such as containers and edge devices.
- Security hardened: Kestrel supports HTTPS and is hardened against web server vulnerabilities.
- Wide protocol support: Kestrel supports common web protocols, (including HTTP/1.1, HTTP/2 and HTTP/3 and WebSockets)
- Integration with ASP.NET Core
- Flexible workloads: Kestrel supports many workloads.
- Extensibility: Customize Kestrel through configuration, middleware, and custom transports.
- Performance diagnostics: Kestrel provides built-in performance diagnostics features, such as logging and metrics.

### The Overlap Between Nginx and Kestrel

We see that both Kestrel and Nginx share some capabilities, such as:

- Supporting multiple HTTP versions (e.g., HTTP/1.1, HTTP/2 and HTTP/3).
- Providing HTTPS for secure connection.

However, their use cases differ significantly:

- _Kestrel:_ Primarily an _application server_, optimized for running ASP.NET core application and _processing business logic_.
- _Nginx:_ Commonly used as a _reverse proxy and load balancer_, have built-in features to manage traffics, and distributing requests to multiple resources.

## Now, Do We Really Need Both?

The short answer is: it depends on your architecture and requirements.

- _Does your architecture require a load balancer?_

  If yes, running an Nginx alongside your application web server (like Kestrel) make sense. Nginx provides built-in features for managing traffic, distributing requests, and handling other network related tasks, saving you from having to implement these features yourself.

- _Do you need a reverse proxy?_

  Kestrel is designed to serve application, and while it can handle secure connections, managing HTTPS for _every applicatioins_ individually can become challenging. Offloading tasks like TLS termination to Nginx centralizes the process, ensuring consistent security without requiring each application to handle it independently.

In fact, the [official documentation](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-9.0) recommends hosting Kestrel behind a proxy like Nginx or Apache. This combination leverages each strengths of both technologies, avoiding the need to reinvent the wheel.

## Conclusion

Do we really need multiple web servers?

The answer depends on your architecture and requirements. Nginx and Kestrel serves different purposes:

- _Nginx_ acts as a reverse proxy and load balancer, handling tasks like traffic management, TLS termination and request distribution.
- _Kestrel_ (or other web application servers like `net/http` in Go) can focus to serve the application logic.

By combining the two, you can offload network-level task—such as redirecting HTTP traffic from port 80 to HTTPS—to Nginx, simplifying the applicatoion's responsibilities.

Do you always need multiple web servers when hosting your service?

- If you're running a simple, single-instance setup that doesn't require load balancing or reverse proxying, a standalone web server like Kestrel may suffice.
- However, if you have a multi-service architecture, combining Nginx with Kestrel or a similiar application server is a better practice. It centralizes networking tasks and ensure your application to just handle application logic.

## References and Further Reading

### **Kestrel and ASP.NET Core**

- [Kestrel Web Server Overview (Microsoft Docs)](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-9.0&viewFallbackFrom=aspnetcore-2.2)
  An in-depth look at the Kestrel web server and its features.
- [When to Use a Reverse Proxy with Kestrel (Microsoft Docs)](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/when-to-use-a-reverse-proxy?view=aspnetcore-9.0)
  Guidance on scenarios where hosting Kestrel behind a reverse proxy is recommended.
- [Hosting and Deploying ASP.NET Core Apps Behind a Proxy (Microsoft Docs)](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-9.0)
  Official recommendations for deploying Kestrel with reverse proxies like Nginx or Apache.

### **Nginx and Reverse Proxying**

- [Nginx Official Site](https://nginx.org/en/)
  The official source for Nginx documentation and feature lists.
- [Benefits of Using Nginx in Front of a Web Server for Go (StackOverflow)](https://stackoverflow.com/questions/17776584/what-are-the-benefits-of-using-nginx-in-front-of-a-webserver-for-go)
  A community-driven discussion on the advantages of combining Nginx with application servers.
- [What is a Reverse Proxy? (Cloudflare)](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)
  A beginner-friendly explanation of reverse proxies and their benefits.

### **HTTP Request Lifecycle**

- [Lifecycle of an HTTP Request (Requestly)](https://requestly.com/blog/life-cycle-of-a-http-request/)
  A simple breakdown of how an HTTP request flows from client to server and back.
- [The HTTP Request Lifecycle (Dev.to)](https://dev.to/dangolant/things-i-brushed-up-on-this-week-the-http-request-lifecycle-)
  A practical exploration of the steps involved in an HTTP request lifecycle.

### **TLS and HTTPS**

- [What Happens in a TLS Handshake? (Cloudflare)](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)
  A detailed yet accessible explanation of the TLS handshake process.
- [The First Few Milliseconds of HTTPS (Moserware)](https://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
  A deep dive into the technical steps that occur during the establishment of an HTTPS connection.
