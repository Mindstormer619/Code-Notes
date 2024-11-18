# Load
> Created: 2023-02-13 16:41

Load is a measurement of strain on the system, indicated using load parameters. E.g. requests per second for a webserver, ratio of reads to writes in a DB, simultaneously active user count in a chatroom, hit rate on a cache etc.

Performance itself is described by the question "what happens when load increases?"
+ How is the performance affected when the load parameter goes up but resources stay constant?
+ How much do you need to increase resources to keep performance unchanged?

Related: [[Response Time Percentiles]]

## 🔍 Case Study: Twitter

In Nov 2012, Twitter had these load parameters for their main operations:
+ Post Tweet: 4.6k rps average, 12k rps peak.
+ View Home Timeline: 300k rps.

12k writes per second is easy, but the issue is _fan-out_ which results from each user following many other users, and being followed by many people. There are two ways of implementing this operation:

1. Direct write but query read. Posting a tweet simply adds it to a tweet collection, but when a user requests their home TL, we query and put together the collection of recent tweets from all the users they follow:
   ```sql
   select tweets.*, users.* from tweets
	join users on tweets.sender_id = users.id
	join follows on follows.followee_id = users.id
	where follows.follower_id = @current_user
   ```
2. Direct read but query write. Maintain a cache for each user's home TL, and when a user _posts_ a tweet, go and update the cache for every one of their followers. The request to read the home TL is then cheap, because it has been computed ahead of time.

The first version of Twitter used Approach 1, but then they switched to Approach 2. This works better because the average rate of published tweets is almost two orders of magnitude lower than the rate of home TL reads. Therefore it is better to do more work at write time than read time.

Currently Twitter uses a hybrid of both approaches, where most users' tweets are fanned out to home timelines of their followers, but for a small number of users with a massive following (celebrities) are excepted from this fan-out.


----

## References
1. [[1. Projects/Book - Designing Data Intensive Applications/README|DDIA]]