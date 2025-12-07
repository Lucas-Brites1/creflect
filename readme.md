# CReflect

A lightweight reflection library for pure C. Zero external dependencies.

## What is it?

CReflect enables generic access to struct fields at runtime — something C doesn't natively support. It serves as the foundation for serialization, ORMs, and frameworks.

## Installation

Copy `reflection.h` to your project:

```c
#include "reflection.h"
```

Or use as a git submodule:

```bash
git submodule add https://github.com/lucasbrites/creflect.git deps/creflect
```

## Basic Usage

```c
#include <stdio.h>
#include "reflection.h"

typedef struct {
    char* name;
    int age;
    double salary;
} User;

int main() {
    User u = {"Lucas", 23, 2150.25};

    // Get field offsets
    t_size name_off = REFLECT_OFFSET(User, name);
    t_size age_off = REFLECT_OFFSET(User, age);

    // Access fields generically
    char* name = REFLECT_GET(&u, name_off, char*);
    int age = REFLECT_GET(&u, age_off, int);

    printf("Name: %s\n", name);  // Lucas
    printf("Age: %d\n", age);    // 23

    // Modify fields generically
    REFLECT_SET(&u, age_off, int, 24);
    printf("New age: %d\n", u.age);  // 24

    return 0;
}
```

## API Reference

### Macros

| Macro | Description |
|-------|-------------|
| `REFLECT_OFFSET(TYPE, MEMBER)` | Returns field offset in bytes |
| `REFLECT_GET(PTR, OFFSET, TYPE)` | Reads field value |
| `REFLECT_SET(PTR, OFFSET, TYPE, VALUE)` | Writes field value |

### Types

```c
typedef enum {
    REFLECT_TYPE_STRING,        // char*
    REFLECT_TYPE_CHAR,          // char
    REFLECT_TYPE_INTEGER,       // int
    REFLECT_TYPE_DOUBLE,        // double
    REFLECT_TYPE_BOOL,          // bool
    REFLECT_TYPE_OBJECT,        // nested struct
    REFLECT_TYPE_ARRAY_STRING,  // char**
    REFLECT_TYPE_ARRAY_INT,     // int*
    REFLECT_TYPE_ARRAY_DOUBLE,  // double*
    REFLECT_TYPE_ARRAY_OBJECT   // struct*
} t_reflect_kind;
```

### Metadata Structures

```c
// Describes a field
typedef struct {
    const char *name;       // Field name
    t_reflect_kind type;    // Field type
    t_size offset;          // Offset in bytes
} t_reflect_field;

// Describes a struct
typedef struct {
    const char *name;           // Struct name
    t_size size;                // sizeof(struct)
    unsigned int field_count;   // Number of fields
    t_reflect_field *fields;    // Array of fields
} t_reflect_object;
```

## Advanced Example

```c
#include <stdio.h>
#include "reflection.h"

typedef struct {
    char* name;
    int age;
    double salary;
} User;

// Define metadata
t_reflect_field user_fields[] = {
    {"name",   REFLECT_TYPE_STRING,  REFLECT_OFFSET(User, name)},
    {"age",    REFLECT_TYPE_INTEGER, REFLECT_OFFSET(User, age)},
    {"salary", REFLECT_TYPE_DOUBLE,  REFLECT_OFFSET(User, salary)},
    {NULL}
};

t_reflect_object user_meta = {
    .name = "User",
    .size = sizeof(User),
    .field_count = 3,
    .fields = user_fields
};

// Print any User using metadata
void print_user(User* u) {
    printf("=== %s ===\n", user_meta.name);

    for (unsigned int i = 0; i < user_meta.field_count; i++) {
        t_reflect_field* f = &user_fields[i];
        void* ptr = reflect_ptr_add(u, f->offset);

        printf("%s: ", f->name);

        switch (f->type) {
            case REFLECT_TYPE_STRING:
                printf("%s\n", *(char**)ptr);
                break;
            case REFLECT_TYPE_INTEGER:
                printf("%d\n", *(int*)ptr);
                break;
            case REFLECT_TYPE_DOUBLE:
                printf("%.2f\n", *(double*)ptr);
                break;
            default:
                printf("???\n");
        }
    }
}

int main() {
    User u1 = {"Lucas", 23, 2150.25};
    User u2 = {"Maria", 30, 3500.00};

    print_user(&u1);
    printf("\n");
    print_user(&u2);

    return 0;
}
```

Output:
```
=== User ===
name: Lucas
age: 23
salary: 2150.25

=== User ===
name: Maria
age: 30
salary: 3500.00
```

## How It Works

CReflect uses a simple but powerful trick: by casting `0` to a struct pointer, we can calculate field offsets without actually accessing memory.

```c
#define REFLECT_OFFSET(TYPE, MEMBER) ((t_size) & ((TYPE *)0)->MEMBER)

// Example: REFLECT_OFFSET(User, age)
// 1. (User *)0         → Pointer to User at address 0
// 2. ->age             → Access age field (no memory read!)
// 3. &                 → Get address of field = 8
// 4. (t_size)          → Cast to number = 8
```

This is the same technique used by the standard `offsetof` macro, but implemented without any dependencies.

## Why Use CReflect?

- **Zero dependencies** - Single header file
- **Portable** - Pure C, works anywhere
- **Lightweight** - No runtime overhead
- **Foundation for frameworks** - Serialization, ORM, REST APIs

## Related Projects

- [cjson](https://github.com/Lucas-Brites1/cjson) - JSON library built on CReflect
- [crest](https://github.com/Lucas-Brites1/crest) - REST framework for C

## License

MIT License - See [LICENSE](LICENSE) for details.

## Author

Lucas Silva Brites

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.
