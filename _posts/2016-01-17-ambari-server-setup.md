---
layout: post
title:  "Non-Interactive Ambari Server setup"
categories:
 - ambari
---

Automation tools like Salt, Ansible, Puppet, ... are great to deploy automatically
Ambari server and agents to the different nodes in your Hadoop cluster.

One step in installing Ambari is to run the setup command, traditionally documented with:

{% highlight shell %}
ambari-server setup
{% endhighlight %}

This requires manual input. However, it can be scripted by providing all the details on the
command line, allowing integration and automation with your preferred automation tool.

In particular, details of an external database can be provided, *eg* for PostgreSQL:

{% highlight shell %}
ambari-server setup \
	-j /YOUR/JAVA/PATH \
	--database=postgres \
 	--databasehost=mypostgresdb.example.com \
  	--databaseport=5432 \
  	--databasename=ambari \
  	--postgresschema=ambari_schema \
  	--databaseusername=ambari \
  	--databasepassword=secretpassword \
	--jdbc-driver=<PATH_TO_JDBC_JAR> \
	--jdbc-db=<JDBC_DATABASE_TYPE>
{% endhighlight %}

Unfortunately [AMBARI-6704] prevents a successful setup if you specify an external database, you'll
have to deploy/add this patch to your deployment strategy before running the setup phase.

There are more options than documented ([Ambari 2.2.0.0]) that could be helpfull to you:

{% highlight shell %}
sage: ambari-server.py [options] action [stack_id os]

Options:
  -h, --help            show this help message and exit
  -f INIT_SCRIPT_FILE, --init-script-file=INIT_SCRIPT_FILE
                        File with setup script
  -r DROP_SCRIPT_FILE, --drop-script-file=DROP_SCRIPT_FILE
                        File with drop script
  -u UPGRADE_SCRIPT_FILE, --upgrade-script-file=UPGRADE_SCRIPT_FILE
                        File with upgrade script
  -t UPGRADE_STACK_SCRIPT_FILE, --upgrade-stack-script-file=UPGRADE_STACK_SCRIPT_FILE
                        File with stack upgrade script
  -j JAVA_HOME, --java-home=JAVA_HOME
                        Use specified java_home.  Must be valid on all hosts
  -v, --verbose         Print verbose status messages
  -s, --silent          Silently accepts default prompt values
  -g, --debug           Start ambari-server in debug mode
  --all                 LDAP sync all Ambari users and groups
  --existing            LDAP sync existing Ambari users and groups only
  --users=LDAP_SYNC_USERS
                        Specifies the path to the LDAP sync users CSV file.
  --groups=LDAP_SYNC_GROUPS
                        Specifies the path to the LDAP sync groups CSV file.
  --database=DBMS       Database to use embedded|oracle|mysql|postgres
  --databasehost=DATABASE_HOST
                        Hostname of database server
  --databaseport=DATABASE_PORT
                        Database port
  --databasename=DATABASE_NAME
                        Database/Service name or ServiceID
  --postgresschema=POSTGRES_SCHEMA
                        Postgres database schema name
  --databaseusername=DATABASE_USERNAME
                        Database user login
  --databasepassword=DATABASE_PASSWORD
                        Database user password
  --sidorsname=SID_OR_SNAME
                        Oracle database identifier type, Service ID/Service
                        Name sid|sname
  --jdbc-driver=JDBC_DRIVER
                        Specifies the path to the JDBC driver JAR file for the
                        database type specified with the --jdbc-db option.
                        Used only with --jdbc-db option.
  --jdbc-db=JDBC_DB     Specifies the database type [postgres|mysql|oracle]
                        for the JDBC driver specified with the --jdbc-driver
                        option. Used only with --jdbc-driver option.
{% endhighlight %}

[Ambari 2.2.0.0]: http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.0.0/bk_Installing_HDP_AMB/content/_setup_options.html
[AMBARI-6704]: https://issues.apache.org/jira/browse/AMBARI-6704
