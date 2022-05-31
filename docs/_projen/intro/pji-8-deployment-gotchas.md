---
layout: post
title:  "Introduction to Projen - 8. Deployment Gotchas"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos "projen-templates 0.0.114"
post_step: 8
---

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>

## Deployment Gotchas

### Source vs Target Pull Request Branches

As a safety precaution, the default build instructions actually do this clever little trick where they use the `build.yml` instructions from the target branch in the pull request, rather than in the source.

This means, you can't go and add a bunch of (potentially) malicious build instructions to a `feature/*` branch, create a PR to `develop`, and have your malicious code run automatically. The build instructions have to already be in `develop` for them to run!

This has the double whammy effect that - to deploy your code to say, a `uat` Deployment Environment from a tagged PR (let's say the PR is from `develop` to `uat`), the `uat` environment code **must already exist in the uat branch** to be able to deploy it.

That's not to say you can't create a second PR to `uat` to create the environment in the build instructions, and then deploy from the first PR. But it probably will confuse you the first time you see this behaviour!

**Update Note: projen-templates 0.0.114 to 0.0.115**: If you upgrade your templates version in a feature branch, and then PR to your `dev` branch, you will probably notice that the build didn't run. This is because the Trigger was changed in `0.0.115`, but your `dev` branch is probably still using `0.0.114`.

This is likely not a major issue, unless you are like me and have a bad habit of including version upgrades in feature branches I'm already working on.

Say I was adding a new feature in branch `feature/my-super-feature`, and I have created a PR to `develop`. About 30 devstack deployments into `feature/my-super-feature`, I decided to upgrade the projen templates, from `0.0.113` to `0.0.115`. Since `develop` hasn't been updated yet, I won't be able to build or deploy `feture/my-super-feature`, and the reason is probably non-obvious.

To fix this, I should create a new PR from `ci/upgrade-projen` to `develop` where I update the projen templates (use `npm projen upgrade-projen`), and then merge it, before making a change to `feature/my-super-feature` again. (Just a label change should do).
This will also be true of any upstream branches.

### Uniquely Named Fields in CDK

Some items, like Lambda Function Names and S3 Bucket Names, are designed to be unique within one AWS Environment. 

That means - if I have a function in `demo` called 'my-fancy-lambda-func', I cannot deploy a function with the same name in `uat` (if they share an AWS Environment). The deployment will fail.

Since I'm a fan of having nicely-labelled AWS resources, I prepend these items with the environment name in non-prod environments.

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>
