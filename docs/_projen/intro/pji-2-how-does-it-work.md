---
layout: post
title:  "Introduction to Projen - 2. How does it work?"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 2
---

<details>
  <summary>Related Articles</summary>
  <ul>
  {% for projen_doc in site.projen %}
    <li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
  {% endfor %}
  </ul>
</details>

## So how do I make it chooch on my PC?

Firstly, follow the startup guide [Here](https://servicevic.atlassian.net/wiki/spaces/PE/pages/4578279603/Application+Development+-+Getting+Started) to install all the important pieces.

Before you create your new repo, make sure to read through the Readme on the templates file itself, as a few key points may have changed: [https://github.com/service-victoria/projen-templates](https://github.com/service-victoria/projen-templates)

Once you've created a new template repo, you will want to run `npx projen` to re-create all the statically generated files.
Most of the files in your new directory are either template files - for you to edit and delete at will (anything in `/src` or `/test`, for example) - or they're projen-managed files, meaning if you edit one, then re-run `npx projen`, it will magically change itself back. [^10]

[^10]: There's a few exceptions to these, for example if you edit a version number in  `package.json`, it will probably respect your change, but that is about it.

### An Experiment - Editing a projen managed file

Let's test that theory! Let's go and try and edit `.eslintrc.json`!
I'm not a big fan of 2-space indents for any language, so let's find the clause 

```json
// .eslint.json
"rules": {
    "indent": [
      "off"
    ],
    "@typescript-eslint/indent": [
      "error",
      2
    ],
    "quotes": [
      "error",
      "single",
      {
        "avoidEscape": true
      }
    ],
},
...
```

And change the rule to `"@typescript-eslint/indent": ["error", 4]`.

Save it (if you can), and then re-run `npx projen`.

Oh no, it swapped back to 2!

### So how do I change settings?

Let's look at how we actually change things around here.
Open the file `.projenrc.ts`. Mine looks like this:

```ts
// .projenrc.ts
import { SvAwsCdkTypeScriptAppBase } from '@service-victoria/projen-templates';
const project = new SvAwsCdkTypeScriptAppBase({
  cdkVersion: '2.23.0',
  defaultReleaseBranch: 'main',
  devDeps: ['@service-victoria/projen-templates'],
  name: 'test-projen-app',

  // deps: [],                /* Runtime dependencies of this module. */
  // description: undefined,  /* The description is just a string that helps people understand the purpose of the package. */
  // packageName: undefined,  /* The "name" in package.json. */
});
project.synth();
```

So there's 3 real interesting bits here.

1. A definition of a `project`, and we see it's a `SvAwsCdkTypeScriptAppBase` - ie, a typescript app that runs on AWS, and uses CDK for deployment. Oh and it's a Service Vic template, so when the team in charge of the library [^20] makes a change to it, you can pull it down, and use the newer definition.

   [^20]: The platform team, they're super friendly :D

2. We have definitions within the `project` constructor that include `deps` and `devDeps`. This is where we do package management? Okay interesting. Put a pin in that maybe.

3. At the end of the file, we call `project.synth()`. This follows the convention of AWS-CDK, where this method call does all the work to change infrastructure code into something else. Okay cool beans.

So to change our indent settings to be a bit less squished, we need to change the config object for `project`, and then run `npx projen` again.

Here's my projenrc once I've done that, to save you reading through all the definitions (I won't lie, the docs hurt me to read too.)

```ts
// .projenrc.ts
import { SvAwsCdkTypeScriptAppBase } from '@service-victoria/projen-templates';
import { EslintOverride } from 'projen/lib/javascript';
const project = new SvAwsCdkTypeScriptAppBase({
    cdkVersion: '2.23.0',
    defaultReleaseBranch: 'main',
    devDeps: ['@service-victoria/projen-templates'],
    name: 'test-projen-app',

    // deps: [],                /* Runtime dependencies of this module. */
    // description: undefined,  /* The description is just a string that helps people understand the purpose of the package. */
    // packageName: undefined,  /* The "name" in package.json. */
});


const eslintOverrides: EslintOverride = {
    files: ['*.ts'],
    rules: {
        '@typescript-eslint/indent': ['error', 4],
    },
};
project.eslint?.addOverride(eslintOverrides);

project.synth();
```

I had to reformat the spacing manually so that ES Lint didn't complain about the indent level :stuck_out_tongue:

If you look at the `.eslintrc.json` file, you'll notice we didn't actually change the original rule, we added a new entry to `overrides`.
But that's fine, it does what we want now, so it's all good :smile_cat:.

<details>
  <summary>Related Articles</summary>
  <ul>
  {% for projen_doc in site.projen %}
    <li><a href="{{ site.baseurl }}{{ projen_doc.url }}">{{ projen_doc.title }}</a></li>
  {% endfor %}
  </ul>
</details>
