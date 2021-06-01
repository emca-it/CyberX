# Upgrades #

 You can check the current version using the API command:
```bash
curl -u $USER:$PASSWORD -X GET http://localhost:9200/_license
```

## Upgrade from version 7.0.5

### General note

1. Indices *.agents, audit, alert* indices currently uses rollover for rotation, after upgrade please use dedicated API for migration: 

```bash
curl -u $USER:$PASSWORD -X POST http://localhost:9200/_logserver/prepareindex/$indexname
```

2. Wiki plugin require open port *tcp/5603*
3. Update alert role to include index-paths: *".alert", "alert_status", "alert_error", ".alertrules_", ".risks", ".riskcategories", ".playbooks"*

### Preferred Upgrade steps

1. Run upgrade script:

   ```bash
   ./install.sh -u
   ```

2. Restart services:

   ```bash
   systemctl restart elasticsearch alert kibana cerebro wiki
   ```

3. Migrate Audit index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/audit' -u logserver
   ```

4. Migrate Alert index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/alert' -u logserver
   ```

5. Migrate Agents index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/.agents' -u logserver
   ```

6. Open tcp/5603 port for wikipedia plugin:

   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   firewall-cmd --reload
   ```

### Alternative Upgrade steps (without install.sh script)

1. Stop services:

   ```bash
   systemctl stop elasticsearch alert kibana cerebro
   ```

2. Upgrade client-node (includes alert engine):

   ```bash
   yum update ./cyberx-client-node-7.0.6-1.el7.x86_64.rpm
   ```

3. Upgrade data-node:

   ```bash
   yum update ./cyberx-data-node-7.0.6-1.el7.x86_64.rpm
   ```

4. Start services:

   ```bash
   systemctl start elasticsearch alert kibana cerebro wiki 
   ```

5. Migrate Audit index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/audit' -u logserver
   ```

6. Migrate Alert index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/alert' -u logserver
   ```

7. Migrate Agents index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST '127.0.0.1:9200/_logserver/prepareindex/.agents' -u logserver
   ```

8. Open tcp/5603 port for wikipedia plugin:

   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   firewall-cmd --relod
   ```

## Upgrade from version 7.0.4

### General note

1. The following indices `.agents`, `audit`, `alert` currently uses rollover for rotation, after upgrade please use dedicated AIP for migration:

   ```bash
   curl -u $USER:$PASSWORD -X POST http://localhost:9200/_logserver/prepareindex/$indexname
   ```

2. The Wiki plugin require open port `tcp/5603`

### Preferred Upgrade steps

1. Run upgrade script:
   
   ```bash
   ./install.sh -u
   ```
   
2. Restart services:

   ```bash
    systemctl restart elasticsearch alert kibana cerebro wiki
   ```

3. Migrate Audit index to new format (the next call will display the current status of the task):

   ```bash
   curl -X POST 'http://localhost:9200/_logserver/prepareindex/audit' -u $USER:$PASSWORD
   ```

4. Migrate Alert index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/alert' -u $USER:$PASSWORD
   ```

5. Migrate Agents index to new format (the next call will display the current status of the task):

   ```bash
    curl -XPOST 'http://localhost:9200/_logserver/prepareindex/.agents' -u $USER:$PASSWORD
   ```

6. Open `tcp/5603` port for Wikipedia plugin:
   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   ```

   ```bash
   firewall-cmd --reload
   ```

### Alternative Upgrade steps (without install.sh script)

1. Stop services:
   
   ```bash
   systemctl stop elasticsearch alert kibana cerebro
   ```
   
2. Upgrade client-node (includes alert engine):
   
   ```bash
   yum update ./cyberx-client-node-7.0.5-1.el7.x86_64.rpm
   ```
   
3. Upgrade data-node:
   
   ```bash
   yum update ./cyberx-data-node-7.0.5-1.el7.x86_64.rpm
   ```
   
