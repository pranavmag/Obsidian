### Lifetime and invalidation

```
#include <string>
#include <string_view>

std::string_view make_label(int id) {
    std::string label = "device-" + std::to_string(id);
    return label;
}
```

```
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    int& first = values.front();

    values.push_back(4);

    first = 99;
}
```

- Is it correct, always incorrect, or dependent on runtime conditions?
- Identify the exact lifetime or invalidation issue.
- Suggest a repair and explain any relevant trade-off.

### My Answers

```
3. Lifetime and invalidation

a) Is it correct, always incorrect, or dependent on runtime conditions? Identify the exact lifetime or invalidation issue. I'm pretty sure it is always incorrect for both examples. For the string_view example it is returning a view to the local variable label but when the function returns label goes out of scope and the caller receives a ghost. The caller gets the string_view but the memory it points to has already been destroyed which is not desired. In the case of the vector example, we create a vector that has just enough memory allocated for three elements and we set the reference to the memory address of the first element of the vector. But right after we push_back a new element which forces a new larger block of memory elsewhere to be allocated meaning that when we set first = 99; we aren't pointing to the first element of values anymore.

b) Suggest a repair and explain any relevant trade-off. For the string_view example, just have the function return a std::string instead so we can return by value. For the vector one if we know the amount of elements we may need we can use the reserve() method beforehand to allocate a block of memory for 4 elements so when we push_back the 4th element no reallocation will occur.
```


### Feedback

Your diagnosis and repair are correct.

`label` is destroyed when the function exits. Returning a `string_view` does not extend its lifetime.

One precision correction: constructing and returning the view is not itself necessarily undefined behavior. The function returns a dangling view; subsequently reading the destroyed string’s characters through it is invalid.

Returning `std::string` gives the caller an owning result. Return-value optimization or moving can avoid an unnecessary character-buffer copy; returning by value is the appropriate design here.

Your answer “always incorrect” is wrong.

You assumed: Three initial elements means capacity is exactly three.

C++ does not guarantee that. The vector’s **size** is three; its capacity is at least three.

The result therefore depends on whether `push_back` reallocates:

| What happens           | Effect on `first`                                                     |
| ---------------------- | --------------------------------------------------------------------- |
| Reallocation occurs    | `first` dangles; assigning through it is undefined behavior           |
| No reallocation occurs | `first` remains valid; assigning through it changes the first element |

Your `reserve()` repair is valid **if called before taking the reference**, with enough capacity for the subsequent insertion:

```
values.reserve(4);
int& first = values.front();
values.push_back(4);
first = 99;
```