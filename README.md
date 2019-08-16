# jetL2 - l2 crypto market data

1. Binance data historical prototype
2. 10+ exchanges historical data
3. realtime stream

![](pipeline.jpg?raw=true)


Components:

1. Ingestor
- establish a connection to exchanges via websocket or FIX protocol
- write raw event from exchanges to Kafka
- service in Rust 
- is running on Kubernetes https://aws.amazon.com/eks/
- should enrich trade event with timestamp, data type, exchange, trading pair

2. Kafka
- is running on https://aws.amazon.com/msk/
- topic per exchange, trading pairs are distributed by partitions
- how many partitions are efficient for one topic
- how many events will be written per second into one partition
- take into account order of the events
- retention policy?

3. Kafka stream
- kafka stream processor is deployed to Kafka cluster
- Scala or Java api only
- parses events into a certain Avro message
- implements stateful logic where snapshot is merged with delta event
- writes result message into a separate Kafka topic
- retention policy?

4. Kafka to S3
- Spark job in python
- running on EMR cluster on demand
- Kafka offsets are stored in Kafka itself
- launched by some trigger, setup EMR cluster and run the job
- reads available Avro events from Kafka
- groups data by Date, Data Type, Trading pair, Exchange
- writes data as Parquet files (should be compressed?)

5. S3
- partitioned and parsed raw trading data
- by Date, Data Type, Trading pair, Exchange
- what will be the main dimensions for requested data? All pairs and particular pairs for an interval? OR All exchanges for - - certain pairs for an interval?
- some partionaly public directory for download ready data. Dimensions can be cached in order to not prepare similar package of data several times
- need to measure volume of data

6. Client API service
- service in Rust to get requests from client
- triggers a Spark job which collect required data
- running on Kubernetes  https://aws.amazon.com/eks/
- what will be the main dimensions for requested data?

7. S3 to download ready data on S3
- Spark job in Python
- triggered by Client API
- running on demand EMR cluster
- collects required data and prepares convenient files in csv format (or FIX)
- writes compressed data to a special S3 bucket which is available for downloading
- how long the data should be stored in this bucket?
- should we have some cache to avoid duplications?
