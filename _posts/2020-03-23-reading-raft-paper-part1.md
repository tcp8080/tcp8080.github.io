---
layout: post
title:  "[6.824 Lab2] reading raft paper (part1)"
date:   2020-03-23 now
categories: go
---
Two step:
1. Firstly, electing a distinguished leader
2. Then, give the leader complete responsibility for managing the replicated log

A leader acceptes log entries from clients, replicates them on other servers, and tell servers when it is safe to apply log entries.

Raft decompoese the consensus problem into three subproblem:
1. Leader election
2. Log replication
3. Safety

### Leader election
Firstly, each server starts up as a **follower**, if it receives any RPC request, it stays as follower.

if it receives nothing at a period of time(called election timeout), the server increments its current term number and becomes a **candidate**,then send RequestVoteRPCs in parallel to all the other servers in the same cluster. (remember, the election timer is random to ensure that split votes are rare and will resolved quickly, and more effectively, we use ranking system)

If the candidate get a majority vote back, it becomes the **leader**, then it sends heartbeat messages to all the other servers to establish its authority and prevent new elections.

How term number works: when other server receives an AppendEntries RPC from the leader, if the term number in the RPC is bigger than the server's current term number, then the leader is recognized, and the receiving server go back to follower state.


### Log replication
log entry: a log index + a term number + a state machine commit

<img src="http://127.0.0.1:4000/pic/logentry.png" width="350"/>

a log entry is **committed** once the leader has replicated it on a majority of the servers (e.g. entry 7 in the pic)

Log Matching Property:
1. If two entries in different logs have the same index and term, then they store the same command
2. If two entries in different logs have the same index and term, then the logs are identical in all preceding entries.

The first property follows from the fact that a leader creates at most one entry with a given log index in a given term, and log entries never change their position in the log
the second prperty is guaranteed by a simple **consistency check** performed by AppendEntries.

<img src="http://127.0.0.1:4000/pic/appendentryrpc.png" width="350"/>

consistency check:
 The leader sends the AppendEntriesRPC with the preLogIndex and preLogTerm, if the follower does not find an entry with the same index and term, then it refuses the new entry. 