4. Start services:
   
   ```bash
   systemctl start elasticsearch alert kibana cerebro wiki
   ```
   
5. Migrate Audit index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/audit' -u $USER:$PASSWORD
   ```
   
6. Migrate Alert index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/alert' -u $USER:$PASSWORD
   ```
   
7. Migrate Agents index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/.agents' -u $USER:$PASSWORD
   ```
   
8. Open `tcp/5603` port for Wikipedia plugin:
   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   ```
   
   ```bash
   firewall-cmd --reload
   ```
## Upgrade from version 7.0.3

### General note

1. Indicators of compromise (IOCs auto-update) require access to the software provider's servers. 

2. GeoIP Databases (auto-update) require access to the software provider's servers.

3. Archive plugin require `ztsd` package to work: 

   ```bash
   yum install zstd
   ```
### Upgrade steps

1. Stop services   

   ```bash
   systemctl stop elasticsearch alert kibana cerebro
   ```

2. Upgrade client-node (includes alert engine):

   ```bash
   yum update ./cyberx-client-node-7.0.4-1.el7.x86_64.rpm
   ```
3. Upgrade data-node:   

   ```bash
   yum update ./cyberx-data-node-7.0.4-1.el7.x86_64.rpm
   ```

4. Start services:
   
   ```bash
   systemctl start elasticsearch alert kibana cerebro
   ```

## Upgrade from version 7.0.2

### General note

- Update the `kibana` role to include index-pattern `.kibana*`
- Update the `alert` role to include index-pattern `.alertrules*` and `alert_status*`
- Install `python36` which is required for the Alerting engine on client-node:
```bash 
yum install python3
```
- AD users should move their saved objects  from the `adrole`.
- Indicators of compromise (IOCs auto-update) require access to the software provider's servers.
- GeoIP Databases (auto-update) require access to the software provider's servers.

### Upgrade steps

- Stop services

```bash
systemctl stop elasticsearch alert kibana
```

- Upgrade client-node (includes alert engine)

```bash
yum update ./cyberx-client-node-7.0.3-1.el7.x86_64.rpm
```

- Login in the GUI CyberX and go to the `Alert List`  on the `Alerts` tab and click `SAVE` button

![](/media/media/image143.png)

- Start  `alert` and `kibana` service

```bash
systemctl start alert kibana
```

- Upgrade data-node

```bash
yum update ./cyberx-data-node-7.0.3-1.el7.x86_64.rpm
```

- Start services

```bash
systemctl start elasticsearch alert
```

**Extra note**

If the Elasticsearch service has been started on the client-node, then it is necessary to update the **client.rpm** and **data.rpm** packages on the client node. 

After update, you need to edit:

```bash
/etc/elasticsearch/elasticsearch.yml
```
and change:
```bash
node.data: false
```
Additionally, check the file:
```bash
elasticsearch.yml.rpmnew
```
and complete the configuration in `elasticsearch.yml` with additional lines.

## Upgrade from version 7.0.1

### General note

- Update the `kibana` role to include index-pattern `.kibana*`
- Update the `alert` role to include index-pattern `.alertrules*` and `alert_status*`
- Install `python36` which is required for the Alerting engine
```bash 
yum install python3 on client-node
```
- AD users should move their saved objects  from the `adrole`.
- Indicators of compromise (IOCs auto-update) require access to the software provider's servers.
- GeoIP Databases (auto-update) require access to the software provider's servers.

### Upgrade steps

- Stop services

```bash
systemctl stop elasticsearch alert kibana
```

- Upgrade client-node (includes alert engine)

```bash
yum update ./cyberx-client-node-7.0.2-1.el7.x86_64.rpm
```

- Login in the GUI CyberX and go to the `Alert List`  on the `Alerts` tab and click `SAVE` button

![](/media/media/image143.png)

- Start  `alert` and `kibana` service

```bash
systemctl start alert kibana
```

