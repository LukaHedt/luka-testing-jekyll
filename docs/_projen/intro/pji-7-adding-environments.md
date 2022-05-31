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
