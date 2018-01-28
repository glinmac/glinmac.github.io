---
layout: post
title:  "Ambari & Widgets"
categories:
  - ambari
comments: true
---

The Ambari UI provides a list of [widgets] for each services and can be customized.
This can be achieved using the widget editor in the UI but can also be achieved through
the Ambari API, allowing to fully automate and deploy new widgets and widget layouts.

This posts gives some notes/hints about using the API to manage widgets in Ambari and a small Python
helper tool can be found [here](https://github.com/glinmac/hdp-tools/tree/master/ambari/widgets).

---
* [Managing widgets](#managing-widgets)
  * [Create a new widget](#create-a-new-widget)
  * [Get a widget](#get-a-widget)
  * [Update a widget](#update-a-widget)
  * [Delete a widget](#delete-a-widget)
* [Managing widget layouts](#managing-widget-layouts)
  * [List of layouts](#list-of-layouts)
  * [Adding a widget to a layout](#adding-a-widget-to-a-layout)

---

## Managing widgets

### Create a new widget

Widgets are defined in JSON. For instance, the NameNode Heap widget has the following definition:

{% highlight json %}
{
  "WidgetInfo" : {
    "author" : "admin",
    "cluster_name" : "hadoop_cluster",
    "widget_name" : "NameNode Heap",
    "description" : "Heap memory committed and Heap memory used with respect to time.",
    "metrics" : [
      {
        "service_name":"HDFS",
        "component_name":"NAMENODE",
        "name":"jvm.JvmMetrics.MemHeapUsedM._avg",
        "metric_path":"metrics/jvm/memHeapUsedM._avg"
      },
      {
        "service_name":"HDFS",
        "component_name":"NAMENODE",
        "name":"jvm.JvmMetrics.MemHeapCommittedM._avg",
        "metric_path":"metrics/jvm/memHeapCommittedM._avg"
      }
    ],
    "properties" : {
      "graph_type":"LINE",
      "display_unit":"MB"
    },
    "scope" : "USER",
    "values" : [
      {
        "name":"jvm.JvmMetrics.MemHeapUsedM",
        "value":"${jvm.JvmMetrics.MemHeapUsedM._avg}"
      },
      {
        "name":"jvm.JvmMetrics.MemHeapCommittedM",
        "value":"${jvm.JvmMetrics.MemHeapCommittedM._avg}"
      }
    ],
    "widget_type" : "GRAPH"
  }
}
{% endhighlight %}

To create a widget, simply post the widget data to the widget API endpoint `/api/v1/clusters/${CLUSTER}/widgets`

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i \
   -X POST \
   -d @widget.json
   http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets

HTTP/1.1 201 Created
...
{% endhighlight %}

The response will contain the ID of the widget created:
{% highlight json %}
{
  "resources" : [
    {
      "href" : "http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets/252",
      "WidgetInfo" : {
        "id" : 252
      }
    }
  ]
}
{% endhighlight %}

### Get a widget

This is simply done by a `GET` request to the widgets endpoint:

* All widgets: `http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets`
* Specific widget: `http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets/252`

### Update a widget

This follows the same process than creation but with a `PUT` request with the new widget definition:

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i \
    -X PUT \
    -d @widget.json
    http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets/252
{% endhighlight %}

### Delete a widget

This can be deleted by sending a `DELETE` request:

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i \
    -X DELETE \
    http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets
{% endhighlight %}

## Managing widget layouts

The widget layouts describes the specific widgets visible for a given service.

### List of layouts

The list of available layouts can be retrieved using a `GET` call to the widget layouts endpoint `/api/v1/clusters/${CLUSTER}/widget_layouts`, eg:

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i  \
    http://localhost:8080/api/v1/clusters/hadoop_cluster/widget_layouts
HTTP/1.1 200 OK
...
{% endhighlight %}

{% highlight json %}
{
  "href" : "http://localhost:8080/api/v1/clusters/hadoop_cluster/widget_layouts",
  "items" : [
    {
      "href" : "http://localhost:8080/api/v1/clusters/hadoop_cluster/widget_layouts/8",
      "WidgetLayoutInfo" : {
        "cluster_name" : "hadoop_cluster",
        "display_name" : "Standard HDFS Dashboard",
        "id" : 8,
        "layout_name" : "admin_hdfs_dashboard",
        "scope" : "USER",
        "section_name" : "HDFS_SUMMARY",
        "user_name" : "admin",
        "widgets" : [
          {
            "href" : "http://localhost:8080/api/v1/clusters/hadoop_cluster/widgets/20",
            "WidgetInfo" : {
              "id" : 20,

...
{% endhighlight %}

You can also use a filter to look for a specific widget given one of its property. For example, filtering by tthe layout's name:

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i  \
    http://localhost:8080/api/v1/clusters/hadoop_cluster/widget_layouts?WidgetLayoutInfo/layout_name=admin_hdfs_dashboard
{% endhighlight %}

### Adding a widget to a layout

Users can map manually a widget to their preferred dashboard or this could also be automated so that they are available by default.

To add a widget to a given layout, this is a matter of updating the list of widget IDs associated with the widget layout.

The data to update:
{% highlight json %}
{
  "WidgetLayoutInfo":
    {
        "display_name":"Standard HDFS Dashboard",
        "id":8,
        "layout_name":"admin_hdfs_dashboard",
        "scope":"USER",
        "section_name":"HDFS_SUMMARY",
        "widgets":
            [{"id":20},{"id":23},{"id":22},{"id":24},{"id":21},{"id":153},{"id":152},{"id":25},{"id":26},{"id":27},{"id":28},{"id":29},{"id":252}]
    }
}
{% endhighlight %}

The update request is a `PUT` request to the specific widget layout endpoint: `/api/v1/clusters/${CLUSTER}/widget_layouts/${LAYOUT_ID}`:

{% highlight bash %}
curl -u admin:admin -H 'X-Requested-By: ambari' -i  \
    -X PUT \
    -d @widget_layout.json \
    http://localhost:8080/api/v1/clusters/hadoop_cluster/widget_layouts/8
{% endhighlight %}

The same can be used to remove or add more widgets to a layout.


[widgets]: https://cwiki.apache.org/confluence/display/AMBARI/Enhanced+Service+Dashboard
