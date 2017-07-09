--- 
layout: content 
title: "Installing Keel" 
subtitle: Keel runs as a single container and is available on DockerHub
description: "Kubernetes service to automate deployment updates"
---

Keel can run anywhere and will do its job as long as it can connect to your Kubernetes environment. Since you have (or plan to have) Kubernetes environment, this guide will show you how to deploy Keel on Kubernetes cluster.

Keel installation consists of:

* [Namespace preparation]({{ page.url }}#creating-namespace)
* [Creating service account]({{ page.url }}#creating-service-account)
* [Deploying Keel]({{ page.url }}#deploying-keel)
* [Configuring Triggers]({{ page.url }}#configuring-triggers)

### Prerequisites

* Kubernetes environment (easiest way to get Kubernetes up and running is probably [Google Container Engine](https://cloud.google.com/container-engine/))
* [kubectl]((https://kubernetes.io/docs/user-guide/kubectl-overview/)) - Kubernetes client

Configuration sample files are available in Keel repository on GitHub [here](https://github.com/rusenask/keel/tree/master/hack).

## Creating namespace

It doesn't matter in which namespace you run Keel but it makes sense to separate your workloads.

Create a `namespace.yaml` file:

```
apiVersion: v1
kind: Namespace
metadata:
  name: keel
```

Now, use kubectl to create it:

```
kubectl create -f namespace.yaml
```  

## Creating service account

Keel will be updating deployments, so let's create a new [service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) in keel namespace:

```
kubectl create serviceaccount keel --namespace=keel
```

## Deploying Keel

Create a `deployment.yaml` and replace environment variables with your settings.
Available configuration options:

* __POLL__ - removing this environment variable will disable poll support (active image SHA digest checking in registry)
* __PUBSUB__  and __PROJECT_ID__ - environment variables to enable automated integration with [Google Container Registry](https://cloud.google.com/container-registry/). Set PROJECT_ID to your gcloud project ID. 
* __WEBHOOK_ENDPOINT__ - provide an endpoint where Keel should send notifications.
* __SLACK_TOKEN__ - provide Slack token where Keel should send notifications, more info on getting token in the [docs](https://get.slack.help/hc/en-us/articles/215770388-Create-and-regenerate-API-tokens)

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  annotations:
    kubernetes.io/service-account.name: keel
  name: keel
  namespace: keel
  labels: 
      name: "keel"
spec:
  replicas: 1
  template:
    metadata:
      name: keel
      labels:
        app: keel      
    spec:
      containers:                    
        - image: karolisr/keel:0.3.0
          imagePullPolicy: Always
          env:          
            - name: POLL
              value: "1"               
            - name: PUBSUB
              value: "1"
            - name: PROJECT_ID
              value: "my-project-id"
            # - name: WEBHOOK_ENDPOINT
            #   value: https://my.webhookrelay.com/v1/webhooks/2fc52b16-75f7-41f2-8e2d-81afbbcae7bq
            # - name: SLACK_TOKEN
            #   value: your-token-here
            # - name: SLACK_CHANNELS
            #   value: general            
          name: keel
          command: ["/bin/keel"]
          ports:
            - containerPort: 9300       
          livenessProbe:
            httpGet:
              path: /healthz
              port: 9300
            initialDelaySeconds: 30
            timeoutSeconds: 10
```

Once you are happy with env variables, create it:

```
kubectl create -f deployment.yaml
```

That's it, to check whether it successfully started - check pods:

```
kubectl -n keel get pods
```

If you have any problems deploying or configuring Keel - submit an issue in our [GitHub page](https://github.com/rusenask/keel/issues).

## Configuring Triggers

Trigger starts evaluation of your current deployments in order to find impacted containers that need an update. Trigger is an event that has basic information such as _image name_ and _image tag_ (version).

### PubSub trigger

If you are using Google Container Engine with Container Registry - search no more, pubsub trigger is for you.

Since Keel requires access for the pubsub in GCE Kubernetes to work - your cluster node pools need to have permissions. If you are creating a new cluster - just enable pubsub from the start. If you have an existing cluster - currently the only way is to create a new node-pool through the gcloud CLI (more info in the [docs](https://cloud.google.com/sdk/gcloud/reference/container/node-pools/create?hl=en_US&_ga=1.2114551.650086469.1487625651):

```
gcloud container node-pools create new-pool --cluster CLUSTER_NAME --scopes https://www.googleapis.com/auth/pubsub
``` 

Make sure that in the deployment.yaml you have set environment variables __PUBSUB=1__ and __PROJECT_ID=your-project-id__. 

###  Webhook Trigger

Keel supports two types of webhooks:

* [DockerHub Webhooks](https://docs.docker.com/docker-hub/webhooks/) - go to your repository on 
  `https://hub.docker.com/r/your-namespace/your-repository/~/settings/webhooks/` and point webhooks
  to `http://your-keel-address.com/v1/webhooks/dockerhub`. 
* Native webhooks (simplified version) - shoot webhooks at `http://your-keel-address.com/v1/webhooks/native` with a payload that has __name__ and __tag__ fields: `{"name": "gcr.io/v2-namespace/hello-world", "tag": "1.1.1"}`

If you don't want to expose your Keel service - recommended solution is [https://webhookrelay.com/](https://webhookrelay.com/) which can deliver webhooks to your internal Keel service through a sidecar container.


### Polling Trigger

Polling is currently not enabled by default. To enable polling support for your deployments - set environment variable
__POLL=1__. 

This will be enabled by default in future releases.

Since only the owners of docker registries can control webhooks - it's sometimes convenient to use
polling. Be aware that registries can be rate limited so it's a good practice to set up reasonable polling intervals. Find out more about setting up polling triggers for your deployments in user guide.


<ul class="actions">
  <li><a href="/user-guide" class="button big">Next - using Keel</a></li>
</ul>