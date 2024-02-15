---
author: Ali Abharya
pubDatetime: 2024-02-15T09:13:38.039Z
modDatetime: 2024-02-15T09:13:38.039Z
title: Named tuple in python
slug: named-tuple
featured: false
draft: false
tags:
  - python
  - clean_code
description: Advantages of using named tuple
---

# NamedTuple in Python

## Introduction
A `namedtuple` is a convenient and lightweight data structure available in the Python standard library's `collections` module. It is essentially a subclass of tuples with named fields, providing a more readable and self-documenting way to define simple classes for storing data. Namedtuples combine the immutability of tuples with the attribute-accessibility of dictionaries, making them a valuable tool for creating simple data containers.

## Usage
To create a named tuple, you can use the `namedtuple` factory function from the `collections` module. The function takes two arguments: the typename (a string representing the name of the named tuple) and the field names (either a single string with space-separated names or an iterable of strings). Here's an example:

```python
  from collections import namedtuple

  # Define a named tuple named 'Person' with fields 'name' and 'age'
  Person = namedtuple('Person', ['name', 'age'])

  # Create an instance of the named tuple
  person_instance = Person(name='John', age=25)

  # Access fields using dot notation
  print(person_instance.name)  # Output: John
  print(person_instance.age)   # Output: 25
```

## Advantages
- Readability: Namedtuples enhance code readability by providing meaningful names to fields, making it easier to  understand the purpose of each element in the tuple.

- Immutability: Like regular tuples, namedtuples are immutable. This ensures data integrity, preventing accidental modification of the stored values.

- Memory Efficiency: Namedtuples are memory-efficient as they are implemented in C and have a smaller memory footprint compared to custom classes or dictionaries.

- Compatibility: Namedtuples are backward-compatible with regular tuples, allowing seamless integration with existing code that expects tuple-like behavior.

## When to use
Namedtuples are particularly useful in scenarios where you need a simple and lightweight data structure to represent a fixed set of fields. They are well-suited for situations such as configuration settings, representing records in a dataset, or any case where immutability and attribute-style access are beneficial.

In summary, namedtuples offer a clean and concise way to define data structures in Python, combining the benefits of tuples with the readability of named fields.

