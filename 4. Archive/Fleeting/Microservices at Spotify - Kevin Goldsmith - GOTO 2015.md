# Microservices at Spotify - Kevin Goldsmith - GOTO 2015
> Created: 2023-06-04 17:27

+ autonomous fullstack teams
	+ each team has the complete freedom to act independently
	+ no dependencies on other teams

Usual infra:

![[Pasted image 20230604172845.png]]

Challenges with above:
+ synchronization of teams -- slows you down

What Spotify does:

![[Pasted image 20230604172955.png]]

it's a dialogue, not synchronization

requires you to structure application in loosely coupled parts

build your components to scale independently. microservices are easier to scale on real world bottlenecks -- identify the bottleneck and scale just that part. Easier to test and deploy as well. Easier to monitor. Less susceptible to large failures. Easier to version. 

downsides: harder to monitor -- surface area on each is smaller, but there are more of them. discovery / documentation of new services are hard. creates increased latency. 

scale. 810 active services at Spotify. 10 systems per squad. 1.7 systems per person. 

Spotify Apollo -- open source.




----

## References
1. ![](https://www.youtube.com/watch?v=7LGPeBgNFuU)