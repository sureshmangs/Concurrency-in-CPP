# Lambda

Lambda functions in C++ provide a way to define anonymous functions directly in your code. They are particularly useful for short-lived, inline operations, especially in conjunction with STL algorithms and asynchronous programming.

## Syntax

The basic syntax of a lambda function is:

```cpp
[capture](parameters) -> return_type {
    // function body
};
```

Capture List: [capture] specifies which variables from the surrounding scope are available inside the lambda. This can be:
[&]: Capture all variables by reference.
[=]: Capture all variables by value.
[variable]: Capture a specific variable by value.
[&variable]: Capture a specific variable by reference.
Parameters: (parameters) specifies the arguments that the lambda takes, similar to function parameters.
Return Type: -> return_type specifies the return type of the lambda. This is optional if the return type can be inferred.
Function Body: The block { ... } contains the code that defines what the lambda does.

### Basic Lambda Function

```cpp
#include <iostream>
using namespace std;

auto greet = []() {
    cout << "Hello World!" << "\n";
};

int main()
{
    greet();

    return 0;
}
```

Output:

```
Hello World!
```

### Lambda with Parameters

```cpp
#include <iostream>
using namespace std;

auto add = [](int a, int b) {
    return a + b;
};

int main()
{
    cout << add(3, 5) << "\n";

    return 0;
}
```

Output:

```
8
```

### Lambda with Capture by Value

```cpp
#include <iostream>
using namespace std;

int main()
{
    int x = 10;

    auto printX = [x]() {
        cout << "x: " << x << "\n";
    };

    printX();

    return 0;
}
```

Output:

```
x: 10
```

### Lambda with Capture by Reference

```cpp
#include <iostream>
using namespace std;

int main()
{
    int x = 10;

    auto incrementX = [&x]() {
        x++;
    };

    incrementX();

    cout << "x: " << x << "\n";

    return 0;
}
```

Output:

```
x: 11
```

### Lambda with Return Type

```cpp
#include <iostream>
using namespace std;

auto multiply = [](double a, double b) -> double {
    return a * b;
};

int main()
{
    cout << multiply(4.5, 2.0) << "\n";

    return 0;
}
```

Output:

```
9
```

### Lambda in STL Algorithms

```cpp
#include <iostream>
#include <vector>
#include<algorithm>
using namespace std;

int main()
{
    vector <int> numbers = {1, 2, 3, 4, 5};
    for_each(numbers.begin(), numbers.end(), [](int num) {
        cout << num << " ";
    });

    return 0;
}
```

Output:

```
1 2 3 4 5 
```

## Advanced Lambda Features

### Mutable Lambda

Allows modification of captured variables:

```cpp
#include <iostream>
using namespace std;

int main()
{
    int x = 10;

    auto lambda = [x]() mutable {
        x += 5;
        cout << "Inside lambda: x: " << x << "\n";
    };

    lambda();
    
    cout << "Inside main: x: " << x << "\n";

    return 0;
}
```

Output:

```
Inside lambda: x: 15
Inside main: x: 10
```

### Lambda with State

Capturing by reference to maintain state:

```cpp
#include <iostream>
using namespace std;

int main()
{
    int counter = 0;

    auto increment = [&counter]() {
        counter++;
        return counter;
    };

    cout << increment()  << "\n";
    cout << increment()  << "\n";

    return 0;
}
```

Output:

```
1
2
```