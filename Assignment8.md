
## This file has few main steps in the history file.

## Create spark-with-kafka directory, navigate to the directory and copy the docker-compose file into the directory
#### mkdir spark-with-kafka
#### cd ~/w205/spark-with-kafka
#### cp ../course-content/07-Sourcing-Data/docker-compose.yml .

## Spin up the container
#### docker-compose up -d

## Check hodoop
#### docker-compose exec cloudera hadoop fs -ls /tmp/

## Create the topic assessment
#### docker-compose exec kafka   kafka-topics     --create     --topic assessment     --partitions 1     --replication-factor 1     --if-not-exists     --zookeeper zookeeper:32181

## Check the topic
#### docker-compose exec kafka   kafka-topics   --describe   --topic assessment   --zookeeper zookeeper:32181

## Publish things to kafka and produced 42 messages.
#### docker-compose exec kafka   bash -c "seq 42 | kafka-console-producer \--request-required-acks 1 \--broker-list kafka:29092 \--topic assessment && echo 'Produced 42 messages.'"

## Spin down the container
#### docker-compose down

## Spin up the container
#### docker-compose up -d

## Pull assessment data
#### curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp

## Create assessment topic
#### docker-compose exec kafka   kafka-topics     --create     --topic assessment     --partitions 1     --replication-factor 1     --if-not-exists     --zookeeper zookeeper:32181

## Check the topic
#### docker-compose exec kafka   kafka-topics   --describe   --topic assessment   --zookeeper zookeeper:32181

## Publish messages to the topic with kafkacat
#### docker-compose exec mids bash -c "cat /w205/spark-with-kafka/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessment && echo 'Produced 100 messages.'"

## Bring up pyspark
#### docker-compose exec spark pyspark

## Read from kafka
#### messages = spark \.read \.format("kafka") \.option("kafka.bootstrap.servers", "kafka:29092") \.option("subscribe","assessment") \.option("startingOffsets", "earliest") \.option("endingOffsets", "latest") \.load() 

## Print the schema
#### messages.printSchema()

## Transform key and value
#### messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")


## View data transformation
#### messages_as_strings.show()
#### messages_as_strings.printSchema()
#### messages_as_strings.count()

## Play with json
#### import json
#### first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)

## Save to hdfs
#### messages_as_strings.write.parquet("/tmp/assessment")

## Show data frame
#### messages_as_strings.rdd.map(lambda x: json.loads(x.value)).toDF().show()

## Taken a df, mapped it onto an rdd, convert back to a spark dataframe and show
#### extracted_assessment = messages_as_strings.rdd.map(lambda x: json.loads(x.value)).toDF()

## Save to hdfs
#### extracted_assessment.write.parquet("/tmp/extracted_assessment")

## View transformation
#### extracted_assessment.show()

## Exit pyspark
#### Ctr+d

## Spin down the container
#### docker-compose down

## Save to history file
#### history > hong-history.txt


