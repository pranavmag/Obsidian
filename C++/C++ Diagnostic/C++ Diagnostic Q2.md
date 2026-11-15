### Ownership and moving

```
#include <memory>

void update(std::unique_ptr<int> value) {
    *value = 42;
}

int main() {
    auto value = std::make_unique<int>(7);
    update(std::move(value));

    int result = *value;
}
```

- What happens to ownership at the function call?
- What is the state of `value` after `update` returns?
- Is the final dereference valid?
- Suppose the intended contract is: “modify an existing integer without taking ownership.” What parameter type would you choose, and why?

### My Answers

```
2. Ownership and moving

a) What happens to ownership at the function call? Ownership is moved at the function call. This is because a unique ptr can only have one owner at a time.

b) What is the state of value after update returns? value should now be a nullptr.

c) Is the final dereference valid? No it is not valid because you can't dereference a nullptr.

d) Suppose the intended contract is: “modify an existing integer without taking ownership.” What parameter type would you choose, and why? If ownership isn't considered then I'd likely just pass in a regular pointer because a regular pointer when you pass through a function call it passes by value so it makes a copy of the pointer but it still points to the same memory address so you can modify the existing integer that way. You just can't reseat that pointer and expect the original pointer from the caller to be modified.
```


### Feedback

**a–c are correct.** Add one missing lifetime step:

1. The function parameter receives ownership.
2. The caller’s `unique_ptr` becomes empty.
3. `update` modifies the integer.
4. The parameter is destroyed when `update` returns, deleting the integer.

So this call does not merely leave the integer owned somewhere else afterward. **The integer has been destroyed.**

Also, `std::move` itself does not transfer ownership. It produces an expression that allows the `unique_ptr` move constructor to perform that transfer.

**d) Your pointer mechanics are correct, but your interface choice is weaker than the contract suggests.**

For “modify an existing integer,” I would choose:

```
void update(int& value);
```

An existing integer is required, so a reference expresses that requirement directly. Use `int*` when absence is a meaningful part of the contract, or another concrete requirement calls for a pointer.

Both can represent non-owning access. **Being non-owning does not by itself decide between a pointer and a reference.** Whether “no object” is permitted is an important next question.

Your explanation about copying a pointer and reseating the local copy was accurate.

A reference says “there is exactly one object, always the same one.” A pointer can also say “maybe no object,” “a different object later,” or “a position in a sequence.” You pick a pointer when you need one of those extra meanings.

What a reference guarantees that a pointer doesn’t, compare the two function bodies:

```
void update(int& value) {
    value += 10;          // nothing to check: an int is always there
}

void update(int* value) {
    if (value) {          // must decide: check, or document "must not be null"
        *value += 10;
    }
}
```

With the reference, “an int must exist” is enforced by the type. A caller can’t write `update(nullptr)`; it won’t compile. The only way to get a “null reference” is to dereference a null pointer first, and that’s undefined behavior on the caller’s side before `update` even runs.

With the pointer, the requirement isn’t expressed by the type, so it has to live somewhere else: a null check, an assertion, or a comment. If “must not be null” is the real contract, a pointer makes the contract weaker than it needs to be. That’s why the feedback recommends `int&` for your case.

(A reference doesn’t guarantee the object is still _alive_, since dangling references are possible. But it does rule out the most common invalid state, which is “nothing at all.”)

