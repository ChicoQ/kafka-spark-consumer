README file for Kafka-Spark-Consumer
===================================

NOTE : This Kafka Spark Consumer code is taken from Kafka spout of the Apache Storm project (https://github.com/apache/storm/tree/master/external/storm-kafka), 
which was originally created by wurstmeister (https://github.com/wurstmeister/storm-kafka-0.8-plus).
Original Storm Kafka Spout Code has been modified to work with Spark Streaming.

 

This utility will help to pull messages from Kafka Cluster using Spark Streaming.
The Kafka Consumer is Low Level Kafka Consumer ( SimpleConsumer) and have better handling of the Kafka Offsets and handle failures.

This code have implemented a Custom Receiver consumer.kafka.client.KafkaReceiver. The KafkaReceiver uses low level Kafka Consumer API (implemented in consumer.kafka packages) to fetch messages from Kafka and 'store' it in Spark.

The logic will detect number of partitions for a topic and spawn that many Kafka Receivers.
Kafka Receivers uses Zookeeper for storing the latest offset for individual partitions, which will help to recover in case of failure .

The consumer.kafka.client.Consumer is the sample Consumer which uses this Kafka Receivers to generate DStreams from Kafka and apply a Output operation for every messages of the RDD.

We use this Kafka Spark Consumer to perform Near Real Time Indexing of Kafka Messages to target Search Cluster and also Near Real Time Aggregation using target NoSQL storage.    

Following are the instructions to build 
========================================

>git clone

>cd kafka-spark-consumer

>mvn install

Integrating Spark-Kafka-Consumer
=================================

If you want to use this Kafka-Spark-Consumer for you target client application, include below dependency in your pom.xml

                <dependency>
                        <groupId>kafka.spark.consumer</groupId>
                        <artifactId>kafka-spark-consumer</artifactId>
                        <version>0.0.1-SNAPSHOT</version>
                </dependency>

				
and use spark-kafka.properties to include below details.

* Kafka ZK details from where messages will be pulled. Speficy ZK Host IP address
	* zookeeper.hosts=host1,host2
* Kafka ZK Port
	* zookeeper.port=2181
* Kafka Broker path in ZK
	* zookeeper.broker.path=/brokers
* Kafka Topic to consume
	* kafka.topic=topic-name

* Consumer ZK Path. This will be used to store the consumed offset. Please specify correct ZK IP and Port
	* zookeeper.consumer.connection=x.x.x.x:2181
* ZK Path for storing Kafka Consumer offset
	* zookeeper.consumer.path=/spark-kafka
* Kafka Consumer ID. This ID will be used for accessing offset details in $zookeeper.consumer.path
	* kafka.consumer.id=12345


Running Spark Kafka Consumer
===========================
Let assume your Kafka Message Processing logic (The implementation of consumer.kafka.client.IIndexer interface) is in custom-processor.jar which is built using the spark-kafka-consumer as dependency.

Launch this using spark-submit

./bin/spark-submit --class consumer.kafka.client.Consumer --master spark://x.x.x.x:7077 --executor-memory 5G /<Path_To>/custom-processor.jar -P /<Path_To>/spark-kafka.properties


-P external properties filename
-p properties filename from the classpath

This will start the Spark Receiver and Fetch Kafka Messages for every partition of the given topic and generates the DStream. The external IIndexer will process every messages in the given RDD. 

 
