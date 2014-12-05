# iRODS Rule Language Cheat Sheet

## Basics

### Comments

````
# <- Comments are written by starting a line with the hash character
## <- Avoid using double hashes (used for other purpose in the one line rule syntax)
````

## Directives

````
# The following line will include the file definitions.re
@include "definitions" 
````

## Boolean operators

````
true  # True
false # False
!     # Not
&&    # And
||    # Or
%%    # Or used in the "##" syntax
````

## Numbers

### Numeric literals
````
1          # integer
1.0        # double
floor(1.5) # Will return integer 1
ceiling(1.5) # Will return integer 2
````
Note: Integer division as found in C, C++, and Java is not implemented. Instead, division between integers is the same as division between doubles.

### Arithmetic operators
````
-  # Negation
^  # Power
*  # Multiplication 
/  # Division 
%  # Modulo
-  # Subtraction 
+  # Addition
>  # Greater than
<  # Less than
>= # Greater than or equal
<= # Less than or equal
````

### Arithmetic functions
````
 exp(<num>)
 log(<num>)
 abs(<num>)
 floor(<num>)
 ceiling(<num>)
 average(<num>,<num>,...)
 max(<num>,<num>,...)
 min(<num>,<num>,...)
 ````

## Strings

...
