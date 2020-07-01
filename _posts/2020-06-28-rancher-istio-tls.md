---
published: true
layout: post
title: enabling Isio Secure Ingress on Rancher
description: simple extra config answer which is required
date: 2020-06-28
tags: [k8s, rancher, istio]
comments: false
---

Rancher has integrated Istio into its management dashboards. During the setup there are several options which you can customise, one is the Ingress controller

By default (at the time of this article) you need to provide an extra answer to the set to enable TLS on the Istio Ingress.

we will look at the settings, and how to confirm the setup.

## TLDR;

- To use `credentialName` to apply TLS on the Istio Ingress, you will need to enable SDS.
- Set this custom answer: `gateways.istio-ingressgateway.sds.enabled=true`

## Background

When you enable Istio in Rancher 2.4.x (2.4.5 in this example), it will show port 31390 will expose HTTPS.

In this [demo setup](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/), we setup a Kubernetes TLS secret and reference it in the `Gateway` deployment using the `credentialName`

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
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
      credentialName: httpbin-credential #The gateway does not know where this is.
    hosts:
    - httpbin.example.com
```

Once this is deployed and you try to call this endpoint, it will close the connection and you will not be able to access the port using `nc -z-v server-ip 31390`

This is due to a missing component called the Secrects Discovery Service (SDS). Which is not deployed by default for Ranchers Istio v1.4

The SDS, monitors the `istio-system` for any new secrets and then mounts these into the gateway for it to use.

# deploying the SDS

The `istio-ingressgateway` component is deployed with 1 container (`istio-proxy`) by default.

We need to provide another answer to the HELM chart

`gateways.istio-ingressgateway.sds.enabled=true`

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/istio-sds/options.PNG)


This will now setup the 'istio-ingressgateway' with 2 containers

- `istio-proxy` - which we already had
- `ingress-sds` - the **Secrets Discovery Service** is now deployed and listening for new certs.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/istio-sds/ingres-sds.PNG)

# confirming its working

Just run the [Istio example code](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/), and you should see this something like this:

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/istio-sds/teapot.PNG)

# Debugging

When we deploy the secret into the `istio-system`, we should be able to look at the `ingress-sds` container logs and see it pick up the TLS secret

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2020/istio-sds/logs-to-show-the-cert-being-applied.PNG)

# related links

- [Istio SDS for tls](https://github.com/rancher/rancher/issues/25315) - this would be cool and render this article obsolete.
- [Enabling SDS Gateway in Istio fails](https://github.com/rancher/rancher/issues/250515) - reference, this is where rancher first provided support for SDS (note the certmanager may need to be added to the answers).
- [Istio By Example - Secure Ingress](https://istiobyexample.dev/secure-ingress/) - excellent article on about the SDS with Istio.
