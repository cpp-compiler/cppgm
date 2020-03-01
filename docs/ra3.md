## CPPGM Reading Assignment C (stdlib)

### Overview

After reading this assignment description, read the Assigned Reading Material enumerated below.

After this assignment you will be aware of the required content and interface of a C++ standard library. You will use this knowledge to develop your implementation of it.

### Standard Library Overview

During translation phase 4, when a C++ source file contains one of the following 105 preprocessing directives:

    #include <algorithm>
    #include <array>
    #include <assert.h>
    #include <atomic>
    #include <bitset>
    #include <cassert>
    #include <ccomplex>
    #include <cctype>
    #include <cerrno>
    #include <cfenv>
    #include <cfloat>
    #include <chrono>
    #include <cinttypes>
    #include <ciso646>
    #include <climits>
    #include <clocale>
    #include <cmath>
    #include <codecvt>
    #include <complex>
    #include <complex.h>
    #include <condition_variable>
    #include <csetjmp>
    #include <csignal>
    #include <cstdalign>
    #include <cstdarg>
    #include <cstdbool>
    #include <cstddef>
    #include <cstdint>
    #include <cstdio>
    #include <cstdlib>
    #include <cstring>
    #include <ctgmath>
    #include <ctime>
    #include <ctype.h>
    #include <cuchar>
    #include <cwchar>
    #include <cwctype>
    #include <deque>
    #include <errno.h>
    #include <exception>
    #include <fenv.h>
    #include <float.h>
    #include <forward_list>
    #include <fstream>
    #include <functional>
    #include <future>
    #include <initializer_list>
    #include <inttypes.h>
    #include <iomanip>
    #include <ios>
    #include <iosfwd>
    #include <iostream>
    #include <iso646.h>
    #include <istream>
    #include <iterator>
    #include <limits>
    #include <limits.h>
    #include <list>
    #include <locale>
    #include <locale.h>
    #include <map>
    #include <math.h>
    #include <memory>
    #include <mutex>
    #include <new>
    #include <numeric>
    #include <ostream>
    #include <queue>
    #include <random>
    #include <ratio>
    #include <regex>
    #include <scoped_allocator>
    #include <set>
    #include <setjmp.h>
    #include <signal.h>
    #include <sstream>
    #include <stack>
    #include <stdalign.h>
    #include <stdarg.h>
    #include <stdbool.h>
    #include <stddef.h>
    #include <stdexcept>
    #include <stdint.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <streambuf>
    #include <string>
    #include <string.h>
    #include <strstream>
    #include <system_error>
    #include <tgmath.h>
    #include <thread>
    #include <time.h>
    #include <tuple>
    #include <typeindex>
    #include <typeinfo>
    #include <type_traits>
    #include <uchar.h>
    #include <unordered_map>
    #include <unordered_set>
    #include <utility>
    #include <valarray>
    #include <vector>
    #include <wchar.h>
    #include <wctype.h>

...then your implementation shall introduce declaration of certain macros and entities into the translation unit, and enable certain language features. Any necessary definitions of entities so included must also be provided by your implementation and, as appropriate, either provided inline or linked separately into the final output program. Collectively these facilities are called the standard library.

#### Instructor Notes (optional)

To implement the standard library for the course, it is recommended to write 105 normal header files of the given names, and a set of associated .cpp files. These files are then checked in with your sources, and as part of your implementation build process these files should then be processed by a small prebuild tool you write into raw strings and then compiled and linked along with your implementation source files to be translated into your implementation program. Then, when your implementation translates programs, during translation phase 4, you can filter #include directives for these header file names, and include the text of the header from the compiled-in raw string rather than accessing the file system. When a header file is so included, the necessary associated cpp(s) can then be marked as used also. The cpp(s) can then be compiled from their raw strings as well, as if they were specified as additional translation units on the command-line. This will mean your standard library is builtin (as source text) in your final implementation, and your implementation will be a single monolithic native linux application.

Keep in mind that these header files and cpp files never need to be compiled with the bootstrap implementation, only by your implementation, so you shouldn't concern yourself necessarily with worrying about how, for example, gcc interfaces with libstdc++, or whether your standard library implementation is gcc-compatible. Your implementation (as a whole) is only required to be standard-compliant. You can use whatever internal interface between your compiler and standard library that you like.

