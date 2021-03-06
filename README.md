DAudit
==============

Identify security risks in your configurations for databases and big data platforms.

![Output](examples/output.png)


Installation
-----
```
$ git clone https://github.com/shouc/daudit.git && cd daudit
$ python3 -m pip install -r requirements.txt
```


Supported Softwares
-----
NoSQL DB:
* [Redis](#redis)
* [MongoDB](#mongodb)
* CouchDB [TODO]
* Cassandra [TODO]
* HBase [TODO]
* LevelDB [TODO]

Relational DB:
* [MySQL / MariaDB](#mysql--mariadb)
* Postgres [TODO]

Other DB:
* Neo4j [TODO]
* Elasticsearch [TODO]
* InfluxDB [TODO]

Big Data Platform:
* [Hadoop](#hadoop)
* [Spark](#spark)

Usage
-----

You can use the following command to print the help message
```
$ python3 main.py -h
==============================================
      _____                   _ _      
     (____ \   /\            | (_)_    
      _   \ \ /  \  _   _  _ | |_| |_  
     | |   | / /\ \| | | |/ || | |  _) 
     | |__/ / |__| | |_| ( (_| | | |__ 
     |_____/|______|\____|\____|_|\___)
      https://github.com/shouc/daudit
==============================================

usage: main.py [-h] {redis,mongodb,mysql,hadoop,spark} ...

This is a tool for detecting configuration issues of Redis, MySQL, etc!

positional arguments:
  {redis,mongodb,mysql,hadoop,spark}
                        commands
    redis               Check configurations of redis
    mongodb             Check configurations of mongodb
    mysql               Check configurations of mysql
    hadoop              Check configurations of hadoop
    spark               Check configurations of spark

optional arguments:
  -h, --help  show this help message and exit
```


Redis
-----
[Advisory](https://redis.io/topics/security)
```
$ python3 main.py redis -h
usage: main.py redis [-h] [--dir DIR]

optional arguments:
  -h, --help  show this help message and exit
  --dir DIR   the dir of redis configuration files, leave blank if you wish the program to automatically detect the location.
```

An example of checking Redis with configuration file /etc/redis/redis.conf
```
$ # both commands are equivalent
$ sudo python3 main.py redis
$ sudo python3 main.py redis --dir /etc/redis 
[INFO]    Evaluating /etc/redis/redis.conf
[INFO]    Checking exposure...
[DEBUG]   Redis is only exposed to internal network (127.0.0.1)
[DEBUG]   Redis is only exposed to internal network (::1)
[INFO]    Checking setting of password...
[ISSUE]   No password has been set. 
[RECOMM]  Consider setting requirepass [your_password] in configuration file(s) based on your context.
[INFO]    Checking commands...
[ISSUE]   config command is exposed to every user.
[RECOMM]  Consider setting rename-command config [UUID] in configuration file(s) based on your context.
[ISSUE]   debug command is exposed to every user.
[RECOMM]  Consider setting rename-command debug [UUID] in configuration file(s) based on your context.
[ISSUE]   shutdown command is exposed to every user.
[RECOMM]  Consider setting rename-command shutdown [UUID] in configuration file(s) based on your context.
[ISSUE]   flushdb command is exposed to every user.
[RECOMM]  Consider setting rename-command flushdb [UUID] in configuration file(s) based on your context.
[ISSUE]   flushall command is exposed to every user.
[RECOMM]  Consider setting rename-command flushall [UUID] in configuration file(s) based on your context.
[ISSUE]   eval command is exposed to every user.
[RECOMM]  Consider setting rename-command eval [UUID] in configuration file(s) based on your context.
```
Checks:
* exposure
* weak/no password
* command renaming

MongoDB
-----
[Advisory](https://docs.mongodb.com/manual/administration/security-checklist/)

```
$ python3 main.py mongodb -h
usage: main.py mongodb [-h] [--dir DIR] [--file FILE]

optional arguments:
  -h, --help   show this help message and exit
  --dir DIR    the dir of configuration files, leave blank if you wish the program to automatically detect it. (e.g. --dir /etc/)
  --file FILE  the name of the configuration file, leave blank if you wish the program to automatically detect it. (e.g. --file xxx.conf)
```

An example of checking MongoDB with configuration file /etc/mongodb.conf
```
$ # both commands are equivalent
$ sudo python3 main.py mongodb
$ sudo python3 main.py mongodb --dir '/etc' --file mongodb.conf 
[INFO]    Evaluating /etc/mongodb.conf
[DEBUG]   Using MongoDB <= 2.4 conf file format (INI)
[INFO]    Checking exposure...
[DEBUG]   The instance is only exposed on internal IP: 127.0.0.1
[INFO]    Checking setting of authentication...
[ISSUE]   No authorization is enabled in configuration file. 
[RECOMM]  Consider setting auth = true in configuration file(s) based on your context.
[INFO]    Checking code execution issue...
[ISSUE]   JS code execution is enabled in configuration file.
[RECOMM]  Consider setting noscripting = true in configuration file(s) based on your context.
[INFO]    Checking object check issue...
[ISSUE]   Object check is not enabled in configuration file.
[RECOMM]  Consider setting noscripting = true in configuration file(s) based on your context.

```
Checks:
* exposure
* authorization
* js code execution
* object check


MySQL / MariaDB
-----

```
$ python3 main.py mysql -h
usage: main.py mysql [-h] [--password PASSWORD] [--username USERNAME] [--host HOST] [--port PORT]

optional arguments:
  -h, --help           show this help message and exit
  --password PASSWORD  Password of root account []
  --username USERNAME  Username of root account [root]
  --host HOST          Username of root account [127.0.0.1]
  --port PORT          Port of MySQL server [3306]
```

An example of checking MySQL:
```
$ # both commands are equivalent
$ python3 main.py mysql
$ python3 main.py mysql --host "127.0.0.1" --port 3306 --username root --password ""
[INFO]    Checking authentication...
[WARNING] User na3 is exposed to the internet (0.0.0.0)
[INFO]    Would you like to perform weak-password check? This may create high traffic load for MySQL server. (i.e. Do not perform this when there is already high traffic.)
Type Y/y to perform this action and anything else to skip [Y]x
[WARNING] User root is exposed to the internet (0.0.0.0)
[INFO]    Would you like to perform weak-password check? This may create high traffic load for MySQL server. (i.e. Do not perform this when there is already high traffic.)
Type Y/y to perform this action and anything else to skip [Y]x
[WARNING] User shou is exposed to the internet (0.0.0.0)
[INFO]    Would you like to perform weak-password check? This may create high traffic load for MySQL server. (i.e. Do not perform this when there is already high traffic.)
Type Y/y to perform this action and anything else to skip [Y]y
[WARNING] Weak password 123 set by user shou with host %
[INFO]    Checking obsolete accounts...
[DEBUG]   Obsolete account 'test' is deleted
[DEBUG]   Obsolete account '' is deleted
[INFO]    Checking useless database...
[DEBUG]   All useless DBs are deleted
[INFO]    Checking load file func...
[DEBUG]   --secure-file-priv is enabled
[INFO]    Checking global grants...
[WARNING] Setting references_priv = N is for user shou with host %
...
[DEBUG]   Skipping privilege checking for root/internal account
[DEBUG]   Skipping privilege checking for root/internal account
[DEBUG]   Skipping privilege checking for root/internal account
[DEBUG]   Skipping privilege checking for root/internal account
[DEBUG]   Skipping privilege checking for root/internal account
[INFO]    Checking database grants...
[DEBUG]   Skipping database privilege checking for root/internal account
[DEBUG]   Skipping database privilege checking for root/internal account
```
Checks:
* authentication (exposure + weak password)
* obsolete accounts
* useless database
* load file func
* global grants
* db grants

Hadoop
-----

```
$ python3 main.py hadoop -h
usage: main.py hadoop [-h] [--dir]

optional arguments:
  -h, --help           show this help message and exit
  --dir DIR    the dir of configuration files, leave blank if you wish the program to automatically detect it. (e.g. --dir /etc/)
```

An example of checking Hadoop:
```
$ # both commands are equivalent
$ python3 main.py hadoop
$ python3 main.py hadoop --dir "/etc/hadoop/"
[INFO]    Evaluating directory /etc/hadoop/
[INFO]    /etc/hadoop/mapred-site.xml not found, skipped.
[INFO]    /etc/hadoop/yarn-site.xml not found, skipped.
[INFO]    Checking web portal cross origin policy
[DEBUG]   CORS is off
[INFO]    Checking SSL
[ISSUE]   SSL is disabled.
[RECOMM]  Consider setting hadoop.ssl.enabled = true in configuration file(s) based on your context.
[INFO]    Checking global access control
[ISSUE]   Everyone can access the instance
[RECOMM]  Consider setting hadoop.security.authentication = kerberos in configuration file(s) based on your context.
[ISSUE]   Authorization is not enabled
[RECOMM]  Consider setting hadoop.security.authorization = true in configuration file(s) based on your context.
[INFO]    Checking web portal access control
[ISSUE]   Everyone can access the web portal
[RECOMM]  Consider setting hadoop.http.authentication.type = kerberos in configuration file(s) based on your context.
[ISSUE]   Anonymous is allowed to access web portal.
[RECOMM]  Consider setting hadoop.http.authentication.simple.anonymous.allowed = false in configuration file(s) based on your context.
[INFO]    Checking registry access control
[DEBUG]   Registry is not enabled. 
[INFO]    Checking hdfs permission
[DEBUG]   HDFS permission system is enabled.
[ISSUE]   HDFS ACLs is not enabled.
[RECOMM]  Consider setting dfs.namenode.acls.enabled = true in configuration file(s) based on your context.
[INFO]    Checking export range
[ISSUE]   NFS is exposed to internet for read and write.
[RECOMM]  Consider setting  / qualify nfs.exports.allowed.hosts in configuration file(s) based on your context.
```
Checks:
* CORS policy
* SSL 
* global access control
* web portal access control
* registry access control
* hdfs access control
* hdfs exposure

Spark
-----
[Advisory](https://spark.apache.org/docs/latest/security.html)
```
$ python3 main.py spark -h
usage: main.py spark [-h] [--file FILE]

optional arguments:
  -h, --help   show this help message and exit
  --file FILE  the name of the configuration file, leave blank if you wish the program to automatically detect it. (e.g. --file /etc/xxx.conf)
```

An example of checking Spark:
```
$ # both commands are equivalent
$ python3 main.py spark
$ python3 main.py spark --file "/etc/spark/spark-defaults.conf"
[INFO]    Assuming no configuration property has been changed by CLI or SparkConf object
[INFO]    Evaluating file /etc/spark/spark-defaults.conf
[INFO]    Checking ACL
[ISSUE]   Access control not enabled for web portal
[RECOMM]  Consider setting spark.acls.enable = true in configuration file(s) based on your context.
[ISSUE]   Access control not enabled for history server
[RECOMM]  Consider setting spark.history.ui.acls.enable = true in configuration file(s) based on your context.
[INFO]    Checking XSS
[DEBUG]   XSS protection is enabled
[DEBUG]   CORB protection is enabled
[INFO]    Checking SSL
[ISSUE]   SSL is not enabled
[RECOMM]  Consider setting spark.ssl.enable = true in configuration file(s) based on your context.
[INFO]    Checking encryption
[ISSUE]   Network encryption is not enabled
[RECOMM]  Consider setting spark.network.crypto.enable = true in configuration file(s) based on your context.
[ISSUE]   Disk encryption is not enabled
[RECOMM]  Consider setting spark.io.encryption.enable = true in configuration file(s) based on your context.
[INFO]    Checking web ui authentication
[ISSUE]   Everyone can visit the instance
[RECOMM]  Consider setting spark.authenticate = true in configuration file(s) based on your context.
[INFO]    Checking logging
[ISSUE]   Logging is not enabled
[RECOMM]  Consider setting spark.eventLog.enabled = true in configuration file(s) based on your context.
```
Checks:
* ACL
* XSS protection 
* CORB protection 
* SSL
* network encryption
* disk encryption
* web ui authentication
* logging