- Upgrade data-node

```bash
yum update ./cyberx-data-node-7.0.2-1.el7.x86_64.rpm
```

- Start services

```bash
systemctl start elasticsearch alert
```

**Extra note**

If the Elasticsearch service has been started on the client-node, then it is necessary to update the **client.rpm** and **data.rpm** packages on the client node. 

After update, you need to edit:

```bash
/etc/elasticsearch/elasticsearch.yml
```
and change:
```bash
node.data: false
```
Additionally, check the file:
```bash
elasticsearch.yml.rpmnew
```
and complete the configuration in `elasticsearch.yml` with additional lines.

## Upgrade from 6.x

Before upgrading to CyberX from 6.x  OpenJDK / Oracle JDK  version 11:

```bash
yum -y -q install java-11-openjdk-headless.x86_64
```

And select default command for OpenJDK /Oracle JDK:

```bash
alternatives --config java
```

The update includes packages:

- cyberx-data-node
- cyberx-client-node

### Pre-upgrade steps for data node

1. Stop the Logstash service

   ```bash
   systemctl stop logstash
   ```

2. Flush sync for indices

   ```bash
   curl -sS -X POST "localhost:9200/_flush/synced?pretty" -u$USER:$PASSWORD
   ```

3. Close all indexes with production data, except system indexes (the name starts with a dot `.`), example of query:

   ```bash
   for i in `curl -u$USER:$PASSWORD "localhost:9200/_cat/indices/winlogbeat*?h=i"` ; do curl -u$USER:$PASSWORD -X POST localhost:9200/$i/_close ; done
   ```

4. Disable shard allocation

   ```bash
   curl -u$USER:$PASSWORD -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": {"cluster.routing.allocation.enable": "none"}}'
   ```
   
1. Check Cluster Status

    ```bash
    export CREDENTIAL="logserver:logserver"
    
    curl -s -u $CREDENTIAL localhost:9200/_cluster/health?pretty
    ```

      Output:

    ```bash
    {
        "cluster_name" : "elasticsearch",
        "status" : "green",
        "timed_out" : false,
        "number_of_nodes" : 1,
        "number_of_data_nodes" : 1,
        "active_primary_shards" : 25,
        "active_shards" : 25,
        "relocating_shards" : 0,
        "initializing_shards" : 0,
        "unassigned_shards" : 0,
        "delayed_unassigned_shards" : 0,
        "number_of_pending_tasks" : 0,
        "number_of_in_flight_fetch" : 0,
        "task_max_waiting_in_queue_millis" : 0,
        "active_shards_percent_as_number" : 100.0
       }
    ```
    
6. Stop Elasticsearch service

    ```bash
    systemctl stop elasticsearch
    ```

### Upgrade CyberX Data Node

1. Upload Package

   ```bash
   scp ./cyberx-data-node-7.0.1-1.el7.x86_64.rpm root@hostname:~/
   ```

