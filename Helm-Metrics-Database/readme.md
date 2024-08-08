## Helm
#### Elasticsearch helm template 
![Helm deployments](Helm-Metrics-Database/Helm.png)


## Metrics:

#### Explaining how Prometheus work.
- Prometheus collects metrics from configured endpoints at specific intervals and stores them in a time-series
database, it allows querying and visualization through its query language called PromQL.

#### What is the Prometheus query you can use in Granfana to properly show usage trend of an application metric that is a counter?
- In Granfana to show the usage trend of a counter metric, I can use **rate()** function, ex: rate(my_counter_metric[5m])


## Database:

### Cassandra
#### Query to db cluster returns different result each time.  Users reported query result has data records that they deleted days ago. Explaining what the likely reason for the behavior and how to avoid it.
- Returning deleted records is likely due to Cassandra's eventual consistency model and hinted handoff. Deleted data might still be present due to replication delays. 
- To avoid: We can use **nodetool repair** to ensure all have the same data.
	
#### MongoDB Sharding Steps
	First enabling Sharding on the db using admin:
	mongos> use admin
	mongos> sh.enableSharding("sanfrancisco")
	
	Choosing a shard key and create the shard key index:
	mongos> use sanfrancisco
	mongos> db.company_name.createIndex({_id: 1})
	
	Shard the collection:
	mongos> sh.shardCollection("sanfrancisco.company_name", {_id: 1})

	verify sharding status:
	mongos> sh.status()