### Assigned Reading Material

The assigned reading material is a subset of the following three documents:

*   [WG14 N1256.pdf](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)
*   [WG14 N1326.pdf](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1326.pdf)
*   [WG21 N3485.pdf](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3485.pdf)

It includes in total 1032 pages in the following five sections:

*   WG14 N1256 Annex B Library Summary (C) - 18 pages
*   WG14 N1256 Clause 7 Library (C) - 239 pages
*   WG14 N1326 - 4 pages
*   WG21 N3485 Annex C.3 C Standard Library (C -> C++ delta) - 4 pages
*   WG21 N3485 Clause 17 to 30 (inclusive) - 767 pages

#### Reading Blocks

We have arranged the reading material into 15 blocks of approximately 70 pages each. If you schedule a 90 minute reading session per day, with two sessions per block - then you should complete one pass in 1 month. The material is somewhat easier to comprehend then the core language specification so one pass should be sufficient.

    Block 1: 58 pages
        WG14 N1256 Annex B Library Summary
        WG14 N1256 7.1 Introduction
        WG14 N1256 7.2 Diagnostics <assert.h>
        WG14 N1256 7.3 Complex Arithmetic <complex.h>
        WG14 N1256 7.4 Character Handling <ctype.h>
        WG14 N1256 7.5 Errors <errno.h>
        WG14 N1256 7.6 Floating-point environment <fenv.h>
        WG14 N1256 7.7 Characteristics of floating types <float.h>
        WG14 N1256 7.8 Format conversion of integer types <inttypes.h>
        WG14 N1256 7.9 Alternative Spellings <iso646.h>
        WG14 N1256 7.10 Size of integer types <limits.h>

    Block 2: 58 pages
        WG14 N1256 7.11 Localization <local.h>
        WG14 N1256 7.12 Mathematics <math.h>
        WG14 N1256 7.13 Nonlocal Jumps <setjmp.h>
        WG14 N1256 7.14 Signal handling <signal.h>
        WG14 N1256 7.15 Variable Arguments <stdarg.h>
        WG14 N1256 7.16 Boolean type and values <stdbool.h>
        WG14 N1256 7.17 Common definitions <stddef.h>
        WG14 N1256 7.18 Integer types <stdint.h>

    Block 3: 63 pages
        WG14 N1256 7.19 Input/output <stdio.h>
        WG14 N1256 7.20 General utilities <stdlib.h>

    Block 4: 78 pages
        WG14 N1256 7.21 String handling <string.h>
        WG14 N1256 7.22 Type-generic math <tgmath.h>
        WG14 N1256 7.23 Date and time <time.h>
        WG14 N1256 7.24 Extended multibyte and wide char <wchar.h>
        WG14 N1256 7.25 Wide char class. and mapping <wctype.h>
        WG14 N1256 7.26 Future library directions

    Block 5: 61 pages
        WG14 N1326 (4 pages)
        WG21 N3485 Annex C.3 C standard library (page 1202-1205)
        WG21 N3485 Clause 17 Library introduction
        WG21 N3485 Clause 18 Language support library

    Block 6: 49 pages
        WG21 N3485 Clause 19 Diagnostics library
        WG21 N3485 Clause 20 General utilities library

    Block 7: 44 pages
        WG21 N3485 Clause 21 Strings library

    Block 8: 57 pages
        WG21 N3485 Clause 22 Localization library

    Block 9: 59 pages
        WG21 N3485 Clause 23.1 Containers library / General
        WG21 N3485 Clause 23.2 Container requirements
        WG21 N3485 Clause 23.3 Sequence containers

    Block 10: 41 pages
        WG21 N3485 Clause 23.4 Associative containers
        WG21 N3485 Clause 23.5 Unordered associative containers
        WG21 N3485 Clause 23.6 Container adapters

    Block 11: 71 pages
        WG21 N3485 Clause 24 Iterators library
        WG21 N3485 Clause 25 Algorithms library

    Block 12: 86 pages 
        WG21 N3485 Clause 26 Numerics library

    Block 13: 88 pages
        WG21 N3485 Clause 27 Input/output library

    Block 14: 48 pages
        WG21 N3485 Clause 28 Regular expressions library

    Block 15: 65 pages
        WG21 N3485 Atomic operations library
        WG21 N3485 Thread support library
