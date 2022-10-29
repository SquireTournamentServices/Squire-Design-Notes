Date: 2022-10-29 13:21

type: #idea
links: 

# TL;DR
Central to the [[sync process]] is the assumption that a tournament can always be recreated between clients. If this is not true, the [[tournament]] sync breaks down.

# About
Not every aspect of a tournament needs to be deterministic. For example, [[player]]s and [[round]]s are stored in `HashMaps`. These order things in a fairly arbitarily. That's ok so long as it doesn't affect how a [[tournament operation]] affects the tournament or the operation's outcome.

One of the biggest cultrips of non-determinism is the [[id]]s for [[player]]s and [[round]]s. All [[id]]s are `Uuid`, basically a fancy 128-bit integer. The easiest why to construct them, in [[Rust]], is to use the `uuid` crate. Their `Uuid`s have a method `Uuid::new_v4()`. It takes nothing, but uses specific system state for "randomness". So, this is completely non-deterministic.

# Examples

## Id generation
Let's say I am running a tournament. I can register [[player]]s, pair the [[round]]s, and everything is running fine. But how to I refer to [[player]]s and [[round]]s? For [[player]]s, you could use their names, but referring to [[rounds by their match number causes problems]]. So, we will use [[id]]s. Great, we can refer to all the components by [[id]].

For ease, I'll use hex integers, since `Uuid`s are just extra large integers, but the "normal" representation is a constant-length, dash-delimited string. So, say we have [[player]]s register. They need [[id]]s. We could just enumerate the, e.g. `0x1`, `0x2`, `0x3`, and so on. Or, we can have the `uuid` crate generate them. Unfortunately, [[enumeration for ids is a bad idea]]. So what about having `uuid` generate them?

Let's say we do that and the players (still) get ideas, `0x1`, `0x2`, `0x3`, etc. Well, these were generated non-deterministically. So, when we send the registeration operation to the backend, the backend will generate different [[id]]s, say `0x11`, `0x12`, `0x13`. Then any operation that refers to player `0x1` will fail on the backend!!

## Pairing rounds
