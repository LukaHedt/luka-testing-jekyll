---
layout: post
title:  "Introduction to Projen - 7. Adding More Deployment Environments"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 7
---

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>

## Adding More Deployment Environments

Adding more environments is just a matter of adding more `addDeploymentStages` entries to `.projenrc.ts`, and corresponding Stack entries to `src/infra/main.ts`.

### Adding a Devstack and UAT environment

Let's add a Devstack and UAT deployment environment. For demonstration's sake, I shall put all 3 environments into the Team Mike sandbox account (so I can delete them later easily), but the only major change between the AWS environments for CI/CD is the AWS AccountID. Keep in mind that Preprod and Prod AWS Environments are set up so users have less free-reign for changing settings.

We will also point out some customisations we can make to different Deployment Environments, and some considerations for each.

#### Adding Deployment Environments in Projen

Let's open `.projenrc.ts` and add the following after the `project` object is created:

```ts
// .projenrc.ts
// ...
// Add a stage called `devstack1`
project.addDeploymentStage('devstack1', ['devstack1-test-projen-app']);

// From Earlier in the Tutorial, but let's modify it
project.addDeploymentStage('dev', ['dev-test-projen-app'], {
    requiredReviewers: ['service-victoria/team-mike'],
});

// Add a stage called `uat`
project.addDeploymentStage('uat', ['uat-test-projen-app'], {
    requiredReviewers: ['service-victoria/team-mike'],
    allowedBranches: ['develop', 'uat', 'main'],
});

```

So our definition of  `devstack1` is the same as our definition for `dev` before. There are no restrictions on whom can deploy code to this environment.

We've also modified our definition of `dev` to list some `requiredReviewers`, namely 'service-victoria/team-mike'. This can be any GitHub user or Team, and all of the people in this list (including all team members if a team is specified) will be sent an email asking for a deployment review when the stage is targeted for a deployment. 

**Note 1:** that only 1 member needs to approve the deployment, and if you're part of the specified team, you can approve your own deployment.
Teams will have to manage their own workflows to make sure all deployments are correctly reviewed before approving.

**Note 2:** A "review" in this case, is not the same as a review on your Pull Request. See the "Deploying Code to a Sandbox Environment" section.

Lastly, we added a Deployment Environment called `uat`, which also requires a review from someone from Team Mike before deployment.
We've also specified a list of  `allowedBranches`, which are the github branches that form the **source** of a pull request. This pattern implies that we cannoy deploy code from a `feature/*` branch directly to UAT, for example, the PR needs to come from a branch called `develop`. (Or `uat` or `main`, but that is more for deploying releases).

Let's re-run `npx projen` to re-generate the GitHub actions for deployment. We should see sections in the build & release workflows that reference the `devstack1` and `uat` environments.

**Note:** at this point, we can merge our code into our `main` branch. We won't yet be able to deploy anything to these environments - we haven't yet created the AWS CDK Stacks that match them, but we need the `build` workflow that targets these environments to already exist before we can deploy infrastructure to them. This may seem confusing, but see the "Deployment Gotchas" section for more details.

#### Adding AWS CDK Stacks

Now we need to add CDK stacks for each environment. We will go into some more detail around stack variables while we are at it, but let's build up from the ground.

1. Firstly, let's create and push a github branch called `develop`. This will be our main target branch for new features or bugfixes. We won't commit anything to it yet.

   ```bash
   git checkout -b develop
   git push --set-upstream origin develop
   ```

2. Next, let's checkout a new, second branch called `feature/my-feature`.

   ```bash
   git checkout -b feature/my-feature
   ```

3. Now let's open `src/infra/main.ts` and modify it to look like so

   ```ts
   // src/infra/main.ts
   import { ServiceVicOrganization } from '@service-victoria/platform-cdk';
   import { App } from 'aws-cdk-lib';
   import { MyStack } from './stack';
   
   const sandboxEnv = {
       account: ServiceVicOrganization.SERVICEVICTORIA_MIKE_SANDBOX,
       region: 'ap-southeast-2',
   };
   
   // New - Non prod Environment details. IRL you will use your app's Non-prod environment, not the Mike Sandbox
   const nonProdEnv = {
       account: ServiceVicOrganization.SERVICEVICTORIA_MIKE_SANDBOX,
       region: 'ap-southeast-2',
   };
   
   const app = new App();
   
   // Adding extra stacks.
   new MyStack(app, 'devstack1-test-projen-app', { env: sandboxEnv });
   new MyStack(app, 'dev-test-projen-app', { env: sandboxEnv });
   new MyStack(app, 'uat-test-projen-app', { env: nonProdEnv });
   
   app.synth();
   
   ```

   As stated before, the `id` string of the stack matches an array value in the `projenrc` file we edited earlier.

4. Let's commit and push these changes to our feature branch, and create a Pull Request into `develop`.

5. Label the PR with `deploy/devstack1`, so we try and deploy the code into AWS under the new `devstack1` environment.

Oh crap, the deployment failed! 

