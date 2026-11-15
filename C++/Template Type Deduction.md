### My Experimentation

```
Case 1: ParamType is a Reference or Pointer, but not a Universal
Reference

#include <iostream>

template<typename T>
void f(T& param) {
    std::cout << param << '\n';
}

int main() {
    int x = 27; // int
    const int y = x; // const int
    const int& z = x; // reference to x as a const int

    f(x); // T is an int, param's type is int&
    f(y); // T is a const int, param's type is a const int&
    f(z); // T is a const int, param's type is a const int&
}

I tried out to make sure that f(y) and f(z) are const int by adding

param = 3;

in the function and as I expected it disallowed it for those two but allows it for f(x) because it is the actual int and not const.

I knew that it changes the actual value of x which I tested

#include <iostream>

template<typename T>
void f(T& param) {
    param = 3;
    std::cout << param << '\n';
}

int main() {
    int x = 27; // int
    const int y = x; // const int
    const int& z = x; // reference to x as a const int

    f(x); // T is an int, param's type is int&
    //f(y); // T is a const int, param's type is a const int&
    //f(z); // T is a const int, param's type is a const int&

    std::cout << x << '\n';

}


So I understand the mechanisms now. I guess I just want to understand deeper of why it works this way and any other tests I can test
```

### Feedback/Explanation

Your experiments are exactly the right way to learn this, and both results are correct. Here’s the deeper reasoning, then some tests that build on what you did.

#### Why deduction absorbs `const` into `T`

When you pass `y` (a `const int`) to `f(T& param)`, the compiler has three possible designs:

1. **Deduce `T = int`**, making `param` an `int&`. That would let `f` modify a const object through `param`. It would break the whole meaning of `const`, so it’s ruled out.
2. **Reject the call.** Safe, but then a generic `f(T&)` couldn’t accept const arguments at all, which would make reference templates much less useful.
3. **Deduce `T = const int`**, making `param` a `const int&`. The call works, and `const` is preserved.

C++ chose option 3. The `const` has to live somewhere, and since the parameter is written as plain `T&`, the only place it can go is into `T`. That’s the reasoning behind your result: `param = 3` fails for `y` and `z` not because of a special rule, but because the deduced type honestly reflects that the argument can’t be modified.

### My Experimentation

```
"C++ chose option 3. The `const` has to live somewhere, and since the parameter is written as plain `T&`, the only place it can go is into `T`. That’s the reasoning behind your result: `param = 3` fails for `y` and `z` not because of a special rule, but because the deduced type honestly reflects that the argument can’t be modified."

You framed it as this and it makes sense. I had a follow up question. So following what you said, since we have T& as param's type, is that why the type of T drops all references? because we don't want to have const T&&? or is there another reason. Also that brings into mind another experiment to test.

if we take that same mental model then


#include <iostream>

template<typename T>
void f(const T& param) {
	param = 3;
	std::cout << param << '\n';
}

int main() {
	int x = 27; // int
	const int y = x; // const int
	const int& z = x; // reference to x as a const int

	f(x); // T is an int, param's type is a const int&
	f(y); // T is an int, param's type is a const int&
	f(z); // T is an int, param's type is a const int&

	std::cout << x << '\n';

}


If we take a const T& as param's type then my theories are written in the comments as so. I also hypothesized that now neither of them would work with the current call with param = 3; in the function because they all represent a const int&. I ran it and it happened as expected, it wouldn't compile and gave me complaints that it is a const int& so not even x can be changed this time.
```

### Feedback/Explanation

Your experiment and all six comments are correct. Here’s the follow-up first, because the real reason references are dropped is a bit different from your guess.

#### Why `T` never deduces to a reference here

It isn’t about avoiding a “reference to reference.” The reason is earlier and more basic: **expressions in C++ never have reference type.** When you use `z` in an expression, including as a function argument, the language adjusts its type: the reference is stripped, and the expression is an lvalue of type `const int`.

So by the time deduction runs, `f(y)` and `f(z)` look identical to the compiler. Both arguments are “an lvalue of type `const int`.” The fact that `z` was declared as a reference doesn’t disappear; it’s expressed as the expression being an **lvalue** (it refers to an existing object). The type and the value category are tracked separately, and references live in the value category.

That’s what the book means by “if expr’s type is a reference, ignore the reference part.” It’s not a special deduction rule as much as a description of how expressions already work.