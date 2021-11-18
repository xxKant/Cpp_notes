# C++ Code Smells

## 01 Construction Separate From Assignment

```cpp
import <string>;

int main()
{
    std::string str;
    // Do some stuff
    str = "Hello World";
    // Work with str
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <string>;

int main()
{
    const std::string str { "Hello World" };
    // Work with str
}

```

To see the difference take a look at the corresponding assembly code of each snippet. Keyword: small string optimization
</details>

## 02 Out Variables

```cpp
import <string>;

void getValue(std::string& outParam);

int main()
{
    std::string value;
    getValue(value);
    // Use value
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <string>;

std::string getValue();

int main()
{
    const auto value { getValue() };
}
```

Further details can be found [here](https://stlab.cc/tips/stop-using-out-arguments.html).
</details>

## 03 Raw Loops

```cpp
void processMore(const std::vector<double>& values);

void processData(const std::vector<double>& values)
{
    bool inRange { true };
    for (const auto& v : values)
    {
        if (v < 5.0 || v > 100.0)
        {
            inRange = false;
            break;
        }
    }

    if (inRange)
    {
        processMore(values);
    }
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <algorithm>;
import <ranges>;
import <vector>;

void processMore(const std::vector<double>& values);

void processData(const std::vector<double>& values)
{
    const auto inRange = [](const double d)
    {
        return d >= 5.0 && d <= 100.0;
    };

    const bool allInRange = std::ranges::all_of(values, inRange);

    if (allInRange)
    {
        processMore(values);
    }
}
```

</details>

## 04 Multi-Step Functions

```cpp
double Data::totalArea()
{
    int value = 0;

    // Pipe area
    for (int i = 0; i < pipes.size(); ++i)
    {
        value += pipes[i].radius * pipes[i].radius * M_PI;
    }

    // Hose area
    for (int i = 0; i < hose.size(); ++i)
    {
        value += hose[i].radius * pipes[i].radius * M_PI;
    }

    // And many more

    return value;
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <numeric>;

constexpr double area(const double r)
{
    return r * r * M_PI;
}

double Data::totalArea() const
{
    const auto accumulateArea = [](const auto lhs, const auto rhs)
    {
        return lhs + area(rhs);
    };

    const auto totalArea = [&](const auto& container)
    {
        // Not yet in ranges :(
        return std::accumulate(std::begin(container),
                               std::end(container),
                               0.0,
                               accumulateArea);
    };

    return totalArea(pipes) + totalArea(hoses); // + Other things
}
```

</details>

## 05 Non-Cannonical Operators

```cpp
struct Data
{
    int x;
    int y;

    bool operator==(Data& rhs)
    {
        return x == rhs.x && y == rhs.y;
    }
};
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
struct Data
{
    int x;
    int y;

    [[nodiscard]] bool operator==(const Data& rhs) const
    {
        return x == rhs.x && y == rhs.y;
    }
};
```

</details>

## 06 Code With Implicit Constructors

```cpp
import <string>;

void useString(const std::string& s);

std::string getString();

int main()
{
    const std::string str = getString();
    useString(str.c_str());
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <string>;
void useString(const std::string& s);

std::string getString();

int main()
{
    const std::string str { getString() };
    useString(str);
}
```

</details>

## 07 Code With Conversions

```cpp
import <string>;

std::string getValue()
{
    std::string s = "Hello World";
    return std::move(s);
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <string>;

std::string getValue()
{
    return "Hello World";
}
```

Keyword: (N)RVO

</details>

## 08 Casting Away `const`

```cpp
int main()
{
    const int i = 4;
    const_cast<int&>(i) = 13;
    return i; // What is returned?
}
```

<details>
<summary>Improved code</summary>
<br>

Modifying a `const` object during its lifetime is undefined behavior.
`const_cast` is another explicit conversion that is a code smell.
</details>

## 09 Static Variables

```cpp
import <string>;

void logError(const std::string& location, const std::string& desc);

void doThings(bool const error)
{
    static const std::string FunctionName("do_things");

    if (error)
    {
        logError("FunctionName", "Error occured!");
    }
}

```

<details>
<summary>Improved code</summary>
<br>

Each time the variable is accessed it must be checked to see if it's been
initialized. Furthermore, `static const` should probably be `constexpr`.

 ```cpp
import <string_view>;

void logError(std::string_view location, std::string_view description);

void doThings(const bool error)
{
    constexpr static std::string_view FunctionName("do_things");

    if (error)
    {
        logError("FunctionName", "Error occured!");
    }
}
```

</details>

## 10 Extern `const`

```cpp
// Data.h
extern int const Value;
```

```cpp
// Data.cpp
#include <Data.h>

int const Value = 5;
```

```cpp
// Value.cpp
#include <Data.h>

int getValue()
{
    return Value;
}
```

<details>
<summary>Improved code</summary>
<br>
Simplified:

 ```cpp
// Data.h
constexpr int Value { 5 };

int getValue()
{
    return Value;
}
```

</details>

## 11 Raw `new` And `delete`

```cpp
void useInt()
{
    int* i = new int(5);
    delete i;
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <memory>;

void useInt()
{
    auto i { std::make_unique<int>(5) };
}
```

</details>

# Exercises

Refactor the following code snippets

## Exercise 01

```cpp
#include <string>

using namespace std;

int main()
{
    int length;
    string greet1 = "Hello";
    string greet2 = " World!";
    string greet3 = greet1 + greet2;

    length = greet3.size();
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <string>;

int main()
{
    const std::string greet { "Hello World!" };
    const auto length { std::size(greet) };
}
```

</details>

## Exercise 02

```cpp
#include <iostream>

int main()
{
    int i, n, fact = 1;

    std::cout << "Enter a whole number: ";
    std::cin >> n;

    for (i = 1; i <= n; ++i)
    {
        fact *= i;
    }

    std::cout << "\nFactorial of " << n << " = " << fact << std::endl;

    return 0;
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <iostream>;

template<typename T>
T readInput()
{
    T obj;
    std::cin >> obj;
    return obj;
}

constexpr int factorial(int value)
{
    int result { 1 };
    while (value > 0)
    {
        result *= value;
        --value;
    }
    return result;
}

int main()
{
    std::cout << "Enter a whole number: ";
    const auto n { readInput<int>() };
    const auto fact { factorial(n) };
    std::cout << "\nFactorial of " << n << " = " << fact << "\n";
}
```

</details>

## Exercise 03

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <iostream>;

template<typename T>
T readInput()
{
    T obj;
    std::cin >> obj;
    return obj;
}

constexpr int factorial(int value)
{
    int result { 1 };
    while (value > 0)
    {
        result *= value;
        --value;
    }
    return result;
}

int main()
{
    std::cout << "Enter a whole number: ";
    const auto n { readInput<int>() };
    const auto fact { factorial(n) };

    std::cout << "Factorial of " << n << " = " << fact << "\n";
}
```

</details>

##

```cpp
#include <vector>
#include <limits>

int range(std::vector<int>& values)
{
    int min = std::numeric_limits<int>::max();
    int max = std::numeric_limits<int>::min();

    for (int i = 0; i < values.size(); ++i)
    {
        if (values[i] < min)
        {
            min = values[i];
        }

        if (values[i] > max)
        {
            max = values[i];
        }
    }

    return max - min;
}
```

<details>
<summary>Improved code</summary>
<br>

 ```cpp
import <ranges>;

template<typename Rng>
auto range(const Rng r)
{
    const auto [min_elem, max_elem] = std::ranges::minmax_element(r);
    return *max_elem - *min_elem;
}
```

</details>

## Source

* [C++ Code Smells - Jason Turner - CppCon 2019](https://www.youtube.com/watch?v=f_tLQl0wLUM)
