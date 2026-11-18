2026-11-17 23:05

Tags: [[C++/Learncpp/Chapter 5|Chapter 5]]

## TL;DR
- constexpr return type must be a literal type (C++20: trivially or constexpr destructible)
- std::string is literal in C++20+ (constexpr dtor), not in C++17
- libstdc++ SSO uses pointer-into-self → non-static local constexpr std::string fails
- Fix: static, global scope, or use inside a constexpr function (transient)
- General rule: self-referencing pointer + non-static local constexpr = error
- Pragmatic fix: std::string_view   

I looked into how SSO works for libstdc++, it is different for other implementations.

```
basic_string (32 bytes, 64-bit)

offset 0 | _M_dataplus._M_p (ptr)
offset 8 | _M_string_length
offset 16 | union {
	_M_local_buf[16] (SSO buffer)
	_M_allocated_capacity (heap capacity)
}
```

```
Short (<= 15 chars): _M_p points to _M_local_buf (the data is inside the local buffer which is in the string object itself).

Long (> 15 chars): _M_p points to heap allocation. Union holds capacity instead of local buffer.

Detection: _M_is_local() -> _M_data() == _M_local_data() (pointer comparison).

```

I started looking into this because I was having some difficulty remembering when to use `constexpr` functions so I looked up the context in which to use both `constexpr` functions and `constexpr` variables and I was led into a couple of different experiments that I tested out.

```
#include <iostream>

  

struct Test {

int x{};

int y{};

};

  

constexpr Test doubleTest(const Test& t) {

Test result{};

result.x = t.x * 2;

result.y = t.y * 2;

return result;

}   

  

std::ostream& operator<<(std::ostream& out, const Test& t) {

out << t.x << ' ' << t.y << '\n';

  

return out;

}

  

int main() {

Test t{1, 2};

  

std::cout << doubleTest(t);

}
```

This is just a simple little program that takes in a Test struct into a `constexpr` function, doubles its fields, and returns that new struct. It reminded me that `constexpr` functions work when literal types (types that can be used in a constant expression) are used because these literal types contain a constexpr constructor and destructor.

Now this is where I started to think about some more experiments to test. I tried adding a `std::string` member to the Test struct like:

```
struct Test {

int x{};

int y{};

std::string s{"Hello"};

};
```

and now compiling with C++ 17 or earlier gives us a compiler error:

```
Error: Constexpr function's return type 'Test' is not a literal type
main.cpp(7, 17): 'Test' is not literal because it has data member 's' of non-literal type 'std::string' (aka 'basic_string<char>')
```

But it actually works in C++ 20 and later because `std::string` became a literal type (I actually spent a lot of time looking through forums to see if it became a literal type. I first consulted some AI's but all of them gave different answers and switched up when I pushed on the topic.) It seems that in C++ 20 the literal type requirement changed from "trivial destructor" to `constexpr` destructor so that is why `std::string` is a literal type in C++ 20.

This is when I ended up getting to SSO for `std::string`. I tried out:

```
#include <string>

int main() {

constexpr std::string s = "sjdak";

}
```

and I thought that it should compile on C++ 20 because the literal type constraints no longer apply to it and it falls within SSO. I was right that it wouldn't compile on C++ 17. It gave the error:

```
Error: Constexpr variable cannot have non-literal type 'const std::string' (aka 'const basic_string<char>')
basic_string.h(93, 11): 'basic_string<char>' is not literal because it is not an aggregate and has no constexpr constructors other than copy or move constructors
```

but I was wrong that it would compile on C++ 20, but not for the same error as C++ 17. There was a new error that I encountered:

```
main.cpp: In function ‘int main()’:
main.cpp:4:31: error: ‘std::string{std::__cxx11::basic_string<char>::_Alloc_hider{((char*)(& s.std::__cxx11::basic_string<char>::<anonymous>.std::__cxx11::basic_string<char>::<unnamed union>::_M_local_buf))}, 5, std::__cxx11::basic_string<char>::<unnamed union>{char [16]{'s', 'j', 'd', 'a', 'k', 0, '\000', '\000', '\000', '\000', '\000', '\000', '\000', '\000', '\000', '\000'}}}’ is not a constant expression
    4 |     constexpr std::string s = "sjdak";
      |                               ^~~~~~~
main.cpp:4:31: error: ‘std::__cxx11::basic_string<char>(((const char*)"sjdak"), std::allocator<char>())’ is not a constant expression because it refers to an incompletely initialized variable
```

I wondered what it meant and I realized that there was a pointer to self issue in this case. This is because the pointer for `basic_string` is pointing either to the heap allocation or the local buffer (in this scenario it points to the local buffer) and if it points to the local buffer that means that the pointer is pointing to a data member within the same object.

This is a non-static local variable meaning that the stack address can't be known at compile time. A non-static local's address is not a compile-time constant. So any type storing a self-referencing pointer can't be a non-static local `constexpr` variable. So I had to change it so a `static constexpr` variable and then it compiled:

```
#include <string>

int main() {

static constexpr std::string s = "sjdak";

}
```

This compiles and works no problem in C++ 20 and later as long as that string falls under SSO. I wanted to do another test so I wrote this:

```
#include <string>

  

struct SelfRef {

int value;

const int* self;

constexpr SelfRef(int v) : value(v), self(&value) {}

};

  

int main() {

constexpr SelfRef s{3};

}
```

This one of course does not compile because of the pointer to self issue:

```
Error: Constexpr variable 's' must be initialized by a constant expression (fix available)
main.cpp(10, 23): Pointer to subobject of 's' is not a constant expression
```

but just making it `static constexpr SelfRef s{3};` resolves it completely just like the scenario with the `std::string`. This one also compiles on C++ 17 because the `value` and `self` data members are literal types. If we make them `std::string` it won't compile on C++ 17 and earlier but will on C++ 20 and later as one last confirmation of our earlier findings:

```
#include <iostream>

#include <string>

  

struct SelfRef {

std::string value;

const std::string* self;

constexpr SelfRef(std::string v) : value(v), self(&value) {}

};

  

int main() {

static constexpr SelfRef s{"hello"};

  

std::cout << *s.self << '\n';

}
```

```
C++ 17

Error: Constexpr constructor's 1st parameter type 'std::string' (aka 'basic_string<char>') is not a literal type
basic_string.h(93, 11): 'basic_string<char>' is not literal because it is not an aggregate and has no constexpr constructors other than copy or move constructors

Error: Constexpr variable cannot have non-literal type 'const SelfRef'
main.cpp(4, 17): 'SelfRef' is not literal because it has data member 'value' of non-literal type 'std::string' (aka 'basic_string<char>')
```

```
C++ 20

bash-5.3$ ./main
hello
```

### basic_string.h

Something further that I looked into was the actual implementation of `basic_string` in `basic_string.h` for gcc the actual file. In that file under `class basic_string` I found:

```
pointer _M_p; // The actual data.
      };

      _Alloc_hider	_M_dataplus;
      size_type		_M_string_length;

      enum { _S_local_capacity = 15 / sizeof(_CharT) };

      union
      {
	_CharT           _M_local_buf[_S_local_capacity + 1];
	size_type        _M_allocated_capacity;
      };
```

which is what led to my earlier claims about it being either the pointer to the heap allocation or the local buffer through the union as well as:

```
if (_M_data() == _M_local_data())
	  {
	    if (_M_string_length > _S_local_capacity)
	      __builtin_unreachable();
	    return true;
	  }
	return false;
```

which is the check to see whether or not the actual data of the string matches the local data that would be in the buffer which tells the compiler whether or not SSO applies.





