# Upgrades #

 You can check the current version using the API command:
```bash
curl -u $USER:$PASSWORD -X GET http://localhost:9200/license
```

## Upgrade from version 7.0.3

1. Stop services   

   ```bash
   systemctl stop elasticsearch alert kibana cerebro
   ```

2. Upgrade client-node (includes alert engine):

   ```bash
   yum update ./PACKAGE_NAME_VARIABLE-client-node-VERSION_TEMPLATE_VARIABLE-1.el7.x86_64.rpm
   ```
3. Upgrade data-node:   

   ```bash
   yum update ./PACKAGE_NAME_VARIABLE-data-node-VERSION_TEMPLATE_VARIABLE-1.el7.x86_64.rpm
   ```

4. Start services:
   
   ```bash
   systemctl start elasticsearch alert kibana cerebro
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