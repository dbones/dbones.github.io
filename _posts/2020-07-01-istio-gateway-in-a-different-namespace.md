---
published: true
layout: post
title: Istio Gateway in a different namspace to the VirtualService
description: its just for a note.
date: 2020-07-01
tags: [k8s, rancher, istio]
comments: false
---

we want to setup the Gateway in a central namespace, and place the virtual service close the the service it is setting up routes too.

OK this is a really simple post, but it was not obvously documented (it does not follow the DNS resolution like services follow)

## High-Level steps

- put gateway in an agreed namespace
- create VirtualService which refernces the gateway using a `namespace/gateway-name` syntax

# setup the gateway in a namespace

in this sample we will place the gateway into the `istio-system` namespace, it does not have to go there, but its an example:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: awesome-io-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: awesome-io # must be the same as secret
    hosts:
    - "awesome.io"
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "awesome.io"
```

# setup the VirtualService

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
        host: weather
```

note 

```
  gateways:
  - istio-system/awesome-io-gateway
```

this is where the magic happens, the pattern is `namespace/gateway-name`

