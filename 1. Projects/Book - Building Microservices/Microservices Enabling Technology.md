---
created: 2023-08-24 21:19
alias: 
---
#### Summary
+ 

----
# Microservices Enabling Technology

Lots of technology is involved in enabling [[Microservices]] in production. However, don't immediately adopt lots of new tech. As you ramp up, look for issues caused by your existing distributed system, then look for tech to mitigate them. Else it's very easy to get completely overwhelmed by the laundry list of new technologies to learn.

> Microservices require an understanding of the supporting technology to such a degree that previous distinctions between logical and physical architecture can be problematic.

## Log Aggregation and [[Distributed Tracing]]

At the bare minimum, implement a [[Log Aggregation]] system. Consider this a prerequisite for microservices.

It allows you to collect and aggregate logs from across services, provides a central place for analysis, and can provide active alerting mechanisms against issues and downtime.

> ☝ Sam Newman recommends Humio for log aggregation.

It's a very good idea to implement [[Correlation IDs]]. This is a single ID used for a related set of service calls. It makes isolating logs for a particular request flow easy.

> ☝ Jaeger: distributed tracing tool. 
> ☝ Lightstep and Honeycomb: to make it easier to explore the state of your running system.

## Containers and [[Kubernetes]]

Containers provide a lightweight way to provision isolated execution for service instances, leading to faster spin up time, making it more cost effective. Container orchestration platforms (Kubernetes) allow managing containers across lots of underlying machines.

Only adopt containers/K8s once the overhead of managing deployment becomes a headache. It's not worth it for very few microservices.

If using a K8s cluster, ensure someone else manages the Kubernetes cluster for you (managed service on cloud provider). Running your own K8s can be a significant amount of work!

> ☝ Container software include Podman, Buildah, and of course, [[Docker]].

## Data Streaming

[[Apache Kafka]] is probably the right bet for streaming data between microservice, offering message permanence, compaction, scale to handle large volumes of messages and stream processing in the form of [[ksqlDB]].

> ☝ Debezium is an open source software to stream from traditional data sources to Kafka, allowing data streaming from things like SQL DBs.

## Public Cloud & [[Serverless]]

The Big Three -- GCP, Azure, AWS. They provide a huge array of managed service and deployment options. Wherever possible, use these managed services -- they are better able to deal with these tasks as they are specialized for it.

> 💡 **Consider the TW Technology Radar.** [This page](https://www.thoughtworks.com/en-in/radar/techniques?blipid=202304013) on using CI/CD infrastructure as a service highly recommends using managed services for CI/CD.



----

## References
+ <% tp.file.cursor(2) %>