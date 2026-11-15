### Refactor: an owning batch of measurements

**Difficulty:** 2/5  
**Type:** Refactor  
**Topic:** Resource ownership, special member functions, interface contracts

This class appears in a small scientific application:

```
class MeasurementBatch {
public:
    explicit MeasurementBatch(int count)
        : count_(count), samples_(new double[count]) {
        for (int i = 0; i < count_; ++i) {
            samples_[i] = 0.0;
        }
    }

    ~MeasurementBatch() {
        delete[] samples_;
    }

    void set(int index, double value) {
        samples_[index] = value;
    }

    double average() {
        double total = 0.0;

        for (int i = 0; i < count_; ++i) {
            total += samples_[i];
        }

        return total / count_;
    }

private:
    int count_;
    double* samples_;
};
```

**Your task:** Identify its problems, then rewrite it as a small C++20 class.

**Requirements and constraints:**

- No explicit `new` or `delete`.
- Copying a batch must produce independent measurements.
- Initially, every measurement is `0.0`.
- Decide and document how invalid indices and empty-batch averages behave.
- Choose suitable parameter and return types; explain your choices.
- Include a few focused tests or assertions, including copying and an empty batch.
- Keep the public interface small.

**What a strong solution demonstrates:** Clear ownership, deliberate edge-case behavior, appropriate constness, and correct copying without unnecessary custom machinery.

Before the code, give me a short list of the original problems. Don’t just replace the allocation and assume the design is finished.


### Mapping out the code

So this code creates a measurement batch. With the constructor it takes in a count and creates that count_ as well as samples_ which is an array of doubles allocated on the heap with the size being count. All of the elements in samples_ are set to 0.0.

The destructor calls delete on the array of samples. We have a setter method that takes the index that you want samples to set the new value and we also take in a value to set. We also have an average function that updates the total for each element value of samples and then finding the average by dividing the total/count_.

### Fixing the code

The first thing I see here that seems wrong is creating dynamic array on the heap through `new double[count]`. The main problem is with copying because it violates the rule of three. Copying must produce independent measurements. For example

```
MeasurementBatch a(3);
MeasurementBatch b = a;
b.set(0, 5.0);
```

In this code our new `MeasurementBatch` b is the same array as our `MeasurementBatch` a which means that the destructor is called for the same array twice which is undefined behavior. Since we have a custom destructor we also need custom copy operations as well.

I think a much better API we could use would be `std::vector<double>` and we want the vector to contain count elements all of which are 0.0. Then we don't need a destructor or custom copy operations. 

Another issue in the original code is that we don't handle negative counts. If we pass in `MeasurementBatch(-5)` then `new double[-5]` would cause an error because you can't have an array of negative length. Our count should probably be a `std::size_t` because it involves array indexing.

Also in the original code our `set()` function has no bounds checking as well as the problem of an empty batch dividing by 0. Our `average()` function frames it as `total / count_` so if our total ends up being 0.0 because all the elements are 0 and our count is 0 then we get `NaN`.

```
#include <cassert>
#include <cstddef>
#include <optional>
#include <stdexcept>
#include <vector>

class MeasurementBatch {
public:
    explicit MeasurementBatch(std::size_t count)
        : samples_(count, 0.0) {}

    void set(std::size_t index, double value) {
        samples_.at(index) = value;
    }

    std::optional<double> average() {
        if (samples_.empty()) {
            return std::nullopt;
        }

        double total = 0.0;

        for (double s : samples_) {
            total += s;
        }

        return total / samples_.size();
    }

private:
    std::vector<double> samples_;
};

int main() {
    MeasurementBatch batch(3);

    assert(batch.average() == 0.0);

    MeasurementBatch a(2);
    MeasurementBatch b = a;
    b.set(0, 3.0);

    assert(*b.average() == 1.5);
    assert(*a.average() == 0.0); // copy should now be independent
}
```

This is the refined code that I have written up. I used a `std::vector<double>` as I said before so that the RAII patterns can be handled for us already and now I didn't need to include a loop to initialize all the elements to 0.0. 

I also use `std::size_t` instead of `int` for these values because it is best for indexing the vector and won't allow for negative values to be passed through because any negative values entered wrap around to very large positive integers. That should be fine because I used the `.at()` method in `set()` because it automatically checks and throws `std::out_of_range` if the index is out of bounds.

I also used `std::optional<double>` as the return type for the `average()` method to handle the divide by zero case. I also got rid of the `count_` private member because it was unnecessary to keep that state now.


### Feedback

**8/10. The core design is good; the submission is incomplete in its tests, and your justification for unsigned parameters is wrong.** I would request changes before approving it.

Your two follow-up answers are correct. For completeness, `snapshot` has type `const std::vector<int>` and owns its own elements. For `second`, the precise lifetime dependency is the `readings` vector object; reallocating its element storage does not destroy that object.

**1. Correctness and undefined behavior**

Your refactored class has no obvious lifetime or bounds-related UB in its normal operations:

- The vector initializes every element to `0.0`.
- `.at()` checks the index before assignment.
- The empty check prevents division by zero.
- The division is floating-point division because `total` is a `double`.

However, this explanation is incorrect:

> “won't allow for negative values to be passed through”

It **does allow negative arguments**:

```
MeasurementBatch batch(-5); // Compiles: converts -5 to std::size_t
```

