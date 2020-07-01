---
published: true
layout: post
title: Remove the Prefix from an Route in Istio
description: this is common where the Istio will have some part of the route to identify which service, and will want to remove the identifier.
date: 2020-07-01
tags: [k8s, rancher, istio]
comments: false
---

This is a increasingly common pattern

```
[client] -> {api.com/service/proxy}  -|
                                      |
    -----------------------------------
    |
    |-----> [Gateway] -> {/proxy} ----|
                                      |
    -----------------------------------
    |
    |-----> [service]
```

the client calls the ingress, where the `{service}` part of the request is used to identify the downstream service, however the service only needs the `/{proxy}`

## High-Level steps

- setup a virtual service with prefixes and a re-write

# virtual service setup

In this VirtualService, below, we setup a weather service, however we remove the `weather` from the url, effively allowing for the server to deal with the rest of the URI

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: weather
  name: weather
spec:
  hosts:
  - "*"
  gateways:
  - istio-system/awesome-io-gateway
  http:
  - match:
    - uri:
        prefix: "/weather/"
    - uri:
        prefix: "/weather"
    rewrite:
      uri: "/"
    route:
    - destination:
        port:
          number: 80
        host: weather-svc
```

note that the prefix is done twice, this is a work-around (https://github.com/istio/istio/issues/8076)


the example of this when it is run would be the following

**Client calls:**

```
curl "http://awesome.io:31380/weather/v1/weatherforecast"
```

**Istio calls the service with**
```
"http://weather-svc/v1/weatherforecast"
```

the `weather-svc` is effectively the kuberneties service entry which resides in the same namespace as the VirtualService, note that it does not have to match the prefix.

just for reference:

```
apiVersion: v1
kind: Service
metadata:
  namespace: weather
  name: weather-svc
  labels:
    app: weather-svc
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: weather-svc
```

