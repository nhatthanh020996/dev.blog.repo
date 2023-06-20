---
layout:     post
title:      "Understanding Variable Behavior in Python and C: A Comparison"
description: "Discover the differences in variable behavior between Python and C programming languages. Explore topics such as passing by value and reference, immutable and mutable objects, and their impact on stored values. Through concise code examples, gain insights into how variables are handled in these languages. Unveil the distinctions in variable manipulation in this concise article."
date:    2023-06-15
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Python
URL: "/does-python-function-use-calling-value-or-calling-reference/"
categories: [ Tech ]
---
## In C.

### 1. Variable in C.
We start with the following simple example:

```C
#include <stdio.h>

int main()
{
    int x;
    printf("address of x: %p", &x);
    printf("\nvalue of x: %d", x);
        
        x = 1
    printf("address of x: %p", &x);
    printf("\nvalue of x: %d", x);
    return 0;
}

// the result will be:
/*
address of x: 0x7ffcd47a5cc4
value of x: 0
address of x: 0x7ffcd47a5cc4
value of x: 1
*/
```
In the above piece of code, when declaring x, we create a space that occupies 4bytes in memory for x variable. This space holds zero value, in this cace zero value equals to 0.

When we assign x to 1, the space of x in memory will hold 1.



### 2. Passing by value in C.
We have a the following function:
```C
#include<stdio.h>

int sum(int a, int b)
{
	return a + b
}

int main()
{
	int x;
	int y;
	x = 1;
	y = 2;
	s = sum(x, y);
	printf("s: %d", s);
	return 0;	 
}
```
In the above example, the sum() function contains a and b as local variables, which are stored in the stack frame. When the program reaches the line s = sum(x, y) in the main() function, a will copy the literal value of x, b will copy the literal value of y, and sum() function will return 3. Since the parameters in sum() function are passed by the literal value of outside function variables, we call this passing by value.


### 3. Passing by reference in C.
We have a the following function:
```C
#include<stdio.h>

void swap(int *a, int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

int main()
{
    int x;
    int y;
    x = 1;
    y = 2;
    swap(&x, &y);
    printf("value of x: %d", x);
    printf("\nvalue of y: %d", y);
    return 0;
}

/*
value of x: 2
value of y: 1
*/
```
In the above piece of code, swap() function receives parameter as pointers instead of value. Thereby, it call this passing by references.


## In Python.

### 1. Variable in Python.
The concept of variable in python is understood differently compared to C. In Python, we have some concepts: name and object. We’ll go through the following example to understand more about these definition, and see the differences between C and Python:

```Python
x = 2337
print('address of x: {}'.format(id(x)))
x = 2338
print('address of x after reassignment: {}'.format(id(x)))

y = x
print('address of y: {}'.format(id(y)))
print('do x and y point to the same object in memory? :{}'.format(x is y))

y += 1
print('address of y after adding 1: {}'.format(id(y)))

"""
address of x: 140080412256560
address of x after reassignment: 140080412256592
address of y: 140080412256592
do x and y point to the same object in memory? :True
address of y after adding 1: 140080412257520
"""
```

Let’s explain the above result:
* `x = 1` an object that hold value 1 will be created in memory. In the CPython implementation, which is the reference implementation of Python, this object is implemented as a structure called PyObject in C.

* PyObject is a fundamental structure in CPython that represents a generic Python object. It contains information about `object’s type`, `reference count`, and other necessary details for proper memory management and manipulation.

* `x = 2338`, `x` would be bound to the new PyObject. In previous PyObject that was bound to `x`, the value of reference count would be decrease to 0, this one would be swept from memory by garbage collector.

* `y = x`, in memory, you would have a new name, but not necessarily a new object:

* `y += 1` After the addition call, you are returned with a new Python object. Now, the memory looks like this:


We have some conclusions:
* in the provided code snippet, `x` and `y` are indeed names or identifiers that can be used to refer to objects in memory. 

* It’s important to note that, assignment statements in python bind the names to the objects in memory, without copying the values themselves.

### 2. Immutable and mutable object in python.
In python, The immutable objects are objects whose value can not be modified after they created, the only way to change the names bound to an immutable object is to bind it to other object. On the other hand, mutable object can be modified after their creation.

In python, there are several immutable objects, such as `int`, `float`, `str`, `bytes`, `tuple`. On the other hand, mutable objects in Python, such as `list` and `dict`.

We make a clearance of immutable objects and mutable object with the following example:
```python
x = "hello world"
x[0] = H

"""
result:

ERROR!
Traceback (most recent call last):
  File "<string>", line 15, in <module>
TypeError: 'str' object does not support item assignment
"""
```

```python
x = [1, 2, 3]
print('the address of x: {}'.format(id(x)))

x = x.append(4)
print('the address of x after modifying: {}'.format(id(x)))

"""
result:

the address of x: 140343201072192
the address of x after modifying: 140343201072192
"""
```

```python
x = [1, 2, 3]
print('the address of x: {}'.format(id(x)))

x.extend([4])
print('the address of x after extending [4]: {}'.format(id(x)))

x += [5]
print('the address of x after extending [5]: {}'.format(id(x)))

x = x + [6]
print('the address of x after extending [6]: {}'.format(id(x)))


"""
the address of x: 139937477218944
the address of x after extending [4]: 139937477218944
the address of x after extending [5]: 139937477218944
the address of x after extending [6]: 139937477234432
"""

"""
in the above result, after executing `x += [5]` line, the a address of x stays
the same, because in behind the scene, the operator `+=` will be implemented 
virtually like:
	class List:
			def __iadd__(self, other)"
					self.extend(other)
					return self

On the other hand, after executing `x = x + [6]` line, the a address of x has
been changed. It's because the name `x` binds to the object that hold the value
[1, 2, 3, 4, 5] + [6] instead of binding to the previous one.
"""
```

The above code snippets are self-explanatory, so there is no need to add unnecessary explanation here.


### 3. Python uses calling by value or reference.
Now, we have sufficient ingredients to answer the question that appear in the title of this article. To be more understandable, we’re going to walk through the following examples.
    
```python
n = [4]
def power(l):
        print("local variables: {}".format(locals()))
        l[0] *= l[0]
        return l

power(n)
print("n: {}".format(n))

"""
result:

local variables: {'l': [4]}
n: [16]
"""

"""
- When program reaches the line `power(n)` function, the stack frame of `power` 
function will create a namespace that is a dictionary with key is names and values 
are the corresponding objects that are bound to those names. In the above result, 
the first local name is `l`, `l` is bound to the object that holds [4].
Consequently, after completely executing power() function, the value of object that 
n is bound is modified accordingly.
"""
```

```python
n = [4]
def power(l):
		print("local variables: {}".format(locals()))
		l = [5]
		print("local variables after reassignment: {}".format(locals()))
		l[0] *= l[0]
		return l

power(n)
print("n: {}".format(n))

"""
result:

local variables: {'l': [4]}
local variables after reassignment: {'l': [5]}
n: [4]
"""

"""
The result of the example is not the same as the previous one, Why's that?
It's because, although `l` is bound to object of [4] initially, but it was
reassigned to [5] later on, it means that `l` was rebound to a new object
that holds [5] as it's value. Consequently, the object `x` was bound to was
not modified.
"""
```

## References
- https://realpython.com/pointers-in-python/
- https://realpython.com/python-memory-management/
- https://realpython.com/python-pass-by-reference/
- https://nedbatchelder.com/text/names1.html