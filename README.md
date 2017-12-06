Kafka Depends on Zookeeper 

Step 1: Start Zookeeper zookeeper-server-start.bat
(may need additional configuration) passing configuration is mandatory 

.zookeeper-server-start.bat ....configzookeeper.properties

**************************************************************************************

Step 2: Start kafka server (if needed change the properties, and you may have to 
change the time out from 6000 to 60000) 

.kafka-server-start.bat ....configserver.properties

// for clustering referring to new properties file 

.kafka-server-start.bat ....configserver-1.properties
.kafka-server-start.bat ....configserver-2.properties
**************************************************************************************


Step 3: create a topic  (You can create multiple topics ) 
	.kafka-topics.bat --create --zookeeper localhost:2181 --topic exitopic1 --partitions 3 --replication-factor 1

	.kafka-topics.bat --create --zookeeper localhost:2181 --topic exitopic2 --partitions 3 --replication-factor 1

// creating topics for clustering 
.kafka-topics.bat --create --zookeeper localhost:2181 --topic server1topic1 --partitions 3 --replication-factor 1
.kafka-topics.bat --create --zookeeper localhost:2181 --topic server1topic2 --partitions 3 --replication-factor 1

.kafka-topics.bat --create --zookeeper localhost:2181 --topic server2topic1 --partitions 3 --replication-factor 1
.kafka-topics.bat --create --zookeeper localhost:2181 --topic server2topic2 --partitions 3 --replication-factor 1


**************************************************************************************


Step 4: to list the topics which are created 

.kafka-topics.bat --zookeeper localhost:2181 --list
**************************************************************************************
Step 5: to describe the topic 
.kafka-topics.bat --zookeeper localhost:2181 --topic exitopic1 --describe
**************************************************************************************

Step 6: Create a producer 
.kafka-console-producer.bat --topic exitopic1 --broker-list localhost:9092

//for multiple cluster 
.kafka-console-producer.bat --topic server1topic1 --broker-list localhost:9093

.kafka-console-producer.bat --topic server2topic1 --broker-list localhost:9094
**************************************************************************************
Step 7: Create a consumer ( you can create multiple consumers for the same producer) 

.kafka-console.consumer.bat --topic server1topic1 --bootstrap-server localhost:9093
**************************************************************************************





package com.exilant;

import java.util.Properties;

import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

// documentation 
// https://kafka.apache.org/documentation/#producerapi
// to configure producer 


public class KafkaProducer {
	public static void main(String[] args) {
		// First lets list the properties 
		
		Properties properties = new Properties();
		
		properties.setProperty("bootstrap.servers", "localhost:9093");
		properties.setProperty("key.serializer", StringSerializer.class.getName());
		properties.setProperty("value.serializer", StringSerializer.class.getName());
		
		properties.setProperty("acks", "1");
		properties.setProperty("retries", "3");
		
		Producer<String, String> producer = new 
				org.apache.kafka.clients.producer.KafkaProducer
				<String, String>(properties);
		
		ProducerRecord<String, String> record1 = new 
				ProducerRecord<String, String>
				("server1topic1", "1", "Hello From Java111");
		
		ProducerRecord<String, String> record2 = new 
				ProducerRecord<String, String>
				("server1topic1", "2", "Hello From Java again");
		
		producer.send(record1);
		producer.send(record2);
		
		
		producer.flush();
		producer.close();
		
		
		System.out.println("Sent... ");
		
		
		
		
	}
}



-----------------------------------------------------------------------------------------


package com.exilant;

import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;

public class KafkaConsumerClient {
	public static void main(String[] args) {
		
		Properties properties = new Properties();
		
		properties.setProperty("bootstrap.servers", "localhost:9093");
		properties.setProperty("key.deserializer", 
				StringDeserializer.class.getName());
		
		properties.setProperty("value.deserializer", 
				StringDeserializer.class.getName());
		
		properties.setProperty("group.id", "Group2");
		properties.setProperty("session.timeout.ms", "30000");
		properties.setProperty("auto.offset.reset", "earliest");
		
		KafkaConsumer<String, String> kafkaConsumer = 
			new KafkaConsumer<String, String>(properties);
		
		// topic -> server1topic1
		kafkaConsumer.subscribe(Arrays.asList("server1topic1"));
		
		// 
		while(true){
			ConsumerRecords<String, String> consumerRecords = 
				kafkaConsumer.poll(1000);
			
			
			System.out.println("Number of Messages Got " 
					+ consumerRecords.count());
			
			for(ConsumerRecord<String, String> consumerRecord: 
					consumerRecords){
				System.out.println("---------------------------------");
				System.out.println("OffSet Value: " + consumerRecord.offset());
				System.out.println("Topic is " + consumerRecord.topic());
				System.out.println("Time Stamp : " + consumerRecord.timestamp());
				System.out.println("Partition which read from " + 
						consumerRecord.partition());
				System.out.println("Key : " + consumerRecord.key());
				System.out.println("Value : " + consumerRecord.value());
			}
			
		}
		
		
		
		
		
	}
}
