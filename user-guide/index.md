--- 
layout: content 
title: "User guide" 
subtitle: your app specific update information is described through labels, annotations and chart configuration
description: "Using Keel"
---

Keel aims to be simple to use and operate in the background so users can focus on important things like
writing code, testing and admiring their creations.

<!-- ![keel overview]({{ site.url }}/images/triggers-policies-providers.png){: .center-image } -->

<div class="image main"> <img src="/images/triggers-policies-providers.png" alt="" /></div>


Keel supports app specific policies, multiple providers, triggers and notification extensions. This guide should provide you a quick
overview of the system.

* [Policies]({{ page.url }}#policies)
* [Providers]({{ page.url }}#providers)
* [Triggers]({{ page.url }}#triggers)
* [Approvals]({{ page.url }}#approvals)
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

- [Kubernetes provider]({{ page.url }}/providers#kubernetes)
- [Helm provider]({{ page.url }}/providers#helm)

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

## Approvals

Users can specify on deployments and Helm charts how many approvals do they have to collect before a resource gets updated. Main features:

* __non-blocking__ - multiple deployments/helm releases can be queued for approvals, the ones without specified approvals will be auto updated.
* __extensible__ - current implementation focuses on Slack but additional approval collection mechanisms are trivial to implement.
* __out of the box Slack integration__ - the only needed Keel configuration is Slack auth token, Keel will start requesting approvals and users will be able to approve.
* __stateful__ - uses [github.com/rusenask/k8s-kv](https://github.com/rusenask/k8s-kv) for persistence so even after updating itself (restarting) it will retain existing info.
* __self cleaning__ - expired approvals will be removed after deadline is exceeded.

Details on how to configure your approvals:

- [Kubernetes provider]({{ page.url }}/approvals#kubernetes)
- [Helm provider]({{ page.url }}/approvals#helm)

## Notifications

Keel can send notifications on successful or failed deployment updates. 

Supported notification extensions:

- [Webhooks]({{ page.url }}/notifications#webhooks)
- [Slack]({{ page.url }}/notifications#slack)

