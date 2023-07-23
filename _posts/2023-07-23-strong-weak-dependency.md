---
layout: post
title:  "Strong/Weak dependencies & Graceful degradation"
date:   2023-07-23 18:30:00 +0800
categories: distributed-systems
---

# Strong/Weak dependencies & Graceful degradation

In my new role, I came across a useful concept of **Strong and weak dependencies** and its application.

A service (e.g. A) is strongly dependent on another service (e.g. C) if it **break** whenever its dependency fails.

```mermaid
graph TD;
    A-->|(strongly) depends on|C;
    B-.->|(weakly) depends on|C;
```

In the graph above, Service C is a dependency of Service A and Service B. C is a weak dependency of B, while C is a strong dependency of A. If C were to fail, A will break. However, B doesn't break, albeit with degraded functionality. 

To be more specific and nuanced, this dependency relationship can be viewed from a service level or to a function level. A function of a service is dependent on another function of another service. So there can exist multiple strong and weak dependency relationships between two services (when we have different functions calling different functions).

This concept of strong/weak dependency helps to create a mental model for understanding and designing services that is capable of **Graceful degradation**. 

i.e. "When building a new feature that calls the API of another service, should this be a weak or a strong dependency? If it's going to be a weak dependency. What will the degraded functionality be like and how do I achieve that (e.g. using timeouts or default/zero data)?"

## Automatic detection of strong/weak dependency

Another cool thing! When service-to-service calls are proxied by service mesh, the service mesh can determine if a service-to-service call is a strong or a weak dependency. This can be done by randomly injecting faults to the downstream service call (e.g. C) and seeing how the originating call to the upstream service (e.g. A or B) respond; whether it breaks or a degraded response.