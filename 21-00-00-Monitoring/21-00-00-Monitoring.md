# Monitoring #

## Skimmer ##

CyberX uses a monitoring module called Skimmer to monitor the performance of its hosts. Metrics and conditions of services are retrieved using the API.

The services that are supported are:

- Elasticsearch;
- Logstash;
- Kibana;
- Metricbeat;
- Pacemaker;
- Zabbix;
- Zookeeper;
- Kafka;
- Httpbeat;
- Elastalert;
- Filebeat

and other.

## Skimmer Installation ##

The RPM package skimmer-x86_64.rpm is delivered with the system installer in the "utils" directory:

		cd $install_direcorty/utils
		yum install skimmer-x86_64.rpm -y

## Skimmer service configuration ##

The Skimmer configuration is located in the `/usr/share/skimmer/skimmer.conf` file.

	# index name in elasticsearch
	index_name = skimmer
	index_daily = true
	# type in elasticsearch index
	index_type = _doc
	# user and password to elasticsearch api
	elasticsearch_auth = logserver:logserver
	# available outputs
	elasticsearch_address = 127.0.0.1:9200
	# logstash_address = 127.0.0.1:6110
	# retrieve from api
	elasticsearch_api = 127.0.0.1:9200
	# logstash_api = 127.0.0.1:9600
	# path to log file
	log_file = /tmp/skimmer.log
	# daemonize
	daemonize = true
	# comma separated OS statistics selected from the list [zombie,vm,fs,swap,net,cpu]
	os_stats = zombie,vm,fs,swap,net,cpu
	# comma separated process names to print their pid
	processes = /usr/sbin/sshd,/usr/sbin/rsyslogd
	# comma separated systemd services to print their status
	systemd_services = elasticsearch,logstash,kibana
	# comma separated port numbers to print if address is in use
	port_numbers = 9200,9300,9600,5601
	# path to directory containing files needed to be csv validated
	csv_path = /tmp/csv_dir

After the changes in the configuration file, restart the service.

	systemctl restart skimmer

### Skimmer GUI configuration ###

To view the collected data by the skimmer in the GUI, you need to add an index pattern.

Go to the "**Management**" -> "**Index Patterns**" tab and press the "**Create Index Pattern**" button. In the "**Index Name**" field, enter the formula `skimmer- *`, and select the "**Next step**" button. In the "**Time Filter**" field, select `@timestamp` and then press "**Create index pattern**"

In the "**Discovery**" tab, select the `skimmer- *` index from the list of indexes. A list of collected documents with statistics and statuses will be displayed.