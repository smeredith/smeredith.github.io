---
layout: page
title: C++
---

# C++

## Coding Conventions

* <http://llvm.org/docs/CodingStandards.html>
* <https://google.github.io/styleguide/cppguide.html>
* <https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md>

## Static Analysis

### cppcheck

    $ cppcheck --enable=all --suppress=missingIncludeSystem -I src/lib *.cpp

### Clang scan-build

    $ mkdir buld-analyze
    $ cd build-analyze
    $ cmake -DCMAKE_C_COMPILER=/usr/share/clang/scan-build-3.6/ccc-analyzer ..
    $ scan-build make

### Clang-tidy

    $ clang-tidy -header-filter='.*' -checks=-*,clang-analyzer-*,google-build-*,google-explicit-constructor,google-readability-*,google-runtime-*,-google-runtime-int,llvm-*,-llvm-header-guard,misc-*,readability-* *.cpp

### Clang-moderize

    $ clang-moderize *.cpp

### Include What You Use

See <https://github.com/include-what-you-use/include-what-you-use>.
Clone the project. Build it out-of-tree. Copy the `include-what-you-use`
binary to `/usr/bin`. Requires `compile_commands.json`.

    $ python ../include-what-you-use/iwyu_tool.py -p . 2> log.txt

It works pretty well for the small projects I tried it on.

## Random

* random\_device
* mt19937
* mt19937\_64
* uniform\_int\_distribution

Example:

    #include <iostream>
    #include <random>
    int main()
    {
        std::random_device rd;
        unsigned int seed = rd();
        std::mt19937 mt(seed); // [0..2^32)
        std::uniform_int_distribution<int> dist(0,99); // [0..99]
        for (int i = 0; i < 16; ++i) {
            std::cout << dist(mt) << endl;
        }
    }

## LLVM/Clang

To install, add llvm to apt:
    <http://llvm.org/apt/>

    $ wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key|sudo apt-key add -
    $ sudo apt-get install clang-3.6 clang-3.6-doc libclang-common-3.6-dev libclang-3.6-dev libclang1-3.6 libclang1-3.6-dbg libllvm-3.6-ocaml-dev libllvm3.6 libllvm3.6-dbg lldb-3.6 llvm-3.6 llvm-3.6-dev llvm-3.6-doc llvm-3.6-examples llvm-3.6-runtime clang-modernize-3.6 clang-format-3.6 python-clang-3.6 lldb-3.6-dev
    $ sudo apt-get install clang
    $ sudo ln -s /usr/lib/llvm-3.6/bin/clang-format /usr/bin/clang-format

Clang tools need compile\_commands.json in the root of the project. To
create, use cmake to create the file in build and use a symbolic link to
add it to the project root.

    $ mkdir build
    $ cd build
    $ cmake .. -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
    $ ln -s $PWD/compile_commands.json ../

### Usual Cmake Options

    add_compile_options(-std=c++14)
    add_compile_options(-fno-rtti)
    add_compile_options(-Wall)
    add_compile_options(-Wextra)
    add_compile_options(-Wshadow)
    add_compile_options(-Wnon-virtual-dtor)
    add_compile_options(-Wold-style-cast)
    add_compile_options(-Wcast-align)
    add_compile_options(-Wunused)
    add_compile_options(-Woverloaded-virtual)
    add_compile_options(-pedantic)
    add_compile_options(-Werror)

### Code Coverage

See <http://gcovr.com/guide.html> and
<http://logan.tw/posts/2015/04/28/check-code-coverage-with-clang-and-lcov/>,
neither of which I could make work.

To emit coverage data using LLVM via CMake, add these options to
CMakeLists.txt:

    add_compile_options(-fprofile-arcs)
    add_compile_options(-ftest-coverage)
    add_compile_options(-g)
    add_compile_options(-O0)

and link to:

    /usr/lib/clang/3.6/lib/linux/libclang_rt.profile-x86_64.a

The coverage data files are put next to the object files.

## Performance

### Benchmarking

See <https://github.com/google/benchmark>

### Profiling with perf

For profiling, use the Linux tool "perf".

