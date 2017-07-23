--- 
layout: content 
title: "Installing Keel" 
subtitle: Keel runs as a single container and is available on DockerHub
description: "Installing Keel on Kubernetes"
---

Keel can run anywhere and will do its job as long as it can connect to your Kubernetes environment. Since you have (or plan to have) Kubernetes environment, this guide will show you how to deploy Keel inside your Kubernetes cluster.


Keel installation consists of:

<!-- * [Namespace preparation]({{ page.url }}#creating-namespace) -->
* [Creating service account]({{ page.url }}#creating-service-account)
* [Creating deployment]({{ page.url }}#creating-deployment)

### Prerequisites

* Kubernetes environment (easiest way to get Kubernetes up and running is probably [Google Container Engine](https://cloud.google.com/container-engine/))
* [kubectl]((https://kubernetes.io/docs/user-guide/kubectl-overview/)) - Kubernetes client

Configuration sample files are available in Keel repository on GitHub [here](https://github.com/rusenask/keel/tree/master/hack).

## Creating service account

Keel will be updating deployments, so let's create a new [service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) in keel namespace:

```
kubectl create serviceaccount keel --namespace=kube-system
```

## Create deployment

Create a `deployment.yaml` and replace environment variables with your settings.
Available configuration options:

* __POLL__ - removing this environment variable will disable poll support.
* __PUBSUB__  and __PROJECT_ID__ - environment variables to enable automated integration with [Google Container Registry](https://cloud.google.com/container-registry/). Set PROJECT_ID to your google cloud project ID. 
* __WEBHOOK_ENDPOINT__ - provide an endpoint where Keel should send notifications.
* __SLACK_TOKEN__ - provide Slack token where Keel should send notifications, more info on getting token in the [docs](https://get.slack.help/hc/en-us/articles/215770388-Create-and-regenerate-API-tokens).
* __HELM_PROVIDER__ - enable [Helm](https://helm.sh/) provider. 

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  annotations:
    kubernetes.io/service-account.name: keel
  name: keel
  namespace: kube-system
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
        - image: karolisr/keel:0.4.0
          imagePullPolicy: Always
          env:          
            - name: POLL
              value: "1"               
            - name: PUBSUB
              value: "1"            
            - name: PROJECT_ID
              value: "my-project-id"
            # Enable/disable Helm provider  
            # - name: HELM_PROVIDER
            #   value: "1" 
            # - name: WEBHOOK_ENDPOINT
            #   value: https://my.webhookrelay.com/v1/webhooks/2fc52b16-75f7-41f2-8e2d-81afbbcae709  
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
kubectl -n kube-system get pods
```

You should see something like this:

```
$ kubectl -n kube-system get pods
NAME                    READY     STATUS    RESTARTS   AGE
keel-2732121452-k7sjc   1/1       Running   0          14s
```

<ul class="actions">
  <li><a href="/user-guide" class="button big">Next - using Keel</a></li>
</ul>

If you have any problems deploying or configuring Keel - [raise an issue](https://github.com/rusenask/keel/issues).