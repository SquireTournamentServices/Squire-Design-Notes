Date: 2022-10-29 15:37

type: #component
links: 

# TL;DR
The Squire player model is relatively simple. They are a name, an optional list of decks, and optional gamer tag. Every player has a unique [[id]], which is used to track their partipation in [[round]]s, amoung other things.

# About
Players one of the simplest components in a [[tournament]]. After registeration for a [[tournament]] closes, their information is largely static. Internally, they are managed via a [[player registry]]. The registry is in charge of creating, querying, and managing players.

# Example

