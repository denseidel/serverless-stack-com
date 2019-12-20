---
layout: post
title: Deploying Through Github
lang: en
date: 2018-03-14 00:00:00
description: We are going to trigger a deployment in Github by pushing a commit to our Serverless project in Git. In the AWS console you can view the build logs and look at the CloudFormation output.
ref: deploying-through-github
comments_id: deploying-through-seed/177
---

Now, we are ready to make our first deployment. You can Git push a new change to master to trigger it.

<img class="code-marker" src="/assets/s.png" />Go back to your project root and run the following.

```bash
$ npm version patch
```

This is simply updating the NPM version for your project. It is a good way to keep track of the changes you are making to your project. And it also creates a quick Git commit for us.

<img class="code-marker" src="/assets/s.png" />Push the change using.

```bash
$ git push
```

Remember this pushes the code to your fork of the repository. Create now a pull request on the original repository, approve the pull request and follow the deployment (in the original repository not your fork) **actions** > **workflow-name** and the run.

![Create pull request screenshot](/assets/part2/create-pull-request.png)

![Create pull request compare changes screenshot](/assets/part2/create-pull-request-2.png)

![Merge pull request screenshot](/assets/part2/merge-pull-request.png)

Now if you head into the **workflow** in Github, you should see a build in progress. Now to see the build logs, you can click on the **workflow**.

Once the build is complete, take a look at the build log and make a note of the following:

- Region: `region`
- Cognito User Pool Id: `UserPoolId`
- Cognito App Client Id: `UserPoolClientId`
- Cognito Identity Pool Id: `IdentityPoolId`
- S3 File Uploads Bucket: `AttachmentsBucketName`
- API Gateway URL: `ServiceEndpoint`

We'll be needing these later in our frontend and when we test our APIs.

![Dev build stack output screenshot](/assets/part2/dev-build-stack-output.png)

Now head over to the app home page. You'll notice that the build was directly promoted to production.

We will in the a next step define two pipelines that allow us to build and deploy the pull request first and another one that deletes this after the merge to master.

Next let's configure our serverless API with a custom domain.
