# Caching Pitfalls
> Created: 2022-09-08 20:03

Remember to cache any empty collections as well -- else you end up querying the same thing every single time, because you cannot distinguish between "this exists and is empty" versus "this does not exist".

----

## References
1. Talk @ GS Software Craft