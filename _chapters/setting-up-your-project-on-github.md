---
layout: post
title: Setting up Your Project on Github Actions
date: 2018-03-12 00:00:00
lang: en
description: To automate our Serverless deployments, we will use Github Actions (https://github.com). We will add actions to our project repository, and set our AWS IAM credentials.
ref: setting-up-your-project-on-github
comments_id: setting-up-your-project-on-seed/175
---

We are going to use [Github Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions) to automate our serverless deployments and manage our environments. This is based on the example to deploy to [circleci](https://seed.run/blog/how-to-build-a-cicd-pipeline-for-serverless-apps-with-circleci).

Start by going to your Github Project [here](https://github.com).

Let's **Add github actions to your project**.

![Add github actions to project screenshot](/assets/part2/add-github-actions-to-project.png)

Now add a simple workflow to your project .

![Create simple workflow screenshot](/assets/part2/create-simple-workflow.png)

First we create the `master` pipeline that triggers when changes are pushed to the master branch. This will deploy the changes to production.
The pipeline includes the setup of

TODO: use the deployed asset of the pull request stage instead of rebuilding it.

{% raw %}

```yaml
name: Master CI/CD

on:
  push:
    branches:
      - master

jobs:
  primary:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v1
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: 12.x
      - name: install dependencies
        run: yarn install
      - name: lint
        run: yarn lint
      - name: test
        run: yarn test
      - name: serverless deploy
        run: npx serverless deploy
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.DEV_AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.DEV_AWS_SECRET_ACCESS_KEY }}
          STRIPE_SECRET_KEY: ${{ secrets.STRIPE_SECRET_KEY }}
```

{% endraw %}

Notice, `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. Github deploys to your AWS account on your behalf. You should create a separate IAM user with exact permissions that your project needs. You can read more about this [here](https://seed.run/docs/customizing-your-iam-policy). But for now we'll simply use the one we've used in this tutorial. For that we will need to configure these in our Github repository secrets section in the next step.

This pipeline installs the dependencies, lints and test the code then deploys it. You'll recall that we had added a couple of tests back in the [unit tests]({% link _chapters/unit-tests-in-serverless.md %}) chapter.

However, before we run the pipeline that, we'll need to add our secret environment variables.

Further improvements could also be:

- [Cache the dependancies between builds](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/caching-dependencies-to-speed-up-workflows)
