---
layout: post
title:  "Introduction to Projen - 4. Github Actions and Key Terms for Deployment"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 4
---

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>

## Github Actions - What the Devil?

Github Actions are the newer, preferred way to ship and deploy code to AWS. The interface is all localised in github, as opposed to another window like bamboo or buildkite.

Each Github Action is defined in a `.yml` file in the `.github/workflows` directory in your project.

They define:

- A set of triggers, ie "when do I run this workflow?"
- What kind of machine does the action need to run on?
- What are the steps of the action? (These are usually either shell commands, or links to other github projects)

I won't go into the full anatomy of them here, but the `projen-templates` file thankfully generates a standard set of actions that will build and test the code you've committed! It also includes steps for deploying to different environments, but we can go into more detail on that momentarily.

### Adding your own Actions

It's possible to add your own actions to the build or release process, but we will not cover this here.

## Deployment - Fundamentals (One Deploy Environment)

As mentioned, cicd deployment is one of the key motivators to using `projen`, it gives a simple, standardised, generated method of putting code into AWS/etc, without a great deal of convoluted steps.

This guide is written in terms of Backend systems, so the instructions here may not apply to all users. Proceed with that understanding.

### Key Terms

There's a few key elements to understanding deployment in this method, and some of them use overlapping terms, so let's ensure we define them correctly.

#### 1. AWS Environment

An AWS Environment refers to an **AWS Account** + an **AWS Region**.
Some AWS resources need to be uniquely named across these two items, such as:

- Lambda Function Names
- S3 Bucket Names
- SSM Secret Names

You will note that inspecting AWS resources' `ARN (Amazon Resource Number)` will show both of these items, as well as some identifier for the resource.

AWS Environments are typically created and controlled by the Platform team.

#### 2. Deployment Environment

A Deployment Environment is *distinct from, yet related to* an AWS Environment.

An AWS Environment may contain multiple Deployment Environments, if desired.

Deployment Environments are configurable via `projen` code (ie, in `.projenrc.ts`), and more generally accessible inside GitHub. 
In projen, they are referred to as `deploymentStages`, but I feel this muddies the distinction between "deployment steps" and "self-contained places I can run my code". 

They are typically set up by a development team to support their own development and testing flow, and there are no limits on how many DE's you create, as long as you structure your CDK code to support it correctly.

#### 3. Deployment Stack/CDK Stack

CDK Stacks are an AWS CDK Concept, and act as a singular unit of deployment. 
Everything in the stack is deployed together.

One or more CDK Stacks may be deployed to a Deployment Environment.

CDK Stacks can be viewed in the AWS Web Console under `Cloudformation`.

#### 4. Deployment

A deployment is when the code from a single commit is pushed to, and made available in, a Deployment Environment.

#### 5. Release

A 'release' in this context, is when code on the `main` branch is tagged with a version number, and (if production environment is set up) the code is deployed to production.
Technically, Deployment and Release are separate steps, but it is implied that released code is also deployed.

<details>
<summary>Related Articles</summary>
<ul>
{% for projen_doc in site.projen %}
<li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
{% endfor %}
</ul>
</details>
