# Function arguments

## How are they passed to function

> All arguments are passed to function by **reference**

BUT

- if the argument is **immutable**. *Any modification* of the argument in the function will create a **new object** under the function scope, that will not be reflected outside the function scope
- if the argument is **mutable**, as expected, any modification of the argument inside the function will be reflected outside the function scope

## Immutable built-in types

- str
- int
- float
- tuple

## Mutable built-in types

- list
- dict
- set
