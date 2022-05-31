---
layout: post
title:  "Introduction to Projen - 1. What is it?"
categories: projen how-to
---

Projen is a new, important part of the development cycle for Service Victoria applications.

This document will act as a guide for anyone new to projen, github actions, or the deployment cycle for CDK apps.
It's primarily built from a Backend - Node - Lambda - Typescript kind of perspective, but you might find it helpful anyway if you're not in these groups :)

This guide is using [Projen Templates | Service Vic](https://github.com/service-victoria/projen-templates) v0.0.115, which was published in late May 2022. The flow may have changed since then.

## What is a `projen` anyway?

Projen is a static configuration generator, designed to help eliminate a lot of the hassle in creating and maintaining configuration files for projects. It's not just us, it's a project that has come out of the AWS CDK team, and they even use it for their own packages!

When I say "configuration", in this case I mean a number of things, and we will use a standard typescript backend app for example:

- TsConfig files
- Jest config files
- EsLint config files
- Prettier config files
- Mergify config files
- CI/CD config files (github actions, etc)
- Gitignore/npmignore files
- Your package.json file!

And probably way way more.

### There are a lot of static config generators, so why is this one special?

Most static config generators are designed to be run once at the start of a project, to create a template repository for you. After that, general config management is up to you.

Not with `projen` though! Projen is designed as a layer on top of other tools like `yarn` or `npm`, `jest`, `tsc`, `gulp` or whatever. It will manage everything for you with *Mostly Sensible Defaults*.

What's more, `projen` gives you a **typed typescript interface** to do all this generation! No more messing about with json files where you're sure to make a type at some point.

Furthermore, since `projen` is just a typescript interface, an organisation (such as Service Victoria) can issue its own template extensions which set up in-house defaults to provide some uniformity to the repositories it creates. Every time you say "I want to create a new repo", you can just pull the latest version of the SV template library, add a little bit of custom configuration, and you're off to the races! You know it will almost certainly deploy properly, and without issue.