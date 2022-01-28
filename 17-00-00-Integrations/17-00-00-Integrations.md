# Integrations #



## OP5 - Naemon logs ##

### Logstash ###

1. In CyberX `naemon_beat.conf` set up `ELASTICSEARCH_HOST`, `ES_PORT`, `FILEBEAT_PORT`

2. Copy CyberX `naemon_beat.conf` to `/etc/logstash/conf.d`

3. Based on "FILEBEAT_PORT" if firewall is running:

   ```bash
   sudo firewall-cmd --zone=public --permanent --add-port=FILEBEAT_PORT/tcp
   sudo firewall-cmd --reload
   ```

   

4. Based on amount of data that elasticsearch will receive you can also choose whether you want index creation to be based on moths or days:

   ```bash
   index => "CyberX-naemon-%{+YYYY.MM}"
   or
   index => "CyberX-naemon-%{+YYYY.MM.dd}"
   ```

   

5. Copy `naemon` file to `/etc/logstash/patterns` and make sure it is readable by logstash process
5. Restart *logstash* configuration e.g.:

   ```bash
   sudo systemct restart logstash
   ```

   

   ### Elasticsearch
1. Connect to Elasticsearch node via SSH and Install index pattern for naemon logs. Note that if you have a default pattern covering *settings* section you should delete/modify that in naemon_template.sh:

   ```bash
   "settings": {
       "number_of_shards": 5,
       "auto_expand_replicas": "0-1"
     },
   ```

   

2. Install template by running:
   `./naemon_template.sh`
### CyberX Monitor ###

1. On CyberX Monitor host install filebeat (for instance via rpm `https://www.elastic.co/downloads/beats/filebeat`)
1. In `/etc/filebeat/filebeat.yml` add:

		#=========================== Filebeat inputs =============================
		filebeat.config.inputs:
		  enabled: true
		  path: configs/*.yml

1. You also will have to configure the output section in `filebeat.yml`. You should have one logstash output:

		#----------------------------- Logstash output --------------------------------
		output.logstash:
		  # The Logstash hosts
		  hosts: ["LOGSTASH_IP:FILEBEAT_PORT"]

	If you have few logstash instances - `Logstash` section has to be repeated on every node and `hosts:` should point to all of them:

		hosts: ["LOGSTASH_IP:FILEBEAT_PORT", "LOGSTASH_IP:FILEBEAT_PORT", "LOGSTASH_IP:FILEBEAT_PORT" ]

1. Create `/etc/filebeat/configs` catalog.
1. Copy `naemon_logs.yml` to a newly created catalog.
1. Check the newly added configuration and connection to logstash. Location of executable might vary based on os:

		/usr/share/filebeat/bin/filebeat --path.config /etc/filebeat/ test config
		/usr/share/filebeat/bin/filebeat --path.config /etc/filebeat/ test output

1. Restart filebeat:

		sudo systemctl restart filebeat # RHEL/CentOS 7
		sudo service filebeat restart # RHEL/CentOS 6

### Elasticsearch ###

At this moment there should be a new index on the Elasticsearch node:

	curl -XGET '127.0.0.1:9200/_cat/indices?v'

Example output:

		health status index                 uuid                   pri rep docs.count docs.deleted store.size pri.store.size
		green  open   CyberX-naemon-2018.11    gO8XRsHiTNm63nI_RVCy8w   1   0      23176            0      8.3mb          8.3mb

If the index has been created, in order to browse and visualise the data, "index pattern" needs to be added in Kibana.

## OP5 - Performance data ##

Below instruction requires that between CyberX node and Elasticsearch node is working Logstash instance.

### Elasticsearch ###
1.	First, settings section in *CyberXtemplate.sh* should be adjusted, either:
	- there is a default template present on Elasticsearch that already covers shards and replicas then settings sections should be removed from the *CyberXtemplate.sh* before executing
	- there is no default template - shards and replicas should be adjusted for you environment (keep in mind replicas can be added later, while changing shards count on existing index requires 
		reindexing it)

			"settings": {
			  "number_of_shards": 5,
			  "number_of_replicas": 0
			}

1. In URL *CyberXperfdata* is a name for the template - later it can be search for or modify with it.

1. The "*template*" is an index pattern. New indices matching it will have the settings and mapping applied automatically (change it if you index name for *CyberX perfdata* is different).

1. Mapping name should match documents type:

    ```
    "mappings": {
    	  "CyberXperflogs"
    ```

    Running CyberXtemplate.sh will create a template (not index) for CyberX perf data documents.

### Logstash ###

1.	The *CyberXperflogs.conf* contains example of *input/filter/output* configuration. It has to be copied to */etc/logstash/conf.d/*. Make sure that the *logstash* has permissions to read the configuration files:
	
	```bash
	chmod 664 /etc/logstash/conf.d/CyberXperflogs.conf
	```
	
	
	
2. In the input section comment/uncomment *“beats”* or *“tcp”* depending on preference (beats if *Filebeat* will be used and tcp if *NetCat*). The port and the type has to be adjusted as well:

   ```bash
   port => PORT_NUMBER
   type => "CyberXperflogs"
   ```

   

3. In a filter section type has to be changed if needed to match the input section and Elasticsearch mapping.

4. In an output section type should match with the rest of a *config*. host should point to your elasticsearch node. index name should correspond with what has been set in elasticsearch template to allow mapping application. The date for index rotation in its name is recommended and depending on the amount of data expecting to be transferred should be set to daily (+YYYY.MM.dd) or monthly (+YYYY.MM) rotation:

   ```bash
   hosts => ["127.0.0.1:9200"]
   index => "CyberX-perflogs-%{+YYYY.MM.dd}"
   ```

5. Port has to be opened on a firewall:

   ```bash
   sudo firewall-cmd --zone=public --permanent --add-port=PORT_NUMBER/tcp
   sudo firewall-cmd --reload
   ```

6. Logstash has to be reloaded:

   ```bash
   sudo systemctl restart logstash
   ```

   or

   ```bash
   sudo kill -1 LOGSTASH_PID
   ```

### CyberX Monitor ###

1. You have to decide wether FileBeat or NetCat will be used. In case of Filebeat - skip to the second step. Otherwise:

   - Comment line:

     ```bash
     54    open(my $logFileHandler, '>>', $hostPerfLogs) or die "Could not open $hostPerfLogs"; #FileBeat
     •	Uncomment lines:
     55 #    open(my $logFileHandler, '>', $hostPerfLogs) or die "Could not open $hostPerfLogs"; #NetCat
     ...
     88 #    my $logstashIP = "LOGSTASH_IP";
     89 #    my $logstashPORT = "LOGSTASH_PORT";
     90 #    if (-e $hostPerfLogs) {
     91 #        my $pid1 = fork();
     92 #        if ($pid1 == 0) {
     93 #            exec("/bin/cat $hostPerfLogs | /usr/bin/nc -w 30 $logstashIP $logstashPORT");
     94 #        }
     95 #    }
     ```

     

   - In process-service-perfdata-log.pl and process-host-perfdata-log.pl: change logstash IP and port:

     ```bash
     92 my $logstashIP = "LOGSTASH_IP";
     93 my $logstashPORT = "LOGSTASH_PORT";
     ```
     
     

2. In case of running single CyberX node, there is no problem with the setup. In case of a peered environment *$do_on_host* variable has to be set up and the script *process-service-perfdata-log.pl/process-host-perfdata-log.pl* has to be propagated on all of CyberX nodes:

   ```bash
   16 $do_on_host = "EXAMPLE_HOSTNAME"; # CyberX node name to run the script on
   17 $hostName = hostname; # will read hostname of a node running the script
   ```

   

3. Example of command definition (*/opt/monitor/etc/checkcommands.cfg*) if scripts have been copied to */opt/plugins/custom/*:

   ```bash
   # command 'process-service-perfdata-log'
   define command{
       command_name                   process-service-perfdata-log
       command_line                   /opt/plugins/custom/process-service-perfdata-log.pl $TIMET$
       }
   # command 'process-host-perfdata-log'
   define command{
       command_name                   process-host-perfdata-log
       command_line                   /opt/plugins/custom/process-host-perfdata-log.pl $TIMET$
       }
   ```

   

4. In */opt/monitor/etc/naemon.cfg service_perfdata_file_processing_command* and *host_perfdata_file_processing_command* has to be changed to run those custom scripts:

   ```bash
   service_perfdata_file_processing_command=process-service-perfdata-log
   host_perfdata_file_processing_command=process-host-perfdata-log
   ```

   

