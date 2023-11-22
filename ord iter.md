Date: 2023-11-21 20:35

type: #idea
links: 

# TL;DR
An iterator that zips two other iterators together and yields elements based on their ordering. This is meant to be used to zip two implicitly order iterators as this iterator will then yield items in order.

# About
The goal of this iterator is to yield an implicitly sorted sequence of elements by zipping together two other implicitly sorted iterators. Here, "implicitly sorted" means that elements are yielded in order without needing to allocate a buffer. An iterator like `0..10` is implicitly sorted as `[1, 4, 7, 9, 1000]`.

Here's a rough implementation of this iterator:
```rust
// L and R are iterator that yield the same type.
struct ZipOrdIter<L, R> {
  left: L,
  right: R,
  left_buf: Option<<L as Iterator>::Item>,
  right_buf: Option<<R as Iterator>::Item>,
}

impl<L, R> Iterator for ZipOrdIter
where L: Iterator,
	  R: Iterator<Item = L::Item>,
	  L::Item: Ord,
{
	type Item = L::Item;

	fn next(&mut self) -> Option<Self::Item> {
		// Assume `left_buf` and `right_buf` are filled on construction
		
		// left_buf is only empty when left iter is empty. Drain the right iter
		let Some(left_item) = left_buf.take() else {
			return std::mem::replace(&mut self.right_buf, self.right.next())
		}
		// And visa versa
		let Some(right_item) = right_buf.take() else {
			return std::mem::replace(&mut self.left_buf, self.left.next())
		}
		match left_item.cmp(&right_item) {
			// Equal -> yield either one, get next value
			// Greater -> yeild right, replace it
			// Less -> yield left, replace it
			// NOTE: In all cases, put the unused value back into the buffer
		}
	}
}
```

# Example
Using this iterator, an iterator over the evens and an iterator over the odds would yield the natural numbers.
Two ranges, say `0..`, would yield each number twice, `[0, 0, 1, 1, 2, 2, ..]`.
