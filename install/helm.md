--- 
layout: content 
title: "Installing with Helm" 
subtitle: Keel runs as a single container and is available on DockerHub
description: "Installing Keel on Kubernetes with Helm"
---

Helm chart is available on [Github here](https://github.com/rusenask/keel/tree/master/chart/keel). There are several parameters that need configuration. Chart creates service account deploys Keel to `kube-system` namespace so Helm provider can be used (requires access to Tiller).

## Install for the first time

Docker image _polling_ is set by default, we also enabling _Helm provider_ support, so Helm releases
can be upgraded when new Docker image is available:

```console
helm upgrade --install keel keel --set helmProvider.enabled="true"
```


## Run upgrades e.g. for docker image change
```console
helm upgrade keel keel --reuse-values --set image.tag="0.4.0"
```

<ul class="actions">
  <li><a href="/user-guide" class="button big">Next - using Keel</a></li>
</ul>

If you have any problems deploying or configuring Keel - [raise an issue](https://github.com/rusenask/keel/issues).