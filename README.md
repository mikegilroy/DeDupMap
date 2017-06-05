# DeDupMap
A playground showcasing the use of generics to create map and flatMap functions that also de-dup (remove duplicates) from the resulting arrays, as well as return only the the duplicates. 

### Example
``` Swift

extension Array {

	/// A map function that iterates over and transforms each element of an array but removes any duplicates from the resulting array, so elements that match will not be added twice.
	func dedupMap<T: Equatable>(_ transform: (Element) -> T) -> [T] {
		var result = [T]()

		for x in self {
			if !result.contains(transform(x)) {
				result.append(transform(x))
			}
		}
		return result
	}

	/// A flatmap function that iterates over and transforms each element of an array, removes any transformed values that are nil, but also removes any duplicates from the resulting array, so elements that match will not be added twice.
	func dedupFlatMap<T: Equatable>(_ transform: (Element) throws -> T?) rethrows -> [T] {
		var result = [T]()

		for x in self {
			if let transformed = try transform(x),
			!result.contains(transformed) {
				result.append(transformed)
			}
		}
		return result
	}

	/// A map function that iterates over and transforms each element of an array but only returns the duplicates from the resulting array.
	func dupMap<T where T: Comparable, T: Hashable>(_ transform: (Element) -> T) -> [T] {
		var nonDups = Set<T>()
		var dups = Set<T>()

		for x in self {
			let transformed = transform(x)
			if nonDups.contains(transformed) {
				dups.insert(transformed)
			} else {
				nonDups.insert(transformed)
			}
		}
		return dups.sorted()
	}
}

// **********************************************************
// ****************** Testing deDupMap **********************
// **********************************************************
// A list of random integers that contains no duplicates
let numbers = [1,2,3,4,5,6,7,8,9, 10, 11, 12,13, 21,33, 126]

// A list of random names that contains one duplicate - "Phoebe"
let names = ["Mike", "Ben", "Phoebe", "Phoebe", "Jenny"]


// First we'll try with a simple Int to String transformation

// Let's try and find the resulting array removing any duplicates using dedupMap
let uniqueNumbers = numbers.dedupMap { "\($0)" }
print("The items in the following array are de-duped numbers: \(uniqueNumbers)")

let uniqueNames = names.dedupMap { "\($0)" }
print("The items in the following array are de-duped names: \(uniqueNames)")


// Now let's try and find the resulting array of any duplicates using dupMap
let duplicateNumbers = numbers.dupMap { "\($0)" }
print("The items in the following array are duplicate numbers: \(duplicateNumbers)")

let duplicateNames = names.dupMap { "\($0)" }
print("The items in the following array are duplicate names: \(duplicateNames)")


// Now we'll try with a more complex transformation such as finding the highest factor of an integer
/**
This function finds and returns the highest factor (excluding itself) of any positive integer.
If a negative or zero integer is passed in the function will return -1 as an error state.
*/
func highestFactor(of number: Int) -> Int {
	if number <= 0 {
		return -1
	}

	if number < 2 {
		return 1
	} else {
		for x in 2...number {
			if number % x == 0 {
				return number / x
			}
		}
	}
	return 1
}

// Let's test our highest factor function works as expected
let factor = highestFactor(of: 21)
print("The highest factor of 21 should be 7 and our function calculates it to be: \(factor)")


// We can use the standard map function to find the non de-duped highest factors from our numbers array
let nonDeDupedHighestFactors = numbers.map(highestFactor)

// We can now also use the dedupMap function to find the highest factors while also removing any duplicates
let highestFactors = numbers.dedupMap(highestFactor)

// We can now also use the dupMap function to  find all the common highest factors for the numbers in the array - i.e. the duplicates
let highestCommonFactors = numbers.dupMap(highestFactor)



// **********************************************************
// ****************** Testing deDupFlatMap ******************
// **********************************************************
/// A list of optional strings that represent possible integers and also includes some duplicates
let possibleIntegers = ["one", "2", "3", nil, "four", "5", "5", nil]

// Lets create a new failable initialiser for Int that accepts an optional String as a parameter
extension Int {

	/// Initialises an Int from an optional String or else returns nil
	init?(_ text: String?) {
		guard let text = text else { return nil }
		self.init(text)
	}
}


// We can now find the deduped and non-nil integer values from the possibleIntegers array using dedupFlatMap
let integers = possibleIntegers.dedupFlatMap { Int($0) }
print("The items in following array are de-duped and removed of nil values: \(integers)")


```
