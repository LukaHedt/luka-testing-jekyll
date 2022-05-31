---
layout: post
title:  "Introduction to Projen - 5. Deploying a single stack to AWS Sandbox"
categories: projen how-to
---

## Deploying Code to a Sandbox Development Environment

We can now create all the bits to deploy to a sandbox environment. We need both CDK Code and to modify the projenrc to add the deployment environments.

### Creating a deployment environment

Open `.projenrc.ts` and add the following:

```ts
// .projenrc.ts
const project = new SvAwsCdkTypeScriptAppBase({
    cdkVersion: '2.25.0',
    // Where does your production code live?
    defaultReleaseBranch: 'main',
    name: 'test-projen-app',
  	// === Cut for berevity ===
});

// === Cut for berevity ===

// Add a Deployment Environment called 'dev' that deploys the CDK Stack called 'dev-test-projen-app'
project.addDeploymentStage('dev', ['dev-test-projen-app']);

project.synth();
```

From here we can re-run `npx projen`, and see that a number of Github Workflows has been modified to add some steps. I won't recount them here, but they are steps to compare the current and future state of the `dev-test-projen-app` CDK stack, and then deploy it to AWS.
In GitHub, we will see a Deployment Environment called `dev` appear once we deploy all these pieces.

Please note that the CDK stack name, Deployment Stack Name, and Application Name do NOT have any hard string dependencies. They are similar only because naming things in groups this way is concise.

We can also add some options dictating any deployment restrictions, but we will come back to this.

### Creating a CDK Stack to deploy

The next step is syncing our infrastructure code with our projen template. This is not done automatically.

Open `src/infra/main.ts`. The template will look like so (after running the formatter to match ES-Lint rules we set earlier):

```ts
// src/infra/main.ts
import { ServiceVicOrganization } from '@service-victoria/platform-cdk';
import { App } from 'aws-cdk-lib';
import { MyStack } from './stack';

// for development, use account/region from cdk cli
// const localEnvironment = {
//   account: process.env.CDK_DEFAULT_ACCOUNT,
//   region: process.env.CDK_DEFAULT_REGION
// }

const nonProdEnvironment = {
    // Replace this with the correct AWS account for your project
    account: ServiceVicOrganization.SERVICEVICTORIA_PLATFORM_NONPRODUCTION,
    region: 'ap-southeast-2',
};

// const prodEnvironment = {
//   // Replace this with the correct AWS account for your project
//   account: ServiceVicOrganization.SERVICEVICTORIA_PLATFORM_PRODUCTION,
//   region: 'ap-southeast-2',
// };

const app = new App();

// declare multiple instances of your stack representing the different deployment environments
new MyStack(app, 'my-stack-dev', { env: nonProdEnvironment });
// new MyStack(app, 'my-stack-prod', { env: prodEnvironment });
// new MyStack(app, 'my-stack-lab', { env: localEnvironment });

app.synth();
```

The `[Specific Settings | App Object + Settings | Synth]` structure of this file may be reminiscent of `.projenrc.ts`, this is intentional.

There's a lot of commented code here, but let's say explicitly what it's implying:

`localEnvironment`: This is an `aws. Environment` object that draws settings from the current machine's `process.env` setup.
This is intended for use with your personal CLI, and some possible credentials obtained via SSO/SAML.
We can see a reference to this object in the line `new MyStack(app, 'my-stack-lab', {env: localEnvironment});`.
With this, we can use the CLI to deploy our app to a Sandbox Environment if we so wish. I will not explain the process for this here.

`nonProdEnvironment`: This object refers to an AWS environment called `SERVICEVICTORIA_PLATFORM_NONPRODUCTION`. This is a constant in the `platform-cdk` library that gives the AWS AccountID for that AWS Environment.

`prodEnvironment`: Similar to non-prod, this points to a different AWS Environment.

`new MyStack( // ...`:  Each line here refers to a specific CDK Stack Object. We expect one of these stacks for each Deployment Environment we are targeting.

Note that the line area `'my-stack-dev'`, `'my-stack-prod'`, etc, must be unique for each Deployment Environment, you cannot have 2 stacks with the same ID.
Furthermore, any stack you want to be deployed via CI/CD (ie github actions) needs to have its ID/Stack Name (ie the string specified where `'my-stack-dev'` is placed) match an exact string in the `addDeploymentStage` call in `.projenrc.ts` (ie `['dev-test-projen-app']`).
Not all stacks in `src/infra/main.ts` need to be listed in `.projenrc.ts`, but if they are not, they will not be picked up by the CI/CD Pipeline.

`app.synth()`: A method call to create the CDK Template files that we will use to deploy our code. Each stack specified will be given its own separate file in `cdk.out`, which will be used when deploying that CDK Stack later.

