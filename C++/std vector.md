2026-06-04 21:10

Tags: [[vector]]

### Introduction

A vector is a container that stores three things:

A pointer to the data in heap memory (8 bytes)
Its size which are the elements currently in use (8 bytes)
Its capacity which are the elements allocated (8 bytes)

So on a 64 bit architecture the vector object usually tends to occupy 24 bytes on the stack while the data the vector holds is on the heap.

So if we have an empty vector

data = null
size = 0
capacity = 0

When you do something like push_back(10) on the vector, it notices

size == capacity
0 == 0

so it adds a capacity of 1
and its size is now 1 and stores one element on the heap

another push_back(20):

no room, so capacity must grow, it usually doubles capacity but the standard doesn't guarantee it so it is implementation-defined.

The only guarantee is that growth is sufficient to achieve amortized O(1) push_back.

capacity = 2
size = 2
and the heap now stores two elements

It does not create capacity for every single new element, it doubles capacity whenever needed.

So when we have like 

for (int i = 0; i < 1000; i++)
{
    v.push_back(i);
}

the vector does not allocate 1000 times, it likely grows capacity like:

1
2
4
8
16
32
64
128
...

Only about 11 allocations. That's why push_back is an Amortized O(1)

This is because sometimes v.push_back(x) could be just placing one element so it's O(1).
But sometimes it could be copying/moving thousands of elements so it could also be O(n).
Average cost over many insertions is O(1).

### Size vs Capacity

Let's say we have an empty vector of ints and we do

v.reserve(100);

Now the size is 0 but the capacity is 100. 

Reserve should be used if you know the amount of elements you wanna allocate. Instead of just looping through and adding the elements one by one and increasing capacity as you go, you can use reserve to make one allocation of the amount you specify. That is a huge performance improvement.

There is another member function called resize().

v.reserve(10) -> size = 0, capacity = 10

v.resize(10) -> size = 10, capacity >= 10

As we can see resize creates actual objects, it creates 10 ints in this case.

### Memory

Elements are contiguous so the memory layout would be like:

Address      Value

1000              1
1004              2
1008              3
1012              4

v[2] would be *(data_ + 2) which is O(1).


### Data

int* ptr = v.data();

ptr[0]
ptr[1]
ptr[2]

works exactly like an array

But you have to be careful. Let's say we have a vector v of ints {1, 2, 3} and we have

int* ptr = &v[0];
Let's say our capacity is 3.

v.push_back(4);

Old memory gets freed:
1000
1004
1008

New memory gets allocated:
5000
5004
5008
5012

Our pointer still points to 1000 which is now dead memory. This is a dangling pointer now.

This can also happen with iterators too. Iterator invalidation is:

v.reserve(3);  
  
v.push_back(1);  
v.push_back(2);  
v.push_back(3);  
  
auto it = v.begin();

v.push_back(4);

If capacity grows then, our iterator is invalid.

Any operation that causes a vector reallocation invalidates all pointers, references, and iterators into the vector.


### emplace_back()

v.push_back(std::string("hello"));
Temporary string created then moved into the vector.

v.emplace_back("hello");
Vector constructs the string directly in its storage.
No temporary object.


### Move Semantics

Move Semantics allows the elements of the vector to be moved into newly allocated storage instead of copying them.

You can use the std::move function to move it.

Lets say we have a vector of strings called v.

When reallocation happens and we have

capacity = 4
size = 4

v.push_back("hello");

The vector must relocate the old strings.

Before C++ 11 it was:
copy string 1
copy string 2
copy string 3
copy string 4

but after C++ 11 it's now:
move string 1
move string 2
move string 3
move string 4

which makes reallocation much cheaper than it was before.


### Vectors and CPU Cache

As we know, memory in vectors is like:

1000 -> 1
1004 -> 2
1008 -> 3
1012 -> 4

When the CPU loads 1000 it often loads 1000-1063 into cache.

I have a feeling this is because of spatial locality because it essentially acts like a dynamic array of contiguous memory so memory addresses close to the one just accessed will be put into cache. It usually does 64 addresses at a time like 1000 - 1063 because that would be a full cache line (usually 64 bytes).


### Vector owns its memory

A vector owns the heap allocation and when the vector is destroyed, the destructor automatically destroys all elements and frees the heap memory due to RAII.

No manual delete [] is needed. 


### Iterators

auto it = v.begin();

What is an iterator?

For vectors it's basically a fancy pointer.

An iterator is essentially an object that identifies a position within a container and allows you to move through the container.

Let's say we have a vector of its called vec and it stores {10, 20, 30} and we say
auto it = vec.begin();

Now it refers to the first element, it should look like:

10 20 30
^
it

We can dereference it like a pointer like * it

std::cout << * it;
and it should print 10

We can also increment it by saying ++it;
and now it should look like:

10 20 30
      ^
      it


### begin() and end()

Let's say we have a vector of its called vec and it stores {10, 20, 30}.

The heap memory should have

10 20 30

and when we say begin() we should now be pointing to the first element.

10 20 30
^

and when we say end() we should now be pointing one past the last element

10 20 30
               ^
Why is this the case?

This is because it makes looping much more clean.

for (auto it = vec.begin(); it != vec.end(); ++it)
{
    std::cout << * it << '\n';
}

The iteration should look like:

it -> 10
it -> 20
it -> 30
it == end()
stop

With that being said, never dereference end(). That should be obvious since it is pointing one past the last element so dereferencing it would be undefined behavior.












