https://github.com/pranavmag/Toy-DB-Storage-Engine

A small experimental storage engine written in C++ to explore how the lower-level storage layer of a database works.

This project is primarily a learning exercise. Rather than trying to build a complete database, I am implementing individual components to understand how they work and how they could eventually fit together.

## Architecture

The current architecture is intentionally simple:

```
              Application
                   │
                   ▼
              ┌─────────┐
              │ FileMgr │
              └────┬────┘
                   │
              Fixed-size
                blocks
                   │
                   ▼
              ┌─────────┐
              │  File   │
              │ (.db)   │
              └─────────┘
                   │
                   ▼
                  Disk
```

### Page

`Page` represents a fixed-size block of bytes in memory.

It provides:

- Reading bytes from a page
- Writing bytes to a page
- Bounds checking to prevent reads/writes outside the page

### File Manager

`FileMgr` handles the persistence of pages to disk.

It is responsible for:

- Opening and creating database files
- Reading a specific block from a file
- Writing a page to a specific block
- Translating block numbers into file offsets
- Caching open file handles
- Closing open files

Blocks are addressed using:

```
offset = block_number × block_size
```

For example, with a block size of 1024 bytes:

```
Block 0 → byte 0
Block 1 → byte 1024
Block 2 → byte 2048
Block 3 → byte 3072
```

This allows the file to be treated as a collection of fixed-size pages rather than one continuous stream of data.

## Project Structure

```
toy-db/
├── README.md
├── CMakeLists.txt
├── include/
│   ├── page.hpp
│   ├── file.hpp
│   └── block.hpp
├── src/
│   ├── page.cpp
│   └── file.cpp
├── tests/
│   ├── page_test.cpp
│   └── file_test.cpp
└── main.cpp
```

## Testing

The project includes unit tests for the `Page` and `FileMgr` components.

There is also a small interactive program that allows manual testing of reading and writing blocks.

## Purpose

The goal of this project is not to create a production-ready database. It is an experiment in understanding database storage internals by implementing the individual pieces myself.

Future components may include things such as a buffer pool, record management, indexes, or other database internals.