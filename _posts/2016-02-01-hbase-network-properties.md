---
layout: post
title:  "HBase network properties"
categories:
 - hbase
comments: true
---

In a *'non-standard'* network environment the following HBase properties may be
helpfull to tune on which interface, port the Master and Region Servers are
listening as well as configuring which hostname are being
registered/advertised to clients.

Service | Property
------- | ---------
HBase Master IPC | `hbase.master.ipc.address`
HBase Master IPC port | `hbase.master.port`
HBase Master UI | `hbase.master.info.bindAddress`
HBase Master UI port | `hbase.master.info.port`
HBase Master Hostname ([HBASE-12954]) |	`hbase.master.hostname`
HBase Region Server IPC | `hbase.regionserver.ipc.address`
HBase Region Server IPC port| `hbase.regionserver.port`
HBase Region Server UI | `hbase.regionserver.info.bindAddress`
HBase Region Server UI port | `hbase.regionserver.info.port`
HBase Region Server Hostname ([HBASE-12954]) | `hbase.regionserver.hostname`

[HBASE-12954]: https://issues.apache.org/jira/browse/HBASE-12954
