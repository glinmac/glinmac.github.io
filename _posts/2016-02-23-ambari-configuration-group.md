---
layout: post
title:  "Ambari & Configuration group"
categories:
 - ambari
 - kafka
comments: true
---

Configuration groups are a great feature of Ambari to override some default
properties on a subset of nodes in your cluster. This can be done through the [Ambari UI]
but can also be automated and driven through the [Ambari API].

Let's take the example of the Kafka service and the `listeners` property.

## Example

By default, the `listeners` property in Ambari is being set to:

```
listeners: PLAINTEXT://localhost:6667
```

As documented, Ambari replaces automatically `localhost` with the canonical hostname of your server.

Let's imagine in our use case, we need to tune this on each broker to be another hostname and that we don't want
to run it on all available interfaces on the server (`0.0.0.0`)
(*eg* you have a management interface, private interface, public interface, ...)

One solution is to create a configuration group for each broker that will override the `listeners`
property. The configuration will still inherit all the default (log retention, ...)
but will only override the interface on which the Kafka broker is listening.

Assuming the service is already installed and default configuration is available, the following JSON
would create a new configuration group for `node1-a.example.com`

## Create a configuration group

This can be achieved by a `POST` request to the cluster's `config_groups` endpoint:

{% highlight shell %}
$ curl -u admin:admin \
    -H 'X-Requested-By: ambari' \
    -d '
[
  {
    "ConfigGroup": {
       "cluster_name": "hadoop_cluster",
       "group_name": "node1-kafka-broker",
       "tag": "KAFKA",
       "description": "Kafka host configuration",
       "hosts": [
          {
             "host_name": "node1-m.example.com"
          }
       ],
       "desired_configs": [
          {
             "type": "kafka-broker",
             "tag": "kbnode1",
             "properties": {
                "listeners": "PLAINTEXT://node1-a.example.com"
             }
          }
       ]
    }
  }
]
' \
    -X POST http://localhost:8080/api/v1/clusters/hadoop_cluster/config_groups
{% endhighlight %}

*et voila*!

## Restart of service

A restart of the Kafka service on the node will restart the broker on the correct interface:

{% highlight shell %}
$ curl -u admin:admin \
    -H 'X-Requested-By: ambari' \
    -d '
{
  "RequestInfo":{
    "command":"RESTART",
    "context":"Restart Kafka node1-m.example.com",
    "operation_level":{
      "level":"HOST",
      "cluster_name":"hadoop_cluster"
    }
  },
  "Requests/resource_filters": [
    {
      "service_name":"KAFKA",
      "component_name":"KAFKA_BROKER",
      "hosts":"node1-m.example.com"
    }
  ]
}' \
    -X POST http://localhost:8080/api/v1/clusters/hadoop_cluster/requests
{% endhighlight %}

You can then wrap this up in some scripts and automatically apply/deploy to other servers
if needed.


[Ambari UI]: https://docs.hortonworks.com/HDPDocuments/Ambari-2.2.0.0/bk_Ambari_Users_Guide/content/_using_host_config_groups.html
[Ambari API]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/config-groups.md
