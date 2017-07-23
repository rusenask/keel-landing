--- 
layout: content 
title: "Kubernetes" 
subtitle: provider that allows Keel to communicate directly with Kubernetes via official API
description: "Using Keel"
---

Kubernetes provider was the first, simplest provider added to Keel. Policies and trigger configuration for each application deployment is done through labels. 

Policies are specified through special label:

```
keel.sh/policy=all
```

A policy to update only minor releases:

```
keel.sh/policy=minor
```

## Deployment example

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

## Polling deployment example

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