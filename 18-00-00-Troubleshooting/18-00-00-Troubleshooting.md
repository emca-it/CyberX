# Troubleshooting #

## Recovery default base indexes ##

Only applies to versions 6.1.5 and older. From version 6.1.6 and later, default indexes are created automatically

If you lost or damage following index:
		
        |Index name		 | Index ID              |
        |----------------|-----------------------|
        | .security      |Pfq6nNXOSSmGhqd2fcxFNg |
        | .taskmanagement|E2Pwp4xxTkSc0gDhsE-vvQ |
        | alert_status   |fkqks4J1QnuqiqYmOFLpsQ |
        | audit          |cSQkDUdiSACo9WlTpc1zrw |
        | alert_error    |9jGh2ZNDRumU0NsB3jtDhA |
        | alert_past     |1UyTN1CPTpqm8eDgG9AYnw |
        | .trustedhost   |AKKfcpsATj6M4B_4VD5vIA |
        | .kibana        |cmN5W7ovQpW5kfaQ1xqf2g |
        | .scheduler_job |9G6EEX9CSEWYfoekNcOEMQ |
        | .authconfig    |2M01Phg2T-q-rEb2rbfoVg |
        | .auth          |ypPGuDrFRu-_ep-iYkgepQ |
        | .reportscheduler|mGroDs-bQyaucfY3-smDpg |
        | .authuser      |zXotLpfeRnuzOYkTJpsTaw |
        | alert_silence  |ARTo7ZwdRL67Khw_HAIkmw |
        | .elastfilter   |TtpZrPnrRGWQlWGkTOETzw |
        | alert          |RE6EM4FfR2WTn-JsZIvm5Q |
        | .alertrules    |SzV22qrORHyY9E4kGPQOtg |



You may to recover it from default installation folder with following steps:

1. Stop Logstash instances which load data into cluster

		systemctl stop logstash

1. Disable shard allocation
	
		PUT _cluster/settings
		{
		  "persistent": {
		    "cluster.routing.allocation.enable": "none"
		  }
		}

1. Stop indexing and perform a synced flush

		POST _flush/synced
1. Shutdown all nodes:

		systemctl stop elasticsearch.service
1. Copy appropriate index folder from installation folder to Elasticsearch cluster data node folder (example of .auth folder)

		cp -rf ypPGuDrFRu-_ep-iYkgepQ /var/lib/elasticsearch/nodes/0/indices/

1. Set appropriate permission

		chown -R elasticsearch:elasticsearch /var/lib/elasticsearch/

1. Start all Elasticsearch instance

		systemctl start elasticsearch

1. Wait for yellow state of Elasticsearch cluster and then enable shard allocation

		PUT _cluster/settings
		{
		  "persistent": {
		    "cluster.routing.allocation.enable": "all"
		  }
		}

1. Wait for green state of Elasticsearch cluster and then start the Logstash instances

		systemctl start logstash

## Too many open files

If you have a problem with too many open files by the Elasticsearch process, modify the values in the following configuration files:

- /etc/sysconfig/elasticsearch
- /etc/security/limits.d/30-elasticsearch.conf
- /usr/lib/systemd/system/elasticsearch.service

Check these three files for:

- LimitNOFILE=65536
- elasticsearch nofile 65537
- MAX_OPEN_FILES=65537

Changes to service file require:

	systemctl daemon-reload

And changes to limits.d require:

	sysctl -p /etc/sysctl.d/90-elasticsearch.conf

## The Kibana status code 500

If the login page is displayed in Kibana, but after the attempt to login, the browser displays "error: 500", and the logs will show entries:

	Error: Failed to encode cookie (sid-auth) value: Password string too short (min 32 characters required).

Generate a new server.ironsecret with the following command:
```bash
echo "server.ironsecret: \"$(</dev/urandom tr -dc _A-Z-a-z-0-9 | head -c32)\"" >> /etc/kibana/kibana.yml
```

## Diagnostic tool

CyberX includes a diagnostic tool that helps solve your problem by collecting system data necessary for problem analysis by the support team.

The diagnostic tool is located in the installation directory: `utils/diagnostic-tool.sh`

Diagnostic tool collect the following information:

- configuration files for Kibana, Elasticsearch, Alert
- logs file for Kibana, Alert, Cerebro, Elasticsearch
- Cluster information from Elasticsearch API

When the diagnostic tool collects data, the credentials are removed from the content of the files.

### Running the diagnostic tool

To run the diagnostic tool, you must provide three parameters:
\- user assigned admin role, default 'logserver'
\- user password;
\- URL of cluster API, default: `http://localhost:9200`

Example of a command:

```bash
./diagnostic-tool.sh $user $password http://localhost:9200
```

The diagnostic tool saves the results to `.tar` file located in the user's home directory.