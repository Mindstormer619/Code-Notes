# Raft Consensus
> Created: 2023-01-01 23:05

## Overview

Node types:
1. Follower: most nodes are followers, they can become candidates for leader.
2. Candidate: node which has published its intention to become the leader of the cluster.
3. Leader: thing which gets the data from the client (sitting outside the cluster) and distributes the log to the rest of the cluster to replicate.

Leader selection:
1. Random timeout (150-300ms) on each follower node. When this timeout expires, they become a candidate
2. As soon as a node becomes a candidate, it publishes a "vote request" to the other nodes in the cluster (and also votes for itself).
3. Follower nodes that receive a vote request and do not already have a leader mapped, will vote for the candidate and set that as their leader.
4. A candidate which receives majority vote, becomes the new Leader.
5. Leader keeps its status alive over time through regular heartbeat messages (which reset the timer on the followers from becoming Candidates themselves).

Data replication:
1. Client contacts the leader with the new data.
2. Leader does not set this data immediately -- rather it writes it to an "uncommitted log" then sends out a message on the next heartbeat to all other nodes asking them to update the data themselves.
3. Nodes receiving the message from the leader send back an acknowledge, and write the data to their own uncommitted log.
4. When the leader receives ACKs _from a majority of nodes_, it sets the data ("commits") then acknowledges to the client. Then it sends a commit message on the next heartbeat.
5. Follower nodes receiving the commit message commit the data.

----

## References
1. https://thesecretlivesofdata.com/raft/