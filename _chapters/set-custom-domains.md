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

In the first part of the tutorial we had added our domain to Route 53. If you haven't done so you can [read more about it here](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/MigratingDNS.html). To automate this we use the [serverless-domain-manager](https://github.com/amplify-education/serverless-domain-manager) / [guide](https://serverless.com/blog/serverless-api-gateway-domain/). For a production deployment make sure your role has [limited rights](https://www.npmjs.com/package/serverless-domain-manager#prerequisites).

> We will use the serverless framework plugins as they go directly to the API instead of using cloud formation as this seems to be [complex](https://binx.io/blog/2018/10/05/automated-provisioning-of-acm-certificates-using-route53-in-cloudformation/) and not working very well for the [domains](https://www.npmjs.com/package/serverless-domain-manager).

First create the ssl certificate through [serverless-certificate-creator](https://github.com/schwamster/serverless-certificate-creator#combine-with-serverless-domain-manager) and then add the domain.

```bash
npm install --save-dev serverless-certificate-creator serverless-domain-manager
```

Update now your `serverless.yaml` it might make sense if the certificate and domains are used in multiple projects to create a seperate stack:

```yaml
plugins:
    - serverless-certificate-creator
    - serverless-domain-manager

    ...

    custom:
        customDomain:
            domainName: api.d10l.de
            certificateName: 'api.d10l.de'
            basePath: ''
            stage: ${self:provider.stage}
            createRoute53Record: true
        customCertificate:
            certificateName: 'api.d10l.de' //required
            idempotencyToken: 'apid10lde' //optional
            hostedZoneIds: 'Z3A1XXXXeXXFVX' //required if hostedZoneNames is not set
            region: us-east-1 // optional - default is us-east-1 which is required for custom api gateway domains of Type Edge (default)
            enabled: true // optional - default is true. For some stages you may not want to use certificates (and custom domains associated with it).
            rewriteRecords: false
```

Now you can run:

```bash
serverless create-cert
serverless create_domain
```

To add more apis to the custome domain you need to add the plugin and the `customDomain` entry into the `serverless.yaml` of the service. Then simply deploy your service without running the create_domain command again. If you have multiple stages you might create differrent custome domains e.g. api.d10l.de for prod and api-dev.d10l.de for dev. for this update the domain name to `api-${self:provider.stage}.d10l.de`, these domains need to be created once for each stage. As a certificate it makes sense to only create on certificate with a wildcard entry like `*.d10.de` or on certificate that has the other stages as an alias.

Serverless Framework will now go through and configure the domain for this API Gateway endpoint, create the SSL certificate and attach it to the domain. This process can take up to 40 mins.

Now that we've automated our deployments, let’s do a quick test to see what will happen if we make a mistake and push some faulty code to production.
