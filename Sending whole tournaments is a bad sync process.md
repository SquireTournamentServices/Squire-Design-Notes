Date: 2022-10-29 12:49

type: #idea
links: 

# TL;DR
While it is possible to send entire [[tournament]]s over the wire, it is both costly and wasteful. Moreover, it makes resolving discrepancies between tournaments nearly impossible in a [[local-first model]] model.

# About
In a [[local-first model]], every client has a copy of the tournament data. The naive approach to syncing between clients is to send whole copies of this data between clients and the backend.

# Example
Imagine we have two clients, A and B, who are syncing with [[SquireCore]]. Perhaps there are two [[admin]]s for a tournament, but they could be anyone really. Say client A sees a player register and add a deck and then starts syncing with [[SquireCore]]. In the simple model of sending whole tournaments, this just requires client A to send their newest tournament state to [[SquireCore]].

However, much of the tournament hasn't changed. They are sending a huge amount of duplicate information. The [[pairing system]], the [[scoring system]], and all of the [[tournament]] settings are completely unchanged. This is not (directly) a big deal for the client. Creating the serialized strings is relatively cheap and they don't *really* need to listen for a response.  But this is a much bigger deal for the backend. Increased request sizes leads to more reasource consumption and longer process times. This reduces the number of requests the backend can handle, which in turn, affects the stablity of clients.

Back to the example though, the backend has recieved client A's latest state... what now? It could make this state the "true" [[tournament]] state, but what about client B? Does the backend need to inform client B that there was a change? If so, who should be informed? Every client that has gotten tournament data from the backend? That's a lot of requests for every update, but suppose we don't send these updates.

Well, client B could easily also see a different player register and add a deck. Like client A, they will want to sync with the backend. So, they send their request, which puts the backend in a bit of a pickle. Who is the "source of truth"? Does the backend need to detect the difference and try to reconcile them? What if there are intractable differences that need [[admin]] input?

Ok, so maybe the backend always syncs with every request. It's a lot of work, but it must be done! Unfortunately, that doesn't solve the problem, either. Because we have to inform everyone after every update, we more or less will be streaming requests to clients. This pretty heavily violates the "sync and local changes shouldn't block each other" paradiagm of the [[local-first model]].