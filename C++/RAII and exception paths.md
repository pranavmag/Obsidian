Our first lesson starts with your own function:

```
if (!s.empty() && seen.insert(s).second) {
    results.push_back(s);
}
```

Suppose insertion into `seen` succeeds, but `push_back` throws because an allocation fails. **Walk me through what happens to `seen`, `results`, and the caller’s input as the exception leaves the function.** You don’t need exception-safety terminology yet—just explain which objects exist, who owns them, and what gets destroyed.

### My Answer

Most likely something like a `std::bad_alloc` is thrown but `seen` already contains that insertion already whereas `results` doesn't get it. Based on what our code for Q5 had, this would mean that every subsequent string that matches the one that got an exception thrown for it will not be added to `results` because our unordered set already contains it so it won't allow it to be inserted.

The exception leaves the function but still no memory leaks will occur because RAII should be handled for these containers so as the destructors are all called they are able to be destroyed properly.

### Feedback

Your cleanup reasoning is correct. Your prediction about subsequent strings is wrong: the loop does not continue when the exception leaves the function.

You correctly identified the temporary mismatch:

- `seen` contains the newly inserted string.
- `results` does not contain it, because `push_back` failed.

But there is no handler inside the function. Execution therefore does not advance to the next iteration. Instead, the exception propagates toward a matching handler in the caller.

During that process, **stack unwinding** destroys the function’s fully constructed local objects in reverse construction order:

1. `seen` is destroyed and releases its resources.
2. `results` is destroyed and releases its resources.

The caller’s input remains unchanged. It was borrowed through a const reference, and the function never modified it. Destroying that reference does not destroy the caller’s vector.

The three main exception guarantees are:

| Guarantee | What failure promises                                                           |
| --------- | ------------------------------------------------------------------------------- |
| Basic     | No resource leaks; affected objects retain valid states, but values may change. |
| Strong    | Failure leaves observable program state unchanged.                              |
| No-throw  | No exception escapes the operation.                                             |

One check before we continue: suppose `seen` and `results` were **members of a long-lived object**, and this code ran in one of its ordinary member functions. If `push_back` threw, would unwinding automatically destroy those members and remove the mismatch? Explain why.

### My Answer

No, unwinding will only destroy automatic objects in the scopes being exited, so the function's local variables and temporary variables not exactly members because members aren't local  variables of the member function. A member function only _uses_ the object through `this`. When the exception leaves the member function, the function’s frame is torn down, but `this` pointed at an object that lives somewhere else, and that object is still there.

### Feedback

**Correct.** Leaving an ordinary member function does not end the containing object’s lifetime. Its members remain, including any partial changes.

One qualification: the owning object could subsequently be destroyed if unwinding exits its own lifetime scope. But that is separate from merely leaving its member function.