---
layout: post
title: Automating React Deployments
date: 2018-03-25 00:00:00
lang: en
description: We want to automatically deploy our Create React App when we push any changes to our Git repository. To do this, we will need to set our project up on Netlify.
ref: automating-react-deployments
comments_id: automating-react-deployments/188
---

Now that we have our backend deployed to production, we are ready to deploy our frontend to production as well! 

We deploy our React app to [S3](https://aws.amazon.com/s3) and we use [CloudFront](https://aws.amazon.com/cloudfront) as a CDN to host our assets. Then we used [Route 53](https://aws.amazon.com/route53) to configure our domain with it. We also had to configure the www version of our domain and this needed another S3 and CloudFront distribution and [Certificate Manager](https://aws.amazon.com/certificate-manager/) to handle our SSL certificate. 

We chose this over more integrated services like [Netlify](https://www.netlify.com) because regulated industries often require more customization than an integrated product provides.

First we setup the infrastructure through cloud formation:

TODO: Check if I should deploy each version into a seperated bucket or if I can use a folder within a bucket.

1. CloudFormation to create: the s3 buckets, `production`, `experiemental`, `dev`
   1. `dev` bucket includes the artifact build for dev (react env variable for dev).
   2. `experimental` bucket includes the artefact build for prod but can be another version that is not jet promoted to the main version
   3. `production` includes the code that runs in production.
2. Cloudformation to creat: CloudFront distributions for `production`, `experimental` and `dev`. 
   1. `dev` maps directly to the s3 bucket and exposes the `${version}-dev.d10l.de`
   2. `production` and `experimental` map to the s3 buckets `production` / `experiemental` under `d10.de`
3. Cloudformation to create: lambda@edge for the a/b testing between `experimental` and `production`: Simple agorithm 10% of the traffic go to the `experiement` the results are tracked through a tracking library that includes the `frontend/app-version` this should correlated to a git tag.  
4. each pull request is build, linted, unit-tested, deployed to `${version}-dev` and integration tested with cypress -> dev goes against the dev backend.
5. prod pipeline: After a pull request is merge
   1. the dev pull request env is deleted (cloudfront distribute mainly)
   2. the prod deployment is triggered by putting the resources into the experimental folder and activating the analytics and the edge function, query the kpi from the tracking and the errors in gerneral in the backend for 15 minutes - promote the version to prod and disable the edge function.


Our workflow looks like this: 

1. when ever we push to `master` a new version is build and stored in s3 as a zip file.
2. then a it is deployed to an A/B Test / Canary release by copying the the files to an s3 bucket `experiemental` and update the cloudfront distribution.  

Now before we can automate our deployments, we'll need to configure environments in our React app. This'll allow the production version of our React app to connect to our production backend.


Create github workflow in your `serverless-stack-client` repository by:

```bash
cd serverless-stack-client
mkdir -p .github/workflows
code .github/workflows/new-pull-request.yaml
```

Start with the simples solution to reach the goal: running code delivered to website. Otherwise the design gets to complex in the head. This means no branches / no A/B Testing: just input a push to the master branch and an output a running website (s3, cloudfront, route53 and ssl). For the details check the manual deployment 

Create required infrastructure: 
- `ssk-notes-ui-prod` s3 bucket
- route53 config - add domain
- cloudfront distribution: orgin policy, 
