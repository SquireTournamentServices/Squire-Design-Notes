Date: 2022-10-29 11:19

type: #process
links: 

# TL;DR
Most Squire services work off of a [[local-first model]]. If order to sync with other services, a clear and concise process is needed. This process is governed by two things, the [[tournament manager]] and the [[tournament operation]].

Key to this process is some level of determinism. Given the same set of [[tournament operation]], the identical tournament must always be produced, even across different platforms.

# About
Let's start at the base level. Tournaments can be completely serialized and then saved to disk, sent over the wire, or deserialized. [[Sending whole tournaments is a bad sync process]] though.

Since sending whole tournaments is out of the question, what's next? Well locally, you mutably intract with a tournament using [[tournament operation]]s. These [[operations encapsulate all of the ways in which you can change with a tournament's state]]. Locally, deternism is not a big deal. [[A lack of determinism is an issue between clients]]. So, we make tournaments and operations determenistic. But, [[complete determinism causes problems]]. The [[solution to a lack of determinism]] requires changes to [[tournament operation]].

At this point, we can safely send complete tournament data between clients and construct identical tournaments on all clients. Now, [[we need to worry about authentication]] and [[data visibility]]. In [[the simple model of clients communicating through a central server]], the server pools changes to a tournament and validates them. For now, this solves the authentication problem. The [[solution to data visibility]] is making a second kind of [[tournament operation]], [[opaque operations]]. Because [[opaque operations]] act as skeleton keys, [[authentication points can accept new opaque operations only from other authentication points]].

This model is not limited to the simple case of a single server connecting multiple clients. This gives us room for a [[local server model]] and an [[integrated server network]]. While these are not without their own challenges, they can still operate using this local-first-and-sync process. For example, [[SquireBot]] can be sharded, intercommunicate, and then backup everything to a [[SquireCore]] server/network.


# Example