Let's replace the contents of `main.ts` with the following:

```ts
// src/infra/main.ts
import { ServiceVicOrganization } from '@service-victoria/platform-cdk';
import { App } from 'aws-cdk-lib';
import { MyStack } from './stack';

const sandboxEnv = {
    account: ServiceVicOrganization.SERVICEVICTORIA_MIKE_SANDBOX,
    region: 'ap-southeast-2',
};

const app = new App();

new MyStack(app, 'dev-test-projen-app', { env: sandboxEnv });

app.synth();
```

Note that the StackID matches what we have specified in `.projenrc.ts`.

If we now press into the definition of `MyStack` (in `src/infra/stack.ts`), we will notice that it doesn't specify any AWS resources to deploy.
For the sake of the demo, let's add an S3 Bucket.

```ts
// src/infra/stack.ts
import { VersionedStack, VersionedStackProps } from '@service-victoria/platform-cdk';
import { RemovalPolicy } from 'aws-cdk-lib';
import { Bucket } from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

export class MyStack extends VersionedStack {
    constructor(scope: Construct, id: string, props: VersionedStackProps = {}) {
        super(scope, id, props);

        // define resources here...
        new Bucket(this, 'test-projen-app-bucket', {
            removalPolicy: RemovalPolicy.DESTROY,
            bucketName: 'test-projen-app-bucket',
        });
    }
}
```

From here, let's run the command  `npx projen build` to see what we get out.

This will run a number of steps, including:

- Compile the application
- Run Unit Tests
- Run ES-Lint
- Run CDK Synth

We can see we got some new repos that git is not tracking:

- cdk.out
- test-reports
- coverage

The last 2 are related to Jest and Unit Testing. We will ignore these for now.

`cdk.out` contains the following:

```bash
-rw-r--r--  1 lukahedt  staff    20 30 May 15:15 cdk.out
-rw-r--r--  1 lukahedt  staff   683 30 May 15:15 dev-test-projen-app.assets.json
-rw-r--r--  1 lukahedt  staff  1511 30 May 15:15 dev-test-projen-app.template.json
-rw-r--r--  1 lukahedt  staff  2625 30 May 15:15 manifest.json
-rw-r--r--  1 lukahedt  staff  2665 30 May 15:15 tree.json
```

As we can see, we get a separate `.assets.json` and `.template.json` file for our named stack. We will get one of each for every CDK stack we specify in `src/infra/main.ts`.

### Deploying to AWS via CI/CD

The next trick is to push all this into a repo on Github.
Now Note: The first commit may be different to your regular workflow, especially if you have `isManualRelease` set to false or undefined. Since we will be commiting and pushing directly to the `main` branch, this would see that the github actions automatically trigger a release for us, and it may even ask us to deploy our code to AWS.
This is intentional, since pushing to `main` would also imply a new release of your code, so don't be alarmed.
That being said, if you've followed all my directions to this point, you may not see it. That's fine too.

0. I have created, at this point, a new private repo in SV's github called `mike-test-projen-app`. It's a completely empty repo with no template or other details.
   I'll follow github's first-commit directions here.

1. Let's create a new branch, call it `feature/test-feature`.
   I don't actually intend to make any code changes here, but we need to make a commit so we can create a PR into `main`.
   Let's edit the Readme to be a little more friendly.

```md
// Readme.md

# test-projen-app

This application is a test app to display the steps involved in deploying my code to AWS.

This application has nary but an S3 Bucket.

```

2. Let's commit that change to our new branch, and create a Pull Request from `feature/test-feature` to `main`. Technically the PR name doesn't matter, but we should follow "Pull Request Lint"'s rules if we wish to keep all our little github checks green. (Please ignore that I have not).

3. We need to create a Github Label on the pull request, entitled `deploy/dev`. This will tell GitHub Actions that we intend for this pull request to be deployed to the Deployment Environment called `dev`.
   ![Labelled PullRequest](/assets/Labelled PullRequest.png)

4. Once the build step completes, you will be prompted (via email, most likely), to deploy the application to the `dev` Deployment Environment.
   ![DeployPrompt](/assets/DeployPrompt.png)

5. Click through the `deploy to dev #2 ` link there, and click `Review Deployments` on the right hand side. (White step below)
6. Tick the 'dev' checkbox on the modal, and click 'Approve and Deploy' (Red step below)
   ![DeployDevTicks](/assets/DeployDevTicks.png)

7. The deploy action will run. On completion, we can open the Team Mike Sandbox AWS Web Console, and the following stack will appear in Cloudformation

![Deploy1WebConsole](/assets/Deploy1WebConsole.png)

8. Now, every time a new commit is added to the Pull Request, you will be prompted to re-deploy once the build step is complete.

Merging PRs can now proceed as normal, which I shall do now.
