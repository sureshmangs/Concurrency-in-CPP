# Chrono

In C++, `chrono` is a library introduced in C++11 that provides tools for handling time and durations. It is part of the Standard Library and includes classes and functions for representing time points, durations, and clocks. The `chrono` library is useful for tasks involving time measurement, timeouts, and delays.

## Key Components of the `chrono` Library

1. **Clocks:**
   - **`std::chrono::system_clock`:** Represents the system-wide real-time clock. It is typically used to get the current time. For real-time clock.
   - **`std::chrono::steady_clock`:** Represents a clock that cannot be adjusted and is guaranteed to be steady. It is used for measuring time intervals. For steady, monotonic time measurement.
   - **`std::chrono::high_resolution_clock`:** Represents the clock with the smallest tick period available on the system. It can be used for high-precision timing. For high-precision timing.

2. **Time Points:**
   - **`std::chrono::time_point`:** Represents a specific point in time. It is parameterized by a clock and a duration.
   - Epoch:
    - The epoch is the reference point from which the time duration is measured.
    - For `std::chrono::system_clock`, the epoch is the "Unix epoch," which is January 1, 1970, 00:00:00 UTC.
    - For `std::chrono::steady_clock`, the epoch is unspecified and is used primarily for measuring elapsed time rather than absolute time.
    - For `std::chrono::high_resolution_clock`, the epoch is also unspecified and similar to `steady_clock` in its usage.

3. **Durations:**
   - **`std::chrono::duration`:** Represents a time interval or duration. It is parameterized by a clock and a period (such as seconds, milliseconds, or microseconds).

4. **Utilities:**
   - **`std::chrono::duration_cast`:** Converts between different duration types.
   - **`std::chrono::time_point_cast`:** Converts between different time point types.

## Examples

### Getting Current Time

```cpp
#include <iostream>
#include <chrono>
using namespace std;

int main()
{
    // Get current time from system clock
    chrono::system_clock::time_point now = chrono::system_clock::now();

    // Epoch: January 1, 1970, 00:00:00 UTC (Unix epoch)
    cout << "Time since epoch: " << now.time_since_epoch().count() << " Nanoseconds" << "\n";

    time_t now_time_t = chrono::system_clock::to_time_t(now);

    cout << "Current time: " << ctime(&now_time_t);

    return 0;
}
```

Output:

```
Current time: Sat Aug 31 12:49:34 2024
```

### Measuring Time Intervals

```cpp
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;

int main()
{
    // Record start time
    auto start = std::chrono::steady_clock::now();

    // Perform some work
    this_thread::sleep_for(chrono::seconds(2));

    // Record end time
    auto end = chrono::steady_clock::now();

    // Calculate elapsed time
    chrono::duration <double> elapsed = end - start;

    cout << "Elapsed time: " << elapsed.count() << " seconds" << "\n";

    return 0;
}
```

Output:

```
Elapsed time: 2.00646 seconds
```

### Using Duration and Time Points

```cpp
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;

int main()
{
    // Define a duration of 5 seconds
    chrono::seconds duration(5);

    // Record the start time point
    auto start_time_point = chrono::steady_clock::now();

    // Wait for 5 seconds
    this_thread::sleep_for(duration);

    // Record the end time point
    auto end_time_point = chrono::steady_clock::now();

    // Calculate the elapsed time
    auto elapsed_time = end_time_point - start_time_point;

    auto elapsed_time_in_seconds        = chrono::duration_cast<chrono::seconds>(elapsed_time).count();
    auto elapsed_time_in_milliseconds   = chrono::duration_cast<chrono::milliseconds>(elapsed_time).count();
    auto elapsed_time_in_microseconds   = chrono::duration_cast<chrono::microseconds>(elapsed_time).count();
    auto elapsed_time_in_nanoseconds    = chrono::duration_cast<chrono::nanoseconds>(elapsed_time).count();

    // Print the elapsed time
    cout << "Elapsed time: " << elapsed_time_in_seconds << " Seconds" << "\n";
    cout << "Elapsed time: " << elapsed_time_in_milliseconds << " Milliseconds" << "\n";
    cout << "Elapsed time: " << elapsed_time_in_microseconds << " Microseconds" << "\n";
    cout << "Elapsed time: " << elapsed_time_in_nanoseconds << " Nanoseconds" << "\n";

    return 0;
}

```

Output:

```
Elapsed time: 5 Seconds
Elapsed time: 5001 Milliseconds
Elapsed time: 5001318 Microseconds
Elapsed time: 5001318325 Nanoseconds
```

### Conversion

```cpp
#include <iostream>
#include <chrono>
using namespace std;

int main()
{
    // Define a duration of 2 seconds
    chrono::seconds sec_duration(2);

    // Convert the duration from seconds to milliseconds
    chrono::milliseconds ms_duration = chrono::duration_cast<chrono::milliseconds>(sec_duration);

    cout << "Duration in seconds: " << sec_duration.count() << " seconds" << "\n";
    cout << "Duration in milliseconds: " << ms_duration.count() << " milliseconds" << "\n\n";


    // Define a duration of 1,000,000,000 nanoseconds (1 second)
    chrono::nanoseconds ns_duration(1000000000);

    // Convert the duration from nanoseconds to seconds
    chrono::seconds sec_duration1 = chrono::duration_cast<chrono::seconds>(ns_duration);

    cout << "Duration in nanoseconds: " << ns_duration.count() << " nanoseconds" << "\n";
    cout << "Duration in seconds: " << sec_duration1.count() << " seconds" << "\n";

    return 0;
}
```

Output:

```
Duration in seconds: 2 seconds
Duration in milliseconds: 2000 milliseconds

Duration in nanoseconds: 1000000000 nanoseconds
Duration in seconds: 1 seconds
```