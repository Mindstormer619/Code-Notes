# Crossing the TDD Chasm and Ensemble Traps with Khaled Souf
> Created: 2023-05-24 21:30

+ testing strategy for TDD
	+ on bigger scale, like large legacy applications, it's hard to put in practice
		+ main struggle is testing strategy
		+ need to create TDD compatible strategy
			+ what is a unit? faster feedback loop creation
		+ freestyling -- without testing at all
+ specific strategies to make easier
	+ legacy is very hard to test
		+ complexity of the dependencies
	+ if in new architecture -- hexagonal architecture
		+ start with slow tests / integration tests
		+ gonna test it as it is -- do not use mocks
	+ outside in
		+ start with outer layer and go inward
	+ start from consumer side
		+ consumer driven contract testing
		+ like doing TDD except before the backend is even built
+ what kind of resistance have you seen to testing / TDD
	+ usually the resistance is for TDD
		+ people are not convinced
		+ less knowledge / surface level
		+ seems too hard "this is not applicable for us"
		+ afraid to lose control or change their habits
	+ solve how
		+ pair program with them
		+ convince them that it works in real life
		+ give them outside "step back" perspective
		+ reassure them that you are there with them
+ mentioned "consumer driven contract testing"
	+ technique from author of "growing object oriented software guided by tests"
		+ Nate Pryce
	+ evolution of TDD
	+ how it works
		+ tool called packt.io???
		+ start from frontend, make some mock or stubbing
		+ mock generates a contract -- when I call that thing it's gonna give me this structure
		+ contract can be taken to the provider on the backend side
		+ used to generate tests
		+ can test the backend as a whole -- e2e testing?
+ burned by slow tests -- what's the tradeoff for you?
	+ experience
		+ 8-9 y ago
		+ started using TDD, BDD, TDD double loop ?
		+ 15 mins to get feedback
		+ sometimes testing and TDD is a very good indicator that there is already a problem
		+ usually problem with ur design
			+ maybe split into multiple apps?
			+ monolith?
		+ make your tests run in parallel
	+ BDD -- Cucumber framework
+ traps to be avoided
	+ biggest -- when ensemble programming -- create a safe space for people
		+ create some constraints so everybody is participating
		+ don't do with zero restrictions at all
		+ e.g. timebox the thing to make switch
		+ "at the end of the week, did everyone speak a roughly equal amount of time"
		+ switching people every 5 or 8 mins
		+ restricting the no of people that can mob
			+ > 5 is too much. 3-5 max
		+ navigator is the one that tells people what to do
			+ should not take decisions without consulting the others.
			+ no one should take decision solo
		+ finding the right balance is important
			+ some people may not want to talk a lot
	+ instead of doing small steps, doing very big steps
		+ taking more than 30 mins
		+ always have some kind of break or something
		+ reflect on the previous session
		+ should not zone in too hard
+ 

----

## References
1. https://www.youtube.com/watch?v=XFHzZMlckzU