---
layout: post
title:  "HBase Encryption at Rest"
categories:
 - hbase
comments: true
---

This provides encryption of data at rest (HFile and WAL).

----
* [Quick overview](#quick_overview)
* [References](#references)
* [Configuration](#configuration)
* [Test](#test)

----

> Written, configured and tested with the help of *João Ascenso* and *Nagesh Yakkanti* during an [Hortonworks](https://hortonworks.com Hackaton Hackaton :).

## Quick overview

How it works, a 10s overview:

* A Master key for the cluster is stored in a key provider only accessible by HBase processes
* Encryption defined per ColumnFamily, 2 options:
  * key explicitly set by the user
  * random key, one per HFile will be created randomly and used (this is the recommended one)
* Encryption keys are wrapped using the Master key and stored in the ColumnFamily schema metadata and in each HFile of the
ColumnFamily
* Key rotation is possible for the master key and the encryption key
* If encryption is activated on a ColumnFamily with existing data, you can trigger a major compaction to
encrypt existing HFiles. New one will automatically be using encryption.
* Default key provider: Java keystore

## References

* [HBASE-7544](https://issues.apache.org/jira/browse/HBASE-7544)
* [HBase Book](http://hbase.apache.org/book.html#hbase.encryption.server)

## Configuration

* Generate the master key (used to wrap keys that are going to be used for encryption)
{% highlight bash %}
$ keytool -keystore /etc/hbase/conf/hbase.jks \
    -storetype jceks \
    -storepass password \
    -genseckey \
    -keyalg AES \
    -keysize 128 \
    -alias hbase   # you can pick another alias if you want
{% endhighlight %}

* Since the file contains the master key, set the appropriate permission
{% highlight bash %}
$ chown hbase:hadoop /etc/hbase/conf/hbase.jks
$ chmod 0600 /etc/hbase/conf/hbase.jks
{% endhighlight %}

* Add properties into `hbase-site.xml`
{% highlight xml %}
<property>
    <name>hbase.crypto.keyprovider</name>
    <value>org.apache.hadoop.hbase.io.crypto.KeyStoreKeyProvider</value>
</property>
<property>
    <name>hbase.crypto.keyprovider.parameters</name>
    <value>jceks:///etc/hbase/conf/hbase.jks?password=*****</value>
</property>
{% endhighlight %}

* Update the alias if you used a different one
{% highlight xml %}
<property>
    <name>hbase.crypto.master.key.name</name>
    <value>my-alias</value>
</property>
{% endhighlight %}

* Setup encryption for WAL
{% highlight xml %}
<property>
    <name>hbase.regionserver.hlog.reader.impl</name>
    <value>org.apache.hadoop.hbase.regionserver.wal.SecureProtobufLogReader</value>
</property>
<property>
    <name>hbase.regionserver.hlog.writer.impl</name>
    <value>org.apache.hadoop.hbase.regionserver.wal.SecureProtobufLogWriter</value>
</property>
<property>
    <name>hbase.regionserver.wal.encryption</name>
    <value>true</value>
</property>
{% endhighlight %}

* Since `hbase-site.xml` contains the master password, set the appropriate permissions to restrict access:
{% highlight bash %}
$ chmod 0400 /etc/hbase/conf/hbase-site.xml
# make sure this is own by hbase (or the user running the hbase service if
different)
$ chown hbase:hbase /etc/hbase/conf/hbase-site.xml
{% endhighlight %}
* restart HBase

## Test

* Create a table for testing as `hbase` user:
{% highlight bash %}
$ hbase shell
> create 'test_encryption', 'cf'
> put 'test_encryption', 'row-1', 'cf:q', 'value-1'
> put 'test_encryption', 'row-2', 'cf:q', 'value-2'
> put 'test_encryption', 'row-3', 'cf:q', 'value-3'
> major_compact 'test_encryption'
{% endhighlight %}

* One can check that the HFile is not encrypted
  * Locate an HFile for the table in HDFS (likely in `/apps/hbase/data/data/default/test_encryption`)
  * Explore properties using `hbase org.apache.hadoop.hbase.io.hfile.HFile` tool:
{% highlight bash %}
hbase org.apache.hadoop.hbase.io.hfile.HFile -m
/apps/hbase/data/data/default/test_encryption/86805efa5bd48a241b895325508a7
499/cf/efb53d6c290e4b1d845af0087d171659
...
encryptionKey=NONE,
...
{% endhighlight %}

  * You could also look at the raw content of the HFile
{% highlight bash %}
hdfs dfs -cat
/apps/hbase/data/data/default/test_encryption_2/f645ed7634aaf56f41a9d11285a
cdb9f/cf/d5e0a2880cc54b2082b591d09c943085 | strings
{% endhighlight %}

* Activate encryption on a column family
{% highlight bash %}
$ hbase shell
> alter 'test_encryption', {NAME => 'cf', 'ENCRYPTION' => 'AES' }
> major_compact
{% endhighlight %}

* Wait for compaction to finish and inspect the results as before
  * Locate an HFile
  * Dump the content
{% highlight bash %}
$ hbase org.apache.hadoop.hbase.io.hfile.HFile -m
/apps/hbase/data/data/default/test_encryption/...
...
encryptionKey=PRESENT,
...
$ hdfs dfs -cat /apps/hbase/data/data/default/test_encryption/…. | strings
# you shouldn’t see the information as before
{% endhighlight %}

A specific encryption key can also be provided:
{% highlight bash %}
> alter 'test_encryption', {NAME => 'cf', 'ENCRYPTION_KEY'=>'E4BCFCBA626BF',
'ENCRYPTION'=>'AES'}
> describe 'test_encryption_2'
DESCRIPTION
ENABLED
'test_encryption_2', {NAME => 'cf', ENCRYPTION => 'AES', DATA_BLOCK_ENCODING =>
'NONE', BLOOMFILTER => 'ROW', REPLI true
CATION_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL
=> 'FOREVER', KEEP_DELETED_CE
LLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true',
ENCRYPTION_KEY => '=\x0A\x03AES\x
10\x10\x1A\x10OG\x92\x17\x15\x919\xD4H]$\xA3da\x8E#"\x10\xF5nY^\xD0\x11~Ic\xD7\xE5\x1
8Jz0\xB6*\x10\x01[9(\xE7\xE1\x
CC\x02\xA6pCdG\x9B0G'}
{% endhighlight %}
