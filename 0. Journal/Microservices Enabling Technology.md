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

## Log Aggregation and Distributed Tracing

At the bare minimum, implement a [[Log Aggregation]] system. Consider this a prerequisite for microservices.

It allows you to collect and aggregate logs from across services, provides a central place for analysis, and can provide active alerting mechanisms against issues and downtime.

> ☝ Sam Newman recommends Humio for log aggregation.



----

## References
+ <% tp.file.cursor(2) %>