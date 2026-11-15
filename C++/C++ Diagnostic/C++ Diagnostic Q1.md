### Copies, references, and const

```
#include <vector>

struct Dashboard {
    std::vector<int> readings{10, 20, 30};

    const std::vector<int>& values() const {
        return readings;
    }
};

int main() {
    Dashboard dashboard;

    auto first = dashboard.values();
    const auto& second = dashboard.values();

    first[0] = 99;
    dashboard.readings[1] = 77;
}
```

- What are the exact types of `first` and `second`?
- What does each contain at the end?
- Does the `const` in `values()` prevent `dashboard.readings` from changing? Explain which object or access path it constrains.
- What must remain alive for `second` to remain usable?

### My Answers

```
1. Copies, references, and const

a) What are the exact types of first and second? Using auto here for first that stores dashboard.values() should be the type of std::vector<int>. Using just plain auto should strip away const qualifiers and references. For second it should be a const std::vector<int>& because it explicitly lists it as a const auto& so the const qualifier and reference should survive. However, I do not know the deeper reason as to why auto by itself doesn't preserve those, I just memorized the rule.

b) What does each contain at the end? first should contain {99, 20, 30}. This is possible because it is not const so first[0] = 99; is allowed and modifies the vector's first element. second should be {10, 77, 30} still. This one I am a bit confused on, because the second variable is const meaning that it shouldn't change but I am not sure if the underlying vector it is pointing to changing changes it. If second stores a reference to the readings and reading changes then it should change probably? Const means it can't change but I have a feeling in this case that is purely just to prevent elements in values from being changed directly while the underlying vector it points to can be changed as much as it needs to be changed.

c) Does the const in values() prevent dashboard.readings from changing? Explain which object or access path it constrains. I think that it shouldn't prevent dashboard.readings from changing. dashboard.readings does not have a const qualifier so it should not have any restrictions. But dashboard.values should not be able to be changed directly. You can do dashboard.readings[0] = 1 to change what dashboard.values shows but you can't do dashboard.values()[0] = 1. dashboard.values is purely dependent on dashboard.readings but doesn't restrict it in any way.

d) What must remain alive for second to remain usable? dashboards.readings must remain alive.
```


### Feedback

#### a) Auto Rules

Your rule about plain `auto` needs narrowing: **by-value deduction drops references and top-level const**, not every const qualifier.

**Top-level const** applies to the object itself: “this variable can’t be changed.”
**Low-level const** applies to something the object _points to_ or _refers to_: “the thing over there can’t be changed through me.”

```
const int a = 5;        // top-level: a itself is const
int* const p1 = &x;     // top-level: the pointer p1 can't be reseated
const int* p2 = &x;     // low-level: the int that p2 points to is const
const int* const p3;    // both: left const is low-level, right const is top-level
```

```
int i = 0;
const int ci = 1;
const int& cr = ci;
const int* pc = &ci;
int* const cp = &i;
const int* const cpc = &ci;

auto a = ci;   // int          — top-level const dropped
auto b = cr;   // int          — reference dropped, then its const becomes top-level and is dropped
auto c = pc;   // const int*   — low-level const kept
auto d = cp;   // int*         — top-level const on the pointer dropped
auto e = cpc;  // const int*   — top-level dropped, low-level kept
```

**Top-level const is dropped** because it describes the _original_ variable, not the new one. If `ci` is const, that doesn’t stop your independent copy `a` from being modified. Changing `a` can’t affect `ci`.

**Low-level const is kept** because a copied pointer still points at the _same_ object. If `auto` dropped it, `c` would become a plain `int*` and you could write `*c = 42`, modifying `ci` through the back door. Keeping low-level const preserves the promise about the shared object.

#### b) Both Final Contents

Both final contents are correct.

```
first  // {99, 20, 30}
second // {10, 77, 30}
```

Your tentative explanation is also correct. Make it definite: `second` provides read-only access to the existing vector. It does not make that vector immutable.

There is no separate vector stored inside `second`, and no snapshot. A reference aliases an object.

```
second[0] = 5;              // Error: modification through const access
dashboard.readings[0] = 5;  // Valid: modification through mutable access
// Reading second[0] now yields 5.
```

#### c) Distinct Const Qualifiers

Your conclusion is correct, but you missed that `values()` contains two distinct uses of `const`

```
const std::vector<int>& values() const
// ^ return access                 ^ member-function access
```

The return type prevents callers from modifying the vector **through the returned reference**.
The trailing `const` prevents the member function from modifying ordinary data members **through `this`**, and permits calls on const `Dashboard` objects.

**Leading `const`** (in the return type): _what can the caller do with what I hand back?_
**Trailing `const`** (after the parameter list): _what can this function do to the object it was called on?_

```
class Dashboard {
public:
    const std::vector<int>& values() const { return data_; }
    void add(int v) { data_.push_back(v); }
private:
    std::vector<int> data_;
};
```

The trailing `const`: protects the object from the function
Every non-static member function has a hidden parameter, `this`. The trailing `const` changes its type:

```
// without trailing const:  this is  Dashboard*
// with trailing const:     this is  const Dashboard*
```

That has two consequences.

**1. Inside the function, members are read-only.** Since `data_` really means `this->data_`, and `this` points to a const `Dashboard`, `data_` is treated as `const std::vector<int>`:

```
const std::vector<int>& values() const {
    data_.push_back(1);   // error: data_ is const in here
    return data_;
}
```

**2. The function can be called on const objects.** A const `Dashboard` can only call member functions that promise not to modify it:

```
void show(const Dashboard& d) {
    d.values();   // OK: values() is const
    d.add(5);     // error: add() isn't const, so it might modify d
}
```

Point 2 is often the more important one in practice. Any time a `Dashboard` is passed around as `const Dashboard&`, which is very common, only its const member functions are usable. A getter that forgets the trailing `const` becomes uncallable in all that code.

The leading `const`: protects the returned data from the caller
The function returns a reference, so the caller gets an alias to the real `data_` inside the object, not a copy. The leading `const` decides what the caller may do through that alias:

Dashboard d;
d.values().push_back(99);   // error: the returned reference is const

```
Dashboard d;
d.values().push_back(99);   // error: the returned reference is const
int first = d.values()[0];  // OK: reading is fine
```

#### d) Vector Object

**Correct.** The vector object must remain alive. Here, its lifetime is tied to the containing `dashboard`.