```bash
Failed resources:
devstack1-test-projen-app | 12:51:13 AM | CREATE_FAILED        | AWS::S3::Bucket    | test-projen-app-bucket (testprojenappbucket23740E86B) test-projen-app-bucket already exists in stack arn:aws:cloudformation:ap-southeast-2:636795152967:stack/dev-test-projen-app/879f5100-dfe6-11ec-b43c-0a8caa13634c

 ❌  devstack1-test-projen-app failed: Error: The stack named devstack1-test-projen-app failed creation, it may need to be manually deleted from the AWS console: ROLLBACK_COMPLETE: test-projen-app-bucket already exists in stack arn:aws:cloudformation:ap-southeast-2:636795152967:stack/dev-test-projen-app/879f5100-dfe6-11ec-b43c-0a8caa13634c
    at prepareAndExecuteChangeSet (/home/runner/.npm/_npx/5b0bc14742efef1b/node_modules/aws-cdk/lib/api/deploy-stack.ts:385:13)
    at processTicksAndRejections (node:internal/process/task_queues:96:5)
    at CdkToolkit.deploy (/home/runner/.npm/_npx/5b0bc14742efef1b/node_modules/aws-cdk/lib/cdk-toolkit.ts:209:24)
    at initCommandLine (/home/runner/.npm/_npx/5b0bc14742efef1b/node_modules/aws-cdk/lib/cli.ts:341:12)
```

As we said earlier, some items like S3 bucket names need to be unique in an AWS Environment, and we already deployed a bucket with the name `test-projen-app-bucket` with the `dev` stack.

Let's modify our stack code so it takes in the Deployment Environment name, and pre-pends it to the bucket names so we don't accidentally get confused.

**NOTE:** The following change will destroy and re-create the existing dev bucket when we re-deploy the dev environment later. Make sure this won't lose any app data before you do so.

```ts
// src/infra/stack.ts
import { VersionedStack, VersionedStackProps } from '@service-victoria/platform-cdk';
import { RemovalPolicy } from 'aws-cdk-lib';
import { Bucket } from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

// Adding these props here
export interface MyStackProps extends VersionedStackProps {
    readonly deployEnvName: string;
}

export class MyStack extends VersionedStack {
    constructor(scope: Construct, id: string, props: MyStackProps) {
        super(scope, id, props);

        // define resources here...
        new Bucket(this, `${props.deployEnvName}-test-projen-app-bucket`, {
            removalPolicy: RemovalPolicy.DESTROY,
            bucketName: `${props.deployEnvName}-test-projen-app-bucket`,
        });
    }
}
```

```ts
// src/infra/main.ts
import { ServiceVicOrganization } from '@service-victoria/platform-cdk';
import { App } from 'aws-cdk-lib';
import { MyStack } from './stack';

const sandboxEnv = {
    account: ServiceVicOrganization.SERVICEVICTORIA_MIKE_SANDBOX,
    region: 'ap-southeast-2',
};

const nonProdEnv = {
    account: ServiceVicOrganization.SERVICEVICTORIA_MIKE_SANDBOX,
    region: 'ap-southeast-2',
};

const app = new App();

new MyStack(app, 'devstack1-test-projen-app', { env: sandboxEnv, deployEnvName: 'devstack1' });
new MyStack(app, 'dev-test-projen-app', { env: sandboxEnv, deployEnvName: 'dev' });
new MyStack(app, 'uat-test-projen-app', { env: nonProdEnv, deployEnvName: 'uat' });

app.synth();
```

We also need to adjust our `test/infra/main.test.ts` file to reflect this definition change. I have deleted this file for now, as well as its neighbouring snapshot file, as it is not helping us.

Committing and pushing these changes should happily deploy the code into `devstack1`. 

If you add the label `deploy/dev` to the same Pull Request, it should also deploy to the development environment, destroying the old bucket and creating a new one with `dev` in the name.

Here's the list of buckets in Mike Sandbox once I've done all this:

![AfterDeployment2StackBuckets]({{ site.baseurl }}/assets/images/projen/intro/AfterDeployment2StackBuckets.png)

If we were to label this PR with `deploy/uat`, we should see that it doesn't attempt to deploy, as we marked UAT as requiring `develop` as the source branch, not `feature/*`.

### A potential Deployment Stack Pattern

This is by no means a well-tested system, but I can offer the following pattern:

- `devstack<1|2|3|...>` - environments in your team's sandbox for individual developers to test their feature development.
- `dev` - An environment in your team's sandbox to test features after they've been merged. This can be to test that CDK changes are all valid, or that two features don't collide.
- `demo` - An environment in the NonProd domain of AWS for your application. Use this for developer testing in a non-sandbox environment.
- `uat` - An environment in the NonProd domain of AWS for your application. Use this for Frontend/Integration/User Acceptance testing with your application.
- `preprod` - A staging environment in either the Prod or Pre-prod domain of AWS for your application. Use this to do minor integration testing, checking that settings all function correctly, and that CDK code changes are valid before pushing to production.
- `prod` - An environment in the Prod domain of AWS for your application. Users consume the application here.

You are under no obligation to use any of this, but it may help get you started.

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>