2. Upgrade CyberX Package

   ```bash
   yum update ./cyberx-data-node-7.0.1-1.el7.x86_64.rpm
   ```

   Output:

   ```bash
   Loaded plugins: fastestmirror
   Examining ./cyberx-data-node-7.0.1-1.el7.x86_64.rpm:       cyberx-data-node-7.0.1-1.el7.x86_64
   Marking ./cyberx-data-node-7.0.1-1.el7.x86_64.rpm as an    update to cyberx-data-node-6.1.8-1.x86_64
   Resolving Dependencies
   --> Running transaction check
   ---> Package cyberx-data-node.x86_64 0:6.1.8-1 will be    updated
   ---> Package cyberx-data-node.x86_64 0:7.0.1-1.el7 will be an update
   --> Finished Dependency Resolution

   Dependencies Resolved
   =======================================================================================================================================================================================
    Package                                        Arch                          Version                            Repository                                                          Size
   ========================================================================  ========================================================================  =======================================
   Updating:
    cyberx-data-node                     x86_64                        7.0.1-1.el7                        /cyberx-data-node-   7.0.1-1.el7.x86_64                     117 M

   Transaction Summary
   =======================================================================================================================================================================================
   Upgrade  1 Package

   Total size: 117 M
   Is this ok [y/d/N]: y
   Downloading packages:
   Running transaction check
   Running transaction test
   Transaction test succeeded
   Running transaction
     Updating   : cyberx-data-node-7.0.1-1.el7.x86_64                                                                                                                       1/2
   Removed symlink /etc/systemd/system/multi-   user.target.wants/elasticsearch.service.
   Created symlink from /etc/systemd/system/multi-   user.target.wants/elasticsearch.service to    /usr/lib/systemd/system/elasticsearch.service.
     Cleanup    : cyberx-data-node-6.1.8-1.x86_64                                                                                                                           2/2
     Verifying  : cyberx-data-node-7.0.1-1.el7.x86_64                                                                                                                       1/2
     Verifying  : cyberx-data-node-6.1.8-1.x86_64                                                                                                                           2/2

   Updated:
     cyberx-data-node.x86_64 0:7.0.1-1.el7

   Complete!
   ```

1. Verification of Configuration Files

    Please, verify your Elasticsearch configuration and JVM configuration in files:

        - /etc/elasticsearch/jvm.options – check JVM HEAP settings and another parameters

    ```bash
    grep Xm /etc/elasticsearch/jvm.options <- old configuration file
    ## -Xms4g
    ## -Xmx4g
    # Xms represents the initial size of total heap space
    # Xmx represents the maximum size of total heap space
    -Xms600m
    -Xmx600m
    ```

    ```bash
    cp /etc/elasticsearch/jvm.options.rpmnew /etc/elasticsearch/jvm.options
    cp: overwrite ‘/etc/elasticsearch/jvm.options’? y
    ```

    ```bash
    vim /etc/elasticsearch/jvm.options
    ```

    - /etc/elasticsearch/elasticsearch.yml – verify elasticsearch configuration file

    - compare exiting /etc/elasticsearch/elasticsearch.yml and /etc/elasticsearch/elasticsearch.yml.rpmnew
    
1. Start and enable Elasticsearch service
    If everything went correctly, we will restart the Elasticsearch instance:

    ```bash
    systemctl restart elasticsearch
    systemctl reenable elasticsearch
    ```
    ```bash
    systemctl status elasticsearch
    ● elasticsearch.service - Elasticsearch
       Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
       Active: active (running) since Wed 2020-03-18 16:50:15 CET; 57s ago
         Docs: http://www.elastic.co
     Main PID: 17195 (java)
       CGroup: /system.slice/elasticsearch.service
               └─17195 /etc/alternatives/jre/bin/java -Xms512m -Xmx512m -Djava.security.manager -Djava.security.policy=/usr/share/elasticsearch/plugins/elasticsearch_auth/plugin-securi...
    
    Mar 18 16:50:15 migration-01 systemd[1]: Started Elasticsearch.
    Mar 18 16:50:25 migration-01 elasticsearch[17195]: SSL not activated for http and/or transport.
    Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: Defaulting to no-operation (NOP) logger implementation
    Mar 18 16:50:33 migration-01 elasticsearch[17195]: SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
    ```
1. Check cluster/indices status and Elasticsearch version

    Invoke curl command to check the status of Elasticsearch:

    ```bash
    curl -s -u $CREDENTIAL localhost:9200/_cluster/health?pretty
    {
      "cluster_name" : "elasticsearch",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 25,
      "active_shards" : 25,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
    }
    ```

    ```bash
    curl -s -u $CREDENTIAL localhost:9200
    {
      "name" : "node-1",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "igrASEDRRamyQgy-zJRSfg",
      "version" : {
        "number" : "7.3.2",
        "build_flavor" : "oss",
        "build_type" : "rpm",
        "build_hash" : "1c1faf1",
        "build_date" : "2019-09-06T14:40:30.409026Z",
        "build_snapshot" : false,
        "lucene_version" : "8.1.0",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }
    ```

