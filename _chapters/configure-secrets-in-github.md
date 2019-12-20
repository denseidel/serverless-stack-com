---
layout: post
title: Configure Secrets in Github
lang: en
date: 2018-03-13 00:00:00
description: To automate our Serverless deployments with Seed (https://seed.run), we will need to set our secrets in the Seed console. Move the environment variables from your .env to the stage we are deploying to.
ref: configure-secrets-in-github
comments_id: configure-secrets-in-seed/176
---

Before we can do our first deployment, we need to make sure to configure our secret environment variables. If you'll recall, we have explicitly not stored these in our code (or in Git). This means that if somebody else on our team needs to deploy, we'll need to pass the `.env` file around. Instead we'll configure [Github Action](https://github.com) to deploy with our secrets for us.

First get the credentials we used in the steps before. Remember it is a best practices to create new credentials with limited rights.

<img class="code-marker" src="/assets/s.png" />Run the following command.

```bash
$ cat ~/.aws/credentials
```

The output should look something like this.

```
[default]
aws_access_key_id = YOUR_IAM_ACCESS_KEY
aws_secret_access_key = YOUR_IAM_SECRET_KEY
```

Add the credentials for the the pipeline using [configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials) action. You need to create credentials for each stage if they are in different accounts or have different roles attached (e.g. can only edit resources with a specific label).

Create the [secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets) in Github. Click on `Settings` > `Secrets` > `Add a new secret`.

Create a Secret for `DEV_AWS_ACCESS_KEY_ID` with the AWS access key id and `DEV_AWS_SECRET_ACCESS_KEY` with the AWS secret access key. As well as the `STRIPE_SECRET_KEY` from your `.env` file.

![Add AWS IAM credentials screenshot](/assets/part2/create-new-github-secret.png)

Ensure they are refrenced in your pipeline definition.

Next, we'll trigger our first deployment on Github.
