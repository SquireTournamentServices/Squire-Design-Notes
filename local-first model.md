Date: 2022-10-29 12:10

type: #idea
links: 

# TL;DR
This is the primary model that Squire client use. Central to this model is that you should be able to run a complete tournament without needing access to the internet during the tournament.

# About
We often take the internet for granted, and for good reason. The entirity of the internet's infastructure is designed to always be available. Moreover, the cost of a temporary outage or increased latency for most services is fairly minimal. That is not the case for tournaments. 

A server outage or (more often) spotty internet access in the tournament hall can cause delays or even stop the tournament altogether. This is unacceptable as it can be completely avoided.

Since the core tournament library, [[SquireLib]], is written in [[Rust]], it be can used almost everywhere. This is why [[SquireWeb]] is written in WASM.

This is not to say a client should forego backing up their local state. There is a whole [[sync process]] for that. However, running and tournament and syncing a tournament should be handled asyncronously and one should not stop or prevent the other from happening. This is something [[Rust]] helps out with tremendiously.

# Example

