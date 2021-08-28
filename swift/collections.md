# Collections

Collection is the superclass of Arrays, Dictionaries, and Sets.

* Each collection inherits `Sequence`
	* Each sequence has an iterator, so each collection by default has an iterator
	* This is why it is possible to use a for-in loop to iterate over collections.
* Each collection must have some way to iterate through all of its items. I believe it can override the iterator with some new methodology.


* To convert one collection in a set


## Sequence
*Sequence* is a list of values that you can step through one at a time. (You iterate through sequences in for-in loops)

Any Sequence (like `Array`) must conform to the `Sequence` protocol. This will require you to add the function `makeIterator()` method. Alternatively, if your type can act as its own iterator, you may conform to the `IteratorProtocol` protocol as well.

Ex:

```
struct Countdown: Sequence, IteratorProtocol {
    var count: Int

    mutating func next() -> Int? {
        if count == 0 {
            return nil
        } else {
            defer { count -= 1 }
            return count
        }
    }
}

let threeToGo = Countdown(count: 3)
for i in threeToGo {
    print(i)
}
// Prints "3"
// Prints "2"
// Prints "1"
```


## IteratorProtocol
*IteratorProtocol* is tightly linked with `Sequence` protocol to keep track of the iteration process  and return one element at a time as it advances through the sequence. Whenever you are using a `for-in` loop with a collection or sequence, you are using that type's iterator.