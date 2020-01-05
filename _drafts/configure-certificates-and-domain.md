---
layout: post
title: Configure Certificate & Domain for
date: 2018-03-05 00:00:00
lang: en
description: To learn how to use a 3rd party API in our AWS Lambda functions, we are going to create a billing API using Stripe.
ref: configure-certificates-and-domain
comments_id: xxx/168
---

### Create a certificate

We will use the serverless framework plugins as they go directly to the API instead of using cloud formation as this seems to be [complex](https://binx.io/blog/2018/10/05/automated-provisioning-of-acm-certificates-using-route53-in-cloudformation/) and not working very well for the [domains](https://www.npmjs.com/package/serverless-domain-manager).

- https://www.npmjs.com/package/serverless-domain-manager
- https://serverless.com/blog/serverless-api-gateway-domain/
