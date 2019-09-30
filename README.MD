# High Level Schema

![Image of Yaktocat](https://github.com/dancamma/data-pipeline-ingestion/raw/master/data_ingestion_pipeline.png)

# Description

I've created a data ingestion pipeline integrated with AWS.

I've based the data ingestion mechanism on the concept of queue which offers the following benefits:

- A buffer mechanism, which makes the pipeline robust to peaks of load
- A retry policy, which guarantees that messages are not lost if some failure in the system occurs.

Here we have many different kinds of sources with very different requirements. Using queues, I can put myself at a higher level of abstraction, which led me to a classification of 2 kinds of sources:

- Push resource, in which the source can push data on-the-fly to the stream (e.g., an application generating events)
- Pull resource, in which events are accumulating, and I have to pull them periodically.

In the system, every source of data has a different format. I've introduced the first layer of streams for this purpose. Each source has its stream, which is responsible for collecting events from the source and translating them in a standard format. To accomplish this, I've chosen AWS Kinesis Firehose with AWS Lambda for record transformation.
Once events are translated in the standard format, they are delivered to another Kinesis Firehose Stream, which is responsible for collecting all events and delivering them to a Data Lake, based on Amazon S3.

Then I have to deliver those data to users through API. Since probably not all the data that are in the data lake need to be delivered, I've introduced a MapReduce job which is responsible for retrieving and selecting data from the data lake and pushing them to an instance of Amazon RDS.

Then, using Amazon AppSync, I'm able to offer this data through a GraphQL API. This is as simple as writing the GraphQL schema and resolvers.

The initial Bulk load can be made directly in the data lake.

# Pros / Cons

The solution is 100% cloud-native. It is 100% managed with all the advantages of being on the AWS platform: scalability, fault tolerance, no need to have a team dedicated to the operation of the system.

However, the costs could be very high if the number of events passing through Firehose is enormous. One possible solution could be to substitute Firehose with a managed solution, such as deployment of Kafka and Kafka Streams.

The system is also modular. Adding a new source of data is as simple as adding a new Firehose stream in the first layer.
