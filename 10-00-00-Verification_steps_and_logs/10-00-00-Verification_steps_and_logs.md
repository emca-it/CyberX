# Verification steps and logs #
## Verification of Elasticsearch service ##


To verify of Elasticsearch service you can use following command:

- Control of the Elastisearch system service via **systemd**:

		# sysetmctl status elasticsearch
  output:

		  ● elasticsearch.service - Elasticsearch
		   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; vendor preset: disabled)
		   Active: active (running) since Mon 2018-09-10 13:11:40 CEST; 22h ago
		 Docs: http://www.elastic.co
		 Main PID: 1829 (java)
		   CGroup: /system.slice/elasticsearch.service
		   └─1829 /bin/java -Xms4g -Xmx4g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -Xss1m ...


- Control of Elasticsearch instance via **tcp port**:

 		 # curl -XGET '127.0.0.1:9200/'

  output:
			
			{
			  "name" : "dY3RuYs",
			  "cluster_name" : "elasticsearch",
			  "cluster_uuid" : "EHZGAnJkStqlgRImqwzYQQ",
			  "version" : {
			    "number" : "6.2.3",
			    "build_hash" : "c59ff00",
			    "build_date" : "2018-03-13T10:06:29.741383Z",
			    "build_snapshot" : false,
			    "lucene_version" : "7.2.1",
			    "minimum_wire_compatibility_version" : "5.6.0",
			    "minimum_index_compatibility_version" : "5.0.0"
			  },
			  "tagline" : "You Know, for Search"
			}

- Control of Elasticsearch instance via **log file**:

		# tail -f /var/log/elasticsearch/elasticsearch.log

- other control commands via ***curl* application**:


		curl -xGET "http://localhost:9200/_cat/health?v"
		curl -XGET "http://localhost:9200/_cat/nodes?v"
		curl -XGET "http://localhost:9200/_cat/indicates"
## Verification of Logstash service ##


To verify of Logstash service you can use following command:

- control Logstash service via **systemd**:

		# systemctl status logstash
output:

		logstash.service - logstash
		   Loaded: loaded (/etc/systemd/system/logstash.service; enabled; vendor preset: disabled)
		   Active: active (running) since Wed 2017-07-12 10:30:55 CEST; 1 months 23 days ago
		 Main PID: 87818 (java)
		   CGroup: /system.slice/logstash.service
		          └─87818 /usr/bin/java -XX:+UseParNewGC -XX:+UseConcMarkSweepGC

- control Logstash service via **port tcp**:

		# curl -XGET '127.0.0.1:9600'
output:

		{
		   "host": "skywalker",
		   "version": "4.5.3",
		   "http_address": "127.0.0.1:9600"
		}
- control Logstash service via **log file**:

		# tail -f /var/log/logstash/logstash-plain.log

### Debuging ###

- dynamically update logging levels through the logging API (service restart not needed):

         curl -XPUT 'localhost:9600/_node/logging?pretty' -H 'Content-Type: application/json' -d'
         {
             "logger.logstash.outputs.elasticsearch" : "DEBUG"
         }
         '
- permanent change of logging level (service need to be restarted):
   - edit file */etc/logstash/logstash.yml* and set the following parameter:
  			
			*log.level: debug*

  - restart logstash service:

         	*systemctl restart logstash*

- checking correct syntax of configuration files:

		*/usr/share/logstash/bin/logstash -tf /etc/logstash/conf.d*

- get information about load of the Logstash:

		*# curl -XGET '127.0.0.1:9600/_node/jvm?pretty=true'*

output: 

	 {                                                          
	  "host" : "logserver-test",                               
	  "version" : "5.6.2",                                     
	  "http_address" : "0.0.0.0:9600",                         
	  "id" : "5a440edc-1298-4205-a524-68d0d212cd55",           
	  "name" : "logserver-test",                               
	  "jvm" : {                                                
	    "pid" : 14705,                                         
	    "version" : "1.8.0_161",                               
	    "vm_version" : "1.8.0_161",                            
	    "vm_vendor" : "Oracle Corporation",                    
	    "vm_name" : "Java HotSpot(TM) 64-Bit Server VM",       
	    "start_time_in_millis" : 1536146549243,                
	    "mem" : {                                              
	      "heap_init_in_bytes" : 268435456,                    
	      "heap_max_in_bytes" : 1056309248,                    
	      "non_heap_init_in_bytes" : 2555904,                  
	      "non_heap_max_in_bytes" : 0                          
	    },                                                     
	    "gc_collectors" : [ "ParNew", "ConcurrentMarkSweep" ]  
	  }                                                        # Verificatoin of CyberX GUI service #

To verify of CyberX GUI service you can use following command:

- control the CyberX GUI service via **systemd**:

		# systemctl status kibana

output:

		● kibana.service - Kibana                                                                                                         
		   Loaded: loaded (/etc/systemd/system/kibana.service; disabled; vendor preset: disabled)                                         
		   Active: active (running) since Mon 2018-09-10 13:13:19 CEST; 23h ago                                                           
		 Main PID: 1330 (node)                                                                                                            
		   CGroup: /system.slice/kibana.service                                                                                           
		           └─1330 /usr/share/kibana/bin/../node/bin/node --no-warnings /usr/share/kibana/bin/../src/cli -c /etc/kibana/kibana.yml 

- control the CyberX GUI via **port tcp/http**:

		# curl -XGET '127.0.0.1:5601/'

output:

		  <script>var hashRoute = '/app/kibana';
		  var defaultRoute = '/app/kibana';
		  var hash = window.location.hash;
		  if (hash.length) {
		    window.location = hashRoute + hash;
		  } else {
		    window.location = defaultRoute;
		  }</script>

- Control the CyberX GUI via **log file**:

		# tail -f /var/log/messages