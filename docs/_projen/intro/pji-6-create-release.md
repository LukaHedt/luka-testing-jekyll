---
layout: post
title:  "Introduction to Projen - 6. Creating a new Release"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 6
---

## Related Posts

{% for projen_doc in site.projen %}
- [{{ projen_doc.title }}]({{ site.baseurl }}{{ projen_doc.url }})
{% endfor %}


## Creating a Release

Now we act to create a Release for our newly minted test application. Since we have not yet set up a 'prod' environment, this release will simply tag our current commit with a version number, and likely attempt to re-deploy our application.

1. On Github, open the 'Actions' tab, and click on the `release` workflow
   ![ReleaseWorkflowPage]({{site.baseurl}}/assets/ReleaseWorkflowPage.png)

2. Click 'Run Workflow' on the right hand side, and click the green 'Run workflow' button. This will trigger a new GitHub Release being created, and you will be asked to deploy your release to `dev` environment. It will also create a Tag for your version, likely `0.0.0` to begin with.
   ![ReleaseActionPage]({{site.baseurl}}/assets/ReleaseActionPage.png)

Subsequent runs with more commits will bump the version higher, and produce a changelog in the release page from different commits. If you are using the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) pattern, the changelog will be nicely formatted.

## Related Posts

{% for projen_doc in site.projen %}
- [{{ projen_doc.title }}]({{ site.baseurl }}{{ projen_doc.url }})
{% endfor %}
