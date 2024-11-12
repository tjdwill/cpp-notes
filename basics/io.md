# C++ I/O

In C++, I/O is done via streams. For the standard input, output, and error Files, you'd
want to use `<iostream>` and the `std::cin`, `std::cout`, and `std::cerr` entities.

For files, there's `[io]fstream`. For building strings, there is `<sstream>` and the related
`[io]stringstream` classes. These types all live within a hierarchy, deriving from
lower-level base classes. More information can be found [on cppreference](https://en.cppreference.com/w/cpp/io).


## Tips

### Parsing by line

Use the [`std::getline()`
function](https://en.cppreference.com/w/cpp/string/basic_string/getline).

### Parsing literal text

When parsing text in which we need to include spaces (ex. parsing a user-input string), you'd want to change the default
configuration of the `istream` entity. By default, istreams skip whitespace, so you want to
change to taking whitespace into consideration:

```c++
#include <sstream>

std::istringstream iss { "Some basic string"  };
iss >> std::noskipws;  // Don't skip whitespace
/* Do processing ... */
iss >> std::skipws;   // restore the default behavior 
```

Note the use of the overloaded right-shift operator `>>` to achieve this.

Another way to do this is by calling `iss.setf(std::ios_base::noskipws)`. [See more
here](https://en.cppreference.com/w/cpp/io/ios_base/fmtflags).

### Parsing floating-point numbers

In order to parse doubles, we use the
[`std::strtod`](https://en.cppreference.com/w/cpp/string/byte/strtof) function. This is a
wrapper to the C function of the same name, so it deals with C-style strings.


Basically, the way it works is that you pass in the string to parse as well as the address
of a string to store the remaining unparsed text. From experience, the function appeas to
stop as soon as it reaches an invalid character.

```c++
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

int main() {
    std::string nums { "1.323 4 54.3 -12. +7.45 nan inf" };
    std::vector<double> vec {};

    std::istringstream iss {nums};

    std::string val {};
    char* remaining {};
    while (iss >> val)
    {
        vec.push_back(std::strtod(val.c_str(), &remaining));
    }

    for (auto const& d: vec)
        std::cout << d << " ";
    std::cout << "\n";
}
```

Outputs 

    1.323 4 54.3 -12 7.45 nan inf 

as expected.
