# Introduction #

CyberX is innovation solution allowing for centralize IT
systems events. It allows for an immediately review, analyze and
reporting of system logs - the amount of data does not matter. 
CyberX is a response to the huge demand for storage and
analysis of the large amounts of data from IT systems. 
CyberX is innovation solution that responds to the need of
effectively processing large amounts of data coming from IT
environments of today's organizations. Based on the open-source
project Elasticsearch valued on the marked, we have created an
efficient solution with powerful data storage and searching
capabilities. The System has been enriched of functionality that
ensures the security of stored information, verification of users,
data correlation and visualization, alerting and reporting.

![](/media/media/image2_js.png) 

CyberX project was created to centralize events of all IT
areas in the organization. We focused on creating a tool that
functionality is most expected by IT departments. Because an effective
licensing model has been applied, the solution can be implemented in
the scope expected by the customer even with very large volume of
data. At the same time, the innovation architecture allows for
servicing a large portion of data, which cannot be dedicated to
solution with limited scalability.

## Elasticsearch ##

Elasticsearch is a NoSQL database solution that is the heart of our
system. Text information send to the system, application and system
logs are processed by Logstash filters and directed to Elasticsearch.
This storage environment creates, based on the received data, their
respective layout in a binary form, called a data index. The Index is
kept on Elasticsearch nodes, implementing the appropriate assumptions
from the configuration, such as:

- Replication index between nodes,
- Distribution index between nodes.

The Elasticsearch environment consists of nodes:

- Data node - responsible for storing documents in indexes,
- Master node - responsible for the supervisions of nodes,
- Client node - responsible for cooperation with the client.

Data, Master and Client elements are found even in the smallest
Elasticsearch installations, therefore often the environment is
referred to as a cluster, regardless of the number of nodes
configured. Within the cluster, Elasticsearch decides which data
portions are held on a specific node.

Index layout, their name, set of fields is arbitrary and depends on
the form of system usage. It is common practice to put data of a
similar nature to the same type of index that has a permanent first
part of the name. The second part of the name often remains the date
the index was created, which in practice means that the new index is
created every day. This practice, however, is conventional and every
index can have its own rotation convention, name convention,
construction scheme and its own set of other features. As a result of
passing document through the Logstash engine, each entry receive a
data field, which allow to work witch data in relations to
time. 

The Indexes are built with elementary part called shards. It is good
practice to create Indexes with the number of shards that is the
multiple of the Elasticsearch data nodes number. Elasticsearch in 7.x version has a new feature called Sequence IDs that guarantee more successful and efficient shard recovery.

Elasticsearch use the *mapping* to describes the fields or properties that documents of that type may have. Elasticsearch in 7.x version restrict indices to a single type.# Kibana #

## Kibana ##
Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack. Kibana gives you the freedom to select the way you give
shape to your data. And you don’t always have to know what you're looking for. Kibana core ships with the classics: histograms, line
graphs, pie charts, sunbursts, and more. Plus, you can use Vega grammar to design your own visualizations. All leverage the full
aggregation capabilities of Elasticsearch.  Perform advanced time series analysis on your Elasticsearch data with our curated time series
UIs. Describe queries, transformations, and visualizations with powerful, easy-to-learn expressions. Kibana 7.x has two new feature - a new "Full-screen" mode to viewing dashboards, and new the "Dashboard-only" mode which enables administrators to share dashboards safely.# Logstash #

## Logstash ##
Logstash is an open source data collection engine with real-time pipelining capabilities. Logstash can dynamically unify data from
disparate sources and normalize the data into destinations of your choice. Cleanse and democratize all your data for diverse advanced
downstream analytics and visualization use cases.

While Logstash originally drove innovation in log collection, its capabilities extend well beyond that use case. Any type of event can be
enriched and transformed with a broad array of input, filter, and output plugins, with many native codecs further simplifying the
ingestion process. Logstash accelerates your insights by harnessing a greater volume and variety of data.

Logstash 7.x version supports native support for multiple pipelines. These pipelines are defined in a *pipelines.yml* file which is loaded by default.
Users will be able to manage multiple pipelines within Kibana. This solution uses Elasticsearch to store pipeline configurations and allows for on-the-fly reconfiguration of Logstash pipelines.# ELK #

## ELK ##
"ELK" is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics 
engine. Logstash is a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, 
and then sends it to a "stash" like Elasticsearch. Kibana lets users visualize data with charts and graphs in Elasticsearch.
The Elastic Stack is the next evolution of the ELK Stack.
