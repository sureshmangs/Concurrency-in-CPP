# bind

In C++, `std::bind` is a function template provided in the `<functional>` header that allows you to create a callable object by binding some of the arguments of a function or callable object. It is a part of the Standard Library and provides a way to simplify function calls, particularly in contexts where you want to partially apply arguments or adapt functions for use with other components, such as algorithms or threading.

## Key Concepts

1. **Binding Arguments:**
   - `std::bind` creates a new callable object by binding specific arguments to a function or method. This means you can fix certain arguments of a function and leave others to be specified later.

2. **Placeholders:**
   - Placeholders (e.g., `std::placeholders::_1`, `std::placeholders::_2`, etc.) are used to indicate the positions of arguments that are not bound yet. These placeholders are replaced with actual arguments when the callable object is invoked.

3. **Flexible Function Calls:**
   - `std::bind` can be used with functions, member functions, and functors. It provides flexibility by allowing you to create custom callables that can be passed around and invoked later.

### Basic Syntax

```cpp
#include <functional>

auto boundFunc = std::bind(callable, arg1, std::placeholders::_1, arg2, ...);
```

Here:
- `callable` is the function or callable object to which you are binding.
- `arg1`, `arg2`, etc., are the arguments you are binding.
- `std::placeholders::_1`, `std::placeholders::_2`, etc., are placeholders for the arguments that are not bound yet.

## Examples

### Binding Free Functions

```cpp
#include <iostream>
#include <functional>
using namespace std;

void printSum(int x, int y)
{
    cout << "x: " << x << "\n";
    cout << "y: " << y << "\n";
    cout << "Sum: " << x + y << "\n";
}

int main()
{
    // Create a boundPrintSum function where the first argument is fixed
    auto boundPrintSum = bind(printSum, 10, placeholders::_1);

    // Call the boundPrintSum function with the remaining argument
    boundPrintSum(20);

    return 0;
}
```

Output:

```
x: 10
y: 20
Sum: 30
```

### Binding Member Functions

```cpp
#include <iostream>
#include <functional>
using namespace std;

class MyClass {
    public:
        void printSum(int x, int y)
        {
            cout << "x: " << x << "\n";
            cout << "y: " << y << "\n";
            cout << "Sum: " << x + y << "\n";
        }
};

int main()
{
    MyClass obj;

    // Bind member function and an instance of MyClass
    auto boundPrintSum = bind(&MyClass::printSum, &obj, 10, placeholders::_1);

    // Call the bound member function
    boundPrintSum(20);

    return 0;
}
```

Output:

```
x: 10
y: 20
Sum: 30
```

### Binding with Lambda Functions

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main()
{
    auto lambda = [](int x, int y) {
        cout << "x: " << x << "\n";
        cout << "y: " << y << "\n";
        cout << "Product: " << x * y << "\n";
    };

    // Bind the lambda function with the first argument fixed
    auto boundLambda = bind(lambda, 5, placeholders::_1);

    // Call the bound lambda function with the remaining argument
    boundLambda(4);

    return 0;
}
```

Output:

```
x: 5
y: 4
Product: 20
```