--- 
layout: content 
title: "User guide" 
subtitle: your app specific update information is described in deployment.yaml labels and annotations, not Keel
description: "Using Keel"
---

Keel is stateless, background service, it cannot store information about your workload update strategies, instead it leverages Kubernetes [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) and [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) to understand how to update your applications. Keel has several different types of triggers and notifications, this page is an overview of these features. 

* [Policies]({{ page.url }}#policies)
* [Deployment example]({{ page.url }}#deployment-example)
* [Polling deployment example]({{ page.url }}#polling-deployment-example)
  - [Cron expression format]({{ page.url }}#cron-expression-format)
* [Notifications]({{ page.url }}#notifications)
  - [Webhook notifications]({{ page.url }}#webhook-notifications)
  - [Slack notifications]({{ page.url }}#slack-notifications)

## Policies

Use policies to define when you want your application updated:

* __all__ - update whenever there is a version bump
* __major__ - update major versions (same as __all__)
* __minor__ - update only minor versions (ignores major)
* __patch__ - update only patch versions (ignores minor and major versions)
* __force__ - force update even if tag is not semver, ie: `latest`

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

_Note: currently Keel only supports watching the same image tag when polling trigger is used, ie: `latest`. Support for semver during polling is coming soon._ 


Add labels:

```
keel.sh/policy: force
keel.sh/trigger: poll
```

To specify custom polling schedule, check [cron expression format]({{ page.url }}#cron-expression-format)

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
        - image: karolisr/webhook-demo
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


### Cron expression format

Custom polling schedule can be specified as cron format or through predefined schedules (recommended solution):

```
keel.sh/pollSchedule=@daily
```

#### Available schedules

```
Entry                  | Description                                | Equivalent To
-----                  | -----------                                | -------------
@yearly (or @annually) | Run once a year, midnight, Jan. 1st        | 0 0 0 1 1 *
@monthly               | Run once a month, midnight, first of month | 0 0 0 1 * *
@weekly                | Run once a week, midnight on Sunday        | 0 0 0 * * 0
@daily (or @midnight)  | Run once a day, midnight                   | 0 0 0 * * *
@hourly                | Run once an hour, beginning of hour        | 0 0 * * * *
```

#### Intervals

You may also schedule a job to execute at fixed intervals. This is supported by formatting the cron spec like this:

`@every <duration>`

where "duration" is a string accepted by [time.ParseDuration](http://golang.org/pkg/time/#ParseDuration).

For example, "@every 1h30m10s" would indicate a schedule that activates every 1 hour, 30 minutes, 10 seconds.

```
keel.sh/pollSchedule=@every 1h30m10s
```

## Notifications

Keel can send notifications on successful or failed deployment updates.  There are two types of notifications - trusted webhooks or Slack messages.

### Webhook notifications

To enabled webhook notifications provide an endpoint via __WEBHOOK_ENDPOINT__ environment variable inside Keel deployment. 

Webhook payload sample:

```
{
	"name": "update deployment",
	"message": "Successfully updated deployment default/wd (karolisr/webhook-demo:0.0.10)",
	"createdAt": "2017-07-08T10:08:45.226565869+01:00"	
}
```

### Slack notifications

First, get a Slack token, info about that can be found in the [docs](https://get.slack.help/hc/en-us/articles/215770388-Create-and-regenerate-API-tokens). Then, provide token via __SLACK_TOKEN__ environment variable. You should also provide __SLACK_CHANNELS__ environment variable with a comma separated list of channels where these notifications should go.

Keel will be sending messages when deployment updates succeed or fail.