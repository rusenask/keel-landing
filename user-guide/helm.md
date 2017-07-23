--- 
layout: content 
title: "Helm guide" 
subtitle: Keel integrates directly with Helm's Tiller reusing existing releases, incrementing revisions
description: "Using Keel"
---


Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application. More information can be found on project's website [https://helm.sh/](https://helm.sh/). 

Keel works directly with Tiller (a daemon that is used by Helm CLI) to manage release upgrades when new images are available. 

## Helm configuration example

Keel is configured through your chart's `values.yaml` file.

Here is an example application `values.yaml` file where we instruct Keel to track and update specific values whenever there is a new version:

```
replicaCount: 1
image:
  repository: karolisr/webhook-demo
  tag: "0.0.8"
  pullPolicy: IfNotPresent
service:
  name: webhookdemo
  type: ClusterIP
  externalPort: 8090
  internalPort: 8090

keel:
  # keel policy (all/major/minor/patch/force)
  policy: all
  # images to track and update
  images:
    - repository: image.repository # [1]
      tag: image.tag  # [2]
```

If Keel gets an event that `karolisr/webhook-demo:0.0.9` is available - it will upgrade release values so Helm can start updating your application.

* [1]  resolves during runtime image.repository -> karolisr/webhook-demo
* [2]  resolves during runtime image.tag -> 0.0.8

## Helm polling example

This example demonstrates Keel configuration for polling.

_Note that even if polling trigger is set - webhooks or pubsub events can still trigger updates_

```
replicaCount: 1
image:
  repository: karolisr/webhook-demo
  tag: "0.0.8"
  pullPolicy: IfNotPresent
service:
  name: webhookdemo
  type: ClusterIP
  externalPort: 8090
  internalPort: 8090

keel:
  # keel policy (all/major/minor/patch/force)
  policy: all
  # trigger type, defaults to events such as pubsub, webhooks
  trigger: poll
  # polling schedule
  pollSchedule: "@every 2m"
  # images to track and update
  images:
    - repository: image.repository # [1]
      tag: image.tag  # [2]
```

