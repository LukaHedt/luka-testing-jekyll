---
layout: post
title:  "Introduction to Projen - 9. A Git Workflow"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 9
---

<details>
  <summary>Related Articles</summary>
  <ul>
  {% for projen_doc in site.projen %}
    <li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
  {% endfor %}
  </ul>
</details>


## A Git Workflow

The following is a git workflow you might find works for you.

You are not bound to it, by any stretch, but it should indicate where code may need to be to deploy it successfully.

### Prereqs:

- Persistent branches named `main`, `uat` and `develop`.
- An environment setup as described above.

### Steps:

1. A new feature branch is created off the `develop` branch, call it `ma1`.
2. The developer makes some initial commits - creating test cases, adding code, adding documentation.
3. The developer creates a Pull Request back to `develop`, and labels it with `deploy/devstack1`, so they can deploy and test their code in AWS.
4. The developer pushes more commits to the branch, perhaps fixing some minor bugs or correcting CDK code. They deploy each commit to AWS when github asks them to.
5. Once they are happy their code works, they assign it to another team member for code review.
6. That team member provides any feedback, and once happy, labels the PR with `deploy/dev`. 
7. They ensure the deployment completes and the code passes sanity checks before merging to `develop` branch. 
8. Here, another feature branch could be created if required, or an existing feature branch should be caught up with the changes.
9. As features are collected, a PR is created from `develop` to `uat` branch. It is tagged with `deploy/demo`.
10. The deployment to `demo` is monitored and tested to ensure the CDK changes are valid, and that the application still passes any integration tests, etc.
11. Once satisfied, the team tags the PR with `deploy/uat` and monitors the deployment. Sanity tests are completed before merging the PR with the `uat` branch.
12. Once the UAT branch has been thoroughly tested, or otherwise used by any consuming teams, a PR is created from `uat` to `main`. The PR is tagged with `deploy/preprod`.
13. Sanity checks are completed on the preprod environment, as well as any integration tests.
14. Once passed, the PR is merged to `main`, and the `release` workflow is run in GitHub.
15. The deployment to `prod` is approved, and then the regular sanity checks are completed in production.

See below diagram for extra context:

![Projen + Git Flow-2]({{site.baseurl}}/assets/images/projen/intro/ProjenGitFlow.png)

<details>
  <summary>Related Articles</summary>
  <ul>
  {% for projen_doc in site.projen %}
    <li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
  {% endfor %}
  </ul>
</details>
