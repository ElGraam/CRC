# CRC

This was created to help me, as a beginner in kernel development, deepen my understanding of CRC (Cyclic Redundancy Check) in the Android Generic Kernel Image (GKI).

## modversions

`modversions` is an advanced mechanism to ensure that module interfaces match without depending on version numbers.

### How It Works

`modversions` operates as follows:

1. Calculates a checksum for every module interface within the kernel.
2. Each module stores the checksum of the interfaces it uses.
3. When loading a module, it compares the expected checksum with the checksum provided by the kernel to verify interface compatibility.

### Checksum Calculation

The checksum is calculated using `scripts/genksyms` and utilizes the CRC32 format.

## Example

Below is an example using `test.h` and `test.c`.

### test.h

```c
#ifndef TEST_H
#define TEST_H

struct a {
    int a;
    long b;
};

int test(struct a *param);

#endif
```

### test.c

```c
#include <linux/module.h>
#include "test.h"

int test(struct a *param) {
    return 0;
}
EXPORT_SYMBOL(test);
```

To obtain the checksum:

```bash
$ gcc -E test.c | genksyms
__crc_test = 0x8ebc093d;
```

For more detailed information:

```bash
$ gcc -E test.c | genksyms -d
Defn for struct a == <struct a { int a ; long b ; } >
Defn for type0 test == <int test ( struct a * ) >
__crc_test = 0x8ebc093d;
Hash table occupancy 3/4096 = 0.000732422
```

## Expanding Compatibility

To maintain interface compatibility, the following methods are commonly used:

### 1. Passing Data Types via Pointers

This method is also known as using an "opaque pointer."

#### How It Works

- Hide the implementation details and treat them simply as pointers in the interface.
- The actual structure's contents need to be known only within the implementation file (`.c`).

#### Example

```c
// public_interface.h
struct my_data; // Forward declaration
void process_data(struct my_data* data);
```

```c
// implementation.c
#include "public_interface.h"

struct my_data {
    int x;
    char y;
    // Internal implementation can change without affecting the external interface
};

void process_data(struct my_data* data) {
    // Implementation
}
```

#### Advantages

- Changing the internal structure of `struct my_data` does not alter the interface.
- Users do not need to know the details of the structure and can treat it as a pointer.

### 2. Using Forward Declarations

Forward declarations allow declaring the existence of a type without providing its complete definition.

#### How It Works

- Avoid providing the full definition of a type in the interface (header file) and merely declare its existence.
- The actual definition is placed within the implementation file.

#### Example

```c
// public_interface.h
struct complex_data; // Forward declaration

struct complex_data* create_data();
void destroy_data(struct complex_data* data);
```

```c
// implementation.c
#include "public_interface.h"

struct complex_data {
    // Actual definition
    double real;
    double imag;
};

struct complex_data* create_data() {
    // Implementation
}

void destroy_data(struct complex_data* data) {
    // Implementation
}
```

#### Advantages

- Changing the internal structure of `struct complex_data` does not affect the public interface.
- Reduces compile-time dependencies, potentially shortening build times.


# References

- [Linux Kernel Documentation](https://www.kernel.org/doc/html/latest/)
- [Android Kernel overview](https://source.android.com/docs/core/architecture/kernel)

