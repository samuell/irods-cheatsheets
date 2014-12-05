# iRODS Rule Language Cheat Sheet

## Basics

### Comments

````php
# <- Comments are written by starting a line with the hash character
## <- Avoid using double hashes (used for other purpose in the one line rule syntax)
````

## Directives

````php
# The following line will include the file definitions.re
@include "definitions" 
````

## Boolean operators

````php
true  # True
false # False
!     # Not
&&    # And
||    # Or
%%    # Or used in the "##" syntax
````

## Numbers

### Numeric literals
````php
1          # integer
1.0        # double
floor(1.5) # Will return integer 1
ceiling(1.5) # Will return integer 2
````
Note: Integer division as found in C, C++, and Java is not implemented. Instead, division between integers is the same as division between doubles.

### Arithmetic operators
````php
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
````php
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
````php
'This is a string.'
"This is also a string"
"Strings are quoted with \"\", or '' characters"
"Some escape characters: \n, \r, \t, \\, \', \", \$, \*"
````

### String operators
````php
++ # String concatenation
````

### String functions

Some examples of string functions:
````php
writeLine("stdout", "This "++" is "++" a string.");
# Output: This is a string.
````

Infix wildcard expression matching operator ```like```
````php
writeLine("stdout", "This is a string." like "This is*");
# Output: true
````

Infix regular expression matching operator ```like regex```
````php
writeLine("stdout", "This is a string." like regex "This.*string[.]");
# Output: true
````

Substring function ```substr()```
````php
writeLine("stdout", substr("This is a string.", 0, 4));
# Output: This
````

Length function ```strlen()```
````php
writeLine("stdout", strlen("This is a string."));
# Output: 17
````

Split function ```split()```
````php
writeLine("stdout", split("This is a string.", " "));
# Output: [This,is,a,string.]
````

Trim left function ```triml(\*str, \*del)```, which trims from ```*str``` the leftmost ```*del```.
````php
writeLine("stdout", triml("This is a string.", " "));
# Output: is a string.
````

Trim right function ```trimr(*str, *del)```, which trims from ```*str``` the rightmost ```*del```.
````php
writeLine("stdout", trimr("This is a string.", " "));
# Output: This is a
````

## Type conversions

### To string

### To other types
