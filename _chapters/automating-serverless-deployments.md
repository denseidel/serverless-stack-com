---
layout: post
title: Automating Serverless Deployments
date: 2018-03-11 00:00:00
lang: en
description: We would like to automatically deploy our Serverless Framework project when we commit any changes to our Git repository. To do this we are going to use a service called Seed (https://seed.run) to automate our serverless deployments. It will configure a CI/CD pipeline and setup our environments.
ref: automating-serverless-deployments
comments_id: automating-serverless-deployments/174
---

So to recap, this is what we have so far:

- A serverless project that has all it's infrastructure completely configured in code
- A way to handle secrets locally
- And finally, a way to run unit tests to test our business logic

All of this is neatly committed in a Git repo.

Next we are going to use our Git repo to automate our deployments. This essentially means that we can deploy our entire project by simply pushing our changes to Git. This can be incredibly useful since you won't need to create any special scripts or configurations to deploy your code. You can also have multiple people on your team deploy with ease.

Along with automating deployments, we are also going to look at working with multiple environments. We want to create clear separation between our production environment and our dev environment. We are going to create a workflow where we continually deploy to our dev (or any non-prod) environment. But we will be using a manual promotion step when we promote to production. We'll also look at configuring custom domains for APIs.

For automating our serverless backend, we use [Github Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-action). Let's get started with setting up your project on GitHub.

Before we start implemnting lets out line our staging concept:

1. **Master**: The code on this stage / branch will run in production. The code is put into production through a pull request. This triggers a canary deployment limiting the bast radius of a release.
2. **Pull Request**: For each pull request a seperated deployment is created and tested. After the tests are successful, this might include a manual aproval step, the branch is merged into master and then deleted. For a manual approval step don't automatically merge to master, but deploy the application, post the link to the pull request and wait for a manual approval, then merge to master.
3. **Local**: This is the local stage for each developer, on this stage either you work with local versions of AWS like localstack or your give each developer a account.

Why no **Dev/Integration** stage - The best way is to test each pull request and reduce the cycle time as much as possible and keep features small and independent.

To simulate a real setup we will create an _Github organisation_ in the Github UI.

![Click create orgnaisation in Github screenshot](/assets/part2/create-github-organisation-1.png)

For this tutorial create an open source project as it is free.

![Click create open source organization in Github screenshot](/assets/part2/create-github-organisation-2.png)

Give it an **Organisation Account Name**, **Contact Email** and select that the organisation belongs to your account. This simplifies the setup as you can access the organisation directly from your existing account.

![Fill the organisation formular screenshot](/assets/part2/create-github-organisation-3.png)

Check that your account is part of the organisation and **finish** the creation.

![Click to finish the process screenshot](/assets/part2/create-github-organisation-4.png)

Next you can either create an new repository or migrate the existing one from your private account. As we create the repository in your existing account we will migrate it.

Go to your existing repository.

![Go to settings in your repository screenshot](/assets/part2/migrate-github-repo-to-organisation-1.png)

Make the repository public. This is required as your open source organisation can only host public repositories.

![Click on make public screenshot](/assets/part2/make-repository-public.png)

Transfer the repository to the organisation you created before.

![Click on transer repository screenshot](/assets/part2/transfer-the-repository.png)

Go to the **main page** and select your **organisation**.

![Switch to organisation screenshot](/assets/part2/switch-to-organisation.png)

Switch to the transfered repository and setup branch protection by go to **setting** and **branches** and click on **add rules**.

![Enable branch protection screenshot](/assets/part2/enable-branch-protection.png)

Create a rule for the **master** branch. To make sure that only reviewed and tested code goes to production enable **Require pull request reviews before merging**. We also enable **includes administrators** to enforce the rules also yourself (as you are an administrator).

![Add branch protection rules screenshot](/assets/part2/add-branch-protection-rules.png)

![Add branch protection rules screenshot](/assets/part2/add-branch-protection-rules-2.png)

To chance code fork the migrated repostitory from your organisation into your own account.

![Fork repository screenshot](/assets/part2/fork-the-organisation-repository.png)

Then find the url of the forked repo and update the remote of your existing local repository.

```bash
git remote set-url origin https://github.com/denseidel/serverless-stack-api.git
```

Now your ready to update the code through pull request.

Start developing your application locally and use unit test and mocks (e.g. localstack,
[aws-sdk-mock](https://serverless.com/blog/serverless-local-development/)) for developing.
Afterwards write automated integration test that can be run on the pull request stage to validate your code.
This is what you learned up to now. Next you create a pull request on the repository.

The `on-pull-request` pipeline is triggered and runs the unit tests and integration test,
deploys it to a test endpoint and provides feedback in the pull request threat.
Next you have two option for full automation go with an A/B canary release that switches
some of the traffic over to the new function (that is then connected to the production persitance layer)
from the pull request and waits if alarms trigger. If no alarms trigger more traffic is switch to the new function until 100%.
Or someone check the feature and evaluates it manually. And either merges the request or not to master.

Finally the pull request will be closed. The `on-pull-request-closed`pipeline removes the test endpoint after the pull request was closed.
The `master` pipeline run on push to master and deploys to master.
The best solution is to run the two latest releases in parallel as a canary release and shift traffic slowly using the production databases
but the function from the branch endpoint. Your code then checks for alarms and if any happen roles back the release and fails the pull request
(last step before closing it).
