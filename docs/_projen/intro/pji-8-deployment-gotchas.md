---
layout: post
title:  "Introduction to Projen - 8. Deployment Gotchas"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
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