1. Install new version of default base template

```bash
curl -k -XPUT -H 'Content-Type: application/json' -u logserver:logserver 'http://127.0.0.1:9200/_template/default-base-template-0' -d@/usr/share/elasticsearch/default-base-template-0.json
```

If everything went correctly, we should see 100% allocated shards in cluster health. However, while connection on port 9200/TCP we  can observe a new version of Elasticsearch.

### Post-upgrade steps for data node

1. Start Elasticsearch service

   ```bash
   systemctl statrt elasticsearch
   ```

2. Delete .auth index

   ```bash
   curl -u$USER:$PASSWORD -X DELETE localhost:9200/.auth
   ```

4. Use `elasticdump` to get all templates and load it back

   - get templates

   ```bash
   /usr/share/kibana/elasticdump/elasticdump  --output=http://logserver:logserver@localhost:9200 --input=templates_elasticdump.json --type=template
   ```
   
   - delete templates
   
   ```bash
   for i in `curl -ss -ulogserver:logserver http://localhost:9200/_cat/templates | awk '{print $1}'`; do curl -ulogserver:logserver -XDELETE http://localhost:9200/_template/$i ; done
   ```

   - load templates

   ```bash
   /usr/share/kibana/elasticdump/elasticdump  --input=http://logserver:logserver@localhost:9200 --output=templates_elasticdump.json --type=template
   ```
   
4. Open indexes that were closed before the upgrade, example of query:

   ```bash
   curl -ss -u$USER:$PASSWORD "http://localhost:9200/_cat/indices/winlogbeat*?h=i,s&s=i" |awk '{if ($2 ~ /close/) system("curl -ss -u$USER:$PASSWORD -XPOST http://localhost:9200/"$1"/_open?pretty")}'
   ```

5. Start the Logstash service

   ```bash
   systemctl start logstash
   ```

6. Enable Elasticsearch allocation

   ```bash
   curl -sS -u$USER:$PASSWORD -X PUT "http://localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": {"cluster.routing.allocation.enable": "none"}}'
   ```

7. After starting on GUI remove aliases .kibana* (double version of index patterns)

   ```bash
   curl -u$USER:$PASSWORD "http://localhost:9200/.kibana_1/_alias/_all" -XDELETE
   ```

### Upgrade CyberX Client Node

1. Upload packages

    - Upload new rpm by scp/ftp:

    ```bash
    scp ./cyberx-client-node-7.0.1-1.el7.x86_64.rpm root@hostname:~/
    ```

    - Backup report logo file.

1. Uninstall old version CyberX GUI

    - Remove old package:

    ```bash
    systemctl stop kibana alert
    ```

    ```bash
    yum remove cyberx-client-node
    Loaded plugins: fastestmirror
    Resolving Dependencies
    --> Running transaction check
    ---> Package cyberx-client-node.x86_64 0:6.1.8-1 will be erased
    --> Finished Dependency Resolution

    Dependencies Resolved

    =======================================================================================================================================================================================
     Package                                           Arch                        Version                        Repository                                                          Size
    =======================================================================================================================================================================================
    Removing:
     cyberx-client-node                      x86_64                      6.1.8-1                        @/cyberx-client-node-6.1.8-1.x86_64                      802 M

    Transaction Summary
    =======================================================================================================================================================================================
    Remove  1 Package

    Installed size: 802 M
    Is this ok [y/N]: y
    Downloading packages:
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction
      Erasing    : cyberx-client-node-6.1.8-1.x86_64                                                                                                                         1/1
    warning: file /usr/share/kibana/plugins/node_modules.tar: remove failed: No such file or directory
    warning: /etc/kibana/kibana.yml saved as /etc/kibana/kibana.yml.rpmsave
      Verifying  : cyberx-client-node-6.1.8-1.x86_64                                                                                                                         1/1

    Removed:
      cyberx-client-node.x86_64 0:6.1.8-1

    Complete!
    ```

1. Install new version

    - Install dependencies:

      ```bash
      yum install net-tools mailx gtk3 libXScrnSaver ImageMagick ghostscript
      ```
      
    - Install new package:

      ```bash
      yum install ./cyberx-client-node-7.0.1-1.el7.x86_64.rpm
      Loaded plugins: fastestmirror
      Examining ./cyberx-client-node-7.0.1-1.el7.x86_64.rpm: cyberx-client-node-7.0.1-1.el7.x86_64
      Marking ./cyberx-client-node-7.0.1-1.el7.x86_64.rpm to be installed
      Resolving Dependencies
      --> Running transaction check
      ---> Package cyberx-client-node.x86_64 0:7.0.1-1.el7 will be installed
      --> Finished Dependency Resolution
    
      Dependencies Resolved
    
      =======================================================================================================================================================================================
       Package                                         Arch                      Version                           Repository                                                           Size
      =======================================================================================================================================================================================
      Installing:
       cyberx-client-node                    x86_64                    7.0.1-1.el7                       /cyberx-client-node-7.0.1-1.el7.x86_64                    1.2 G
    
      Transaction Summary
      =======================================================================================================================================================================================
      Install  1 Package
    
      Total size: 1.2 G
      Installed size: 1.2 G
      Is this ok [y/d/N]: y
      Downloading packages:
      Running transaction check
      Running transaction test
      Transaction test succeeded
      Running transaction
        Installing : cyberx-client-node-7.0.1-1.el7.x86_64                                                                                                                     1/1
      Generating a 2048 bit RSA private key
      ..............................................................................................+++
      ...........................................................................................................+++
      writing new private key to '/etc/kibana/ssl/kibana.key'
      -----
      Removed symlink /etc/systemd/system/multi-user.target.wants/alert.service.
      Created symlink from /etc/systemd/system/multi-user.target.wants/alert.service to /usr/lib/systemd/system/alert.service.
      Removed symlink /etc/systemd/system/multi-user.target.wants/kibana.service.
      Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /usr/lib/systemd/system/kibana.service.
      Removed symlink /etc/systemd/system/multi-user.target.wants/cerebro.service.
      Created symlink from /etc/systemd/system/multi-user.target.wants/cerebro.service to /usr/lib/systemd/system/cerebro.service.
        Verifying  : cyberx-client-node-7.0.1-1.el7.x86_64                                                                                                                     1/1
    
      Installed:
        cyberx-client-node.x86_64 0:7.0.1-1.el7
    
      Complete!
      ```

1. Start CyberX GUI
   
    Add service:
    - Kibana
    - Cerebro
    - Alert
    
    to autostart and add port ( 5602/TCP ) for Cerebro. 
    Run them and check status:
    
    ```bash
    firewall-cmd –permanent –add-port 5602/tcp
    firewall-cmd –reload
    ```


```bash
    systemctl enable kibana cerebro alert
    Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /usr/lib/systemd/system/kibana.service.
    Created symlink from /etc/systemd/system/multi-user.target.wants/cerebro.service to /usr/lib/systemd/system/cerebro.service.
    Created symlink from /etc/systemd/system/multi-user.target.wants/alert.service to /usr/lib/systemd/system/alert.service.
