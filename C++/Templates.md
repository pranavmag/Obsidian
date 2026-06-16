2026-06-15 21:46

Tags: 

### Function Templates

A function template is a blueprint for generating functions:

```
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}
```

When you call

int x = max(2, 3);
double y = max(4.24, 6.36);

The compiler will generate the functions based on the data type:

int max(int, int)
double max(double, double);

Template argument deduction figured out that T = int.


### Class Templates

```
template<typename T>
class Vector {
public:
    void push_back(const T& value);
private:
    T* data_;
};


Vector<int> numbers;
Vector<std::string> names;
```

The compiler generates separate classes with each being its own type.

For function templates the compiler usually deduces the type but for class templates you usually have to specify it.


### Template Specialization

Sometimes the generic implementation isn't good enough. If we want to have different behavior for one data type through your templated function, you can have your generic implementation and also a specialized implementation for that specific data type.

Primary Template:
```
template<typename T>
struct Printer {
    static void print() {
        std::cout << "Generic\n";
    }
};
```

Full Specialization:
```
template<>
struct Printer<int> {
    static void print() {
        std::cout << "Integer\n";
    }
};
```


### Partial Specialization

Class templates can be partially specialized as well. Let's say we want special behavior for pointers.

Primary Template:
```
template<typename T>
struct TypeInfo {
    static void print() {
        std::cout << "Normal type\n";
    }
};
```

Partial Specialization:
```
template<typename T>
struct TypeInfo<T*> {
    static void print() {
        std::cout << "Pointer type\n";
    }
};
```

Now this can match any pointer type, so it's not a full specialization towards one specific type.

For functions there is no partial specialization because that is essentially just overload resolution anyways.


