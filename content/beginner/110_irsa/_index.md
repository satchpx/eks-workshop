---
title: "IAM Roles for Service Accounts"
date: 2021-07-20T00:00:00-03:00
weight: 110
pre: '<i class="fa fa-film" aria-hidden="true"></i> '
draft: false
tags:
  - intermediate
  - CON205
_build:
  list: false
  render: false
---

### Fine-Grained IAM Roles for Service Accounts

{{< youtube lyMKskPXbEA >}}

In Kubernetes version 1.12, support was added for a new **ProjectedServiceAccountToken** feature, which is an OIDC JSON web token that also contains the service account identity, and supports a configurable audience.

Amazon EKS now hosts a public OIDC discovery endpoint per cluster containing the signing keys for the ProjectedServiceAccountToken JSON web tokens so external systems, like IAM, can validate and accept the Kubernetes-issued OIDC tokens.

OIDC federation access allows you to assume IAM roles via the Secure Token Service (STS), enabling authentication with an OIDC provider, receiving a JSON Web Token (JWT), which in turn can be used to assume an IAM role. Kubernetes, on the other hand, can issue so-called projected service account tokens, which happen to be valid OIDC JWTs for pods. Our setup equips each pod with a cryptographically-signed token that can be verified by STS against the OIDC provider of your choice to establish the pod’s identity.

new credential provider ”<span style="color:orange">**sts:AssumeRoleWithWebIdentity**</span>”
