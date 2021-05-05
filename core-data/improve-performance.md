# Improve performance

This page involves quick tips that should help to make your Core Data code more performant.

* Add `fetchBatchSize` property to your `NSFetchRequest`s. 
	* Not specifying a `fetchBatchSize` will be zero by default. This will return all results available.
	* Choose a number that is about double of the amount of objects that would appear on the screen at any given time.
	* As you scroll through your `List` or `UITableView`, your app should automatically fetch more data as it needs it.
* When adding compound predicates to your app, the order matters. Place more restrictive predicate sfirst. 
	* Ex: `"(active == YES) AND (name CONTAINS[cd] %@)"` is more efficient than `"(name CONTAINS[cd] %@) AND (active == YES)"`