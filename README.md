# groundtruth

*A real-world systems language benchmark, written by sysl's authors and not built to flatter it.*

groundtruth measures how fast the same everyday program runs when it is written in each of today's
systems languages. A few of its tasks are the classic compute kernels. Most are the work application code
does all day: building and searching hash maps, splitting and joining strings, parsing text,
allocating and freeing many small objects, calling through interfaces and closures, and sorting records.

It follows the rules of Heng Li's [plb2](https://github.com/attractivechaos/plb2): the same algorithm in
every language, written plainly and timed the same way. It goes further than plb2 in two ways. It has
more tasks, chosen to cover what real programs spend their time on, and it includes every systems
language that has a usable compiler.

**Status:** being built. There are no results yet. This page describes what the benchmark will measure
and how.

## Why another benchmark

- **The Computer Language Benchmarks Game** compares heavily tuned entries: hand-written SIMD, thread
  pools and custom allocators. It mostly measures how much effort each entry had.
- **plb2** is fair, but its four tasks (nqueen, matmul, sudoku, bedcov) are loops over arrays of
  integers. They hardly touch allocation, strings or hash maps.
- **Are We Fast Yet?** covers objects and collections, but it was designed for dynamic-language VMs, and
  most of the languages it has ported are garbage-collected.

groundtruth asks a plainer question: if a competent programmer writes the obvious program in each
language, how fast does it run?

## Languages

groundtruth includes any language that compiles ahead of time to native code, is statically typed, is
used to write systems software, and has a released compiler that anyone can install.

| Language | Compiler | Memory model |
|---|---|---|
| Ada | gnat | manual / controlled types |
| C | clang | manual |
| C++ | clang++ | RAII / manual |
| C3 | c3c | manual |
| Crystal | crystal | GC |
| D | ldc2 | GC (optional) |
| Go | go | GC |
| Hare | harec + qbe | manual |
| Nim | nim (ORC) | reference counting |
| Odin | odin | manual |
| Pascal | fpc (Free Pascal) | manual |
| Rust | rustc | ownership |
| Swift | swiftc | reference counting |
| sysl | sysl | reference counting |
| V | v | autofree / GC |
| Zig | zig | manual |

The memory model is listed because it explains many of the results. A table of times is easy to misread
without it.

A language is added when it meets the bar above and someone writes all of its tasks. A language that
cannot build a task gets an empty cell, with the reason given. It never gets a cell that quietly runs
different code.

## Tasks

| Task | What it exercises |
|---|---|
| nqueen | bit manipulation, recursion (from plb2) |
| matmul | nested loops over float arrays (from plb2) |
| sudoku | arrays, backtracking (from plb2) |
| bedcov | sorting, interval queries (from plb2) |
| binary-trees | allocating and freeing many small objects |
| word-freq | reading text, splitting into words, hash map counts, sorting the result |
| json | parsing a JSON document into a tree, walking it, and writing it back out |
| csv | parsing quoted CSV fields and aggregating columns |
| dispatch | a tree of shapes called through the language's interface/trait/vtable |
| closures | map, filter and fold with captured state over large collections |
| sort-records | sorting structs by several keys with a comparator |
| string-build | building large strings by formatting numbers and appending |
| graph | Dijkstra over an adjacency list with a binary heap |
| lru | an LRU cache: a hash map plus a doubly linked list, with many evictions |
| interp | a small bytecode interpreter running a fixed program |

The four plb2 tasks are ported from plb2, which is released into the public domain under CC0. They are
credited here and in each file's header.

## Rules

1. **Same algorithm, same data structures.** Each task has a specification and a reference
   implementation. A port may use its language's idioms, but it may not change the algorithm or skip work.
2. **The standard library is allowed; third-party packages are not.** Everyday code uses the language's
   own hash map, string type and sort. That is what this benchmark measures.
3. **Single-threaded. No hand-written SIMD intrinsics, custom allocators or arena tricks** that an
   ordinary program would not use. The compiler is free to vectorise.
4. **Safe by default.** Each language's safety checks stay on wherever its normal release build keeps
   them. Any `unsafe`, unchecked or `noalias` escape is listed in that port's notes, with the reason.
5. **Release builds as a user would make them:** the language's highest standard optimisation level,
   `-mcpu=native` or its equivalent, and LTO where the toolchain offers it. Every flag is recorded.
6. **Verified output.** Each task prints a checksum, and a run that prints the wrong one is not timed.

## Measurement

- **Timing.** Each binary is timed with [hyperfine](https://github.com/sharkdp/hyperfine): warm-up runs,
  then at least 10 timed runs. The median is reported, with its spread.
- **Memory.** Peak resident memory comes from `/usr/bin/time`.
- **Summary.** Each language's time on each task is divided by C's time on the same task. The geometric
  mean of those ratios is reported, along with every per-task ratio, because a geomean on its own hides
  the tasks that matter.
- **Records.** The machine, OS, every compiler version and every flag are recorded with the results,
  which are committed to the repo as JSON.
- **Reproducibility.** One script installs nothing, checks that the toolchains are present, builds,
  verifies and times. Anyone can rerun the whole benchmark.

## On sysl

sysl's authors wrote groundtruth, and sysl is one of its columns. We will not tune tasks or rules to suit
sysl:

- Every task where sysl is slower is published, with the ratio and the reason as far as we understand it.
- The sysl ports follow the same rules as everyone else's, and are no more optimised than the others.
- Every other language's port is open to review and replacement by people who know that language better
  than we do. **If a port is not what an experienced programmer in that language would write, please open
  an issue or a pull request.**

## License

ISC. The tasks ported from plb2 remain CC0.
