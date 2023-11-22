Date: 2023-11-21 19:27

type: #component
links: 

# TL;DR
This is the system through which new [[tournament]]s are associated with a [[user profile]]'s recommended list of tournaments.

# About
When a new tournament is created, we must determine was users might be interested in that tournament. Every [[user profile]] has a collection of [[tournament rule]]s. These are created by users to communicate what kinds of tournament they are interested in. This system is entirely opt-in. Every user starts with zero tournament rules, and any user without rules does not get tournaments recommended to them.

# Specifications
This system functions entirely to calculate a list of users that are interested in a tournament at the time of that tournament's creation. This means that it does not work retroactively. If a user changes their [[tournament rule]]s, already-recommend tournaments will not be taken away. They must remove recommend tournaments. 

That said, users are allowed to change their rules at any time and the system will use those new rules for any new tournaments.

The simplest version of this system iterates through the database's user table for every new tournament, checks if the tournament matches the user's rules, and then update the profile if it is a match. This is far from ideal as it adds a lot of I/O overhead and is computationally intensive.
# Current Design
The system is currently being designed, with two designs being the main focus:
 - The [[on-demand rec system]]
 - The [[batched rec system]]
 
