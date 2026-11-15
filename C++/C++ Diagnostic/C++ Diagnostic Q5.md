### From scratch: collect unique labels

**Difficulty:** 2/5  
**Type:** From scratch  
**Topic:** Value semantics, standard containers, clear contracts

Implement a function that accepts a read-only sequence of strings and returns the distinct, nonempty strings in their original order of first appearance.

Example:

```
Input:  {"render", "", "audio", "render", "Audio", "audio"}
Output: {"render", "audio", "Audio"}
```

**Requirements and constraints:**

- Target C++20.
- Choose the function signature yourself and justify it.
- Comparison is case-sensitive; do not trim whitespace.
- Do not modify the input.
- The returned strings must remain valid after the input is destroyed.
- No global mutable state or explicit memory management.
- Test empty input, all-empty input, duplicates, and the example above.
- State the time complexity of your implementation. You do not need the fastest possible solution.

**What a strong solution demonstrates:** An understandable interface, correct ownership of returned data, sensible container choices, and awareness of complexity.


### Mapping out the solution

Okay so we need to take an read-only sequence of strings as the input so I think that we should take a `const std::vector<std::string>&` and the return type of the function should be a `std::vector<std::string>` because we don't want the function to modify the vector that was passed in and the caller should be able to get the full vector passed by value.

I think that we should store the vector of distinct strings in a new vector called `results` and use a `std::unordered_set<std::string>` to keep track of what elements have been seen. We also should have a empty string check when we check for the elements as well. If we haven't seen this string yet and if it isn't an empty string then we can `push_back()` the element to the `results` vector.

```
#include <cassert>
#include <iostream>
#include <string>
#include <unordered_set>
#include <vector>

std::vector<std::string> distinct_strings(const std::vector<std::string>& string_sequence) {
	std::vector<std::string> results{};
	std::unordered_set<std::string> seen{};

	for (const auto& s : string_sequence) {
		if (!s.empty() && seen.insert(s).second) {
			results.push_back(s);
		}
	}

	return results;
}

int main() {
	std::vector<std::string> string_sequence{ "render", "", "audio", "render", "Audio", "audio" };

	std::vector<std::string> distinct = distinct_strings(string_sequence);

	for (const auto& s : distinct) {
		std::cout << s << '\n';
	}

	assert(distinct_strings({}).empty());
	assert(distinct == std::vector<std::string>({ "render", "audio", "Audio" }));
}
```

As you can see here I used `seen.insert(s).second` rather than

```
auto it = seen.find(s);
if (!s.empty() && it == seen.end()) {
    seen.insert(s);
    results.push_back(s);
}
```

The second one is what I originally did but I realized that we don't have to split it up into `find + insert` and we can just check the return type of the insert immediately by checking `.insert(element).second`. If the element was inserted then it returns true and if the element was already present then it returns false. The `empty()` check comes first because we don't want an empty string to be inserted.

Time Complexity: Average case **O(n · L)**, worst case **O(n² · L)**, where _n_ is the number of input strings and _L_ is the maximum string length.

For each of the _n_ strings, the loop does:

- `s.empty()`: O(1)
- `find` and `insert`: hashing the string reads all its characters, O(L). Then the set checks the strings in that hash bucket, and with a hash function a bucket holds O(1) strings on average, each compared in O(L). So O(L) per operation.
- `push_back(s)`: copying the string is O(L). The vector occasionally reallocates, but that’s amortized O(1) per push.

For the worst case scenario, hash containers only guarantee constant-time lookups on average. If many distinct strings hash to the same bucket, each lookup has to compare against all of them, up to O(n) comparisons of O(L) each. Over _n_ strings, that’s O(n² · L).


### Feedback

**Grade: 9/10.** Your complexity bounds are appropriate, and your explanation shows you understand hashing and collision costs. The implementation remains correct.

**Vector insertion is amortized constant time in element operations—not in characters copied.** Here, copying the incoming string costs \(O(L)\). Occasional vector reallocations move existing strings; with this ordinary `std::string` setup, those moves do not require copying every character. Your overall expected \(O(nL)\) bound remains appropriate.

**Expected bucket occupancy depends on hash distribution and load factor.** Say “with a well-distributed hash and controlled load factor,” rather than suggesting every hash function produces constant expected bucket sizes.

One mathematical edge case: if every string is empty, \(L=0\), but traversing the input still takes \(O(n)\). Bounds that cover that case explicitly are:

- Expected: \(O(n(1+L))\)
- Worst case: \(O(n^2(1+L))\)

Your \(O(nL)\) and \(O(n^2L)\) notation is conventional if you explicitly assume \(L\geq1\).

**Most important lesson:** State what your complexity model counts. “Constant-time insertion” into a container does not make hashing or copying a variable-length string constant time.