The constructor then receives a huge positive count. It will generally fail with a size or allocation exception, rather than deliberately report “negative count.”

Your `.at()` argument only applies to `set()`. It provides no protection for the constructor.

Using `std::size_t` is a defensible choice because it matches the vector’s size interface. **It is not input validation.** The Core Guidelines explicitly address this in **ES.106**, which cautions against using unsigned types to exclude negative values. [isocpp.github.io](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?clid=618&version=1&utm_source=chatgpt.com)

You have two reasonable contracts:

- Accept `std::size_t` as a container-style count; validate signed external input before conversion.
- Accept a signed count and explicitly reject negative values before constructing the vector.

Choose deliberately. You do not need to change the signature merely to satisfy a blanket preference for signed or unsigned types.

**2. Ownership and resource management**

This is your strongest part. `std::vector<double>` gives the class independent copying and automatic cleanup. Removing both the custom destructor and redundant `count_` is correct.

That follows the **Rule of Zero**, reflected in **C.20**: let members implement the resource-management operations when their behavior already matches the class’s needs. [isocpp.github.io](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?utm_source=chatgpt.com)

Two refinements to your original diagnosis:

- `new[]` itself was not the correctness bug. The bug was owning an allocation while allowing default shallow copying.
- Copy assignment was also broken: it could leak the destination’s existing allocation before leaving both objects pointing to the same array.

“Since we have a custom destructor we also need custom copy operations” is too absolute. A class can instead prohibit copying. **For this exercise**, independent copying is required, so replacing the raw resource owner is the appropriate choice.

**3. Interface and design**

You missed const-correctness:

```
std::optional<double> average() const
```

Computing the average does not modify the batch. Without `const`, this legitimate use fails:

```
const MeasurementBatch batch(3);
auto result = batch.average();
```

This is an interface restriction with no justification. **Con.2** recommends making observational member functions const. [isocpp.github.io](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?utm_source=chatgpt.com)

Your `std::optional<double>` choice is appropriate: an empty batch has no average, and the return type exposes that possibility. This is a C++17 facility that fits our C++20 target well.

Document the two observable contracts briefly:

```
// Throws std::out_of_range if index is outside the batch.

// Returns std::nullopt if the batch is empty.
```

These comments communicate behavior callers need to know.

**4. Modern idioms**

Your range-based loop is clear and appropriate. You do not need to replace it with an algorithm to make the code “modern.”

The denominator is implicitly converted from `std::size_t` to `double`. An explicit conversion can clarify that choice:

```
return total / static_cast<double>(samples_.size());
```

This is a minor improvement, not a correctness fix. A cast makes the conversion explicit; it does not eliminate possible precision loss for enormous counts.

**5. Readability**

The class is small and easy to follow. `sample` would be clearer than `s` in the loop, but that is a minor naming comment.

Keep test-only headers such as `<cassert>` in the test source when you separate the class from its tests.

**6. Performance**

Nothing here needs optimization for this exercise. The average takes linear time and constant auxiliary space, which is appropriate.

A scientific production API might need a policy for nonfinite inputs and more careful summation. That is outside the current requirements; do not expand the exercise into numerical analysis.

**7. Testability and missing tests**

You explicitly omitted a required case: **there is no empty-batch test**.

The tests also omit invalid indices, despite bounds handling being one of your design decisions.

Add focused coverage for:

- Empty batch returns `std::nullopt`.
- Index equal to the size throws `std::out_of_range`.
- Setting an element in an empty batch throws.
- Copy assignment produces independent measurements.
- Calling `average()` on a const batch compiles and works.

Your copy-construction test is useful: modifying `b` and checking both averages demonstrates independence.

However:

```
assert(*b.average() == 1.5);
```

dereferences the optional before checking whether it contains anything. A broken implementation returning `nullopt` would make the test itself invalid. Your earlier comparison style avoids that:

```
assert(b.average() == 1.5);
```

Exact floating-point equality is fine for these particular expected values: `0.0` and `1.5` are exactly representable. Do not generalize that to arbitrary calculated averages.

**The single most important lesson:** A type choice must be justified by what it actually guarantees. `std::vector` guarantees ownership and copying behavior; `std::size_t` does not guarantee rejection of negative input.


### Final Code

```
#include <cassert>
#include <cstddef>
#include <optional>
#include <stdexcept>
#include <vector>

class MeasurementBatch {
public:
    explicit MeasurementBatch(std::size_t count)
        : samples_(count, 0.0) {}

    void set(std::size_t index, double value) {
        // Throws std::out_of_range if index is outside the batch.
        samples_.at(index) = value;
    }

    std::optional<double> average() const {
        // Returns std::nullopt if the batch is empty.
        if (samples_.empty()) {
            return std::nullopt;
        }

        double total = 0.0;

        for (double s : samples_) {
            total += s;
        }

        return total / static_cast<double>(samples_.size());
    }

private:
    std::vector<double> samples_;
};

int main() {
    MeasurementBatch batch(3);
    assert(batch.average() == 0.0);

    MeasurementBatch a(2);
    MeasurementBatch b = a;
    b.set(0, 3.0);

    assert(*b.average() == 1.5);
    assert(*a.average() == 0.0); // copy should now be independent

    MeasurementBatch empty(0);
    assert(!empty.average().has_value());
}
```
