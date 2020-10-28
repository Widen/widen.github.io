---
title: Open Source Guidelines
author: Mark Feltner
published: true
permalink: /open-source
comments: false
layout: page
toc: true
---

# Community Guidelines

Anyone is welcome to contribute in a positive way to our open-source
repositories.

As we develop our own guidelines, please refer to the following as references
on how we expect contributors and maintainers to conduct themselves:

- [Contributor Covenant](https://www.contributor-covenant.org/)
- [Google's Community Guidelines]( https://opensource.google/conduct/ )

Keep in mind every commit message, comment, and issue are both public and
linked to the Widen brand.

# Project Guidelines

See [http://github.com/Widen/new-public-project](http://github.com/Widen/new-public-project)
for boilerplate for starting an effective open-source project.

## What can I release as open-source?

Your project shall not contain any proprietary information related to
the core business functionality of Widen.

- Examples of good things to open source:
    - Utilities
    - Integrations with our APIs
    - Code samples
    - Style guides

If you have any questions, please reach out to your manager or the Developer
Relations Committee for guidance.

## Licensing

Widen recommends the ISC. MIT and Apache are also viable. Talk with the [Widen Developer Relations Committee](mailto://engineering@widen.com) if you have any questions.

## Infrastructure

Projects should not depend on internal Widen infrastructure. Making something
that interacts with our public APIs and sites is fine, but a project that
launches EC2 instances in our cloud is not.

## Secrets

No secrets in code or commit messages. This includes commits from the past –
it might be worth erasing the entire commit history, or using an automated
tool to scrub past commits.
