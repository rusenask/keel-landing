--- 
layout: content 
title: "Installing with Helm" 
subtitle: Keel runs as a single container and can upgrade itself when new version is available
description: "Installing Keel on Kubernetes with Helm"
---

Helm chart is available on [KubeApps](https://kubeapps.com/charts/stable/keel) stable charts and   [Github here](https://github.com/rusenask/keel/tree/master/chart/keel).

Keel installation with Helm:

* [Installing the Chart with Kubernetes provider support]({{ page.url }}#installing-the-chart-with-kubernetes-provider-support)
* [Installing the Chart with Helm provider support]({{ page.url }}#installing-the-chart-with-helm-provider-support)
* [Setting up Helm release to be automatically updated by Keel]({{ page.url }}#setting-up-helm-release-to-be-automatically-updated-by-keel)
* [Uninstalling the chart]({{ page.url }}#uninstalling-the-chart)
* [Configuration]({{ page.url }}#configuration)

## Installing the Chart with Kubernetes provider support

Docker image _polling_ and _Kubernetes_ provider are set by default, then Kubernetes _deployments_ can be upgraded when new Docker image is available:

```console
helm repo up
helm upgrade --install keel stable/keel
```

## Installing the Chart with Helm provider support

Docker image _polling_ is set by default, but we need to enable _Helm provider_ support, then Helm _releases_ can be upgraded when new Docker image is available:

```console
helm upgrade --install keel stable/keel --set helmProvider.enabled="true"
```

### Setting up Helm release to be automatically updated by Keel

Add the following to your app's `values.yaml` file and do `helm upgrade ...`:

```
keel:
  # keel policy (all/major/minor/patch/force)
  policy: all
  # trigger type, defaults to events such as pubsub, webhooks
  trigger: poll
  # polling schedule
  pollSchedule: "@every 3m"
  # images to track and update
  images:
    - repository: image.repository # it must be the same names as your app's values
      tag: image.tag # it must be the same names as your app's values
```

The same can be applied with `--set` flag without using `values.yaml` file:

```
helm upgrade --install whd webhookdemo --reuse-values \
  --set keel.policy="all",keel.trigger="poll",keel.pollSchedule="@every 3m" \
  --set keel.images[0].repository="image.repository" \
  --set keel.images[0].tag="image.tag"
```

You can read in more details about supported policies, triggers and etc in the [User Guide](https://keel.sh/user-guide/).

Also you should check the [Webhooh demo app](https://github.com/webhookrelay/webhook-demo) and it's chart to have more clear
idea how to set automatic updates.


## Uninstalling the Chart

To uninstall/delete the `keel` deployment:

```console
$ helm delete keel
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists has the main configurable parameters (polling, triggers, notifications, service) of the _Keel_ chart and they apply to both Kubernetes and Helm providers:

| Parameter                         | Description                            | Default                                                   |
| --------------------------------- | -------------------------------------- | --------------------------------------------------------- |
| polling.enabled                   | Docker registries polling              | `true`                                                    |
| helmProvider.enabled              | Enable/disable Helm provider           | `false`                                                   |
| gcr.enabled                       | Enable/disable GCR Registry            | `false`                                                   |
| gcr.projectID                     | GCP Project ID GCR belongs to          |                                                           |
| gcr.pubsub.enabled                | Enable/disable GCP Pub/Sub trigger     | `false`                                                   |
| webhook.enabled                   | Enable/disable Webhook Notification    | `false`                                                   |
| webhook.endpoint                  | Remote webhook endpoint                |                                                           |
| slack.enabled                     | Enable/disable Slack Notification      | `false`                                                   |
| slack.token                       | Slack token                            |                                                           |
| slack.channel                     | Slack channel                          |                                                           |
| service.enable                    | Enable/disable Keel service            | `false`                                                   |
| service.type                      | Keel service type                      | `LoadBalancer`                                            |
| service.externalPort              | Keel service port                      | `9300`                                                    |
| webhookRelay.enabled              | Enable/disable WebhookRelay integration| `false`                                                   |
| webhookRelay.key                  | WebhookRelay key                       |                                                           |
| webhookRelay.secret               | WebhookRelay secret                    |                                                           |
| webhookRelay.bucket               | WebhookRelay bucket                    |                                                           |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
helm install --name keel -f values.yaml stable/keel
```
> **Tip**: You can use the default [values.yaml](values.yaml)


<ul class="actions">
  <li><a href="/user-guide" class="button big">Next - using Keel</a></li>
</ul>

If you have any problems deploying or configuring Keel - [raise an issue](https://github.com/rusenask/keel/issues).