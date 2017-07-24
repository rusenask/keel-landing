--- 
layout: content 
title: "Providers" 
subtitle: providers are responsible for making sure that your applications with Keel policies get updated
description: "Providers are direct integrations with schedulers or other tools like Helm to update applications based on specified policies"
---

Providers are direct integrations into schedulers or other tools (ie: Helm). Providers are handling events created by triggers. Each provider can handle events in different ways, for example Kubernetes provider identifies impacted deployments and starts rolling update while Helm provider communicates with Tiller, identifies releases by Chart and then starts update. 

Available providers:

- [Kubernetes provider]({{ page.url }}#kubernetes)
  * [Deployment example]({{ page.url }}#deployment-example)
  * [Polling deployment example]({{ page.url }}#polling-deployment-example)
- [Helm provider]({{ page.url }}#helm)
  * [Helm configuration example]({{ page.url }}#helm-configuration-example)
  * [Helm configuration polling example]({{ page.url }}#helm-configuration-polling-example)

## Kubernetes

Kubernetes provider was the first, simplest provider added to Keel. Policies and trigger configuration for each application deployment is done through labels. 

Policies are specified through special label:

```
keel.sh/policy=all
```

A policy to update only minor releases:

```
keel.sh/policy=minor
```

### Deployment example

Here is an example application `deployment.yaml` where we instruct Keel to update container image whenever there is a new version:

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: wd
  namespace: default
  labels: 
      name: "wd"
      keel.sh/policy: all
spec:
  replicas: 1
  template:
    metadata:
      name: wd
      labels:
        app: wd        

    spec:
      containers:                    
        - image: karolisr/webhook-demo:0.0.2
          imagePullPolicy: Always            
          name: wd
          command: ["/bin/webhook-demo"]
          ports:
            - containerPort: 8090       
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8090
            initialDelaySeconds: 30
            timeoutSeconds: 10
          securityContext:
            privileged: true      
```

If Keel gets an event that `karolisr/webhook-demo:0.0.3` is available - it will update deployment and therefore start a rolling update.

### Polling deployment example

While the deployment above works perfect for both webhook and Google Cloud Pubsub triggers sometimes you can't control these events and the only available solution is to check registry yourself. This is where polling trigger comes to the rescue.

_Note: when image with non-semver style tag is supplied (ie: `latest`) Keel will monitor SHA digest. If tag is semver - it will track and notify providers when new versions are available._ 


Add labels:

```
keel.sh/policy: force # add this to enable updates of non-semver tags
keel.sh/trigger: poll
```

To specify custom polling schedule, check [cron expression format]({{ page.url }}#cron-expression-format)

_Note that even if polling trigger is set - webhooks or pubsub events can still trigger updates_

Example deployment file for polling:

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: wd
  namespace: default
  labels: 
      name: "wd"
      keel.sh/policy: force
      keel.sh/trigger: poll      
  annotations:
      keel.sh/pollSchedule: "@every 10m"
spec:
  replicas: 1
  template:
    metadata:
      name: wd
      labels:
        app: wd        

    spec:
      containers:
        - image: karolisr/webhook-demo:latest # this would start repository digest checks
          imagePullPolicy: Always            
          name: wd
          command: ["/bin/webhook-demo"]
          ports:
            - containerPort: 8090       
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8090
            initialDelaySeconds: 30
            timeoutSeconds: 10
          securityContext:
            privileged: true      
```            

## Helm 


Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application. More information can be found on project's website [https://helm.sh/](https://helm.sh/). 

Keel works directly with Tiller (a daemon that is used by Helm CLI) to manage release upgrades when new images are available. 

### Helm configuration example

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

### Helm configuration polling example

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

