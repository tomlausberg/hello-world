#Optimisation
Optimal approach to Optimisation:
1. Do not optimise
2. Use compiler flags
3. Find optimal algorithm, library
4. Find optimal data structure
5. Use profiling to determine slow parts of program
6. Optimise slow parts
7. Use parallelisation or vectorisation

### Timing & Profiling
- `time` is a simple shell command which returns the total & CPU runtime of a program. It is the simplest way to time your program
```sh
$ time program.exe
real    0m0.872s            #total time
user    0m0.813s            #cputime of the program
sys     0m0.016s            #cputime of other software whilst the program is running.
```

- **Manual instrumentation**: In c++11 the `<chrono>` header enables "manual instrumentation" or timing in a program. To make sure that your timings are accurate, make sure that your program times multiple repetitions. _See previous chapter for more information_
- **Profiling**  : `gprof` is widely used tool for linux which enables profiling of a program. To use it simply compile the program with `-pg`, run it and view the output with `gprof`. Gprof gives you information on how long different functions run for in your program.

```sh
$ g++ myProgram.cpp -pg -o program.exe
$ ./program.exe
$ gprof ./program.exe gmon.out > prof.txt
```

### Choice of data structures

When choosing data structures it is important to use **STL containers** wherever possible. Differing data structures have different strengths:
- Array: random access
- Linked lists: fast insertion in the middle
- Trees: middle ground, fastest if both features are needed

### Choice of algorithm

The way to find an efficient algorithm is to use a library. Many libraries are available at [Netlib.org](http://www.netlib.org/).

The main advantage of professional libraries are that they are:
- bug free, thoroughly tested
- optimised
- well documented (at least better documented then your code)
- support most architectures

### Assembly Calls and Compiler intrinsics (is this relevant?)


### Optimisation options

- `-DNDEBUG`: switch off all asserts and other tests
- `-O1`,...,`-O5`: -O flags are shortcuts to turn on many different compiler optimisation techniques. Levels 3 and higher increase the risk of wrong code being generated.
- `-Os`: optimises a program for size instead of speed
- `-O0 -g`: switch off all optimisations, for debugging
- `-Og:` enables optimisations that don't effect debugging (gcc >= 4.8)
- `-fopt-info`: gives you information on which optimisations have been done (gnu compilers)

### Common compiler optimisations

---

##### Subexpression elimination

<img src="graphics/Optimisation1.png" height="210" align="top">
<img src="graphics/Optimisation2.png" height="210" align="top">

The compiler can easily change simple expressions to eliminate unnecessary additions or loading of elements from arrays. The example below shows where the compiler has trouble. This is due to the function call in the expression. The compiler doesn't optimise the function call away because the function call my change after every call. *e.g. RNG function*

<img src="graphics/Optimisation3.png" height="230" align="top">
---
##### Strength reduction

<img src="graphics/Optimisation4.png" height="230" align="top">



### Template Meta Programs

A template meta program is a program is a program that your program at compile time using templates. The c++ can be used to optimise code by unrolling loops and deciding branches/ if-statements whilst compiling. The technique was first used by Todd Veldhuisen in the blitz++ library.

The following code snippet shows a template meta program for calculating the factorial of a number at compile time. The templated class contains simply a enum used for storing the variable.

//__TODO c++14 TMP variables??__

```c++
template<int i>
class FACTOR{
  public:
      enum {RESULT = i * FACTOR<i-1>::RESULT};
};

class FACTOR<1>{
  public:
      enum {RESULT = 1};
};

     int a = FACTOR<5>::RESULT;
}
```
