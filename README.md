# pycstruct

[![AppVeyor](https://ci.appveyor.com/api/projects/status/github/midstar/pycstruct?svg=true)](https://ci.appveyor.com/api/projects/status/github/midstar/pycstruct)
[![Coverage Status](https://coveralls.io/repos/github/midstar/pycstruct/badge.svg?branch=HEAD)](https://coveralls.io/github/midstar/pycstruct?branch=HEAD)
[![Documentation](https://readthedocs.org/projects/pycstruct/badge/?version=latest)](https://pycstruct.readthedocs.io/en/latest/?badge=latest)

pycstruct is a python library for converting binary data to and from ordinary
python dictionaries.

Data is defined similar to what is done in C language structs.

Typical usage of this library is read/write binary files or binary data
transmitted over a network.

It supports all traditional data types (integer, unsigned integer, boolean and
float) between 1 to 8 bytes large, arrays (lists), strings (UTF-8), bitfields
and enums.

Structs can be embedded inside other structs.

Individual elements can be stored / read in any byte order.

Checkout the full documentation [here](https://pycstruct.readthedocs.io/en/latest/).

# Example

Following C has a structure (person) with a set of elements
that are written to a binary file.

```c
#include <stdio.h>
#include <stdbool.h>
#include <string.h>

#pragma pack(1) // To secure no padding is added in struct

struct person 
{ 
    char name[50];
    unsigned int age;
    float height;
    bool is_male;
    unsigned int nbr_of_children;
    unsigned int child_ages[10];
};


void main(void) {
    struct person p;
    memset(&p, 0, sizeof(struct person));

    strcpy(p.name, "Foo Bar");
    p.age = 42;
    p.height = 1.75; // m
    p.is_male = true;
    p.nbr_of_children = 2;
    p.child_ages[0] = 7;
    p.child_ages[1] = 9;

    FILE *f = fopen("simple_example.dat", "w");
    fwrite(&p, sizeof(struct person), 1, f);
    fclose(f);
}
```

To read the binary file using pycstruct following code 
required.

```python
import pycstruct

person = pycstruct.StructDef()
person.add('utf-8', 'name', length=50)
person.add('uint32', 'age')
person.add('float32','height')
person.add('bool8', 'is_male')
person.add('uint32', 'nbr_of_children')
person.add('uint32', 'child_ages', length=10)

f = open('simple_example.dat','rb')
inbytes = f.read()
result = person.deserialize(inbytes)
f.close()

print(str(result))
```

The produced output will be::

    {'name': 'Foo Bar', 'is_male': True, 'nbr_of_children': 2, 
     'age': 42, 'child_ages': [7, 9, 0, 0, 0, 0, 0, 0, 0, 0], 
     'height': 1.75}

To write a binary file from python using the same structure
using pycstruct following code is required.

```python
import pycstruct

person = pycstruct.StructDef()
person.add('utf-8', 'name', length=50)
person.add('uint32', 'age')
person.add('float32','height')
person.add('bool8', 'is_male')
person.add('uint32', 'nbr_of_children')
person.add('uint32', 'child_ages', length=10)

mrGreen = {}
mrGreen['name'] = "MR Green"
mrGreen['age'] = 50
mrGreen['height'] = 1.93
mrGreen['is_male'] = True
mrGreen['nbr_of_children'] = 3
mrGreen['child_ages'] = [13,24,12]

buffer = person.serialize(mrGreen)

f = open('simple_example_mr_green.dat','wb')
f.write(buffer)
f.close()
```