5. In addition *service_perfdata_file_template* and *host_perfdata_file_template* can be changed to support sending more data to Elasticsearch. For instance, by adding *$HOSTGROUPNAMES$* and *$SERVICEGROUPNAMES$* macros logs can be separated better (it requires changes to Logstash filter config as well)

5. Restart naemon service:

   ```bash
   sudo systemctl restart naemon # CentOS/RHEL 7.x
   sudo service naemon restart # CentOS/RHEL 7.x
   ```

   

6. If *FileBeat* has been chosen, append below to *filebeat.conf* (adjust IP and PORT):

   ```bash
   filebeat.inputs:
   type: log
   enabled: true
   paths:
     - /opt/monitor/var/service_performance.log
     - /opt/monitor/var/host_performance.log
   tags: ["CyberXperflogs"]
     output.logstash:
   # The Logstash hosts
     hosts: ["LOGSTASH_IP:LOGSTASH_PORT"]
   ```

   

7. Restart FileBeat service:

   ```bash
   sudo systemctl restart filebeat # CentOS/RHEL 7.x
   sudo service filebeat restart # CentOS/RHEL 7.x
   ```

   

   ### Kibana

At this moment there should be new index on the Elasticsearch node with performance data documents from CyberX Monitor. 
Login to an Elasticsearch node and run: `curl -XGET '127.0.0.1:9200/_cat/indices?v'` Example output:

	health status index                      pri rep docs.count docs.deleted store.size pri.store.size
	green  open   auth                       5   0          7         6230      1.8mb          1.8mb
	green  open   CyberX-perflogs-2018.09.14    5   0      72109            0     24.7mb         24.7mb

After a while, if there is no new index make sure that: 

- Naemon is runnig on CyberX node
- Logstash service is running and there are no errors in: */var/log/logstash/logstash-plain.log* 
- Elasticsearch service is running an there are no errors in: */var/log/elasticsearch/elasticsearch.log*

If the index has been created, in order to browse and visualize the data “*index pattern*” needs to be added to Kibana. 

1. After logging in to Kibana GUI go to *Settings* tab and add *CyberX-perflogs-** pattern. Chose *@timestamp* time field and click *Create*. 
2. Performance data logs should be now accessible from Kibana GUI Discovery tab ready to be visualize.

## OP5 Beat

The op5beat is small agent for collecting metrics from op5 Monitor.

The op5beat  is located in the installation directory: `utils/op5integration/op5beat`

### Installation for Centos7 and newer

1. Copy the necessary files to the appropriate directories:

   ```bash
   cp -rf etc/* /etc/
   cp -rf usr/* /usr/
   cp -rf var/* /var/
   ```

   

2. Configure and start  op5beat service (systemd):

   ```bash
   cp -rf op5beat.service /usr/lib/systemd/system/
   systemctl daemon-reload
   systemctl enable op5beat
   systemctl start op5beat
   ```

   

### Installation for Centos6 and older

1. Copy the necessary files to the appropriate directories:

   ```bash
   cp -rf etc/* /etc/
   cp -rf usr/* /usr/
   cp -rf var/* /var/
   ```

2. Configure and start  op5beat service:

   - sysV init:

     ```bash
     cp -rf op5beat.service /etc/rc.d/init.d/op5beat
     chkconfig op5beat on
     service op5beat start
     ```

   - supervisord (optional):

     ```bash
     yum install supervisor
     cp -rf supervisord.conf /etc/supervisord.conf
     ```

    
## The Grafana instalation ##

1. To install the Grafana application you should:

   - add necessary repository to operating system:

     ```bash
     [root@localhost ~]# cat /etc/yum.repos.d/grafan.repo
       [grafana]
       name=grafana
       baseurl=https://packagecloud.io/grafana/stable/el/7/$basearch
       repo_gpgcheck=1
       enabled=1
       gpgcheck=1
       gpgkey=https://packagecloud.io/gpg.key https://grafanarel.s3.amazonaws.com/RPM-GPG-KEY-grafana
       sslverify=1
       sslcacert=/etc/pki/tls/certs/ca-bundle.crt
       [root@localhost ~]#
     ```

     

   - install the Grafana with following commands: 

     ```bash
      [root@localhost ~]# yum search grafana
       Loaded plugins: fastestmirror
       Loading mirror speeds from cached hostfile
        * base: ftp.man.szczecin.pl
        * extras: centos.slaskdatacenter.com
        * updates: centos.slaskdatacenter.com
       =========================================================================================================== N/S matched: grafana ===========================================================================================================
       grafana.x86_64 : Grafana
       pcp-webapp-grafana.noarch : Grafana web application for Performance Co-Pilot (PCP)
     
         Name and summary matches only, use "search all" for everything.
     
       [root@localhost ~]# yum install grafana
     ```

     

   - to run application use following commands:

     ```bash
      [root@localhost ~]# systemctl enable grafana-server
       Created symlink from /etc/systemd/system/multi-user.target.wants/grafana-server.service to /usr/lib/systemd/system/grafana-server.service.
       [root@localhost ~]#
       [root@localhost ~]# systemctl start grafana-server
       [root@localhost ~]# systemctl status grafana-server
       ● grafana-server.service - Grafana instance
          Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; enabled; vendor preset: disabled)
          Active: active (running) since Thu 2018-10-18 10:41:48 CEST; 5s ago
            Docs: http://docs.grafana.org
        Main PID: 1757 (grafana-server)
          CGroup: /system.slice/grafana-server.service
                  └─1757 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid cfg:default.paths.logs=/var/log/grafana cfg:default.paths.data=/var/lib/grafana cfg:default.paths.plugins=/var...
     
        [root@localhost ~]#
     ```

     

2. To connect the Grafana application you should:

   - define the default login/password (line 151;154 in config file):

     ```bash
     [root@localhost ~]# cat /etc/grafana/grafana.ini
     148 #################################### Security ####################################
     149 [security]
     150 # default admin user, created on startup
     151 admin_user = admin
     152
     153 # default admin password, can be changed before first start of grafana,  or in profile settings
     154 admin_password = admin
     155
     ```

     

   - restart *grafana-server* service:

     ```bash
     systemctl restart grafana-server
     ```

     

   - Login to Grafana user interface using web browser: *http://ip:3000*

     ![](/media/media/image112.png)

   - use login and password that you set in the config file.

   - Use below example to set conection to Elasticsearch server:

     ![](/media/media/image113.png)

## The Beats configuration ##

### Kibana API ###

