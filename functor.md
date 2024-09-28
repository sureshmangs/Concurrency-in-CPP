# Functor

In C++, a functor (or function object) is an object that can be called as if it were a function. This is achieved by defining a class or struct that overloads the `operator()`.

```cpp
#include <iostream>
using namespace std;

// Define a functor by creating a class and overloading the operator()
class Adder {
    private:
    int value;

    public:
        Adder(int value) : value(value) {}

        // Overload the function call operator
        int operator()(int x)
        {
            return x + value;
        }

        int getValue()
        {
            return value;
        }
};

int main() {
    Adder addFive(5); // Create a functor object that adds 5
    
    cout << addFive.getValue() << "\n";

    // Use the functor like a regular function
    cout << addFive(10) << "\n";

    return 0;
}
```

Output:

```
5
15
```

Key Points about Functors:
- Stateful: Functors can maintain state. In the example above, the Adder functor maintains an internal state value, which affects its behavior when called.

- Customizable Behavior: You can create functors with different internal states and behaviors, allowing for flexible and reusable code.

- Efficiency: Functors can be more efficient than function pointers in some cases because they can be inlined by the compiler and avoid some overhead.

- Usage: Functors are commonly used in the C++ Standard Library with algorithms that accept callable objects, such as std::sort or std::for_each.

Example: 

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Define a functor for comparison
class CompareDescending {
public:
    bool operator()(int a, int b) const {
        return a < b;
    }
};

int main()
{
    vector <int> numbers = {1, 5, 3, 4, 2};

    // Use the functor with sort
    sort(numbers.begin(), numbers.end(), CompareDescending());

    for (int num : numbers)
        cout << num << " ";

    return 0;
}
```

Output:

```
1 2 3 4 5 
```