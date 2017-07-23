--- 
layout: content 
title: "User guide" 
subtitle: your app specific update information is described through labels, annotations and values.yaml
description: "Using Keel"
---

Keel aims to be simple to use and operate in the background so users can focus on important things like
writing code, testing and admiring their creations.

Keel supports app specific policies, multiple providers, triggers and notification extensions. This guide should provide you a quick
overview of the system.

* [Policies]({{ page.url }}#policies)
* [Providers]({{ page.url }}#providers)
* [Triggers]({{ page.url }}#triggers)
* [Notifications]({{ page.url }}#notifications)

## Policies

Use policies to define when you want your application to be updated. Providers can have different mechanisms of getting configuration for your application, but policies are consistent across all of them. Following [semver](http://semver.org/) 
best practices allows you to safely automate application updates.


Available policies:

* __all__ - update whenever there is a version bump
* __major__ - update major & minor & patch versions (same as __all__)
* __minor__ - update only minor & patch versions (ignores major)
* __patch__ - update only patch versions (ignores minor and major versions)
* __force__ - force update even if tag is not semver, ie: `latest`

## Providers

Providers are direct integrations into schedulers or other tools (ie: Helm). Providers are handling events created by triggers. Each provider can handle events in different ways, for example Kubernetes provider identifies impacted deployments and starts rolling update while Helm provider communicates with Tiller, identifies releases by Chart and then starts update. 

Available providers:

- [Kubernetes provider]({{ page.url }}/kubernetes)
- [Helm provider]({{ page.url }}/helm)

While the goal is often the same, different providers can choose different update strategies.

## Triggers

Triggers are entry points into the Keel. Their task is to collect information regarding updated images and send events to providers.

Available triggers:

- [Webhooks]({{ page.url }}/triggers#webhooks)
  * [Native Webhooks]({{ page.url }}/triggers#native-webhooks)
  * [DockerHub Webhooks]({{ page.url }}/triggers#dockerhub-webhooks)
  * [Quay Webhooks]({{ page.url }}/triggers#quay-webhooks)
  * [Receiving webhooks without public endpoint]({{ page.url }}/triggers#receiving-webhooks-without-public-endpoint)
- [Google Cloud GCR registry]({{ page.url }}/triggers#google-cloud-gcr-registry) 
- [Polling]({{ page.url }}/triggers#polling)

## Notifications

Keel can send notifications on successful or failed deployment updates. 

Supported notification extensions:

- [Webhooks]({{ page.url }}/notifications#webhooks)
- [Slack]({{ page.url }}/notifications#slack)

