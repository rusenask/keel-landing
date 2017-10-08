--- 
layout: content 
title: "FAQ" 
subtitle: Keel extracts configuration from your cluster and does it's best to reduce complexity
description: "frequently asked questions"
---

Here are some frequently asked questions and answers:

__Wait, does Keel monitor my git repository for changes?__

No, Keel is only interested in your Docker image repositories like DockerHub, Quay, etc... Once the image update is detected via webhooks, polling or GCR pubsub Keel updates affected resources.

__How does Keel know what repositories it should monitor?__

Keel scans your Kubernetes environment, checking deployments and Helm charts looking for Keel configuration. In Kubernetes provider case it's looking for a label `keel.sh/policy=minor` where value can be `all/major/minor/patch`. And if you are using Helm - it expects to find Keel config in values.yaml of your chart, more info here: [{{ site.url }}/user-guide/providers/#helm]({{ site.url }}/user-guide/providers/#helm)

__If I have a private repository, how does Keel get credentials to authenticate to it?__

Keel uses the same pod secrets that Kubernetes uses to pull the image so no additional configuration is required. 

__Can Keel monitor Helm chart updates?__

No, Keel monitors only the image, if the chart is updated there is no way for Keel to know it. There were some requests to implement this feature so Keel could add support for this in the future.

__Can I still have a final say whether to approve/reject updates?__

Yes. You can specify how many approvals an update needs and vote via Slack, more info here: [{{ site.url }}/user-guide/approvals]({{ site.url }}/user-guide/approvals)

__Don't you know I could just do it manually instead? Kubectl apply -f is so easy bleah bleah__ 

Ok!

__How can I help?__

Check Keel's issues list and feel free to contribute. Donations are more than welcome on [Patreon](https://www.patreon.com/keel) or [Paypal](https://www.paypal.me/keelhq).