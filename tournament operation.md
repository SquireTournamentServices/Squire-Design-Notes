Date: 2022-10-29 12:32

type: #component
links: 

# TL;DR
A tournament operation is an "atomic" tournament action. All mutations to a tournament are encapsulated in tournaments.

# About
Tournament operations encapsulate what an action is and who is performing such an action. For example, the `TournOp::RegisterPlayer` action encodes that a [[player]] is registering for a tournament themselves while the `JudgeOp::RegisterPlayer` action encodes that a judge is registering a player for the tournament and which [[judge]] is doing so.

Tournament operations are broken down into three primary categories: `PlayerOp`, `JudgeOp`, and `AdminOp`. Each category corresponds to a level of permission.

There are also [[opaque operations]]. These are operations that conceal information. The best example of this would be the `AddDeck` operations. In tournaments that are closed decklist, only [[judge]] and [[admin]] clients should have access to this information. Player and [[spectator]] clients should not be able to see information relating the [[decklists]].

# Example

