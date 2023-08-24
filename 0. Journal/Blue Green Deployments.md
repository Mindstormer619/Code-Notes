---
created: 2023-08-24 20:55
alias: 
---
#### Summary
+ 

----
# Blue Green Deployments

It's good to have completely automated deployments. This is a deployment technique meant to minimize downtime.

How does it work? Start with two identical production environments, Blue and Green. Say Blue is live. At this stage, we deploy our changes to Green and do our final stage testing here. Once the software works in Green, switch the router so that all incoming requests come to Green -- Blue is now idle. Now Blue is the staging environment for the final testing step for your next deployment. When you are ready for your next release, you switch from Green to Blue in the same way that you did from Blue to Green earlier.

Rapid rollback abilities: if anything goes wrong, simply switch the router back to Blue, and then you can examine what went wrong with the changes in Green while Blue continues to serve production traffic. To deal with missed transactions 

----

## References
+ Fowler: https://martinfowler.com/bliki/BlueGreenDeployment.html © 2010
+ Book: Continuous Delivery, by Dave Farley and Jez Humble.
+ Recommended by Chaitanya.