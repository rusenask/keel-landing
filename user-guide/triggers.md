--- 
layout: content 
title: "Triggers" 
subtitle: Triggers act as entrypoints into Keel, passively and actively (polling) checking for new images
description: "Available triggers in Keel"
---

Trigger is an event that has basic information such as _image name_ and _image tag_ (version).

- [Webhooks]({{ page.url }}#webhooks)
  * [Native Webhooks]({{ page.url }}#native-webhooks)
  * [DockerHub Webhooks]({{ page.url }}#dockerhub-webhooks)
  * [Quay Webhooks]({{ page.url }}#quay-webhooks)
  * [Receiving webhooks without public endpoint]({{ page.url }}#receiving-webhooks-without-public-endpoint)
- [Google Cloud GCR registry]({{ page.url }}#google-cloud-gcr-registry) 
- [Polling]({{ page.url }}#polling)

## Webhooks

Webhooks are "user-defined HTTP callbacks". They are usually triggered by some event, such as pushing image to a registry.

###  Native Webhooks

Native webhooks (simplified version) are accepted at `/v1/webhooks/native` endpoint with a payload that has __name__ and __tag__ fields: 

```
{
  "name": "gcr.io/v2-namespace/hello-world", 
  "tag": "1.1.1"
}
```

### DockerHub Webhooks

DockerHub uses webhooks to inform 3rd party systems about repository related events such as pushed image.

[DockerHub Webhooks](https://docs.docker.com/docker-hub/webhooks/) - go to your repository on 
`https://hub.docker.com/r/your-namespace/your-repository/~/settings/webhooks/` and point webhooks
to `/v1/webhooks/dockerhub` endpoint. Any number of repositories 
can send events to this endpoint.


### Quay Webhooks 

Documentation how to setup quay webhooks for __Repository Push__ events is available here: [https://docs.quay.io/guides/notifications.html](https://docs.quay.io/guides/notifications.html). These webhooks should be delivered to `/v1/webhooks/quay` endpoint. Any number of repositories 
can send events to this endpoint.


### Receiving webhooks without public endpoint

If you don't want to expose your Keel service - recommended solution is [https://webhookrelay.com/](https://webhookrelay.com/) which can deliver webhooks to your internal Keel service through a sidecar container.

Example sidecar container configuration for your `deployments.yaml`:

```
        - image: webhookrelay/webhookrelayd:0.6.0
          name: webhookrelayd
          command: ["/relayd"]
          env:                         
            - name: KEY
              valueFrom:
                secretKeyRef:
                  name: webhookrelay-credentials
                  key: key                
            - name: SECRET
              valueFrom:
                secretKeyRef:
                  name: webhookrelay-credentials
                  key: secret
            - name: BUCKET
              value: dockerhub      
```

## Google Cloud GCR registry

If you are using Google Container Engine with Container Registry - search no more, pubsub trigger is for you.

Since Keel requires access for the pubsub in GCE Kubernetes to work - your cluster node pools need to have permissions. If you are creating a new cluster - just enable pubsub from the start. If you have an existing cluster - currently the only way is to create a new node-pool through the gcloud CLI (more info in the [docs](https://cloud.google.com/sdk/gcloud/reference/container/node-pools/create?hl=en_US&_ga=1.2114551.650086469.1487625651)):

```
gcloud container node-pools create new-pool --cluster CLUSTER_NAME --scopes https://www.googleapis.com/auth/pubsub
``` 

Make sure that in the deployment.yaml you have set environment variables __PUBSUB=1__ and __PROJECT_ID=your-project-id__. 


### Polling

Since only the owners of docker registries can control webhooks - it's often convenient to use
polling. Be aware that registries can be rate limited so it's a good practice to set up reasonable polling intervals.
While configuration for each provider can be slightly different - each provider has to accept several polling parameters:

* Explicitly enable polling trigger
* Supply polling schedule (defaults to 1 minute intervals)

### Cron expression format

Custom polling schedule can be specified as cron format or through predefined schedules (recommended solution). 

Available schedules:


Entry                  | Description                                | Equivalent To
-----                  | -----------                                | -------------
@yearly (or @annually) | Run once a year, midnight, Jan. 1st        | `0 0 0 1 1 *`
@monthly               | Run once a month, midnight, first of month | `0 0 0 1 * *`
@weekly                | Run once a week, midnight on Sunday        | `0 0 0 * * 0`
@daily (or @midnight)  | Run once a day, midnight                   | `0 0 0 * * *`
@hourly                | Run once an hour, beginning of hour        | `0 0 * * * *`


#### Intervals

You may also schedule a job to execute at fixed intervals. This is supported by formatting the cron spec like this:

`@every <duration>`

where _duration_ is a string accepted by [time.ParseDuration](http://golang.org/pkg/time/#ParseDuration).

For example, _@every 1h30m10s_ would indicate a schedule that activates every 1 hour, 30 minutes, 10 seconds.


> **Tip**: If you want to disable polling support for your Keel installation - set environment variable
__POLL=0__.


<ul class="actions">
  <li><a href="/user-guide" class="button big">Next - using Keel</a></li>
</ul>

If you have any problems deploying or configuring Keel - [submit an issue](https://github.com/rusenask/keel/issues).