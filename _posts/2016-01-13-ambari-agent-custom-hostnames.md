---
layout: post
title:  "Custom hostnames with Ambari Agent"
categories:
  - ambari
comments: true
---

This is a [neat feature](http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.0.0/bk_ambari_reference_guide/content/_how_to_customize_the_name_of_a_host.html) of Ambari that allows you to control the
hostname with which an Ambari Agent registers to the Ambari Server as well as controlling its known
public hostname.

This can be in particular useful when you need to exactly control on which interface the Ambari and
Hadoop services need to be configured.

