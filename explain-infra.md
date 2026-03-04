---
name: explain-infra
description: Explains infrastructure setup with visual diagrams. Use when explaining how the server, nginx, or deployment works, or when the user asks "how does this work?" about infrastructure.
---

When explaining infrastructure, always include:

1. **Draw a diagram**: Use ASCII art to show the request flow, nginx proxy routing, and how traffic reaches each app/site hosted on the DigitalOcean VM. For example:

```
Internet
    │
    ▼
[ DigitalOcean VM ]
    │
[ nginx (reverse proxy) ]
    ├──> app-one.com  →  :3000
    ├──> app-two.com  →  :4000
    └──> site.com     →  /var/www/site
```

2. **Walk through the request**: Explain step-by-step what happens from the moment a request hits the server to when the response is returned — DNS resolution, nginx matching the server block, proxying to the destination, and the response path back.

3. **Highlight a gotcha**: What's a common mistake or misconception about this setup? (e.g. forgetting to reload nginx after config changes, SSL termination at nginx vs the app, sticky sessions, etc.)

Keep explanations practical and grounded in the actual stack. Tailor diagrams to the specific app or route being discussed when possible.