**Build Flags from Chandler Carruth?s CppCon 2015 Talk
(<https://www.youtube.com/watch?v=nXaxk27zwlk>).**

    -O3
    -std=c++14
    -stdlib=libc++
    -lc++abi
    -Wl,-rpath=/home/smeredith/lib64
    -fno-exceptions
    -fno-rtti
    -Wall
    -pedantic
    -Werror
    -isystem /home/smeredith/include
    -pthread
    -fno-omit-frame-pointer

Can use a file for flags and pass this to compiler via: `$(< flags)`.
Link with `-lbenchmark`.

#### Overview

    % perf stat ./a.out

* task-clock: CPU time
* Other stats.

#### Call Graph

    % perf record ./a.out
    % perf report

* Reports flat profile of which functions and how long in each.
* Sampling.

For call graph, use `-g` switch to record and report. Interrupts and
looks at stack. Need to build with `-fno-omit-frame-pointer`.

    % perf record -g ./a.out
    % perf report -g

* Shows list of callees.
* Expand a callee to see a list of callers of it (not the callees).
* Self is time spent in reported function.
* Children is time spent in the callees.
* From report, can hit *a* to see annotated assembly.

      % perf report -g 'graph,0.5,caller'

"graph" is one kind of graph which shows you percentages of total time.
The other is "fractal", which shows you time as a percentage of parent,
which is confusing because parent may be tiny fraction of total and you
lose that context. (Nothing to do with fractals). "0\.5" is a filter for
the lowest times. "caller" inverts the graph: expand to see which
functions a function calls, not the callers.

#### Defeat the Optimizer

(Doesn't work on VS)

    // Write to all memory.
    static void escape(void *p)
    {
        // "volatile" indicates that the asm has side effects so not to optimize it away.
        asm volatile("" : : "g"(p) : "memory");
    }

    // Read and write all memory in entire system.
    static void clobber()
    {
        asm volatile("" : : : "memory");
    }

To use:

    std::vector<int> v;
    v.reserve(1);
    escape(v.data());    // Don't optimize away v even though it is not used.
    v.push_back(42);
    clobber();

But these functions don?t generate any code so won?t affect benchmarks.

### Profiling use gprof

* build using `-pg`
* did not get this to work with clang

### Heap Profiler

<http://milianw.de/blog/heaptrack-a-heap-memory-profiler-for-linux>

    $ git clone git://anongit.kde.org/heaptrack
    $ cd heaptrack
    $ mkdir build
    $ cd build
    $ cmake -DCMAKE_BUILD_TYPE=Release ..
    $ make install

## Operators

Make unary operators members.

    += -= *= /= %=
    |= &= ^= <<= >>=
    Pre-increment, post-increment.

Make binary operators with const params free functions unless they change one of the values.

    == !=
    < <= > =>
    + - * / %
    | & ^ << >>

Implement operator\*() and friends in terms of operator\*=() and
friends. Like:

    T operator*(T lhs, const T& rhs) {
        lhs *= rhs;
        return lhs;
    }

Notice that the first param is passed by value so a copy is taken.

Implement assignment operator using the copy-and-swap idiom:

    T& T::operator=(T rhs) {
        swap(rhs);
        return *this;
    }

or copy-and-move:

    T& T::operator=(T rhs) {
        *this = std::move(rhs);
        return *this;
    }

Note that the above two techniques are unifying assignment operators
because they cover both copy assignment and move assignment. The way the
copy version works is obvious: the value of the argument is copied to
the parameter. For the move, when bound to an rvalue, the compiler
elides the copy as an optimization.

For stream operations, use free functions:

    std::ostream& operator<<(std::ostream& os, const T& obj)
    {
        // write obj to stream
        return os;
    }

    std::istream& operator>>(std::istream& is, T& obj)
    {
        // read obj from stream
        if( /* no valid object of T found in stream */ )
            is.setstate(std::ios::failbit);
        return is;
    }

Implement comparison operators as free functions. You need `operator<()`
and `operator==()` and can implement the rest with those.

Implement `operator++(int)` in terms of `operator++()`.

    T& T::operator++() {
        // increment the thing
        return *this;
    }

    T T::operator++(int) {
        T old(*this);
        ++*this;
        return old;
    }

* could use std::rel\_ops to define the trivial ones, but be careful as it might add operators to things you didn't intend
* boost::totally\_ordered can be used to do the same thing in a more sane fashion

## No Raw Loops

* This concept comes from Scott Meyers in 2001 and Sean Parent in his talk C++ Seasoning <https://channel9.msdn.com/Events/GoingNative/2013/Cpp-Seasoning>
* See also *A Tour of C++*, Bjarne Stroustrup, p166 "Know your standard-library algorithms and prefer them to hand-crafted loops."
* range-for is probably better than for\_each()

## No Inheritance for Implementation Reuse

* <http://www.peterprovost.org/blog/2008/06/30/Inherit-to-Be-Reused2c-Not-to-Reuse/>
* <http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html>
* See "Remove Unnecessary Inheritance" section in <http://www.gotw.ca/publications/mill04.htm>
* see <https://medium.com/@cscalfani/goodbye-object-oriented-programming-a59cda4c0e53#.6y5q4zrei>
* See Item 36-38 in C++ Coding Standards, 101 Rules

## STL

Interview on the design of the STL and the flaws of OOP:
<http://www.stlport.org/resources/StepanovUSA.html>

## Type Deduction

* <https://www.youtube.com/watch?v=wQxj20X-tIU>

auto type deduction uses template type deduction

* throw away ref
* if now const or volitile, throw those away
* auto never becomes a ref type
* auto creates a new value
* a braced initializer has no type, but ends up as std::initializer\_list

lambda capture

* uses auto rules
* value type doesn?t throw away const or volatile

Observing Deduced Types

* Avoid std::type\_info::name.

      // Delcare template class but don't define it.
      template<typename T>
      class TD;

      // T and param's type displayed in compiler error messages.
      template<typename T>
      void f(T& param)
      {
          TD<T> tType;
          TD<decltype(param)> paramType;
      }

      // Works also for auto.
      auto y = rx;
      TD<decltype(y)> tType;

## Move Semantics

* <https://skillsmatter.com/skillscasts/2188-move-semanticsperfect-forwarding-and-rvalue-references>
* <http://thbecker.net/articles/rvalue_references/section_01.html>
* <http://jlebar.com/2016/5/28/A_Practical_Introduction_to_C%2B%2B11_Rvalue_References.html>

## Special Member Functions

* <http://stackoverflow.com/questions/24342941/what-are-the-rules-for-automatic-generation-of-move-operations#24512883>
* <https://msdn.microsoft.com/en-us/library/dn457344.aspx>

![special member functions](images/special-members.png)

