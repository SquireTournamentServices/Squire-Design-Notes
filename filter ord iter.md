Date: 2023-11-22 01:56

type: #idea
links: 

# TL;DR
Another zipping iterator, but, unlike [[ord iter]], this iterator filters out values that do not match. Because it assumes the held iterators are sorted, this iterator can avoid any unnecessary checks or buffering.

# About
This iterator hold two iterators that yield the same value. For this to be useful, the held iterators need to yield their items in order. By making this assumption, we can easily find all items that the iterators have in common. See the examples for a visual.

Here's a rough implementation of this iterator:
```rust
// L and R are iterators that yield the same items.
struct FilterZipOrdIter<L, R>{
  left: L,
  right: R,
  left_buf: Option<<L as Iterator>::Item>,
  right_buf: Option<<R as Iterator>::Item>,
}

impl<L, R> Iterator for FilterZipOrdIter<L, R>
where L: Iterator,
	  R: Iterator<Item = L::Item>,
	  L::Item: Ord,
{
	type Item = L::Item;

	fn next(&mut self) -> Option<Self::Item> {
		// Assume the buffers are filled on construction.

		loop {
			let left_item = self.left_buf.as_mut()?;
			let right_item = self.right_buf.as_mut()?;
			match left_item.cmp(right_item) {
				// Equal, pull the next items into the buffers and yield
				// Less, pull the left iter into its buffer
				// Greater, pull the right iter into its buffer
			}
		}
	}
}
```
# Example
Let's say you have the following two (sorted) arrays, and you wish to find all common elements between them (i.e. their intersection).
```
[1, 3, 4, 5, 6]
[2, 3, 4, 5, 7]
```
Clearly, they have `[3, 4, 5]` in common. A simple approach would be to directly zip them together and find all matching pairs. That works here; however, we don't care if two elements have the indices. We care about common sets. What happens if we change the lists slightly?
```
[1, 2, 3, 4, 5]
[2, 3, 4, 5, 7]
```
Now, they have `[2, 3, 4, 5]` in common. Instead, we need read from the left and right at the same time and compare the items. If they match, we can yield that item. If they don't match, we pull the next item from the iterator whose item was smaller. We can now compare again and repeat. Once an iterator has yield all of its elements, the whole iterator is done.