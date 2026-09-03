2026-09-01 22:56

Tags: 

Before smart pointers were a thing we could have something like a pointer containing an address and an object on the heap that needs to be destroyed eventually.

`Node* node = new Node();`

The problem here is if we forget to delete or do a double delete or we may have some ownership ambiguity if it's passed through multiple functions.

Smart pointers ensure that ownership is explicit and they follow the RAII pattern that ties resource lifetime to object lifetime.

### Unique Ptr

A unique ptr allows one smart pointer to own an object at a time. 

`std::unique_ptr<int> p = std::make_unique<int(42);`

So p owns the int and when p dies the destructor of p automatically destroys the int.

```
std::unique_ptr<int> a = std::make_unique<int>(42);

std::unique_ptr<int> b = a; // ERROR
```

This doesn't work because you can't have two objects owning the int. This prevents double deletes and if b eventually wants the int, a must transfer ownership.

```
std::unique_ptr<int> a = std::make_unique<int>(42);

std::unique_ptr<int> b = std::move(a);
```

Now a is a nullptr while b owns the int. The object was not copied, it was moved and we can't expect a to own anything now. Unique ptr establishes one unique owner.

### Shared Ptr

```
auto p = std::make_shared<int>(42);

auto q = p;
```

Unlike with unique ptr, this is allowed with shared ptr. When we need ownership to be shared we can do this where both p and q own the object. The object will not be destroyed if p or q are destroyed, because the other one needs it. So in the case of shared ptr, it uses reference counting.

The shared ptr contains a pointer to the managed object and information that connects it to the control block. The control block contains things such as reference count, weak-reference information, deleter, and allocator information. 

The object that is being owned lives as long as at least one owning shared ptr exists. As more shared ptrs gain shared ownership of that object the strong reference count goes up, and when a shared ptr disappears then the strong reference count goes down.



