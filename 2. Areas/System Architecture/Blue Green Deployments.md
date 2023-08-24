---
created: 2023-08-24 20:55
alias: 
---
#### Summary
+ 

----
# Blue Green Deployments

It's good to have completely automated deployments. This is a deployment technique meant to minimize downtime.

How does it work? Start with two identical production environments, Blue and Green. Say Blue is live. At this stage, we deploy our changes to Green and do our final stage testing here. **Once the software works in Green, switch the router so that all incoming requests come to Green -- Blue is now idle.** Now Blue is the staging environment for the final testing step for your next deployment. When you are ready for your next release, you switch from Green to Blue in the same way that you did from Blue to Green earlier.

Rapid rollback abilities: if anything goes wrong, simply switch the router back to Blue, and then you can examine what went wrong with the changes in Green while Blue continues to serve production traffic. To deal with missed transactions while Green was live, you may be able to feed transactions to both environments in such a way as to keep the Blue environment as a backup when Green is live. Or you may be able to put the application in read-only mode before cut-over, run it for a while in read-only mode, and then switch it to read-write mode.

Unlike most other rollback schemes, the actual mechanism for rolling back is the same as for a normal release (a load balancer flip), so it's continually exercised and validated. To roll back, you literally do a deploy, but you just skip the step where you alter the bits on the dark cluster. This is highly unlikely to fail, since the only thing that has to not go wrong is the part where the software update is skipped, which basically boils down to a conditional.

The setups need to be as identical as possible, and it needs to be easily switchable between Green and Blue. This is highly essential for successful Blue-Green deploys.

One difficulty can be aligning with DB schema changes. For this, first separate the deployment of the schema change from application upgrade changes. Upgrade the schema first (see [[Database Refactoring]]) with backwards compatibility. Then deploy and check the application. Remember to remove the DB support for the old version after a successful deployment.

Origins of the name:
> We thought about calling these side-by-side environments A and B, until someone pointed out that if the application broke and it happened to be deployed in the B environment, the first question would be “Why weren’t you using the A environment?” Because clearly A is better than B! We needed labels for the domains that didn’t have an obvious hierarchy, and we settled on colours. If your domains are called Blue, Green, Orange, Yellow, etc. then there isn’t an obviously “best” one. We avoided having a Red domain because that just sounded dangerous. (“You were running in RED??”)
> ~ *Daniel Terhorst-North and Jez Humble*: [https://gitlab.com/-/snippets/1846041](https://gitlab.com/-/snippets/1846041)

----

## References
+ Fowler: https://martinfowler.com/bliki/BlueGreenDeployment.html © 2010
+ Book: Continuous Delivery, by Dave Farley and Jez Humble.
+ Recommended by Chaitanya. [[2023-08-15]]
+ https://news.ycombinator.com/item?id=21434914