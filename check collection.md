Date: 2023-11-21 20:22

type: #component
links: 

# TL;DR
These collect [[rule check]]s to efficiently calculate what rules match a given tournament. This type of collection is a general concept and the data structure used to perform queries might vary by rule type. However, all collections allow for the quick deletion and insertion of checks and efficiently return an iterator over matching rules.

# About
To start, every collection splits checks into two groups: catch-all/any checks and bounded checks (i.e rules that actually filter). The "any" rules are stored in a sorted list. The bounded rules are stored in different structures depending on what type of check is being stored. 

Regardless of the check type, the bounded check data structure should be able to return an implicitly sorted iterator in either constant time or, at worst, `O(log(N))` time, where `N` is the number of bounded checks when given a query for matches.

From there, the iterator over the matched bounded checks can be zipped with the "any" checks via a [[ord iter]]. This will result in an iterator that is implicitly sorted.

# Example