```

```bash
    systemctl start kibana cerebro alert
    systemctl status kibana cerebro alert
    ● kibana.service - Kibana
       Loaded: loaded (/usr/lib/systemd/system/kibana.service; enabled; vendor preset: disabled)
       Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
     Main PID: 12399 (node)
       CGroup: /system.slice/kibana.service
               └─12399 /usr/share/kibana/bin/../node/bin/node --no-warnings --max-http-header-size=65536 /usr/share/kibana/bin/../src/cli -c /etc/kibana/kibana.yml
    
    Mar 19 14:46:52 migration-01 systemd[1]: Started Kibana.
    
    ● cerebro.service - Cerebro
       Loaded: loaded (/usr/lib/systemd/system/cerebro.service; enabled; vendor preset: disabled)
       Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
     Main PID: 12400 (java)
       CGroup: /system.slice/cerebro.service
               └─12400 java -Duser.dir=/opt/cerebro -Dconfig.file=/opt/cerebro/conf/application.conf -cp -jar /opt/cerebro/lib/cerebro.cerebro-0.8.4-launcher.jar
    
    Mar 19 14:46:52 migration-01 systemd[1]: Started Cerebro.
    
    ● alert.service - Alert
       Loaded: loaded (/usr/lib/systemd/system/alert.service; enabled; vendor preset: disabled)
       Active: active (running) since Thu 2020-03-19 14:46:52 CET; 2s ago
     Main PID: 12401 (elastalert)
       CGroup: /system.slice/alert.service
               └─12401 /opt/alert/bin/python /opt/alert/bin/elastalert
    
    Mar 19 14:46:52 migration-01 systemd[1]: Started Alert.
