---
layout: post
title: "mosquitto_sub with timestamp"
date:   2017-02-23 08:31:49
categories: ["iot", "mqtt", "oneliners"]
Description: Subscribe to Mosquitto MQTT messages with mosquitto_sub and append a timestamp.
---

[mosquitto_sub](https://mosquitto.org/man/mosquitto_sub-1.html) is an MQTT 
version 3.1 client for subscribing to topics.

It works pretty well except it does not support to display a timestamp of the events.

The following "simple" command subscribes to all topics of the mosquitto MQTT server and
executes the date command for each message.

{% highlight bash %}
mosquitto_sub -v -t /# | xargs -d$'\n' -L1 bash -c 'date "+%Y-%m-%d %T.%3N $0"'
{% endhighlight %}

Update:
Thanks to Stewart C. Russell, who suggested to use `ts` from `moreutils`
package, it can be achieved with a shorter expression:

{% highlight bash %}
mosquitto_sub -v -t /# | ts
{% endhighlight %}

