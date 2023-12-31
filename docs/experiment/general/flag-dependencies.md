---
title: Flag Dependencies
description: An overview of flag dependencies, and how it is used to implement mutual exclusion and holdout groups.
---

Flag dependencies define relationships between flags to ensure evaluation order. The result of each flag's evaluation is then passed to all subsequent evaluations to decide if dependent flags should [evaluate](./evaluation/implementation.md#flag-dependencies) based on the result of the dependency.

Flag dependencies are currently used to implement mutual exclusion and holdout groups.

## Mutual exclusion groups

!!!info "Learn more about [mutual exclusion group creation, management, and best practices](https://help.amplitude.com/hc/en-us/articles/360061270712-Set-up-and-run-mutually-exclusive-experiments) at Amplitude's help center."

![Diagram of a mutual exclusion group with three slots and three experiments. One experiment in each slot.](../../assets/images/experiment/mutex-group.drawio.svg)

A mutual exclusion group ensures that, on evaluation, at most one of the experiments within the group is assigned. In Amplitude Experiment, a mutual exclusion group defines multiple slots, each with a percent of traffic allocated to that slot. The mutual exclusion group is actually just a flag with a variant for each slot. Experiments in the group add a dependency on one or more slots (variants) of the mutual exclusion group flag.

The variant result of a mutual exclusion group's evaluation isn't returned and not assigned as a user property.

## Holdout groups

!!!info "Learn more about [holdout group creation, management, and best practices](https://help.amplitude.com/hc/en-us/articles/13508918823835) at Amplitude's help center."

![Diagram of a holdout exclusion group with 10% of traffic held out and three experiments.](../../assets/images/experiment/holdout-group.drawio.svg)

A holdout group withholds a percentage of traffic from a group of experiments, allowing measurement of the long-term and combined impact of multiple experiments. In Amplitude Experiment, a holdout group is implemented using a flag with two variants: `holdout` and `on`, where the `holdout` variant is allocated the holdout percentage defined on creation. Experiments in the group depend on the holdout group's variant `on`.

The variant result of a holdout group's evaluation isn't returned but is assigned as a user property to enable holdout analysis.

## Local evaluation support

Flag dependencies (mutual exclusion and holdout groups) is only supported after certain version of SDKs. 

!!!warning "Older local evaluation SDK versions"
    Prior local evaluation SDK versions don't consider mutual exclusion or holdout groups at all. In other words, **two experiments in a mutual exclusion group evaluated with an old local evaluation SDK version aren't mutually exclusive**.

| SDK | Local Evaluation Flag Dependencies Support |
| --- | --- |
| [:material-nodejs: Node.js](../sdks/nodejs-sdk.md) | `1.4.0+` |
| [:material-language-ruby: Ruby](../sdks/ruby-sdk.md) | `1.1.0+` |
| [:material-language-java: JVM](../sdks/jvm-sdk.md) | `1.1.0+` |
| [:fontawesome-brands-golang: Go](../sdks/go-sdk.md) | `1.1.0+` |
| [:material-language-python: Python](../sdks/python-sdk.md) | `1.1.0+` |