Reference link: [https://www.elastic.co/guide/en/kibana/master/api.html](https://www.elastic.co/guide/en/kibana/master/api.html "https://www.elastic.co/guide/en/kibana/master/api.html")

After installing any of beats package you can use ready to use dashboard related to this beat package. For instance dashboard and index pattern are available in */usr/share/filebeat/kibana/6/* directory on Linux.

Before uploading index-pattern or dashboard you have to authorize yourself: 

1. Set up *login/password/kibana_ip* variables, e.g.:

		login=my_user
		password=my_password
		kibana_ip=10.4.11.243

1. Execute command which will save authorization cookie:

		curl -c authorization.txt -XPOST -k "https://${kibana_ip}:5601/login" -d "username=${username}&password=${password}&version=6.2.3&location=https%3A%2F%2F${kibana_ip}%3A5601%2Flogin"

1.	Upload index-pattern and dashboard to *Kibana*, e.g.:

		curl -b authorization.txt -XPOST -k "https://${kibana_ip}:5601/api/kibana/dashboards/import" -H 'kbn-xsrf: true' -H 'Content-Type: application/json' -d@/usr/share/filebeat/kibana/6/index-pattern/filebeat.json
		curl -b authorization.txt -XPOST -k "https://${kibana_ip}:5601/api/kibana/dashboards/import" -H 'kbn-xsrf: true' -H 'Content-Type: application/json' -d@/usr/share/filebeat/kibana/6/dashboard/Filebeat-mysql.json

1.	When you want to upload beats index template to Ealsticsearch you have to recover it first (usually you do not send logs directly to Es rather than to Logstash first):

		/usr/bin/filebeat export template --es.version 6.2.3 >> /path/to/beats_template.json

1.	After that you can upload it as any other template (Access Es node with SSH):

		curl -XPUT "localhost:9200/_template/CyberXperfdata" -H'Content-Type: application/json' -d@beats_template.json

## Wazuh integration ##

CyberX can integrate with the Wazuh, which is lightweight agent is designed to perform a number of tasks with the objective of detecting threats and, when necessary, trigger automatic responses. The agent core capabilities are:

- Log and events data collection
- File and registry keys integrity monitoring
- Inventory of running processes and installed applications
- Monitoring of open ports and network configuration
- Detection of rootkits or malware artifacts
- Configuration assessment and policy monitoring
- Execution of active responses

The Wazuh agents run on many different platforms, including Windows, Linux, Mac OS X, AIX, Solaris and HP-UX. They can be configured and managed from the Wazuh server.
#### Deploying Wazuh Server ####

https://documentation.wazuh.com/3.13/installation-guide/installing-wazuh-manager/linux/centos/index.html

#### Deploing Wazuh Agent ####

https://documentation.wazuh.com/3.13/installation-guide/installing-wazuh-agent/index.html

#### Filebeat configuration ####

## BRO integration ##

## 2FA authorization with Google Auth Provider (example)

### Software used (tested versions):

- NGiNX (1.16.1 - from CentOS base reposiory)
- oauth2_proxy ([https://github.com/pusher/oauth2_proxy/releases](https://github.com/pusher/oauth2_proxy/releases) - 4.0.0)

### The NGiNX configuration:
1. Copy the [ng_oauth2_proxy.conf](/files/ng_oauth2_proxy.conf) to `/etc/nginx/conf.d/`;

   ```bash
   server {
       listen 443 default ssl;
       server_name logserver.local;
       ssl_certificate /etc/kibana/ssl/logserver.org.crt;
       ssl_certificate_key /etc/kibana/ssl/logserver.org.key;
       ssl_session_cache   builtin:1000  shared:SSL:10m;
       add_header Strict-Transport-Security max-age=2592000;
   
     location /oauth2/ {
       proxy_pass       http://127.0.0.1:4180;
       proxy_set_header Host                    $host;
       proxy_set_header X-Real-IP               $remote_addr;
       proxy_set_header X-Scheme                $scheme;
       proxy_set_header X-Auth-Request-Redirect $request_uri;
       # or, if you are handling multiple domains:
       # proxy_set_header X-Auth-Request-Redirect $scheme://$host$request_uri;
     }
     location = /oauth2/auth {
       proxy_pass       http://127.0.0.1:4180;
       proxy_set_header Host             $host;
       proxy_set_header X-Real-IP        $remote_addr;
       proxy_set_header X-Scheme         $scheme;
       # nginx auth_request includes headers but not body
       proxy_set_header Content-Length   "";
       proxy_pass_request_body           off;
     }
   
     location / {
       auth_request /oauth2/auth;
       error_page 401 = /oauth2/sign_in;
   
       # pass information via X-User and X-Email headers to backend,
       # requires running with --set-xauthrequest flag
       auth_request_set $user   $upstream_http_x_auth_request_user;
       auth_request_set $email  $upstream_http_x_auth_request_email;
       proxy_set_header X-User  $user;
       proxy_set_header X-Email $email;
   
       # if you enabled --pass-access-token, this will pass the token to the backend
       auth_request_set $token  $upstream_http_x_auth_request_access_token;
       proxy_set_header X-Access-Token $token;
   
       # if you enabled --cookie-refresh, this is needed for it to work with auth_request
       auth_request_set $auth_cookie $upstream_http_set_cookie;
       add_header Set-Cookie $auth_cookie;
   
       # When using the --set-authorization-header flag, some provider's cookies can exceed the 4kb
       # limit and so the OAuth2 Proxy splits these into multiple parts.
       # Nginx normally only copies the first `Set-Cookie` header from the auth_request to the response,
       # so if your cookies are larger than 4kb, you will need to extract additional cookies manually.
       auth_request_set $auth_cookie_name_upstream_1 $upstream_cookie_auth_cookie_name_1;
   
       # Extract the Cookie attributes from the first Set-Cookie header and append them
       # to the second part ($upstream_cookie_* variables only contain the raw cookie content)
       if ($auth_cookie ~* "(; .*)") {
           set $auth_cookie_name_0 $auth_cookie;
           set $auth_cookie_name_1 "auth_cookie__oauth2_proxy_1=$auth_cookie_name_upstream_1$1";
       }
   
       # Send both Set-Cookie headers now if there was a second part
       if ($auth_cookie_name_upstream_1) {
           add_header Set-Cookie $auth_cookie_name_0;
           add_header Set-Cookie $auth_cookie_name_1;
       }
   
       proxy_pass https://127.0.0.1:5601;
       # or "root /path/to/site;" or "fastcgi_pass ..." etc
     }
   }
   ```

   

1. Set `ssl_certificate` and `ssl_certificate_key` path in ng_oauth2_proxy.conf

When SSL is set using nginx proxy, Kibana can be started with http. 
However, if it is to be run with encryption, you also need to change `proxy_pass` to the appropriate one.

### The [oauth2_proxy](/files/oauth2_proxy.cfg) configuration:

1. Create a directory in which the program will be located and its configuration:

    ```bash
    mkdir -p /usr/share/oauth2_proxy/
    mkdir -p /etc/oauth2_proxy/
    ```

2. Copy files to directories:

    ```bash
    cp oauth2_proxy /usr/share/oauth2_proxy/
    cp oauth2_proxy.cfg /etc/oauth2_proxy/
    ```

3. Set directives according to OAuth configuration in Google Cloud project

    ```bash
            cfg
            client_id =
            client_secret =
            # the following limits domains for authorization (* - all)
        ​	email_domains = [
        ​	  "*"
        ​	]
    ```

4. Set the following according to the public hostname:

    ```bash
    cookie_domain = "kibana-host.org"
    ```

5. In case 	og-in restrictions for a specific group defined on the Google side:
	- Create administrative account: https://developers.google.com/identity/protocols/OAuth2ServiceAccount ; 
	- Get configuration to JSON file and copy Client ID;
	- On the dashboard of the Google Cloud select "APIs & Auth" -> "APIs";
	- Click on "Admin SDK" and "Enable API";
	- Follow the instruction at [https://developers.google.com/admin-sdk/directory/v1/guides/delegation#delegate_domain-wide_authority_to_your_service_account](https://developers.google.com/admin-sdk/directory/v1/guides/delegation#delegate_domain-wide_authority_to_your_service_account) and give the service account the following permissions:

			https://www.googleapis.com/auth/admin.directory.group.readonly
			https://www.googleapis.com/auth/admin.directory.user.readonly

	- Follow the instructions to grant access to the Admin API [https://support.google.com/a/answer/60757](https://support.google.com/a/answer/60757)
	- Create or select an existing administrative email in the Gmail domain to flag it `google-admin-email`
	- Create or select an existing group to flag it `google-group`
	- Copy the previously downloaded JSON file to `/etc/oauth2_proxy/`.
	- In file [oauth2_proxy](/files/oauth2_proxy.cfg) set the appropriate path:

    ```bash
    google_service_account_json =
    ```

### Service start up

- Start the NGiNX service 

- Start the oauth2_proxy service

```bash
/usr/share/oauth2_proxy/oauth2_proxy -config="/etc/oauth2_proxy/oauth2_proxy.cfg"
```

In the browser enter the address pointing to the server with the CyberX installation

## Cerebro - elasticsearch web admin tool

Cerebro is the Elasticsearch administration tool that allows you to perform the following tasks:

- monitoring and management of indexing nodes, indexes and shards:

![](/media/media/image217.PNG)

- monitoring and management of index shapshoots :

![](/media/media/image220.PNG)

- informing about problems with indexes and shards:

![](/media/media/image219.PNG)

### Cerebro Configuration

Configuration file: `/opt/cerebro/conf/application.conf`

- Authentication

   ```bash

			auth = {
			  type: basic
			    settings: {
			      username = "logserver"
			      password = "logserver"
			    }
			}
   ```

- A list of known Elasticsearch hosts

   ```bash
			hosts = [
			  {
			    host = "https://192.168.3.11:9200"
			    name = "energy-logserver"
			    auth = {
			      username = "logserver"
			      password = "logserver"
			    }
			  }
			]

			play.ws.ssl {
			  trustManager = {
			    stores = [
			      { type = "PEM", path = "/etc/elasticsearch/ssl/rootCA.crt" }
			    ]
			  }
			} 
			play.ws.ssl.loose.acceptAnyCertificate=true
   ```

- SSL access to cerebro

   ```bash
      http = {
        port = "disabled"
      }
      https = {
        port = "5602"
      }
      
      # SSL access to cerebro - no self signed certificates
      #play.server.https {
      #  keyStore = {
      #    path = "keystore.jks",
      #    password = "SuperSecretKeystorePassword"
      #  }
      #}
      
      #play.ws.ssl {
      #  trustManager = {
      #    stores = [
      #      { type = "JKS", path = "truststore.jks", password =       SuperSecretTruststorePassword"  }
      #    ]
      #  }
      #}
   ```

- service restart

   ```bash
   	systemctl start cerebro
   ```

- register backup/snapshot repository for Elasticsearch

   ```bash
      curl -k -XPUT "https://127.0.0.1:9200/_snapshot/backup?pretty" -H 'Content-Type: plication/      json' -d'
      {
        "type": "fs",
        "settings": {
          "location": "/var/lib/elasticsearch/backup/"
        }
      }' -u logserver:logserver
   ```

- login using curl/kibana

   ```bash
      curl -k -XPOST 'https://192.168.3.11:5602/auth/login' -H 'mimeType: application/      -www-form-urlencoded' -d 'user=logserver&password=logserver' -c cookie.txt
      curl -k -XGET 'https://192.168.3.11:5602' -b cookie.txt
   ```

## Elasticdump

Elasticdump is a tool for moving and saving indices.

### Location

```bash
/usr/share/kibana/elasticdump/elasticdump
```

### Examples of use

#### Copy an index from production to staging with analyzer and mapping

```bash
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=analyzer
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=mapping
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=data
```

#### Backup index data to a file:

```bash
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=/data/my_index_mapping.json \
  --type=mapping
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=/data/my_index.json \
  --type=data
```

#### Backup and index to a gzip using stdout

```bash
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=$ \
  | gzip > /data/my_index.json.gz
```

#### Backup the results of a query to a file

```bash
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=query.json \
  --searchBody="{\"query\":{\"term\":{\"username\": \"admin\"}}}"
```

#### Copy a single shard data

```bash
elasticdump \
  --input=http://es.com:9200/api \
  --output=http://es.com:9200/api2 \
  --params="{\"preference\":\"_shards:0\"}"
```

#### Backup aliases to a file

```bash
elasticdump \
  --input=http://es.com:9200/index-name/alias-filter \
  --output=alias.json \
  --type=alias
```

#### Import aliases into ES

```bash
elasticdump \
  --input=./alias.json \
  --output=http://es.com:9200 \
  --type=alias
```

#### Backup templates to a file

```bash
elasticdump \
  --input=http://es.com:9200/template-filter \
  --output=templates.json \
  --type=template
```

#### Import templates into ES

```bash
elasticdump \
  --input=./templates.json \
  --output=http://es.com:9200 \
  --type=template
```

#### Split files into multiple parts

```bash
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=/data/my_index.json \
  --fileSize=10mb
```

#### Import data from S3 into ES (using s3urls)

```bash
elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input "s3://${bucket_name}/${file_name}.json" \
  --output=http://production.es.com:9200/my_index
```

#### Export ES data to S3 (using s3urls)

```bash
elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input=http://production.es.com:9200/my_index \
  --output "s3://${bucket_name}/${file_name}.json"
```

#### Import data from MINIO (s3 compatible) into ES (using s3urls)

```bash
elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input "s3://${bucket_name}/${file_name}.json" \
  --output=http://production.es.com:9200/my_index
  --s3ForcePathStyle true
  --s3Endpoint https://production.minio.co
```

#### Export ES data to MINIO (s3 compatible) (using s3urls)

```bash
elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input=http://production.es.com:9200/my_index \
  --output "s3://${bucket_name}/${file_name}.json"
  --s3ForcePathStyle true
  --s3Endpoint https://production.minio.co
```

#### Import data from CSV file into ES (using csvurls)

```bash
elasticdump \

  # csv:// prefix must be included to allow parsing of csv files

  # --input "csv://${file_path}.csv" \

  --input "csv:///data/cars.csv"
  --output=http://production.es.com:9200/my_index \
  --csvSkipRows 1    # used to skip parsed rows (this does not include the headers row)
  --csvDelimiter ";" # default csvDelimiter is ','

```

#### Copy a single index from a elasticsearch:

```bash
elasticdump \
  --input=http://es.com:9200/api/search \
  --input-index=my_index \
  --output=http://es.com:9200/api/search \
  --output-index=my_index \
  --type=mapping
```

#### Copy a single type:

```bash
elasticdump \
  --input=http://es.com:9200/api/search \
  --input-index=my_index/my_type \
  --output=http://es.com:9200/api/search \
  --output-index=my_index \
  --type=mapping
```

### Usage

```bash
elasticdump --input SOURCE --output DESTINATION [OPTIONS]
```

### All parameters

```bash
--input
                    Source location (required)
--input-index
                    Source index and type
                    (default: all, example: index/type)
--output
                    Destination location (required)
--output-index
                    Destination index and type
                    (default: all, example: index/type)
--overwrite
                    Overwrite output file if it exists
                    (default: false)                    
--limit
                    How many objects to move in batch per operation
                    limit is approximate for file streams
                    (default: 100)
--size
                    How many objects to retrieve
                    (default: -1 -> no limit)
--concurrency
                    The maximum number of requests the can be made concurrently to a specified transport.
                    (default: 1)       
--concurrencyInterval
                    The length of time in milliseconds in which up to <intervalCap> requests can be made
                    before the interval request count resets. Must be finite.
                    (default: 5000)       
--intervalCap
                    The maximum number of transport requests that can be made within a given <concurrencyInterval>.
                    (default: 5)
--carryoverConcurrencyCount
                    If true, any incomplete requests from a <concurrencyInterval> will be carried over to
                    the next interval, effectively reducing the number of new requests that can be created
                    in that next interval.  If false, up to <intervalCap> requests can be created in the
                    next interval regardless of the number of incomplete requests from the previous interval.
                    (default: true)                                                                                       
--throttleInterval
                    Delay in milliseconds between getting data from an inputTransport and sending it to an
                    outputTransport.
                     (default: 1)
--debug
                    Display the elasticsearch commands being used
                    (default: false)
--quiet
                    Suppress all messages except for errors
                    (default: false)
--type
                    What are we exporting?
                    (default: data, options: [settings, analyzer, data, mapping, alias, template, component_template, index_template])
--filterSystemTemplates
                    Whether to remove metrics-*-* and logs-*-* system templates 
                    (default: true])
--templateRegex
                    Regex used to filter templates before passing to the output transport 
                    (default: ((metrics|logs|\\..+)(-.+)?)
--delete
                    Delete documents one-by-one from the input as they are
                    moved.  Will not delete the source index
                    (default: false)
--searchBody
                    Preform a partial extract based on search results
                    when ES is the input, default values are
                      if ES > 5
                        `'{"query": { "match_all": {} }, "stored_fields": ["*"], "_source": true }'`
                      else
                        `'{"query": { "match_all": {} }, "fields": ["*"], "_source": true }'`
--searchWithTemplate
                    Enable to use Search Template when using --searchBody
                    If using Search Template then searchBody has to consist of "id" field and "params" objects
                    If "size" field is defined within Search Template, it will be overridden by --size parameter
                    See https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html for 
                    further information
                    (default: false)
--headers
                    Add custom headers to Elastisearch requests (helpful when
                    your Elasticsearch instance sits behind a proxy)
                    (default: '{"User-Agent": "elasticdump"}')
--params
                    Add custom parameters to Elastisearch requests uri. Helpful when you for example
                    want to use elasticsearch preference
                    (default: null)
--sourceOnly
                    Output only the json contained within the document _source
                    Normal: {"_index":"","_type":"","_id":"", "_source":{SOURCE}}
                    sourceOnly: {SOURCE}
                    (default: false)
--ignore-errors
                    Will continue the read/write loop on write error
                    (default: false)
--scrollId
                    The last scroll Id returned from elasticsearch. 
                    This will allow dumps to be resumed used the last scroll Id &
                    `scrollTime` has not expired.
--scrollTime
                    Time the nodes will hold the requested search in order.
                    (default: 10m)
--maxSockets
                    How many simultaneous HTTP requests can we process make?
                    (default:
                      5 [node <= v0.10.x] /
                      Infinity [node >= v0.11.x] )
--timeout
                    Integer containing the number of milliseconds to wait for
                    a request to respond before aborting the request. Passed
                    directly to the request library. Mostly used when you don't
                    care too much if you lose some data when importing
                    but rather have speed.
--offset
                    Integer containing the number of rows you wish to skip
                    ahead from the input transport.  When importing a large
                    index, things can go wrong, be it connectivity, crashes,
                    someone forgetting to `screen`, etc.  This allows you
                    to start the dump again from the last known line written
                    (as logged by the `offset` in the output).  Please be
                    advised that since no sorting is specified when the
                    dump is initially created, there's no real way to
                    guarantee that the skipped rows have already been
                    written/parsed.  This is more of an option for when
                    you want to get most data as possible in the index
                    without concern for losing some rows in the process,
                    similar to the `timeout` option.
                    (default: 0)
--noRefresh
                    Disable input index refresh.
                    Positive:

                         1. Much increase index speed
                            are requirements
                                                Negative:
                                                     1. Recently added data may not be indexed
                                                         with big data indexing,
                                                                            where speed and system health in a higher priority
                                                                            than recently added data.
                                                        --inputTransport
                                                                            Provide a custom js file to use as the input transport
                                                        --outputTransport
                                                                            Provide a custom js file to use as the output transport
                                                        --toLog
                                                                            When using a custom outputTransport, should log lines
                                                                            be appended to the output stream?
                                                                            (default: true, except for `$`)
                                                        --transform
                                                                            A method/function which can be called to modify documents
                                                                            before writing to a destination. A global variable 'doc'
                                                                            is available.
                                                                            Example script for computing a new field 'f2' as doubled
                                                                            value of field 'f1':
                                                                                doc._source["f2"] = doc._source.f1 * 2;
                                                                            May be used multiple times.
                                                                            Additionally, transform may be performed by a module. See [Module Transform](#module-transform) below.
                                                        --awsChain
                                                                            Use [standard](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/) location and ordering for resolving credentials including environment variables, config files, EC2 and ECS metadata locations
                                                                            _Recommended option for use with AWS_
                                                                            Use [standard](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/) 
                                                                            location and ordering for resolving credentials including environment variables, 
                                                                            config files, EC2 and ECS metadata locations _Recommended option for use with AWS_
                                                        --awsAccessKeyId
                                                        --awsSecretAccessKey
                                                                            When using Amazon Elasticsearch Service protected by
                                                                            AWS Identity and Access Management (IAM), provide
                                                                            your Access Key ID and Secret Access Key.
                                                                            --sessionToken can also be optionally provided if using temporary credentials
                                                        --awsIniFileProfile
                                                                            Alternative to --awsAccessKeyId and --awsSecretAccessKey,
                                                                            loads credentials from a specified profile in aws ini file.
                                                                            For greater flexibility, consider using --awsChain
                                                                            and setting AWS_PROFILE and AWS_CONFIG_FILE
                                                                            environment variables to override defaults if needed
                                                        --awsIniFileName
                                                                            Override the default aws ini file name when using --awsIniFileProfile
                                                                            Filename is relative to ~/.aws/
                                                                            (default: config)
                                                        --awsService
                                                                            Sets the AWS service that the signature will be generated for
                                                                            (default: calculated from hostname or host)
                                                        --awsRegion
                                                                            Sets the AWS region that the signature will be generated for
                                                                            (default: calculated from hostname or host)
                                                        --awsUrlRegex
                                                                            Regular expression that defined valied AWS urls that should be signed
                                                                            (default: ^https?:\\.*.amazonaws.com.*$)
                                                        --support-big-int   
                                                                            Support big integer numbers
                                                        --big-int-fields   
                                                                            Sepcifies a comma-seperated list of fields that should be checked for big-int support
                                                                            (default '')
                                                        --retryAttempts  
                                                                            Integer indicating the number of times a request should be automatically re-attempted before failing
                                                                            when a connection fails with one of the following errors `ECONNRESET`, `ENOTFOUND`, `ESOCKETTIMEDOUT`,
                                                                            ETIMEDOUT`, `ECONNREFUSED`, `EHOSTUNREACH`, `EPIPE`, `EAI_AGAIN`
                                                                            (default: 0)

--retryDelay   
                    Integer indicating the back-off/break period between retry attempts (milliseconds)
                    (default : 5000)            
--parseExtraFields
                    Comma-separated list of meta-fields to be parsed  
--maxRows
                    supports file splitting.  Files are split by the number of rows specified
--fileSize
                    supports file splitting.  This value must be a string supported by the **bytes** module.     
                    The following abbreviations must be used to signify size in terms of units         
                    b for bytes
                    kb for kilobytes
                    mb for megabytes
                    gb for gigabytes
                    tb for terabytes
                    
                    e.g. 10mb / 1gb / 1tb
                    Partitioning helps to alleviate overflow/out of memory exceptions by efficiently segmenting files
                    into smaller chunks that then be merged if needs be.
--fsCompress
                    gzip data before sending output to file.
                    On import the command is used to inflate a gzipped file
--s3AccessKeyId
                    AWS access key ID
--s3SecretAccessKey
                    AWS secret access key
--s3Region
                    AWS region
--s3Endpoint        
                    AWS endpoint can be used for AWS compatible backends such as
                    OpenStack Swift and OpenStack Ceph
--s3SSLEnabled      
                    Use SSL to connect to AWS [default true]
                    
--s3ForcePathStyle  Force path style URLs for S3 objects [default false]
                    
--s3Compress
                    gzip data before sending to s3  
--s3ServerSideEncryption
                    Enables encrypted uploads
--s3SSEKMSKeyId
                    KMS Id to be used with aws:kms uploads                    
--s3ACL
                    S3 ACL: private | public-read | public-read-write | authenticated-read | aws-exec-read |
                    bucket-owner-read | bucket-owner-full-control [default private]

--retryDelayBase
                    The base number of milliseconds to use in the exponential backoff for operation retries. (s3)
--customBackoff
                    Activate custom customBackoff function. (s3)
--tlsAuth
                    Enable TLS X509 client authentication
--cert, --input-cert, --output-cert
                    Client certificate file. Use --cert if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--key, --input-key, --output-key
                    Private key file. Use --key if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--pass, --input-pass, --output-pass
                    Pass phrase for the private key. Use --pass if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--ca, --input-ca, --output-ca
                    CA certificate. Use --ca if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--inputSocksProxy, --outputSocksProxy
                    Socks5 host address
--inputSocksPort, --outputSocksPort
                    Socks5 host port
--handleVersion
                    Tells elastisearch transport to handle the `_version` field if present in the dataset
                    (default : false)
--versionType
                    Elasticsearch versioning types. Should be `internal`, `external`, `external_gte`, `force`.
                    NB : Type validation is handled by the bulk endpoint and not by elasticsearch-dump
--csvDelimiter        
                    The delimiter that will separate columns.
                    (default : ',')
--csvFirstRowAsHeaders        
                    If set to true the first row will be treated as the headers.
                    (default : true)
--csvRenameHeaders        
                    If you want the first line of the file to be removed and replaced by the one provided in the `csvCustomHeaders` option
                    (default : true)
--csvCustomHeaders  A comma-seperated listed of values that will be used as headers for your data. This param must
                    be used in conjunction with `csvRenameHeaders`
                    (default : null)
--csvWriteHeaders   Determines if headers should be written to the csv file.
                    (default : true)
--csvIgnoreEmpty        
                    Set to true to ignore empty rows. 
                    (default : false)
--csvSkipLines        
                    If number is > 0 the specified number of lines will be skipped.
                    (default : 0)
--csvSkipRows        
                    If number is > 0 then the specified number of parsed rows will be skipped
                    NB:  (If the first row is treated as headers, they aren't a part of the count)
                    (default : 0)
--csvMaxRows        
                    If number is > 0 then only the specified number of rows will be parsed.(e.g. 100 would return the first 100 rows of data)
                    (default : 0)
--csvTrim        
                    Set to true to trim all white space from columns.
                    (default : false)
--csvRTrim        
                    Set to true to right trim all columns.
                    (default : false)
--csvLTrim        
                    Set to true to left trim all columns.
                    (default : false)   
--csvHandleNestedData        
                    Set to true to handle nested JSON/CSV data. 
                    NB : This is a very optioninated implementaton !
                    (default : false)
--csvIdColumn        
                    Name of the column to extract the record identifier (id) from
                    When exporting to CSV this column can be used to override the default id (@id) column name
                    (default : null)   
--csvIndexColumn        
                    Name of the column to extract the record index from
                    When exporting to CSV this column can be used to override the default index (@index) column name
                    (default : null)
--csvTypeColumn        
                    Name of the column to extract the record type from
                    When exporting to CSV this column can be used to override the default type (@type) column name
                    (default : null)              
--help
                    This page
```



### Elasticsearch's Scroll API

Elasticsearch provides a scroll API to fetch all documents of an index starting from (and keeping) a consistent snapshot in time, which we use under the hood. This method is safe to use for large exports since it will maintain the result set in cache for the given period of time.

NOTE: only works for --output

### Bypassing self-sign certificate errors

Set the environment NODE_TLS_REJECT_UNAUTHORIZED=0 before running elasticdump

### An alternative method of passing environment variables before execution

NB : This only works with linux shells

NODE_TLS_REJECT_UNAUTHORIZED=0 elasticdump --input="https://localhost:9200" --output myfile

## Curator - Elasticsearch index management tool

Curator is a tool that allows you to perform index management tasks, such as:

- Close Indices
- Delete Indices
- Delete Snapshots
- Forcemerge segments
- Changing Index Settings
- Open Indices
- Reindex data

And other.

### Curator installation

Curator is delivered with the client node installer. 

### Curator configuration

Create directory for configuration:

```bash
mkdir /etc/curator
```

Create directory for Curator logs file:

```bash
mkdir /var/log/curator
```

### Running Curator

The curator executable is located in the directory:

```bash
/usr/share/kibana/curator/bin/curator
```

Curator requires two parameters:

- config - path to configuration file for Curator
- path to action file for Curator

Example running command:

```bash
/usr/share/kibana/curator/bin/curator --config /etc/curator/curator.conf /etc/curator/close_indices.yml
```

### Sample configuration file

---
Remember, leave a key empty if there is no value.  None will be a string, not a Python "NoneType"

```bash
client:
  hosts:
    - 127.0.0.1
  port: 9200
#  url_prefix:
#  use_ssl: False
#  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth: $user:$passowrd
  timeout: 30
  master_only: True

logging:
  loglevel: INFO
  logfile: /var/log/curator/curator.log
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```

### Sample action file

- close indices

  ```bash
  actions:
    1:
      action: close
      description: >-
        Close indices older than 30 days (based on index name), for logstash-
        prefixed indices.
      options:
        delete_aliases: False
        timeout_override:
        continue_if_exception: False
        disable_action: True
      filters:
      - filtertype: pattern
        kind: prefix
        value: logstash-
        exclude:
      - filtertype: age
        source: name
        direction: older
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: 30
        exclude:
  ```

- delete indices

  ```bash
  actions:
    1:
      action: delete_indices
      description: >-
        Delete indices older than 45 days (based on index name), for logstash-
        prefixed indices. Ignore the error if the filter does not result in an
        actionable list of indices (ignore_empty_list) and exit cleanly.
      options:
        ignore_empty_list: True
        timeout_override:
        continue_if_exception: False
        disable_action: True
      filters:
      - filtertype: pattern
        kind: prefix
        value: logstash-
        exclude:
      - filtertype: age
        source: name
        direction: older
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: 45
        exclude:
  ```

- forcemerge segments

  ```bash
  actions:
    1:
      action: forcemerge
      description: >-
        forceMerge logstash- prefixed indices older than 2 days (based on index
        creation_date) to 2 segments per shard.  Delay 120 seconds between each
        forceMerge operation to allow the cluster to quiesce.
        This action will ignore indices already forceMerged to the same or fewer
        number of segments per shard, so the 'forcemerged' filter is unneeded.
      options:
        max_num_segments: 2
        delay: 120
        timeout_override:
        continue_if_exception: False
        disable_action: True
      filters:
      - filtertype: pattern
        kind: prefix
        value: logstash-
        exclude:
      - filtertype: age
        source: creation_date
        direction: older
        unit: days
        unit_count: 2
        exclude:
  ```

- open indices

  ```bash
  actions:
    1:
      action: open
      description: >-
        Open indices older than 30 days but younger than 60 days (based on index
        name), for logstash- prefixed indices.
      options:
        timeout_override:
        continue_if_exception: False
        disable_action: True
      filters:
      - filtertype: pattern
        kind: prefix
        value: logstash-
        exclude:
      - filtertype: age
        source: name
        direction: older
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: 30
        exclude:
      - filtertype: age
        source: name
        direction: younger
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: 60
        exclude:
  ```

- replica reduce

  ```bash
  actions:
    1:
      action: replicas
      description: >-
        Reduce the replica count to 0 for logstash- prefixed indices older than
        10 days (based on index creation_date)
      options:
        count: 0
        wait_for_completion: False
        timeout_override:
        continue_if_exception: False
        disable_action: True
      filters:
      - filtertype: pattern
        kind: prefix
        value: logstash-
        exclude:
      - filtertype: age
        source: creation_date
        direction: older
        unit: days
        unit_count: 10
        exclude:
  ```

  

## Cross-cluster Search

**Cross-cluster search** lets you run a single search request against one or more remote clusters. For example, you can use a cross-cluster search to filter and analyze log data stored on clusters in different data centers.

### Configuration

1. Use `_cluster` API to add least one remote cluster:

   ```bash
   curl -u user:password -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
   {
     "persistent": {
       "cluster": {
         "remote": {
           "cluster_one": {
             "seeds": [
               "192.168.0.1:9300"
             ]
           },
           "cluster_two": {
             "seeds": [
               "192.168.0.2:9300"
             ]
           }
         }
       }
     }
   }'
   ```

   

2. To search data in index `twitter` located on the `cluster_one` use following command:

   ```bash
   curl -u user:password -X GET "localhost:9200/cluster_one:twitter/_search?pretty" -H 'Content-Type: application/json' -d'
   {
     "query": {
       "match": {
         "user": "kimchy"
       }
     }
   }'
   ```

   

3. To search data in index `twitter` located on multiple clusters, use following command:

   ```bash
   curl -u user:password -X GET "localhost:9200/twitter,cluster_one:twitter,cluster_two:twitter/_search?pretty" -H 'Content-Type: application/json' -d'
   {
     "query": {
       "match": {
         "user": "kimchy"
       }
     }
   }'
   ```

4. Configure index pattern in Kibana GUI to discover data from multiple clusters:

   ```bash
   cluster_one:logstash-*,cluster_two:logstash-*
   ```

   ![](/media/media/image133.png)

### Security

Cross-cluster search uses the Elasticsearch transport layer (default 9300/tcp port) to exchange data.  To secure the transmission, encryption must be enabled for the transport layer. 

Configuration is in the `/etc/elasticsearch/elastisearch.yml` file:

```bash
# Transport layer encryption
logserverguard.ssl.transport.enabled: true
logserverguard.ssl.transport.pemcert_filepath: "/etc/elasticsearch/ssl/certificate.crt"
logserverguard.ssl.transport.pemkey_filepath: "/etc/elasticsearch/ssl/certificate.key"
logserverguard.ssl.transport.pemkey_password: ""
logserverguard.ssl.transport.pemtrustedcas_filepath: "/etc/elasticsearch/ssl/rootCA.crt"

logserverguard.ssl.transport.enforce_hostname_verification: false
logserverguard.ssl.transport.resolve_hostname: false

```

 Encryption must be enabled on each cluster.

## Sync/Copy

The Sync/Copy module allows you to synchronize or copy data between two Elasticsearch clusters.
You can copy or synchronize selected indexes or indicate index pattern.

### Configuration

Before starting Sync/Copy, complete the source and target cluster data in the `Profile` and `Create profile`tab:

- Protocol - http or https;
- Host - IP address ingest node;
- Port - communication port (default 9200);
- Username - username that has permission to get data and save data to the cluster;
- Password - password of the above user
- Cluster name

![](/media/media/image134.png)

You can view or delete the profile in the `Profile List` tab.

### Synchronize data

To perform data synchronization, follow the instructions:

- go to the `Sync` tab;
- select `Source Profile`
- select `Destination Profile`
- enter the index pattern name in `Index pattern to sync`
- or use switch `Toggle to select between Index pattern or name` and enter indices name.
- to create synchronization task, press `Submit` button

![](/media/media/image135.png)

### Copy data

To perform data copy, follow the instructions:

- go to the `Copy` tab;
- select `Source Profile`
- select `Destination Profile`
- enter the index pattern name in `Index pattern to sync`
- or use switch `Toggle to select between Index pattern or name` and enter indices name.
- to start copying data press the `Submit` button

![](/media/media/image136.png)

### Running Sync/Copy

Prepared Copy/Sync tasks can be run on demand or according to a set schedule.
To do this, go to the `Jobs` tab. With each task you will find the `Action` button that allows:

- running the task;
- scheduling task in Cron format;
- deleting task;
- download task logs.

![](/media/media/image137.png)

## XLSX Import

The XLSX Import module allow to import your `xlsx` and `csv` file to indices.

### Importing steps

1. Go to XLSX Import module and select your file and sheet:

   ![](/media/media/image138.png)

2. After the data has been successfully loaded, you will see a preview of your data at the bottom of the window. 

3. Press `Next` button.

4. In the next step, enter the index name in the `Index name` field, you can also change the pattern for the document ID and select the columns that the import will skip.

   ![](/media/media/image139.png)

5. Select the `Configure your own mapping` for every field. You can choose the type and apply more options with the advanced JSON.
   The list of parameters can be found here, https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-params.html

6. After the import configuration is complete, select the `Import` button to start the import process.

7. After the import process is completed, a summary will be displayed. Now you can create a new index pattern to view your data in the Discovery module.

   ![](/media/media/image140.png)

## Logtrail

LogTrail module allow to view, analyze, search and tail log events from multiple indices in realtime. Main features of this module are:

- View, analyze and search log events from a centralized interface
- Clean & simple devops friendly interface
- Live tail
- Filter aggregated logs by hosts and program
- Quickly seek to logs based on time
- Supports highlighting of search matches
- Supports multiple Elasticsearch index patterns each with different  schemas
- Can be extended by adding additional fields to log event
- Color coding of messages based on field values

Default Logtrail configuration, keeps track of event logs for Elasticsearch, Logstash, Kibana and Alert processes.
The module allows you to track events from any index stored in Elasticsearch.

### Configuration

The LogTrail module uses the Logstash pipeline to retrieve data from any of the event log files and save its contents to the Elasticsearch index.

### Logstash configuration

Example for the file `/var/log/messages`

1. Add the Logstash configuration file in the correct pipline (default is "logtrail"):

    ```bash
    vi /etc/logstash/conf.d/logtrail/messages.conf
    ```

    ```bash
    input {
        file {
            path => "/var/log/messages"
            start_position => beginning
            tags => "logtrail_messages"
        }
    }
    filter {
            if "logtrail_messages" in [tags] {
                    grok {
                            match => {
                                    #"message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:syslog_message}"
    # If syslog is format is "<%PRI%><%syslogfacility%>%TIMESTAMP% %HOSTNAME% %syslogtag%%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
                                    "message" => "<?%{NONNEGINT:priority}><%{NONNEGINT:facility}>%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:syslog_message}"
                                    }
                            }
                    date {
                            match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
                    }
                    ruby {
                            code =>  "event.set('level',event.get('priority').to_i - ( event.get('facility').to_i * 8 ))"
                    }
            }
    }
    output {
        if "logtrail_messages" in [tags] {
            elasticsearch {
                hosts => "http://localhost:9200"
                index => "logtrail-messages-%{+YYYY.MM}"
                user => "logstash"
                password => "logstash"
            }
        }
    }
    ```

2. Restart the Logstash service

   ```bahs
   systemctl restart logstash
   ```


### Kibana configuration

1. Set up a new pattern index `logtrail-messages*`  in the CyberX configuration. The procedure is described in the chapter [First login](/02-00-00-Data_source_and_application_management/02-00-00-Data_source_and_application_management.md).

2. Add a new configuration section in the LogTrail configuration file:

   ```bash
   vi /usr/share/kibana/plugins/logtrail/logtrail.json
   ```

   ```bash
   {
     "index_patterns" : [
       {
         "es": {
           "default_index": "logstash-message-*",
           "allow_url_parameter": false
         },
         "tail_interval_in_seconds": 10,
         "es_index_time_offset_in_seconds": 0,
         "display_timezone": "Etc/UTC",
         "display_timestamp_format": "MMM DD HH:mm:ss",
         "max_buckets": 500,
         "default_time_range_in_days" : 0,
         "max_hosts": 100,
         "max_events_to_keep_in_viewer": 5000,
         "fields" : {
           "mapping" : {
               "timestamp" : "@timestamp",
               "display_timestamp" : "@timestamp",
               "hostname" : "hostname",
               "program": "program",
               "message": "syslog_message"
           },
           "message_format": "{{{syslog_message}}}"
         },
         "color_mapping" : {
           "field": "level",
           "mapping" : {
             "0": "#ff0000",
             "1": "#ff3232",
             "2": "#ff4c4c",
             "3": "#ff7f24",
             "4": "#ffb90f",
             "5": "#a2cd5a"
           }
         }
       }
     ]
   }
   
   ```

3. Restate the Kibana service

   ```bash
   systemctl restart kibana
   ```

### Using Logtrail

To access of the LogTrail module, click the tile icon from the main menu bar and then go to the „LogTrail” icon.

![](/media/media/image144.png)

The main module window contains the content of messages that are automatically updated.

![](/media/media/image145.png)

Below is the search and options bar.

![](/media/media/image146.png)

It allows you to search for event logs, define the systems from which events will be displayed, define the time range for events and define the index pattern.

## Tenable.sc

Tenable.sc is vulnerability management tool, which make a scan systems and environments to find vulnerabilities. The Logstash collector can connect to Tebable.sc API to get results of the vulnerability scan and send it to the Elasticsarch index. Reporting and analysis of the collected data is carried out using a prepared dashboard `[Vulnerability] Overview Tenable`

![](/media/media/image166.png)

### Configuration

- enable pipeline in Logstash configuration:

  ```bash
  vim /etc/logstash/pipelines.yml
  ```

  uncomment following lines:

  ```bash
  - pipeline.id: tenable.sc
    path.config: "/etc/logstash/conf.d/tenable.sc/*.conf"
  ```

- configure connection to Tenable.sc manager:

  ```bash
  vim /etc/logstash/conf.d/tenable.sc/venv/main.py
  ```

  set of the connection parameters:

  - TENABLE_ADDR - IP address and port Tenable.sc manger;
  - TENABLE_CRED - user and password;
  - LOGSTASH_ADDR = IP addresss and port Logstash collector;

  example:

  ```bash
  TENABLE_ADDR = ('10.4.3.204', 443)
  TENABLE_CRED = ('admin', 'passowrd')
  LOGSTASH_ADDR = ('127.0.0.1', 10000)
  ```

## Qualys Guard

Qualys Guard is vulnerability management tool, which make a scan systems and environments to find vulnerabilities. The Logstash collector can connect to Qualys Guard API to get results of the vulnerability scan and send it to the Elasticsarch index. Reporting and analysis of the collected data is carried out using a prepared dashboard `[Vulnerability] Overview Tenable`

![](/media/media/image166.png)

### Configuration

- enable pipeline in Logstash configuration:

  ```bash
  vim /etc/logstash/pipelines.yml
  ```

  uncomment following lines:

  ```bash
  - pipeline.id: qualys
    path.config: "/etc/logstash/conf.d/qualys/*.conf"
  ```

- configure connection to Qualys Guard manager:

  ```bash
  vim /etc/logstash/conf.d/qualys/venv/main.py
  ```

  set of the connection parameters:

  - LOGSTASH_ADDR - IP address and port of the Logstash collector;

  - hostname - IP address and port of the Qualys Guard manger;
  - username - user have access to Qualys Guard manger;
  - password - password for user have access to Qualys Guard manger.

  example:

  ```bash
  LOGSTASH_ADDR = ('127.0.0.1', 10001)
  
  # connection settings
  conn = qualysapi.connect(
      username="emcas5ab1",
      password="Lewa#stopa1",
      hostname="qualysguard.qg2.apps.qualys.eu"
  )
  ```

## Embedding dashboard in iframe

It is possible to send alerts containing HTML *iframe* as notification content. For example:

```html
<a href="https://siem-vip:5601/app/kibana#/discover/72503360-1b25-11ea-bbe4-d7be84731d2c?_g=%28refreshInterval%3A%28display%3AOff%2Csection%3A0%2Cvalue%3A0%29%2Ctime%3A%28from%3A%272021-03-03T08%3A36%3A50Z%27%2Cmode%3Aabsolute%2Cto%3A%272021-03-04T08%3A36%3A50Z%27%29%29" target="_blank" rel="noreferrer">https://siem-vip:5601/app/kibana#/discover/72503360-1b25-11ea-bbe4-d7be84731d2c?_g=%28refreshInterval%3A%28display%3AOff%2Csection%3A0%2Cvalue%3A0%29%2Ctime%3A%28from%3A%272021-03-03T08%3A36%3A50Z%27%2Cmode%3Aabsolute%2Cto%3A%272021-03-04T08%3A36%3A50Z%27%29%29</a>
```

If you want an existing HTTP session to be used to display the iframe content, you need to set the following parameters in the `/etc/kibana/kibana.yml` file:

```yaml
login.isSameSite: "Lax"
login.isSecure: true
```

Possible values for *isSameSite* are: **"None", "Lax", "Strict", false**

For *isSecure*: **false or true**

