# Real-time Trade Monitor

A sample dashboard which uses [Hazelcast](https://github.com/hazelcast/hazelcast)
to ingest trades from Apache Kafka into a distributed map. It also performs
an aggregation on the trades, storing the results into a separate map.

These two maps are utilized by a live dashboard which offers drill down
functionality into viewing individual trades that make up the aggregation.

## Run the Demo on Hazelcast Viridian:

1. Open the Hazelcast Viridian console, click *Connect Client*, and go to the *Advanced setup* tab. Leave this window open. You'll need some of these details in the next step.

2. Download the `client.keystore` and `client.truststore` files and save them to `/realtime-trade-monitor/trade-queries/src/main/resources` and `/realtime-trade-monitor/webapp/src/main/resources`.

3. Add the following details to the `hazelcast-client.yaml` configuration files in the `/realtime-trade-monitor/trade-queries/src/main/resources` and `/realtime-trade-monitor/webapp/src/main/resources` directories:

- Cluster name
- Discovery token
- Keystore and truststore password
- Absolute path to the `client.keystore` and `client.truststore` files

3. You need a Kafka cluster that is reachable by Hazelcast Viridian. For demo purposes, the easiest way is
   to create the simplest Kafka cluster on https://confluent.cloud with defaults.

4. Create topic `trades` with 4 partitions. If you use https://confluent.cloud, go to Topics section in the UI.

5. Put all kafka consumer/producer properties in `trade-producer/src/main/resources/kafka.properties` and
   `trade-queries/src/main/resources/kafka.properties`. If you use https://confluent.cloud you can find them in:
   `Data Integration > Client > New Client section`. The CLUSTER_API_KEY and CLUSTER_API_SECRET placeholders
   must be replaced with the values available at: `Data integration > API Keys`.

For the Confluent.cloud the config should look like this:
```
# Required connection configs for Kafka producer, consumer, and admin
bootstrap.servers=change-me.us-west-2.aws.confluent.cloud:9092
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username='<CLUSTER_API_KEY>'   password='<CLUSTER_API_SECRET>';
sasl.mechanism=PLAIN
# Required for correctness in Apache Kafka clients prior to 2.6
client.dns.lookup=use_all_dns_ips

# Best practice for Kafka producer to prevent data loss
acks=all
```

6. Build the project

```
mvn package
```

7. Start the Kafka producer

```
java -jar trade-producer/target/trade-producer-5.0.jar "" <trades per sec>
```

8. Run the queries

* Load static data into map: (Stock names)
```
java -jar trade-queries/target/trade-queries-5.0.jar load-symbols
```

* Ingest trades from Kafka

```
java -jar trade-queries/target/trade-queries-5.0.jar ingest-trades ""
```
* Aggregate trades by symbol
```
java -jar trade-queries/target/trade-queries-5.0.jar aggregate-query ""
```

9. Start the front end

```
java -jar webapp/target/webapp-5.0.jar 
```

10. Browse to [http://localhost:9000](http://localhost:9000) to see the dashboard
