Please use a fresh Ubuntu 16.04 machine. All instructions have been tested on fresh Ubuntu 16.04

# System Installation

1. Use sudo (Must switch to root privilege)  

  ``` sudo -i ```

2. Install Java

  ``` apt-get update```

  ``` add-apt-repository ppa:webupd8team/java ``` 

  ``` apt-get update``` 
  
  ``` apt-get install -y oracle-java8-installer ```     
  // You must accept Oracle License
  
  ``` apt-get install -y oracle-java8-set-default``` 

3. Very Java

  ``` javac -version``` 

4. Install Docker

  ``` apt-get install -y curl``` 
  
  ``` curl -sSL https://get.docker.com/ | sh``` 
  
  ``` curl -L "https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose``` 
  
  ``` chmod +x /usr/local/bin/docker-compose``` 

5. Verify Docker Installation

  > docker ps

# Mongo Cluster Setup

1. Create workspace

  > mkdir xanadu_install
  > cd xanadu_install

2. Copy docker-compose.yaml to workspace directory, xanadu_install

3. Setup cluster

  >_ docker-compose up -d    
// Please wait for 3-4 mins to let Mongo Install

4. Verify Cluster Setup

    >_ docker ps  // You should see 10 docker containers running now.

# Mongo DB Setup

1. Connect to MongoS  

  >_ docker exec -it xanaduinstall_mongo-s-01_1 bash 

2. Connect to MongoS shell 

  >_ mongo —-host 127.0.0.1 —-port 27017 

3. Check status

  mongos> sh.status() 

Now, ensure that cluster is stable and has 2 shards. You should see something like this in the response. You must see 2 shards like the one here in bold.  --- Sharding Status ---
  sharding version: {
	"_id" : 1,
	"minCompatibleVersion" : 5,
	"currentVersion" : 6,
	"clusterId" : ObjectId("58abf6926635fbcc6a3e08f8")
}
  shards:
	{  "_id" : "rep1",  "host" : "rep1/mongo-d-01:27017,mongo-d-02:27017" }
	{  "_id" : "rep2",  "host" : "rep2/mongo-d1-01:27017,mongo-d1-02:27017" }
  active mongoses:
	"3.2.3" : 1
  balancer:
	Currently enabled:  yes
	Currently running:  no
	Failed balancer rounds in last 5 attempts:  0
	Migration Results for the last 24 hours:
		No recent migrations
  databases:  

4. Now, let’s enable a DB, say Xanadu and enable sharing for it.  

  mongos> use admin; 
  mongos> sh.enableSharding(“Xanadu”); {“ok”: 1} 

5. Assign a shard key  

  mongos> use Xanadu; 
  mongos> db.user.createIndex({_id: 1});  

6. Now, shard collection  

  mongos> sh.shardCollection( "Xanadu.user", { "_id" : 1 } );
 { "collectionsharded" : "Xanadu.user", "ok" : 1 }  

# Run Java Program

1. Keep xanadu_test_mongo.jar and config.properties to workspace directory, xanadu_install 

2. Run program  

  >_ java -jar xanadu_test_mongo.jar 

3. Monitor logs -

2017-02-21T00:39:07.253 INFO [com.xanadu.poc.Main] - =>recordCount: 30=>workerCount:2
2017-02-21T00:39:07.422 INFO [org.mongodb.driver.cluster] - Cluster created with settings {hosts=[127.0.0.1:27017], mode=SINGLE, requiredClusterType=UNKNOWN, serverSelectionTimeout='30000 ms', maxWaitQueueSize=500}
2017-02-21T00:39:07.525 INFO [org.mongodb.driver.connection] - Opened connection [connectionId{localValue:1}] to 127.0.0.1:27017
.
.
.
.
2017-02-21T00:39:08.153 INFO [org.mongodb.driver.connection] - Closed connection [connectionId{localValue:8}] to 127.0.0.1:27017 because the pool has been closed.
2017-02-21T00:39:08.154 INFO [com.xanadu.poc.Main] - =>main() Reading records complete @:1487666348154
2017-02-21T00:39:08.154 INFO [com.xanadu.poc.Main] - =>main() Time taken writer:307 ms
2017-02-21T00:39:08.154 INFO [com.xanadu.poc.Main] - =>main() Time taken reader:24 ms