```
## Changing OpenJDK version

### Logstash

OpenJDK 11 is supported by Logstash from version 6.8 so if you have an older version of Logstash you must update it.

To update Logstash, follow the steps below:

1. Back up the following files
	
    - /etc/logstash/logstash.yml 
    - /etc/logstash/piplines.yml
    - /etc/logstash/conf.d

2. Use the command to check custom Logstash plugins:

    ```bash
    /usr/share/bin/logstash-plugin list --verbose
    ```
    and note the result

3. Install a newer version of Logstash according to the instructions:

	https://www.elastic.co/guide/en/logstash/6.8/upgrading-logstash.html
	
	or 
	
	https://www.elastic.co/guide/en/logstash/current/upgrading-logstash.html

4. Verify installed plugins:

    ```bash
    /usr/share/bin/logstash-plugin list --verbose
    ```

5. Install the missing plugins if necessary:

    ```bash
    /usr/share/bin/logstash-plugin install plugin_name
    ```

6. Run Logstash using the command:

    ```bash
    systemctl start logstash
    ```

### Elasticearch

CyberX can use OpenJDK version 10 or later.
If you want to use OpenJSK version 10 or later, configure the Elasticsearch service as follows:

1. After installing OpenJDK, select the correct version that Elasticsearch will use:

    ```bash
    alternative --config java
    ```

2. Open the `/etc/elasticsearch/jvm.options` file in a text editor:

    ```bash
    vi /etc/elasticsearch/jvm.options
    ```

3. Disable the OpenJDK version 8 section:

    ```bash
    ## JDK 8 GC logging

    #8:-XX:+PrintGCDetails
    #8:-XX:+PrintGCDateStamps
    #8:-XX:+PrintTenuringDistribution
    #8:-XX:+PrintGCApplicationStoppedTime
    #8:-Xloggc:/var/log/elasticsearch/gc.log
    #8:-XX:+UseGCLogFileRotation
    #8:-XX:NumberOfGCLogFiles=32
    #8:-XX:GCLogFileSize=64m
    ```

4. Enable the OpenJDK version 11 section

    ```bash
    ## G1GC Configuration
    # NOTE: G1GC is only supported on JDK version 10 or later.
    # To use G1GC uncomment the lines below.
    10-:-XX:-UseConcMarkSweepGC
    10-:-XX:-UseCMSInitiatingOccupancyOnly
    10-:-XX:+UseG1GC
    10-:-XX:InitiatingHeapOccupancyPercent=75
    ```

5. Restart the Elasticsearch service

    ```bash
    systemctl restart elasticsearch
    ```