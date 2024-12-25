# Web Server vs Web Server

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
