Date: 2022-10-29 12:26

type: #component
links: 

# TL;DR
The tournament manager is a structure in [[SquireLib]] that helps manage a tournament. It acts as a container for a tournament's current state and all its past [[tournament operation]]s. It also manages things like [[rollbacks]] and the [[sync process]].

# About
The tournament manager has one primary usecase, to manage a tournament's state through
- Tracking [[tournament operation]]s
- Managing most of the [[sync process]]
- Restricting mutable access to the tournament state


# Example

