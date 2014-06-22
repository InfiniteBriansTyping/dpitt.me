---
layout: post
title:  "Paper: Generalized Scale Independence Through Incremental Precomputation"
date:   2014-05-05 08:59:20
categories: distributed-systems
---

_As data sizes grow, even by orders of magnitude, a scale-independent system enforces strict invariants on the cost of query execution._

The authors of this paper also [wrote a paper on PIQL](http://arxiv.org/abs/1111.7166), which I have not read, that “_...enforces strict invariants on the cost of query execution_" to protect against bottlenecks in large systems.  They called this approach to data independence _scale independence_.  _Generalized Scale Independence Through Incremental Precomputation_ introduces a second mechanism to their model for scale independence, where pre-computation is used at write time, to save from “_...reading unbounded amounts of data_”.

My first thought as I made it through the beginning of this paper was prepared queries, or something like the precomputation of all possible cartesians of a finite dataset.  To partly borrow an example from the paper:

> A database that contains documents, and these documents are tagged.  There are tables like so: `tags(doc_id, tag, timestamp)` and `docs(doc_id, text)`.  A query to get the five newest docs which had tag-a and tab-b could be precomputed by storing all possible arrangements of two tags, then storing those in the order of their timestamp.  

This, apparently, is called _intersection precomputation_, and is a very common practice.  I had a brief moment of clarity while rescanning the paper looking for a clear comparison of _intersection_ precomputation, and the _scale independence_ precomputation that the paper is centered around.  It is not the preparing of known queries to prevent heavy computation on complex reads, but rather a precomputation to _decouple_ the size of the query workload from the size of the overall dataset.  The former does significantly reduce average response time for complex queries, but it does not however, produce a predictable response time as the dataset grows exponentially.  The latter creates cartesians similar to intersection precomputation, but in this case these cartesians represent an entire view.  In PIQL, the qualification of these views are determined by detecting hotspots, and in these hotspots, finding violators of the aforementioned invariants.  The formulas for these invariants are especially dense, and I'm not going to begin to attempt to wrap my head around them; the cool thing here is, though, the idea of representing a complete view in a single, scale independent read.  

If we look at our previous example of docs and tags, consider that the service has now grown to millions of documents and tags. The previous intersection precomputation of tag cartesians is unbounded, and now quite massive.  The wins of this precomputation are all but lost. However, if it is known through the detection of hotspots that the previous query is very common, we can begin to incrementally maintain a complete view of the 5 most recent docs with tag-a and tag-b.  This query does not grow as the dataset grows, as the majority of its data is static.  We can incrementally maintain dozens, or even hundreds of these complete views at write time, most likely asynchronously.  All of these views are independent of the overall dataset size.  This may cause a considerably higher workload for writes, but it will radically decrease response time for reads.  This tradeoff is favorable to any read-heavy service, such as most of the large social media sites like Facebook or Twitter.

This paper was written by Michael Armbrust, Armando Fox, Eric Liang, Michael J. Franklin, Tim Kraska, and David A. Patterson.  You can find it here: [Generalized Scale Independence Through Incremental Precomputation](https://amplab.cs.berkeley.edu/publication/generalized-scale-independence-through-incremental-precomputation/)



### Update:

I was excited about the wrong thing, and [skipped the most important part of the whole paper](https://twitter.com/danielscottt/status/463522240307277824).
