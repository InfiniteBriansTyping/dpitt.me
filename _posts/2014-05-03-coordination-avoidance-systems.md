---
layout: post
title:  "Paper: Coordination-Avoiding Database Systems"
date:   2014-05-03 08:59:20
categories: distributed-systems
---

Classic "strong consistency" systems, ACID-based transactional databases and the like, use transaction isolation to ensure some form of correctness.  Isolation in this context simply means no updates happen concurrently, each transaction is isolated by a locking mechanism, and these transactions are coordinated to all replicas synchronously.  Obviously, these synchronous transactions seriously limit scalability.  From this problem arose _NoSQL_ systems like Cassandra, Riak, Dynamo, etc.  These systems allow any number of concurrent updates, allow those updates to be sent to any node, and asynchronously replicate these updates (or their datasets) to their neighbors.  This is the basis of _eventual consistency_.  As a result of these non-isolated, asynchronous updates, determining _correctness_ is very difficult.  There are many attempts to "force" some type of correctness with merge operations, or something much harsher like LWW - Last Write Wins.  Other _correctness_ sumation attempts include determining causality with a vector clock, or using data types that are commutative or monotonic.  Although quite similar, these last two properties, commutativity and monotonicity, are not exactly the same thing.  A simple way to think of each is with a counters example:

> Consider an application that, rather than querying for all users, then counting the results, it uses a counter for a quick, and single-row read to surmise the current number of Users.  Each time a new User is saved, a counter is updated with a +1.

If a User is never deleted, and the counter only increases, then the counter is monotonic.  Monotonicity can be described as a function which never changes direction.  The counter only goes up in value.

> Now consider that in our application, we _can_ delete users, so the counter can go up or down.  Each time a user is added: +1, and deleted: -1.  This results in a series of updates that look like this: 

> 2 Users are added, 1 is deleted, then 2 more are added, resulting a total of 3 Users. No matter what order these updates are received in, the result is _always_ 3:

> 0+1+1-1+1+1 = 3

> 0-1+1+1+1+1 = 3

> 0+1+1+1+1-1 = 3

> etc...

These counters _commute_, because no matter what order the updates are received in, the final result is always the same.

#### ___Invariant Confluence___

This paper introduces a new approach to strong consistency while avoiding coordination using something the researchers call invariant confluence.  Invariant Confluence implements this avoidance by predetermining a constraint to the schema.  That is, coordination is not needed if all asynchronous transactions merge into a state that is valid against this constraint.  Lets go back to our application's users for an example.  As Users are added and removed to our system, we can safely say there should never be a negative amount of users.  We can never have less than we started with: 0.  So, using _Invariant Confluence_ we can say `Users.count` should never be < 0.  Using this constraint on the schema, we can then allow our transactions to occur coordination-free, and test their resultant state if merged.  If they do not violate our constraint, they can merge into a valid state; if not we must forego the coordination avoidance, or the correctness of the final state.  

This technique can be used to surmise the scenarios in which coordination is necessary, and allows the the avoidance in all other transaction merges.  This contributes to a much more scalable, strongly-consistent system, which sacrifices much less availability.

This paper was written by a group from Berkley and the University of Sydney, and you can find it here: [Coordination-Avoiding Database Systems](http://arxiv.org/abs/1402.2237).
