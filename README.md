# Recommended C style and coding rules

This document describes C code style used by Tilen MAJERLE in his projects and libraries.
It's based off of [this guide](https://github.com/MaJerle/c-code-style).

## Table of Contents

- [Recommended C style and coding rules](#recommended-c-style-and-coding-rules)
  - [Table of Contents](#table-of-contents)
  - [The single most important rule](#the-single-most-important-rule)
  - [Conventions used](#conventions-used)
  - [General rules](#general-rules)
  - [Comments](#comments)
  - [Functions](#functions)
  - [Error-handling](#error-handling)
  - [Variables](#variables)
  - [Structures, enumerations, typedefs](#structures-enumerations-typedefs)
  - [Compound statements](#compound-statements)
  - [Switch statement](#switch-statement)
  - [Macros and preprocessor directives](#macros-and-preprocessor-directives)
  - [Header/source files](#headersource-files)

## The single most important rule

Let's start with the quote from [GNOME developer](https://developer.gnome.org/documentation/guidelines/programming/coding-style.html) site.

> The single most important rule when writing code is this: *check the surrounding code and try to imitate it*.
>
> As a maintainer it is dismaying to receive a patch that is obviously in a different coding style to the surrounding code. This is disrespectful, like someone tromping into a spotlessly-clean house with muddy shoes.
>
> So, whatever this document recommends, if there is already written code and you are patching it, keep its current style consistent even if it is not your favorite style.

## Conventions used

The keywords *MUST*, *MUST NOT*, *REQUIRED*, *SHALL*, *SHALL NOT*, *SHOULD*, *SHOULD NOT*, *RECOMMENDED*, *NOT RECOMMENDED*, *MAY*, and
   *OPTIONAL* in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174]

## General rules

Here are listed most obvious and important general rules. Please check them carefully before you continue with other chapters.

- Use C99 standard
- gcc preferred over clang, but nothing else
- Do not use tabs, use spaces instead
- Use `2` spaces per indent level
- Use `1` space between keyword and opening bracket
```c
/* OK */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)

/* Wrong */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

- Do not use space between function name and opening bracket
```c
int32_t a = sum(4, 3);        /* OK */
int32_t a = sum (4, 3);       /* Wrong */
```

- Never use `__` or `_` prefix for variables/functions/macros/types. This is reserved for C language itself
  - Prefer `prv_` name prefix for strictly module-private (static) functions
  - Prefer `libname_int_` or `libnamei_` prefix for library internal functions, that should not be used by the user application while they MUST be used across different library internal modules
- Use only lowercase characters for variables/functions/types with optional underscore `_` char
- Opening curly bracket is always at the same line as keyword (`for`, `while`, `do`, `switch`, `if`, ...)
```c
size_t i;
for (i = 0; i < 5; ++i) {       /* OK */
}
for (i = 0; i < 5; ++i){      /* Wrong */
}
for (i = 0; i < 5; ++i)       /* Wrong */
{
}
```

- Use single space before and after comparison and assignment operators
```c
int32_t a;
a = 3 + 4;        /* OK */
for (a = 0; a < 5; ++a) /* OK */
a=3+4;          /* Wrong */
a = 3+4;        /* Wrong */
for (a=0;a<5;++a)     /* Wrong */
```

- Use single space after every comma
```c
func_name(5, 4);    /* OK */
func_name(4,3);     /* Wrong */
```

- Do not initialize `global` variables to any default value (or `NULL`), implement it in the dedicated `init` function (if REQUIRED).
```c
static int32_t a;     /* Wrong */
static int32_t b = 4;   /* Wrong */
static int32_t a = 0;   /* Wrong */
```
> In embedded systems, it is very common that RAM memories are scattered across different memory locations in the system.
> It quickly becomes tricky to handle all the cases, especially when user declares custom RAM sections.
> Startup script is in-charge to set default values (.data and .bss) while other custom sections may not be filled with default values, which leads to variables with init value won't have any effect.
>
> To be independent of such problem, create init function for each module and use it to set
> default values for all of your variables, like so:

```c
static int32_t a;     /* OK */
static int32_t b = 4;   /* Wrong - this value may not be set at zero 
              if linker script&startup files are not properly handled */

void my_module_init(void) {
  a = 0;
  b = 4;
}
```

- Declare all local variables of the same type in the same line
```c
void my_func(void) {
  /* 1 */
  char a;       /* OK */
  
  /* 2 */
  char a, b;      /* OK */
  
  /* 3 */
  char a;
  char b;       /* Wrong, variable with char type already exists */
}
```

- Always declare local variables at the beginning of the block, before first executable statement
- Always add trailing comma in the last element of structure (or its children) initialization (this helps clang-format to properly format structures). Unless structure is very simple and short
```c
typedef struct {
  int a, b;
} str_t;

str_t s = {
  .a = 1,
  .b = 2,   /* Comma here */
}

/* Examples of "complex" structure, with or with missing several trailing commas, after clang-format runs the formatting */
static const my_struct_t my_var_1 = {
  .type = TYPE1,
  .type_data =
    {
      .type1 =
        {
          .par1 = 0,
          .par2 = 1, /* Trailing comma here */
        }, /* Trailing comma here */
    },  /* Trailing comma here */
};

static const my_struct_t my_var_2 = {.type = TYPE2,
                   .type_data = {
                     .type2 =
                       {
                         .par1 = 0,
                         .par2 = 1,
                       },
                   }};  /* Missing comma here */
static const my_struct_t my_var_3 = {.type = TYPE3,
                   .type_data = {.type3 = {
                             .par1 = 0,
                             .par2 = 1,
                           }}}; /* Missing 2 commas here */

/* No trailing commas - good only for small and simple structures */
static const my_struct_t my_var_4 = {.type = TYPE4, .type_data = {.type4 = {.par1 = 0, .par2 = 1}}};
```

- Declare counter variables in `for` loop
```c
/* OK */
for (size_t i = 0; i < 10; ++i)

/* OK, if you need counter variable later */
size_t i;
for (i = 0; i < 10; ++i) {
  if (...) {
    break;
  }
}
if (i == 10) {

}

/* Wrong */
size_t i;
for (i = 0; i < 10; ++i) ...
```

- Avoid variable assignment with function call in declaration, except for single variables
```c
void a(void) {
  /* Avoid function calls when declaring variable */
  int32_t a, b = sum(1, 2);

  /* Use this */
  int32_t a, b;
  b = sum(1, 2);

  /* This is ok */
  uint8_t a = 3, b = 4;
}
```

- Except `char`, `float` or `double`, always use types declared in `stdint.h` library, eg. `uint8_t` for `unsigned 8-bit`, etc.
- Do not use `stdbool.h` library. Use `1` or `0` for `true` or `false` respectively
```c
/* OK */
uint8_t status;
status = 0;

/* Wrong */
#include <stdbool.h>
bool status = true;
```

- Never compare against `true`, eg. `if (check_func() == 1)`, use `if (check_func()) { ... }`
- Always compare pointers against `NULL` value
```c
void* ptr;

/* ... */

/* OK, compare against NULL */
if (ptr == NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

- Always use *pre-increment (and decrement respectively)* instead of *post-increment (and decrement respectively)*
```c
int32_t a = 0;
...

a++;      /* Wrong */
++a;      /* OK */

for (size_t j = 0; j < 10; ++j) {}  /* OK */
```

- Always use `size_t` for length or size variables
- Always use `const` for pointer if function should not modify memory pointed to by `pointer`
- Always use `const` for function parameter or variable, if it should not be modified
```c

/* When d could be modified, data pointed to by d could not be modified */
void my_func(const void* d) {

}

/* When d and data pointed to by d both could not be modified */
void my_func(const void* const d) {

}

/* Not REQUIRED, it is advised */
void my_func(const size_t len) {

}

/* When d should not be modified inside function, only data pointed to by d could be modified */
void my_func(void* const d) {

}
```

- When function may accept pointer of any type, always use `void *`, do not use `uint8_t *`
  - Function MUST take care of proper casting in implementation
```c
/*
 * To send data, function should not modify memory pointed to by `data` variable
 * thus `const` keyword is important
 *
 * To send generic data (or to write them to file)
 * any type may be passed for data,
 * thus use `void *`
 */
/* OK example */
void send_data(const void* data, size_t len) { /* OK */
  /* Do not cast `void *` or `const void *` */
  const uint8_t* d = data;/* Function handles proper type for internal usage */
}

void
send_data(const void* data, int len) {  /* Wrong, not not use int */
}
```

- Always use brackets with `sizeof` operator
- Never use *Variable Length Array* (VLA). Use dynamic memory allocation instead with standard C `malloc` and `free` functions or if library/project provides custom memory allocation, use its implementation
- Use Heap Allocation functions (e.g. `malloc`) as AS RARE AS POSSIBLE!
```c
/* OK */
#include <stdlib.h>
void
my_func(size_t size) {
  int32_t* arr;
  arr = malloc(sizeof(*arr) * n); /* OK, Allocate memory */
  arr = malloc(sizeof *arr * n);  /* Wrong, brackets for sizeof operator are missing */
  if (arr == NULL) {
    /* FAIL, no memory */
  }

  free(arr);  /* Free memory after usage */
}

/* Wrong */
void
my_func(size_t size) {
  int32_t arr[size];  /* Wrong, do not use VLA */
}
```

- Always compare variable against zero, except if it is treated as `boolean` type
- Never compare `boolean-treated` variables against zero or one. Use NOT (`!`) instead
```c
size_t length = 5;  /* Counter variable */
uint8_t is_ok = 0;  /* Boolean-treated variable */
if (length)     /* Wrong, length is not treated as boolean */
if (length > 0)   /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */
if (length == 0)  /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */

if (is_ok)      /* OK, variable is treated as boolean */
if (!is_ok)     /* OK, -||- */
if (is_ok == 1)   /* Wrong, never compare boolean variable against 1! */
if (is_ok == 0)   /* Wrong, use ! for negative check */
```

- Always use `/* comment */` for comments, even for *single-line* comment
- Always include check for `C++` with `extern` keyword in header file
- Use English names/text for functions, variables, comments
- Use *lowercase* characters for variables
- Use *underscore* if variable contains multiple names, eg. `force_redraw`. Do not use `forceRedraw`
- Never cast function returning `void *`, eg. `uint8_t* ptr = (uint8_t *)func_returning_void_ptr();` as `void *` is safely promoted to any other pointer type
  - Use `uint8_t* ptr = func_returning_void_ptr();` instead
- Always use `<` and `>` for C Standard Library include files, eg. `#include <stdlib.h>`
- Always use `""` for custom libraries, eg. `#include "my_library.h"`
- When casting to pointer type, always align asterisk to type, eg. `uint8_t* t = (uint8_t*)var_width_diff_type`
- Always respect code style already used in project or library

## Comments

- The most important rule concerning comments: Only comment if your code is not self-explanatory!
- Comments are important but may result in bloat of text: Use them only if needed and efficiently

- Comments starting with `//` are not allowed. Always use `/* comment */`, even for single-line comment
```c
//This is comment (wrong)
/* This is comment (ok) */
```

- For multi-line comments use `space+asterisk` for every line
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

## Functions

- Every function which may have access from outside its module, MUST include function *prototype* (or *declaration*)
- Function name MUST be lowercase, optionally separated with underscore `_` character
```c
/* OK */
void my_func(void);
void myfunc(void);

/* Wrong */
void MYFunc(void);
void myFunc();
```

- When function returns pointer, align asterisk to return type
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t a, int32_t b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```
- Align all function prototypes (with the same/similar functionality) for better readability
```c
/* OK, function names aligned */
void    set(int32_t a);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t a);
const char * get(void);
```

- Function implementation MUST include return type and optional other keywords in separate line
```c
/* OK */
int32_t
foo(void) {
  return 0;
}

/* OK */
static const char*
get_string(void) {
  return "Hello world!\r\n";
}

/* Wrong */
int32_t foo(void) {
  return 0;
}
```

## Error handling

- Exceptions are handled via integer return values
- Avoid negative values
- As in Kernel development, returning zero means success, non-zero values indicate errors
- It's recommended to describe your codes by macros in a header file, enums are also sufficient

```c
// error_codes.h
#ifndef ERROR_CODES_H
#define ERROR_CODES_H

#define SUCCESS 0
#define ERR_FILE_NOT_FOUND 1
#define ERR_INVALID_INPUT 2
#define ERR_MEMORY_ALLOCATION_FAILED 3
// Add more error definitions as needed...
#endif 
```

## Variables

- Make variable name all lowercase with optional underscore `_` character
```c
/* OK */
int32_t a;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t A;
int32_t myVar;
int32_t MYVar;
```

- Group local variables together by `type`
```c
void foo(void) {
  int32_t a, b;   /* OK */
  char a;
  char b;     /* Wrong, char type already exists */
}
```

- Do not declare variable after first executable statement
```c
void foo(void) {
  int32_t a;
  a = bar();
  int32_t b;    /* Wrong, there is already executable statement */
}
```

- You may declare new variables inside next indent level
```c
int32_t a, b;
a = foo();
if (a) {
  int32_t c, d;   /* OK, c and d are in if-statement scope */
  c = foo();
  int32_t e;    /* Wrong, there was already executable statement inside block */
}
```

- Declare pointer variables with asterisk aligned to type
```c
/* OK */
char* a;

/* Wrong */
char *a;
char * a;
```

- When declaring multiple pointer variables, you may declare them with asterisk aligned to variable name
```c
/* OK */
char *p, *n;
```

## Structures, enumerations, typedefs

- Structure or enumeration name MUST be lowercase with optional underscore `_` character between words
- Structure or enumeration may contain `typedef` keyword
- All structure members MUST be lowercase
- All enumeration members SHOULD be uppercase

When structure is declared, it may use one of `3` different options:

1. When structure is declared with *name only*, it *MUST not* contain `_t` suffix after its name.
```c
struct struct_name {
  char* a;
  char b;
};
```
2. When structure is declared with *typedef only*, it *has to* contain `_t` suffix after its name.
```c
typedef struct {
  char* a;
  char b;
} struct_name_t;
```
3. When structure is declared with *name and typedef*, it *MUST NOT* contain `_t` for basic name and it *MUST* contain `_t` suffix after its name for typedef part.
```c
typedef struct struct_name {  /* No _t */
  char* a;
  char b;
  char c;
} struct_name_t;  /* _t */
```

Examples of bad declarations and their suggested corrections
```c
/* a and b MUST be separated to 2 lines */
/* Name of structure with typedef MUST include _t suffix */
typedef struct {
  int32_t a, b;
} a;

/* Corrected version */
typedef struct {
  int32_t a;
  int32_t b;
} a_t;

/* Wrong name, it MUST not include _t suffix */
struct name_t {
  int32_t a;
  int32_t b;
};

/* Wrong parameters, MUST be all uppercase */
typedef enum {
  MY_ENUM_TESTA,
  my_enum_testb,
} my_enum_t;
```

- When initializing structure on declaration, use `C99` initialization style
```c
/* OK */
a_t a = {
  .a = 4,
  .b = 5,
};

/* Wrong */
a_t a = {1, 2};
```

- When new typedef is introduced for function handles, use `_fn` suffix
```c
/* Function accepts 2 parameters and returns uint8_t */
/* Name of typedef has `_fn` suffix */
typedef uint8_t (*my_func_typedef_fn)(uint8_t p1, const char* p2);
```

## Compound statements

- Every compound statement MUST include opening and closing curly bracket, even if it includes only `1` nested statement
- Every compound statement MUST include single indent; when nesting statements, include `1` indent size for each nest
```c
/* OK */
if (c) {
  do_a();
} else {
  do_b();
}

/* Wrong */
if (c)
  do_a();
else
  do_b();

/* Wrong */
if (c) do_a();
else do_b();
```

- In case of `if` or `if-else-if` statement, `else` MUST be in the same line as closing bracket of first statement
```c
/* OK */
if (a) {

} else if (b) {

} else {

}

/* Wrong */
if (a) {

}
else {

}

/* Wrong */
if (a) {

}
else
{

}
```

- In case of `do-while` statement, `while` part MUST be in the same line as closing bracket of `do` part
```c
/* OK */
do {
  int32_t a;
  a = do_a();
  do_b(a);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

- Indentation is REQUIRED for every opening bracket
```c
if (a) {
  do_a();
} else {
  do_b();
  if (c) {
    do_c();
  }
}
```

- Compound statement MUST include curly brackets, even in the case of a single statement. Examples below show bad practices
```c
if (a) do_b();
else do_c();

if (a) do_a(); else do_b();
```

- Empty `while`, `do-while` or `for` loops MUST include brackets
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

- If `while` (or `for`, `do-while`, etc) is empty (it can be the case in embedded programming), use empty single-line brackets
```c
/* Wait for bit to be set in embedded hardware unit */
volatile uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* Wait bit 13 to be ready */
while (*addr & (1 << 13)) {}    /* OK, empty loop contains no spaces inside curly brackets */
while (*addr & (1 << 13)) { }     /* Wrong */
while (*addr & (1 << 13)) {     /* Wrong */

}
while (*addr & (1 << 13));      /* Wrong, curly brackets are missing. Can lead to compiler warnings or unintentional bugs */
```
- Always prefer using loops in this order: `for`, `do-while`, `while`
- Avoid incrementing variables inside loop block if possible, see examples

```c
/* Not recommended */
int32_t a = 0;
while (a < 10) {
  .
  ..
  ...
  ++a;
}

/* Better */
for (size_t a = 0; a < 10; ++a) {

}

/* Better, if inc may not happen in every cycle */
for (size_t a = 0; a < 10; ) {
  if (...) {
    ++a;
  }
}
```

- Inline `if` statement MAY be used only for assignment or function call operations
```c
/* OK */
int a = condition ? if_yes : if_no; /* Assignment */
func_call(condition ? if_yes : if_no); /* Function call */
switch (condition ? if_yes : if_no) {...}   /* OK */

/* Wrong, this code is not well maintenable */
condition ? call_to_function_a() : call_to_function_b();

/* Rework to have better program flow */
if (condition) {
  call_to_function_a();
} else {
  call_to_function_b();
}
```

### Switch statement

- Add *single indent* for every `case` statement
- Use additional *single indent* for `break` statement in each `case` or `default` statement
```c
/* OK, every case has single indent */
/* OK, every break has additional indent */
switch (check()) {
  case 0:
    do_a();
    break;
  case 1:
    do_b();
    break;
  default:
    break;
}

/* Wrong, case indent missing */
switch (check()) {
case 0:
  do_a();
  break;
case 1:
  do_b();
  break;
default:
  break;
}

/* Wrong */
switch (check()) {
  case 0:
    do_a();
  break;    /* Wrong, break MUST have indent as it is under case */
  case 1:
  do_b();   /* Wrong, indent under case is missing */
  break;
  default:
    break;
}
```

- Always include `default` statement
```c
/* OK */
switch (var) {
  case 0:
    do_job();
    break;
  default:
    break;
}

/* Wrong, default is missing */
switch (var) {
  case 0:
    do_job();
    break;
}
```

- If local variables are REQUIRED, use curly brackets and put `break` statement inside.
  - Put opening curly bracket in the same line as `case` statement
```c
switch (a) {
  /* OK */
  case 0: {
    int32_t a, b;
    char c;
    a = 5;
    /* ... */
    break;
  }

  /* Wrong */
  case 1:
  {
    int32_t a;
    break;
  }

  /* Wrong, break shall be inside */
  case 2: {
    int32_t a;
  }
  break;
}
```

## Macros and preprocessor directives

- Always use macros instead of literal constants, especially for numbers
- All macros MUST be fully uppercase, with optional underscore `_` character, except if they are clearly marked as function which may be in the future replaced with regular function syntax
```c
/* OK */
#define SQUARE(x)     ((x) * (x))

/* Wrong */
#define square(x)       ((x) * (x))
```

- Always protect input parameters with parentheses
```c
/* OK */
#define MIN(x, y)       ((x) < (y) ? (x) : (y))

/* Wrong */
#define MIN(x, y)       x < y ? x : y
```

- Always protect final macro evaluation with parenthesis
```c
/* Wrong */
#define MIN(x, y)       (x) < (y) ? (x) : (y)
#define SUM(x, y)       (x) + (y)

/* Imagine result of this equation using wrong SUM implementation */
int32_t x = 5 * SUM(3, 4);  /* Expected result is 5 * 7 = 35 */
int32_t x = 5 * (3) + (4);  /* It is evaluated to this, final result = 19 which is not what we expect */

/* Correct implementation */
#define MIN(x, y)       ((x) < (y) ? (x) : (y))
#define SUM(x, y)       ((x) + (y))
```

- When macro uses multiple statements, protect these using `do {} while (0)` statement
```c
typedef struct {
  int32_t px, py;
} point_t;
point_t p;          /* Define new point */

/* Wrong implementation */

/* Define macro to set point */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)  /* 2 statements. Last one should not implement semicolon */

SET_POINT(&p, 3, 4);    /* Set point to position 3, 4. This evaluates to... */
(&p)->px = (3); (&p)->py = (4); /* ... to this. In this example this is not a problem. */

/* Consider this ugly code, however it is valid by C standard (not recommended) */
if (a)            /* If a is true */
  if (b)          /* If b is true */
    SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
  else
    SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */

/* Evaluates to code below. Do you see the problem? */
if (a)
  if (b)
    (&p)->px = (3); (&p)->py = (4);
  else
    (&p)->px = (5); (&p)->py = (6);

/* Or if we rewrite it a little */
if (a)
  if (b)
    (&p)->px = (3);
    (&p)->py = (4);
  else
    (&p)->px = (5);
    (&p)->py = (6);

/*
 * Ask yourself a question: To which `if` statement does the `else` keyword belong?
 *
 * Based on first part of code, answer is straight-forward. To inner `if` statement when we check `b` condition
 * Actual answer: Compilation error as `else` belongs nowhere
 */

/* Better and correct implementation of macro */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)  /* 2 statements. No semicolon after while loop */
/* Or even better */
#define SET_POINT(p, x, y)  do {  \   /* Backslash indicates statement continues in new line */
  (p)->px = (x);          \
  (p)->py = (y);          \
} while (0)               /* 2 statements. No semicolon after while loop */

/* Now original code evaluates to */
if (a)
  if (b)
    do { (&p)->px = (3); (&p)->py = (4); } while (0);
  else
    do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* Every part of `if` or `else` contains only `1` inner statement (do-while), hence this is valid evaluation */

/* To make code perfect, use brackets for every if-ifelse-else statements */
if (a) {          /* If a is true */
  if (b) {        /* If b is true */
    SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
  } else {
    SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */
  }
}
```

- Avoid using `#ifdef` or `#ifndef`. Use `defined()` or `!defined()` instead
```c
#ifdef XYZ
/* do something */
#endif /* XYZ */
```

- Always document `if/elif/else/endif` statements
```c
/* OK */
#if defined(XYZ)
/* Do if XYZ defined */
#else /* defined(XYZ) */
/* Do if XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
/* Do if XYZ defined */
#else
/* Do if XYZ not defined */
#endif
```

- Do not indent sub statements inside `#if` statement
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* do when ABC defined */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* Do when XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
  #if defined(ABC)
    /* do when ABC defined */
  #endif /* defined(ABC) */
#else /* defined(XYZ) */
  /* Do when XYZ not defined */
#endif /* !defined(XYZ) */
```

## Header/source files

- Leave single empty line at the end of file

- Use the same license as already used by project/library IF NECESSARY
```c

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:      FirstName LASTNAME <optional_email@example.com>
 */
```

- Header file MUST include guard `#ifndef`
- Include external header files with STL C files first followed by application custom files
- Header file MUST include only every other header file in order to compile correctly, but not more (.c should include the rest if REQUIRED)
- Header file MUST only expose module public variables/types/functions
- Use `extern` for global module variables in header file, define them in source file later
```
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* This is global variable declaration in header */

#endif

/* file.c ... */
int32_t my_variable;    /* Actually defined in source */
```
- Never include `.c` files in another `.c` file, use the linker!
- `.c` file should first include corresponding `.h` file, later others, unless otherwise explicitly necessary
- Do not include module private declarations in header file

- Header file example (no license for sake of an example)
```c
/* License comes here */
#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* Include headers */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/* File content here */

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */
```