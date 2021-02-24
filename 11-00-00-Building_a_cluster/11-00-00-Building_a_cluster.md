# Building a cluster #
## Node roles ##

Every instance of Elasticsearch server is called a *node*.
A collection of connected nodes is called a *cluster*.
All nodes know about all the other nodes in the cluster 
and can forward client requests to the appropriate node. 

Besides that, each node serves one or more purpose:

- Master-eligible node - A node that has *node.master* set to true (default), which makes it eligible to be elected as the master node, which controls the cluster
- Data node - A node that has *node.data* set to true (default). Data nodes hold data and perform data related operations such as CRUD, search, and aggregations 
- Client node - A client node has both *node.master* and *node.data* set to false. It can neither hold data nor become the master node. It behaves as a “*smart router*” and is used to forward cluster-level requests to the master node and data-related requests (such as search) to the appropriate data nodes 
- Tribe node - A tribe node, configured via the *tribe.** settings, is a special type of client node that can connect to multiple clusters and perform search and other operations across all connected clusters.
## Naming convention ##

Elasticsearch require little configuration before before going into work.

The following settings must be considered before going to production:

- **path.data** and **path.logs** - default locations of these files are:
`/var/lib/elasticsearch`and `/var/log/elasticsearch`.
- **cluster.name** - A node can only join a cluster when it shares its 
`cluster.name` with all the other nodes in the cluster. The default name 
is "elasticsearch", but you should change it to an appropriate name which 
describes the purpose of the cluster. You can do this in `/etc/elasticsearch/elasticsearch.yml` file.
- **node.name** - By default, Elasticsearch will use the first seven characters of the randomly 
generated UUID as the node id. Node id is persisted and does not change when a node restarts.
It is worth configuring a more human readable name: `node.name: prod-data-2`
in file `/etc/elstaicsearch/elasticsearch.yml`
- **network.host** - parametr specifying network interfaces to which Elasticsearch can bind. 
Default is `network.host: ["_local_","_site_"]`.
- **discovery** - Elasticsearch uses a custom discovery implementation called "Zen Discovery".
There are two important settings:
    - `discovery.zen.ping.unicast.hosts` - specify list of other nodes in the cluster that are 
      likely to be live and contactable;
    - `discovery.zen.minimum_master_nodes` - to prevent data loss, you can configure this setting
      so that each master-eligible node knows the minimum number of master-eligible nodes that must
      be visible in order to form a cluster.
- **heap size** - By default, Elasticsearch tells the JVM to use a heap with a minimum (Xms) and maximum (Xmx)
size of 1 GB. When moving to production, it is important to configure heap size to ensure that 
Elasticsearch has enough heap available

## Config files ##


To configure the Elasticsearch cluster you  must specify some parameters
in the following configuration files on every node that will be connected to the cluster:

- `/etc/elsticsearch/elasticserach.yml`:
    - `cluster.name:name_of_the_cluster` - same for every node;
    - `node.name:name_of_the_node` - uniq for every node;
    - `node.master:true_or_false`
    - `node.data:true_or_false`
    - `network.host:["_local_","_site_"]`
    - `discovery.zen.ping.multicast.enabled`
    - `discovery.zen.ping.unicast.hosts`
- `/etc/elsticsearch/log4j2.properties`:
  - `logger: action: DEBUG` - for easier debugging.

## Example setup ##

Example of the Elasticsearch cluster configuration:
- file `/etc/elasticsearch/elasticsearch.yml`:

		cluster.name: tm-lab
		node.name: "elk01"
		node.master: true
		node.data: true
		network.host: 127.0.0.1,10.0.0.4
		http.port: 9200
		discovery.zen.ping.multicast.enabled: false
		discovery.zen.ping.unicast.hosts: ["10.0.0.4:9300","10.0.0.5:9300","10.0.0.6:9300"]


- to start the Elasticsearch cluster execute command:

		# systemctl restart elasticsearch

- to check status of the Elstaicsearch cluster execute command:
    - check of the Elasticsearch cluster nodes status via tcp port:

			# curl -XGET '127.0.0.1:9200/_cat/nodes?v'
	
			host         	  ip           heap.percent ram.percent load node.role master name
			10.0.0.4 	 10.0.0.4     18           91 		   0.00 -         -      elk01
			10.0.0.5 	 10.0.0.5     66           91 		   0.00 d        *      elk02
			10.0.0.6 	 10.0.0.6     43           86         	   0.65 d        m     elk03
			10.0.0.7 	 10.0.0.7     45           77         	   0.26 d        m     elk04
    
    - check status of the Elasticsearch cluster via log file:
    
			# tail -f /var/log/elasticsearch/tm-lab.log (cluster.name)

## Adding a new node to existing cluster ##

Install the new CyberX instance. The description of the installation can be found in the chapter "First configuration steps"

Change the following parameters in the configuration file:

- `cluster.name:`name_of_the_cluster same for every node;
- `node.name:`name_of_the_node uniq for every node;
- `node.master:`true_or_false
- `node.data:`true_or_false
- `discovery.zen.ping.unicast.hosts:`["10.0.0.4:9300","10.0.0.5:9300","10.0.0.6:9300"] - IP addresses and instances of nodes in the cluster.

If you add a node with the role `data`, delete the contents of the `path.data` directory, by default in `/var/lib/elasticsearch`

Restart the Elasticsearch instance of the new node:
	

```/
systemctl restart elasticsearch
```

## Cluster HOT-WARM-COLD architecture

Let's assume we have 3 zones **hot, warm, cold**, with three servers in each zone.

Each of the servers in all zones must have an active Data Node license.

1. We have 3 shards for each index.

2. When indexes are created, they stay in the HOT zone for 3 days.

3. After 3 days, the "rollower" mechanism is activated and the indexes are moved to the "WARM" zone, for example `logs_write` index:

   ```json
   POST /logs_write/_rollover
   {
     "conditions" : {
       "max_age": "3d",
       "max_docs": 100000000,
       "max_size": "5gb"
     },
     "settings": {
       "index.number_of_shards": 3
       "index.routing.allocation.require._name": "server-warm-1"
     }
   }
   ```

4. After the next 7 days, the indexes are moved to the COLD zone as follows:

   - write in index is blocked and relocation to COLD zone is set:

      ```json
      PUT active-logs-1/_settings
      {
        "index.blocks.write": true,
        "index.routing.allocation.require._name": "serwer-cold-1"
      }
      ```

   - the number of shards is reduced to 1:

      ```bash
      POST active-logs-1/_shrink/inactive-logs-1
      ```

   - the number of segments is reduced to 1:

      ```bash
      POST inactive-logs-1/_forcemerge?max_num_segments=1
      ```

5. As a result, after 10 days, the `inactive-logs-1` index is on the server in the COLD zone and has 1 shard and 1 segment.