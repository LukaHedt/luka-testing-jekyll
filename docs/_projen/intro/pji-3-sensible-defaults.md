---
layout: post
title:  "Introduction to Projen - 3. Some Sensible Defaults"
author: "Luka Hedt (luka.hedt@twobulls.com)"
date: "2022-05-31 16:00:00 +1000"
tags: projen how-tos
post_step: 3
---

## Related Posts

{% for projen_doc in site.projen %}
- [{{ projen_doc.title }}]({{ site.baseurl }}{{ projen_doc.url }})
{% endfor %}

## Some Sane Defaults - Make my VsCode startup easy please!

```ts
// .projenrc.ts
import { SvAwsCdkTypeScriptAppBase } from '@service-victoria/projen-templates';
import { EslintOptions, EslintOverride, PrettierOptions, TrailingComma } from 'projen/lib/javascript';

// Set prettier to enforce these apparently non-standard rules
const prettierOptions: PrettierOptions = {
    settings: {
        printWidth: 120, // No more than 120 chars in 1 line.
        tabWidth: 4, // Code indent is 4 spaces!
        singleQuote: true, // Use single-quotes for strings.
        semi: true, // Add the damn semicolons!
        trailingComma: TrailingComma.ES5, // I like trailing commas when possible!
    },
};

// Set ESLint to be nice.
// ESLint will usually tell you if you're using something imported in 'devDeps' inside your deployed app.
// We need to tell it which bits are deployed app.
const eslintOptions: EslintOptions = {
    dirs: ['src/app'],
    devdirs: ['src/infra', 'test'],
};

const project = new SvAwsCdkTypeScriptAppBase({
    cdkVersion: '2.25.0',
    // Where does your production code live?
    defaultReleaseBranch: 'main',
    name: 'test-projen-app',

    // You get to press a button in github to make a new release! It's not automatic!
    // Stops every commit to `main` becoming a new version.
    isManualRelease: true,
    // Projen will complain if you have modified a projen-managed file before you committed,
    // and even make a PR to fix the change! This turns off the change, since the action projen
    // creates to do this uses a secret that SV doesn't make available.
    mutableBuild: false,
    // Probably not critical, but npm gives me enough issues when there isn't a cache involved :p
    enableDependencyCaching: false,

    // For some reason pj adds the `dd-trace` library, which has some strange behaviour with some peer dependencies, which also blocks deployments.
    // This turns that off
    datadogJestInstrumentation: false,

    // There's a feature where any file that ends in `.lambda.ts` will auto-generate some infra code for it.
    // I'm personally not a fan, but YMMV.
    lambdaAutoDiscover: false,

    // The following combo, plus this vscode extension: rvest.vs-code-prettier-eslint
    // Will make ESLint and Prettier all work happy together, and even let you use the
    // `Format Document` command in vscode to make your work pass linting.
    // Turn on ES Lint explicitly.
    eslint: true,
    eslintOptions,
    // Turn on prettier explicitly
    prettier: true,
    prettierOptions,

    devDeps: [
        // Required to keep using the SV Template
        '@service-victoria/projen-templates',
        // Required for use with projen, v9 seems to break for some reason.
        'ts-node@^10',
        // Adds the `lambda` event types, which are helpful!
        '@types/aws-lambda',
        // Required for the ESLint/`Format Document` command thing
        'prettier-eslint@^10.1.0',
    ],
    deps: [
        // I like dependency injection. Don't you?
        'typed-inject',
        // Lambda config must flow from ParameterStore. Please use SDKv3.
        '@aws-sdk/client-ssm',
        // This adds standard CDK constructs, as well as aws account information!
        '@service-victoria/platform-cdk@^2.12.0',
        // A logging framework. Anything that logs JSON is probably fine, but this one is nice.
        'winston',
        'aws-lambda',
        'datadog-lambda-js',
        // Svic Platform-CDK Peer Deps. Yarn isn't good with pre-release versions so we need to pin the version.
        '@aws-cdk/aws-kinesisfirehose-alpha@2.4.0-alpha.0',
        '@aws-cdk/aws-kinesisfirehose-destinations-alpha@2.4.0-alpha.0',
        '@aws-cdk/aws-glue-alpha@2.4.0-alpha.0',
    ],
});

// We don't really need to override any rules, but just in case
const eslintOverrides: EslintOverride = {
    files: ['*.ts'],
    rules: {},
};
project.eslint?.addOverride(eslintOverrides);

project.synth();

```

## Related Posts

{% for projen_doc in site.projen %}
- [{{ projen_doc.title }}]({{ site.baseurl }}{{ projen_doc.url }})
{% endfor %}