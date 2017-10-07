--- 
layout: content 
title: "Approvals" 
subtitle: Configure your deployments & charts 
description: "Configuring approvals for your updates"
---

Keel allows to postpone deployment or Helm chart updates until they are manually approved. This allows to have a semi automated workflows where user still has to make a final decision whether to update a resource or not. 

Configuration is slightly different for Kubernetes and Helm providers.

<!-- TOC -->

- [Enabling approvals](#enabling-approvals)
- [Configuring via Kubernetes deployments](#configuring-via-kubernetes-deployments)
- [Configuring via Helm charts](#configuring-via-helm-charts)
- [Approving through Slack example](#approving-through-slack-example)
- [HTTP endpoint](#http-endpoint)

<!-- /TOC -->

## Enabling approvals

Approvals are enabled by default but currently there is only one way to approve/reject updates:
Slack - commands like:

* ```keel get approvals``` - get all pending/approved/rejected approvals
* ```keel approve <identifier>``` - approve specified request.
* ```keel reject <identifier>``` - reject specified request.

Make sure you have set `export SLACK_TOKEN=<your slack token here>` environment variable for Keel deployment.

## Configuring via Kubernetes deployments

The only required configuration for Kubernetes deployment to enable approvals is to add `keel.sh/approvals: "1"` with a number (string! as the underlying type is map[string]string) of required approvals.

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
      keel.sh/trigger: poll      
      keel.sh/approvals: "1"
```

## Configuring via Helm charts

To enable approvals for a Helm chart update Keel config section in `values.yaml` with a required number of approvals:

```
replicaCount: 1
image:
  repository: karolisr/webhook-demo
  tag: "0.0.13"
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
  pollSchedule: "@every 1m"
  # approvals required to proceed with an update
  approvals: 1
  # images to track and update
  images:
    - repository: image.repository
      tag: image.tag
```

## Approving through Slack example

Keel will send notifications to your Slack group about pending approvals. Approval process is as simple as replying to Keel:

- Approve: `keel approve default/whr:0.4.12`
- Reject it: `keel reject default/whr:0.4.12`

Example conversation:

<div class="image main"> <img src="/images/approvals.png" alt="" /></div>

## HTTP endpoint

You can also view pending/rejected/approved update request on `http://localhost:9300/v1/approvals`Keel endpoint (make sure you have service exported). Example response:

```json
[
	{
		"provider": "helm",
		"identifier": "default/wd:0.0.15",
		"event": {
			"repository": {
				"host": "",
				"name": "index.docker.io/karolisr/webhook-demo",
				"tag": "0.0.15",
				"digest": ""
			},
			"createdAt": "0001-01-01T00:00:00Z",
			"triggerName": "poll"
		},
		"message": "New image is available for release default/wd (0.0.13 -> 0.0.15).",
		"currentVersion": "0.0.13",
		"newVersion": "0.0.15",
		"votesRequired": 1,
		"deadline": "2017-09-26T09:14:54.979211563+01:00",
		"createdAt": "2017-09-26T09:14:54.980936804+01:00",
		"updatedAt": "2017-09-26T09:14:54.980936824+01:00"
	}
]
```