---
layout: post
title: Set Custom Domains
date: 2018-03-15 00:00:00
lang: en
description: We will use the Serverless framework and our ci/cd to configure our API Gateway endpoints in our Serverless project with custom domains.
ref: set-custom-domains
comments_id: set-custom-domains-through-seed/178
---

Our serverless API uses API Gateway and it gives us some auto-generated endpoints. We would like to configure them to use a scheme like `api.my-domain.com` or something similar.

In the first part of the tutorial we had added our domain to Route 53. If you haven't done so you can [read more about it here](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/MigratingDNS.html). To automate this we use the [serverless-domain-manager](https://github.com/amplify-education/serverless-domain-manager)

```bash
npm install serverless-domain-manager
```

https://www.npmjs.com/package/serverless-domain-manager#prerequisites

![Click Add Custom Domain button for prod endpoint](/assets/part2/click-add-custom-domain-button-for-prod-endpoint.png)

Seed will now go through and configure the domain for this API Gateway endpoint, create the SSL certificate and attach it to the domain. This process can take up to 40 mins.

Now that we've automated our deployments, let’s do a quick test to see what will happen if we make a mistake and push some faulty code to production.
