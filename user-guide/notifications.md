--- 
layout: content 
title: "Notifications" 
subtitle: Keel supports several notification providers such as webhooks & Slack
description: "Using Keel"
---

Keel has several different types of notifications, this page will guide you how to configure and consume these events.

* [Notifications]({{ page.url }}#notifications)
  - [Webhook notifications]({{ page.url }}#webhook-notifications)
  - [Slack notifications]({{ page.url }}#slack-notifications)

## Notifications

Keel can send notifications on successful or failed deployment updates.  There are two types of notifications - trusted webhooks or Slack messages.

Notification types:

__Pre-deployment update__ - fired before doing update. Can be used to drain running tasks.

```
{
	"name":"preparing to update deployment",
	"message":"Preparing to update deployment <your deployment namespace/name> (gcr.io/webhookrelay/webhook-demo:0.0.10)",
	"createdAt":"2017-07-23T23:51:46.478440258+01:00",
	"type":"preparing deployment update",
	"level":"LevelDebug"
}
```

__Successful deployment update__ - fired after successful update.

```
{
	"name":"deployment update",
	"message":"Successfully updated deployment <your deployment namespace/name> (gcr.io/webhookrelay/webhook-demo:0.0.10)",
	"createdAt":"2017-07-23T23:51:46.478440258+01:00",
	"type":"deployment update",
	"level":"LevelSuccess"
}
```

__Failed deployment update__ - fired after failed event.

```
{
	"name":"deployment update",
	"message":"Deployment <your deployment namespace/name> (gcr.io/webhookrelay/webhook-demo:0.0.10) update failed, error: <error here> ", 
	"createdAt":"2017-07-23T23:51:46.478440258+01:00",
	"type":"deployment update",
	"level":"LevelError"
}
```

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

First, get a Slack token, info about that can be found in the [docs](https://get.slack.help/hc/en-us/articles/215770388-Create-and-regenerate-API-tokens). Then, provide token via __SLACK_TOKEN__ environment variable. You should also provide __SLACK_CHANNELS__ environment variable with a comma separated list of channels where these notifications should be delivered to.

Keel will be sending messages when deployment updates succeed or fail.