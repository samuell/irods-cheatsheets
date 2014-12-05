# iRODS Rule Language Cheat Sheet

## Basics

### Comments

````c
# <- Comments are written by starting a line with the hash character
## <- Avoid using double hashes (used for other purpose in the one line rule syntax)
````

## Directives

````c
# The following line will include the file definitions.re
@include "definitions" 
````

## Boolean operators

````c
true  # True
false # False
!     # Not
&&    # And
||    # Or
%%    # Or used in the "##" syntax
````

## Numbers

### Numeric literals
````c
1          # integer
1.0        # double
floor(1.5) # Will return integer 1
ceiling(1.5) # Will return integer 2
````
Note: Integer division as found in C, C++, and Java is not implemented. Instead, division between integers is the same as division between doubles.

### Arithmetic operators
````c
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
````c
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

### Basic string syntax
````c
'This is a string.'
"This is also a string"
"Strings are quoted with \"\", or '' characters"
"Some escape characters: \n, \r, \t, \\, \', \", \$, \*"
````

### String operators
````c
++ # String concatenation
````

### String functions

Some examples of string functions:
````c
writeLine("stdout", "This "++" is "++" a string.");
# Output: This is a string.
````

Infix wildcard expression matching operator ```like```
````c
writeLine("stdout", "This is a string." like "This is*");
# Output: true
````

Infix regular expression matching operator ```like regex```
````c
writeLine("stdout", "This is a string." like regex "This.*string[.]");
# Output: true
````

Substring function ```substr()```
````c
writeLine("stdout", substr("This is a string.", 0, 4));
# Output: This
````

Length function ```strlen()```
````c
writeLine("stdout", strlen("This is a string."));
# Output: 17
````

Split function ```split()```
````c
writeLine("stdout", split("This is a string.", " "));
# Output: [This,is,a,string.]
````

Trim left function ```triml(\*str, \*del)```, which trims from ```*str``` the leftmost ```*del```.
````c
writeLine("stdout", triml("This is a string.", " "));
# Output: is a string.
````

Trim right function ```trimr(*str, *del)```, which trims from ```*str``` the rightmost ```*del```.
````c
writeLine("stdout", trimr("This is a string.", " "));
# Output: This is a
````

## Type conversions

### To string

### To other types
