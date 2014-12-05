## iRODS Rule Language

The iRODS Rule Language is a language provided by iRODS to define policies and
actions in the system. The iRODS Rule Language is tightly integrated with
other components of iRODS. Many frequently used policies and actions can be
configured easily by writing simple rules, yet the language is flexible enough
to allow complex policies or actions to be defined.

Everything is a rule in the iRODS Rule Language. A typical rule looks like:
````php
acPostProcForPut { 
  on($objPath like "*.txt") { 
    msiDataObjCopy($objPath,"$objPath.copy"); 
  } 
}
````
In this rule, the rule name ```acPostProcForPut``` is an
event hook defined in iRODS. iRODS automatically applies this rule when
certain events are triggered. The ```on(...)``` clause is a rule condition. The
```{...}``` block following the rule condition is a sequence of actions that is
executed if the rule condition is true when the rule is applied. And the
customary hello world rule looks like: 
````php
HelloWorld { 
  writeLine("stdout", "Hello, world!"); 
}
````
In the 3.0 release of iRODS, a new rule engine is included that comes with an
array of new features and improvements in robustness, error reporting, etc.,
and takes care of some corner cases where it was ambiguous, such as special
characters in strings, by following the conventions of mainstream programming
languages.

In the following sections, we go over some features of the new rule engine,
with a focus on changes and improvements in the new rule engine.

## Comments

The new rule engine parses characters between the ```#``` token and the end of
line as comments. Therefore, a comment does not have to occupy its own line.
For example,

````php
* A=1; # comments 
````

Although the parser is able to parse comments starting with ```##```, it is not recommended to start comments with ```##```, as ```##``` is also used in the one line rule syntax as the actions connector. It is recommended to start comments with ```#```.

## Directives

Directives are used to provide the rule engine with compile time intructions.
The ```@include``` directive allows including a different rule base file into the
current rule base file, similar to ```#include``` in C. For example, if we have a
rule base file "definitions.re", then we can include it with the following
directive ```@include "definitions"```

## Boolean and Numeric

### Boolean Literals

Boolean literals include ```true``` and ```false```.

### Boolean Operators

Boolean operators include 
````php
!  # not
&&`# and
|| # or
%% # or used in the ## syntax
````
For example:

````php
true && true
false && true
true || false
false || false
true %% false
false %% false
! true
````

### Numeric Literals

Numeric literals include integral literals and double literals. An integral
literal does not have a decimal while a double literal does. For example, 

````php
1 # integer 
1.0 # double
````

In the iRODS Rule Language, an integer can be converted to a double. The
reverse is not always true. A double can be converted to an integer only if
the fractional part is zero. The new rule engine, however, provides two
functions that can be used to truncate the fractional part of a double:
```floor()``` and ```ceiling()```.

Integers and doubles can be converted to booleans using the ```bool()``` function.
```bool()``` converts ```1``` to ```true``` and ```0``` to ```false```.

### Arithmetic Operators

Arithmetic operators include, ordered by precedence:

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

### Arithmetic Functions

Arithmetic functions include:

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

For example:
````php
exp(10)
log(10)
abs(-10)
floor(1.2)
ceiling(1.2)
average(1,2,3)
max(1,2,3)
min(1,2,3)
````

## Strings

### String Literals

One of the main changes to the rule language is quotes and escapes in strings.
The new rule engine requires by default that every string literal is quoted.
The quotes can be either matching single quotes ```'This is a string.'``` or double
quotes 

````php
"This is a string."`
``` 

If a programmer needs to quote strings containing single (double) quotes using single (double) quotes, then the quotes in the
strings should be escaped using a backslash ```"\"```, just as in the C Programming
Language. For example, 

````php
writeLine("stdout", "\"\"");
# output "" 
````

Single quotes inside double quotes are viewed as regular characters, and vice versa. They can be either escaped or not escaped. For example, 

````php
writeLine("stdout", "'");
# output ' 
````

````php
writeLine("stdout", "\'");
# output ' 
````

The rule engine also supports various escaped characters: 
````php
\n, \r, \t, \', \", \$, \*
````

An asterisk should always be escaped if it is a regular
character and is followed by letters.

### Converting Values of Other Types to Strings

The ```str()``` function converts a value of type BOOLEAN, INTEGER, DOUBLE,
DATETIME, or STRING to string. For example

````php
writeLine("stdout", str(123));
# output 123
````

In addition

````php
timestrf(*time, *format)
````

converts a datetime stored in ```*time``` to a string, according to the ```*format```
parameter.

````php
timestr(*time)
````

converts a datetime stored in ```*time``` to a string, according to the default
format.

The default format is

````php
%b %d %Y %H:%M:%S
````

The format string uses the same directives as the standard C library.

### Converting Strings to Values of Other Types

String can be converted to values of type BOOLEAN, INTEGER, DOUBLE, DATETIME,
or STRING. For example, 

````php
int("123")
double("123")
bool("true")
````

In addition

````php
datetimef(*str, *format)
````

converts a string stored in ```*str``` to a datetime, according to the ```*format```
parameter.

````php
datetime(*str)
````

converts a string stored in ```*str``` to a datetime, according to the default
format. It can also be used to convert an integer or a double to a datetime.

The following are examples of string datetime conversion

````php
datetime(*str)
datetimef(*str, "%Y %m %d %H:%M:%S")
timestr(*time)
timestrf(*time, "%Y %m %d %H:%M:%S")
````

### String Functions

The new rule engine supports the infix string concatenation operator ```"++"```

````php
writeLine("stdout", "This "++" is "++" a string.");
# output This is a string.
````

**Infix wildcard expression matching operator**: ```like```
````php
writeLine("stdout", "This is a string." like "This is*");
# Output: true
````

**Infix regular expression matching operator**: ```like regex```
````php
writeLine("stdout", "This is a string." like regex "This.*string[.]");
# Output: true
````

**Substring**: ```substr()```
````php
writeLine("stdout", substr("This is a string.", 0, 4));
# Output: This
````

**Length**: ```strlen()```
````php
writeLine("stdout", strlen("This is a string."));
# Output: 17
````

**Split**: ```split()```
````php
writeLine("stdout", split("This is a string.", " "));
# Output: [This,is,a,string.]
````

**Trim left**: ```triml(*str, *del)```, which trims from ```*str``` the leftmost ```*del```.
````php
writeLine("stdout", triml("This is a string.", " "));
# Output: is a string.
````

**Trim right**: ```trimr(*str, *del)```, which trims from ```*str``` the rightmost ```*del```.
````php
writeLine("stdout", trimr("This is a string.", " "));
# Output: This is a
````

### Variable Expansion

In a quoted string, an asterisk followed immediately by a variable name
(without whitespaces) makes an expansion of the variable. For example, 

````php
"This is *x."
````
is equivalent to 
````php
"This is "++str(*x)++"."
````

### Rules for Quoting Action Arguments

A parameter to a microservice is of type string if the expected type is
```MS_STR_T``` or ```STRING```. When a microservice expects a parameter of type string and
the argument is a string constant, the argument has to be quoted. For example,

````php
writeLine("stdout", "This is a string.");
````

When a microservice expects a parameter of type string and the argument is not of type string, a type error
may be thrown. For example,

````php
*x = 123; 
strlen(*x); 
````

This error can be fixed by either using the "str" function 
````php
strlen(str(*x));
````
or putting ```*x``` into quotes 
````
strlen("*x");
````
Action names and keywords such as ```for```, ```while```, ```assign``` are not arguments. Therefore, they do not have to be quoted.

### Wildcard and Regular Expression

The new rule engine supports both the wildcard matching operator ```like``` and a
new regular expression matching operator ```like regex```. (It is an operator, not
two separate keywords.) Just as the old rule engine does, the new rule engine
supports the ```*``` wildcard. For example, 

````php
"abcd" like "ab*"
````

In case of ambiguity with variable expansion, the ```*``` has to be escaped. For example
````php
"abcd" like "a\\*d"
````
because ```"a*d"``` is interpreted as ```"a"++str(*d)+""```

When wildcard is not expressive enough, regular expression matching operator
can be used. For example

````php
"abcd" like regex "a.c."
````

A regular expression matches the whole string. It follows the syntax of the
POSIX API.

### Quoting Code

Sometime when you want to pass a string representation of code or regular
expressions into an action, it is very tedious to escape every special
character in the string. For example

````php
writeLine(\"stdout\", \*A)
````

or

````php
 *A like regex "a\*c\\\\\\[\\]" # matches the regular expression a*c\\\[\]
````

In this case you can use "``" instead of the regular quotes. The rule
engine does not further look for variables, etc. in strings between two "``"s.
With "``", the examples can be written as: 

````php
``writeLine("stdout", *A)``
````
and
````php
*A like regex ``a*c\\\[\]``
````


## Dot Expression

**This feature is added after the 3.2 release**

The dot oeprator provides a simple syntax for creating and accessing key
values pair.

To write to a key value pair, use the dot operator on the left hand side:

````php
*A.key = "val"
````

If the key is not a syntactically valid identifier, quotes can be used, escape
rules for strings also apply:

````php
*A."not an identifier" = "val"
````

If the variable ```*A``` is undefined, a new key value pair data structure will be
created.

To read from a key value pair, use the dot operator as binary infix operation
in any expression.

Currently key value pairs only support the string type for values.

The str() function is extended to support converting a key value pair data
structure to an options format:

````php
*A.a=A;
*A.b=B;
*A.c=C; 
str(*A); # a=A++++b=B++++c=C
````

## Constant

**This feature is added after the 3.2 release**

A constant can be defined as a function that returns a constant. A constant
defintion has the following syntax:
````php
<constant name> = <constant value>
````
where the constant value can be on of the following:

  * an integer
  * a double
  * a string (with no variable expansion in it)
  * a boolean

A constant name can be used in a pattern and is replaced by its value (whereas
a nonconstant is treated as a constructor). For example,

With
````php
CONSTANT = 1
````

the following expression
````php
match CONSTANT with
   CONSTANT => "CONSTANT"
   *_ => "NOT CONSTANT"
````

returns ```CONSTANT```. With a nonconstant function definition such as
````php
CONSTANT = time()
````

it returns ```NOT CONSTANT```.

## Variable

The new rule engine expands variables on demand, like in C, instead of
expanding variables before a rule is executed. For example, suppose we have
the following rule 

````php
ifExec(*A==1,assign(*A,0),assign(*A,1),nop,nop) 
````
Expanding ```*A``` (say, to 1) before the rule is executed would result in something like
````php
ifExec(1==1,assign(1,0),assign(1,1),nop,nop) 
````
As a result, extra code has to be written for system microservices to avoid this. With the new rule engine, the value of ```*A`` is retrieved from a runtime environment only when the rule engine tries to evaluate it.

## Function

### Function Definition

The new rule engine allows defining functions. Functions can be thought of as
microservices written in the rule language. The syntax of a function
definition is 

````php
<name>(<param>, ..., <param>) = <expr> 
````
For example
````php
square(*n) = *n * *n 
````
Function names should be unique (no function-function or function-rule
name conflict). Functions can be defined in a mutually exclusive manner. For
example
````php
odd(*n) = if *n==0 then false else 
even(*n-1) even(*n) = if *n==1 then true else odd(*n-1) 
````
Here we cannot use ```&&``` or ```||``` because they do not short
circuit like in C or Java.

### Calling a Function

To use a function, call it as if it was a microservice.

## Rule

### Rule Definition

The syntax of a rule with a nontrivial rule condition is as follows:
````php
<name>(<param>, ..., <param>) { 
  on(<expr>) { <actions> } 
}
````

If the rule condition is trivial or unnecessary, the rule can be written in
the simpler form: 
````php
<name>(<param>, ..., <param>) { <actions> }
````
Multiple rules with the same rule name and parameters list can be combined in
a more concise syntax where each set of actions is enumerated for each set of
conditions: 
````php
<name>(<param>, ..., <param>) { 
  on(<expr>) { <actions> } ...
  on(<expr>) { <actions> } 
}
````

### Function Name and Rule Name

Function and rule names have to be valid identifiers. Identifiers start with
letters followed by letters or digits. For example,
````php
ThisIsAValidFunctionNameOrRuleName
````
There should not be whitespaces in a
function name or a rule name. For example
````php
This Is Not A Valid Function Name or Rule Name
````

### Rule Condition

In the new rule engine, rule conditions should be expressions of type ```boolean```. The rule is executed only when the rule condition evaluates to true. Which means that there are three failure conditions:

  1. Rule condition evaluates to false.
  2. Some actions in rule condition fails which causes the evaluation of the whole rule condition to fail.
  3. Rule condition evaluates to a value whose type is not boolean.

For example, if we want to run a rule when the microservice "msi" succeeds, we
can write the rule as

````php
rule { 
  on (msi >= 0) { ... } 
} 
````
Conversely, if we want to run a rule when the
microservice fails, we need to write the rule as 
````php
rule { 
  on (errorcode(msi) < 0) { ... } 
}
````
The errormsg microservice captures the error message, allows
further processing of the error message, and avoiding the default logging of
the error message
````php
rule { 
  on (errormsg(msi, *msg) < 0 ) { ... } 
} 
````
By failure condition 3, the following rule condition always fails because msi returns an integer value 
````php
on(msi) { ... }
````

### Generating and Capturing Errors

In a rule, we can also prevent the rule from failing when a microservice fails
````php
errorcode(msi)
````
The errormsg microservice captures the error message, allows
further processing of the error message, and avoiding the default logging of
the error message
````php
errormsg(msi, *msg)
````
In a rule, the fail and failmsg microservices can be used to generate errors 
````php
fail(*errorcode)
````
generates an error with an error code
````php
failmsg(*errorcode, *errormsg)
````
generates an error with an error code and an error message. For example
````php
fail(-1)
failmsg(-1, "this is a user generated error message")
````

The msiExit microservice is similar to failmsg
````php
msiExit("-1", "msi")
````

### Calling a Rule

To use a rule, call it as if it was a microservice.

## Data Types and Pattern Matching

### System Data Types

#### Lists

The new rule engine provides built-in support for lists. A list can be created
using the ```list()``` microservice. For example
````php
list("This","is","a","list")
````
The elements of a list should have the same type. Elements of a list can be
retrieved using the "elem" microservice. The index starts from 0. For example,
````php
elem(list("This","is","a","list"),1)
````
evaluates to "is". 

If the index is out of range it fails with error code -1. 

The ```setelem()``` takes in three parameters, a list, an index, and a value, 
and returns a new list that is identical with the list given by the first 
parameter except that the element at the index given by the second parameter 
is replace by the value given by the third parameter.
````php
setelem(list("This","is","a","list"),1,"isn't")
````
evaluates to
````php
list("This","isn't","a","list").
````
If the index is out of range it fails with an error code. The ```size``` 
microservice takes in on parameter, a list, and returns the size of the list. 

For example
````php
size(list("This","is","a","list"))
````
evaluates to ```4```. 

The ```hd()``` microservice returns the first element of a list and
the ```tl()``` microservice returns the rest of the list. 

If the list is empty then it fails with an error code. 

````php
hd(list("This","is","a","list"))
````
evaluates to "This" and 
````php
tl(list("This","is","a","list"))
````
evaluates to 
````php
list("is","a","list")
````
The ```cons()``` microservice returns a list by combining an
element with another list. For example
````php
cons("This",list("is","a","list"))
````
evaluates to
````php
list("This","is","a","list").
````

#### Tuples

The new rule engine supports built-in data type tuple. 
````php
( <component>, ..., <component> ) 
````
Different components may have different types.

#### Interactions with Packing Instructions

Complex lists such as lists of lists can be constructed locally, but mapping
from complex list structures to packing instructions are not yet supported.
The supported lists types that can be packed are integer lists and string
lists. When remote execute or delay execution is called while there is a
complex list in the current runtime environment, an error will be generated.
For example, in the following rule test {

````php
test {
   *A = list(list(1,2),list(3,4));
   *B = elem(*A, 1);
   delay("<PLUSET>1m</PLUSET>") {
       writeLine("stdout", *B);
   }
}
````

Even though ```*A``` is not used in the delay execution block, the rule will still generate an error. One solution to this is to create a rule with only necessary values. 

````php
test {
   *A = list(list(1,2),list(3,4));
   *B = elem(*A, 1);
   test(*B);
}
test(*B) {
   delay("<PLUSET>1m</PLUSET>") {
       writeLine("stdout", *B);
   }
}
````

### Inductive Data Type

The features discussed in this section are currently '''under development'''.

The new rule engine allows defining inductive data types. An inductive data
type is a data type for values that can be defined inductively, i.e. more
complex values can be constructed from simpler values using constructors. The
general syntax for inductive data type definition is

````php
data <name> [ ( <type parameter list> ) ] =
    |  : <data constructor type>
    ...
    | <data constructor name> : <data constructor type>
````

For example, a data type that represents the natural numbers can be defined as

````php
data nat =
    | zero : nat
    | succ : nat -> nat
````
  
Here the type name defined is “nat.” The type parameter list is empty. If the type parameter list is empty, we may omit it. There are two data constructors. The first constructor “zero” has type “nat,” which means that “zero” is a nullary constructor of nat. We use “zero” to represent “0”. The second constructor “succ” has type “nat -> nat” which means that “succ” is unary constructor of nat. We use “succ” to represent the successor. With these two constructors we can represent all natural numbers: ```zero, succ(zero), succ(succ(zero)), ...``` As another example, we can define a data type that represents binary trees of natural numbers

````php
  data tree =
      | empty : tree
      | node : nat * tree * tree -> tree
````
  
The ```empty``` constructor constructs an empty tree, and the ```node``` constructor
takes in a ```nat``` value, a left subtree, and a right subtree and constructs a
tree whose root node value is the "nat" value. The next example shows how to
define a polymorphic data type. Suppose that we want to generalize our binary
tree data type to those trees whose value type is not "nat." We give the type
tree a type parameter X

````php
data tree(X) =
    | empty : tree(X)
    | node : X * tree(X) * tree(X) -> tree(X)
````
  
With a type parameter, ```tree``` is not a type, but a unary type constructor. A
type constructor constructs types from other types. For example, the data type
of binary trees of natural numbers is "tree(nat)." By default, the rule engine
parses all types with that starts with uppercase letter as type variables.

Just as data constructors, type constructor can also take multiple parameters.
For example

````php
data pair(X, Y) =
    | pair : X * Y -> pair(X, Y)
````
  
Given the data type definition of "pair", we can construct a pair using "pair"
the data constructor. For example,

````php
*A = pair(1, 2);
````

### Pattern Matching

#### Pattern Matching In Assignment

Patterns are similar to expressions. For example
````php
pair(*X, *Y)
````
There are a few restrictions. First, only data constructors and free variables may appear in
patterns. Second, each variable only occurs once in a pattern (sometimes
called linearity). To retrieve the components of ```*A```, we can use patterns on
the left hand side of an assignment. For example
````php
pair(*X, *Y) = *A;
````
When this action is executed, ```*X``` will be assigned to 1 and ```*Y``` will be assigned to ```2```.
Patterns can be combined with let expressions. For example
````php
fib(*n) = if *n==0 then pair(-1, 0)
    else if *n==1 then (0, 1)
    else let pair(*a, *b) = fib(*n - 1) in
        pair(*b, *a + *b)
````

#### Pseudo Data Constructors

For other types, the new rule engine allows the programmer to define pseudo
data constructors on them for pattern matching purposes. Pseudo data
constructors are like data constructors but can only be used in patterns.
Pseudo data constructor definitions are like function definitions except that
a pseudo data constructor definition starts with a tilde and must return a
tuple. The general syntax is 
````php
<name>(<param>, ..., <param>) = <expr>
````
A pseudo data constructor can be thought of an inverse function that maps the values in
the codomain to values in the domain. For example, we can define the following
pseudo data constructor
````php
lowerdigits(*n) = let *a = *n % 10 in ((*n - *a) / 10 % 10, *a)
````
The assignment
````php
lowerdigits(*a, *b) = 256;
````
results in ```*a``` assigned ```5``` and ```*b``` assigned ```6```.

## Control Structures

### Actions

The iRODS rule engine has a unique concept of recovery action. Every action in
a rule definition may have a recovery action. The recovery action is executed
when the action fails. This allows iRODS rules to rollback some side effects
and restore most of the system state to a previous point. The new rule engine
supports a more general notion of an action recovery block. An action recovery
block has the form 

````php
  {
     A1 ::: R1
     A2 ::: R2
     ...
     An ::: Rn
  }
````

The basic semantics is that if ```Ax``` fails then ```Rx, R(x-1), ... R1```` will be executed. The programmer
can use this mechanism to restore the system state to the point before this action recovery block is executed.

The new rule engine make the distinction between expressions and actions. An
expression does not have a recovery action. An action always has a recovery
action. If a recovery action is not specified for an action, the rule engine
use "nop" as the default recovery action.

Examples of expressions include the rule condition, and the conditional
expressions in the ```if```, ```while```, and ```for``` actions.

There is no intrinsic difference between an action and an expression. An
expression becomes an action when it occurs at an action position in an action
recovery block. An action recovery block, in turn, is an expression.

The principle is that an expression should only be used not as an action if it
is side-effect free. In the current version, this property is not checked by
the rule engine. The programmer has to make sure that it holds for the rule
base.

From this perspective, the only difference between functions and rules is
nondeterminism.

### if

The new rule engine has a few useful extensions to the "if" keyword that makes
programming in the rule language more convenient.

In addition to the traditional way of using "if" in the rule language, which
will be referred to as the "logical if", where you use if as an action which
either succeeds or fails with an error code, the new rule engine supports
another way of using if, which will be referred to as the "functional if". The
"functional if" may return a value of any type if it succeeds. The two
different usages have different syntax. The "logical if" has the same syntax
as before
````php
if <expr> then { <actions> } else { <actions> }
````

while the "functional if" has the following syntax
````php
if <expr> then <expr> else <expr>
````

For example, the following are "functional if"s
````php
if true then 1 else 0 
if *A==1 then true else false 
````
To compare, if written in the "logical if" form, the
second example would be
````php
if (*A==1) then { true; } else { false; }
````

To make the syntax of "logical if" more concise, the new rule engine allows
the following abbreviation (where the greyed out part can be abbreviated): 
````php
if (...) then { ... } else { ... } 
if (...) then { ... } else { if (...) then {...} else {...} }
````
Multiple abbreviations can be combined for
example:
````php
if (*X==1) { *A = "Mon"; } 
else if (*X==2) {*A = "Tue"; } 
else if (*X==3) {*A = "Wed"; } 
...
````

### foreach

The new rule engine allows defining a different variable name for the iterator
variables in the foreach action. For example
````php
foreach(*E in *C) {
  writeLine("stdout", *E); 
} 
````
This is equivalent to
````php
foreach(*C) {
  writeLine("stdout", *C); 
}
````

This new feature allows the collection to be a complex expression. For
example

````php
foreach(*E in list("This", "is", "a", "list")) { 
  writeLine("stdout", *E); 
} 
````
This is equivalent to

````php
*C = list("This", "is", "a", "list");
foreach(*C) {
   writeLine("stdout", *C);
}
````

### The ```let``` Expression

As function definitions are based on expressions rather than action sequences,
we cannot put an assignment directly inside an expression. For example, the
following is not a valid function definition quad(*n) = *t = *n * *n; *t * *t
To solve this problem, the let expression provides scoped values in an
expression. The general syntax for the let expression is let <assignment> in
<expr> For example, quad(*n) = let *t = *n * *n in *t * *t The variable on the
left hand side of the assignment in the let expression is a let-bound
variable. The scope of such a variable is within the let expression. A let
bound variable should not be reassigned inside the let expression.

### The ```match``` Expression

If a data type has more than one data structure, then the "match" expression
is useful

````php
match <expr> with
   | <pattern> => <expr>
   ...
   | <pattern> => <expr>
````
  
For example, given the ```nat``` data type we defined earlier, we can define the
following function using the ```match``` expression
````php
add(*x, *y) =
   match *x with
       | zero => *y
       | succ(*z) => succ(add(*z, *y))
````

zero => *y  
---  

For another example, given the "tree" data type we defined earlier, we can
define the following function

````php
size(*t) =
   match *t with
       | empty => 0
       | node(*v, *l, *r) => 1 + size(*l) + size(*r)
````


## Recovery Chain For Control Structures

### Sequence

Actions:
````php
A1##A2##...##An
````

Recovery:
````php
R1##R2##...##Rn
````

Rulegen syntax: 
````php
  A1:::R1
  A2:::R2
  ...
  An:::Rn
````

If Ax fails, then ```Rx, ..., R1``` are executed

### Branch

Action: 
````php
if(cond, A11##A12##...##A1n, A21##A22##...##A2n,
         R11##R12##...##R1n, R21##R22##...##R2n)
````

Recovery:
````php
R
````

Rulegen syntax:
````php
if(cond) then {
    A11:::R11
    A12:::R12
    ...
    A1n:::R1n
} else {
    A21:::R21
    A22:::R22
    ...
    A2n:::R2n
}:::R
````

If Axy fails, then ```Rxy, ..., Rx1, R``` are executed. If ```cond``` fails, then ```R``` is
executed.

### Loop

#### ```while```

Action: 
````php
  while(cond, A1##A2##...##An
              R1##R2##...##Rn)
````
Recovery:
````php
R
````

Rulegen syntax: 
````php
  while(cond) {
      A1:::R1
      A2:::R2
      ...
      An:::Rn
  }:::R
````

If ```Ax``` fails, then ```Rx, ..., R1, R``` are executed. If ```cond``` fails, then ```R``` is
executed. Here ```R``` should deal with the loop invariant. The recovery chain in
the loop restores the loop invariant, and the ```R``` restores the machine status
from the loop invariant to before the loop is executed.

#### ```foreach```

Action: 
````php
foreach(coll, A1##A2##...##An 
              R1##R2##...##Rn) 
````

Recovery: 
````php
R
````

Rulegen syntax: 
````php
foreach(coll) {
    A1:::R1
    A2:::R2
    ...
    An:::Rn
}:::R
````

If ```Ax``` fails, then ```Rx, ..., R1, R``` are executed.

#### "for"

Action:
````php
for(init, cond, incr, A1##A2##...##An
                      R1##R2##...##Rn)
````
Recovery:
````php
R
````
Rulegen syntax:
````php
for(init; cond; incr) {
    A1:::R1
    A2:::R2
    ...
    An:::Rn
}:::R
````
If ```Ax``` fails, then ```Rx, ..., R1, R``` are executed. If ```init```, ```cond```, or ```incr``` fails, then ```R``` is executed.

## Types

### Introduction

Types are useful for capturing errors before rules are executed. At the same
time, a restrictive type system may also rule out meaningful expressions. As
the rule language is a highly dynamic language, the main goal of introducing a
type system is the following:

  * To enable discovering some errors statically without ruling out most valid rules written for the old rule engine.
  * To help remove some repetitive type checking and conversion code in microservices by viewing types as contracts of what kinds of values are passed between the rule engine and microservices.

The type system is designed so that the rule language is dynamically typed
when no type information is given, while providing certain static guarantees
when some type information is given. The key is combining static typing with
dynamic typing, so that we only need to check the statically typed part of a
program statically and leave the rest of the program to dynamic typing. The
idea is based on Coercion, Soft Typing, and Gradual Typing.

The types implemented in the old rule engine are very dynamic, which allows
values such as integer to be converted back and forth to strings. The new rule
engine tries to make this a little bit more rigorous. The central idea is
coercion insertion. The new rule engine has a fixed coercion relation among
its primitive types. Any coercion has to be based on this coercion relation.
The coercion relation does not have to be fixed but has to satisfy certain
properties. This way, we can potentially adjust the coercion relation without
breaking other parts of the type system.

The new rule engine distinguishes between two groups of microservices. System
provided microservices such as string operators are called internal micro
services. The rest are called external micro services. Most internal micro
services are statically typed. They come with type information which the type
check can make use of to check for errors statically. Currently, all external
micro services are dynamically typed.

The new rule engine supports a simple form of polymorphism, based on the
Hindley-Milner (HM) polymorphism. The idea of HM is that any function type can
be polymorphic, but all type variables are universally quantified at the top
level. As the rule language does not have higher-order functions, many
simplifications can be made. To support certain internal microservices, the
type system allows type variables to be bounded, but only to a set of base
types.

The type system implemented in the new rule engine is an extension to the
existing iRODS types by viewing the existing iRODS types as opaque types.

### Types

The function parameter and return types can be
````php
  <btype> ::= boolean
            | integer
            | double
            | string
            | time
            | path
````
````php
  <stype> ::= <tvar>                   identifiers starting with uppercase letters
            | iRODS types              back quoted string
            | <btype>
            | ?                        dynamic type
            | <stype> * … * <stype>    tuple types 
            | c[(<stype>, …, <stype>)] inductive data types
````
A function type is
````php
  <ftype> ::= <quanti>, …, <quanti>, <ptype> * <ptype> * … * <ptype> [*|+|?] -> <stype>
````
where 
* the ```<stype>``` on the right is the return type
* the optional ```*```, ```+```, or ```?``` indicates varargs
* the ```<ptype>``` are parameter types of the form
````php
  <ptype> ::= [(input|output)*|dynamic|actions|expression] [f] <stype>
````
where the optional
* ```input``` indicates that this is an io parameter
    - ```output``` indicates that this is an output parameter
    - ```dynamic``` indicates that this is an io parameter or output parameter determined dynamically
    - ```actions``` indicates that this is a sequence of actions
    - ```expression``` indicates that this is an expression
    - ```f``` indicates that a coercion can be inserted at compile time based on the coercion relation
* the ```<quanti>``` are quantifiers of the form
````php
  <quanti> ::= forall <tvar> [in {<btype> … <btype>}]
````
where ```<tvar>``` is a type variable, and the optional set of types provides a bound for the type variable.

### Typing Constraint

Type constraints are used in the new rule engine to encode typing requirements
that need to be checked at compile time or at runtime. The type constraints
are solved against a type coercion relation, a model of whether one type can
be coerced to another type and how their values should be converted. The type
coercion relation can be considered as a directed graph, with types as its
vertices, and type coercion functions as its edges.

The current coercion relation, denoted by ```->``` below, consists of the
following rules.

````php
SCIntegerDouble: integer -> double
SCDynamicLeft: dynamic -> y
SCDynamicRight: x -> dynamic
````

All coercion functions in the coercion relation are total.

### Types by Examples

For example binary arithmetic operators such as addition and subtraction are
given type: 
````php
forall X in {integer double}, f X * f X -> X
````
This indicates that the operator takes in two parameters of the same type and returns a value of
the same type as its parameters. The parameter type is bounded by {integer
double}, which means that the micro service applies to only integers or
doubles, but the "f" indicates that if anything can be coerced to these types,
they can also be accepted with a runtime conversion inserted. Examples:

(a) double + double => X = double 
````php
1.0+1.0 
````
(b) int + double => X = double 
````php
1+1.0
````
(c) integer + integer => X = {integer double} 
````php
1+1 
````
(d) unknown + double => X = double
Assuming that *A is a fresh variable
````php
*A+1.0
````
The type checker generate a constraint that the type of ```*A`` can be coerced to double.

(e) unknown + unknown => X = {integer double}

Assuming that *A and *B are fresh variables

  * A+*B The type checker generate a constraint that the type of *A can be coerced to either integer or double.

Some typing constraints can be solved within certain context. For example, if
we put (e) in to the following context
````php
*B = 1.0;
*B = *A + *B;
````
then we can eliminate the possibility that ```*B``` is an integer, thereby narrowing the type variable ```X``` to double.

Some typing constraints can be proved unsolvable. For example,
````php
*B = *A + *B;
*B == "";
````
by the second action we know that ```*B``` has to have type string. In this case the rule engine reports a type error.

However, if some typing constraints are not solvable, they are left to be
solved at runtime.

### Variable Typing

As in C, all variables in the rule language have a fixed type that can not be updated through an assignment.
For example, the following does not work:

````php
  testTyping1 {
      *A = 1;
      *A = "str";
  }
````

Once a variable ```*A``` is assigned a value ```X``` the type of the variable is given by a typing constraint

````php
type of X can be coerced to type of *A
````

For brevity, we sometimes denote the "can be coerced to" relation by "<=". For example,

````php
type of X <= type of *A
````

The reason why the type of *A is not directly assigned to the type of *X is to allow the following usage

````php
testTyping2 {
    *A = 1; # integer <= type of *A
    *A = 2.0; # double <= type of *A
}
````

Otherwise, the programmer would have to write

````php
testTyping3 {
    *A = 1.0;
    *A = 2.0;
}
````

to make the rule pass the type checker.

As a more complex example, the following generates a type error:

````php

testTyping4 {
    *A = 1; # integer <= type of *A
    if(*A == "str") { # type error
    }
}
````

If the value of a variable is dynamically typed, then a coercion is inserted. The following example works, with a runtime coercion:

````php
testTyping2 {
   msi(*A);
   if(*A == “str”) { # insert coercion type of *A <= string
   }
}
````

### Type Declaration

In the new rule engine, you can declare the type of a rule function or a microservice. If the type of an action is declared, then the rule engine will do more static type checking. For example, although

````php
concat(*a, *b) = *a ++ *b
add(*a, *b) = concat(*a, *b)
````

does not generate a static type error, ```add(0, 1)``` will generate a dynamic type error. This can be solved (generate static type errors instead of dynamic type errors) by declaring the types of the functions

````php
concat : string * string -> string
concat(*a, *b) = *a ++ *b
add : integer * integer -> integer
add(*a, *b) = concat(*a, *b)
````

## Microservices

### Automatic Evaluation of Arguments

The new rule engine automatically evaluates expressions within arguments of actions, which is useful when a program needs to pass the result of an expression in as an argument to an action. For example, in the old rule engine, if we want to pass the result of an expression “1+2” as an argument to microservice “msi”, then we need to either write something like:

````php
*A=1+2;
msi(*A);
````

or pass ```1+2``` in as a string to ```msi``` and write code in the microservice which parses and evaluates the expression. With the new rule engine, the programmer can write:

````php  
msi(1+2);
````

and the rule engine will evaluates the expression
````php
1+2
````
and pass the result in.

### The Return Value of User Defined Microservices

Both the old rule engine and the new rule engine view the return value of user defined microservices as an integer "errorcode." If the return value of a microservice is less than zero, both rule engines interpret it as a failure, rather than an integer value; and if the return value is greater than zero, both rule engines interpret it as an integer. Therefore the following expression

````php  
msi >= 0
````

either evaluates to true or fail, since when "msi" returns a negative integer, the rule engine interprets the value as a failure.
In some applications, there is need for capturing all possible return values as regular integers. The "errorcode" microservice provided by the new rule engine can be used to achieve this. In the previous example, we can modify the code to

````php
errorcode(msi) >= 0
````
This expression will not fail on negative return values from msi.

### The Administration Microservices

The rule engine has three groups of rule administration microservices. The changes made by the first group are system wide. They affect subsequent execution of the irule command and other parts of the iRODS system such as delayed execution. The changes made by second group only affects the current irule command. Subsequent execution of the irule command and other parts of the iRODS system will not be affected by these changes. The third group reads rules from or writes rules to files or the database.
The first group includes

````php
msiAdmAppendToTopOfCoreRE
msiAdmChangeCoreRE
msiAdmShowCoreRE
````

The second group includes

````php
msiAdmAddAppRuleStruct
msiAdmClearAppRuleStruct
msiAdmShowIRB
````

The third group includes
````php
msiAdmWriteRulesFromStructIntoFile
msiAdmReadRulesFromFileIntoStruct
msiAdmInsertRulesFromStructIntoDB
msiAdmRetrieveRulesFromDBIntoStruct
````

## Improvements to the Rule Engine

### Error Messages

The new rule engine generates error messages that provide information on
parser errors, type errors, and runtime errors. Most of the error messages
contain error locations in the source code, whether dynamically generated or
given in the core.re file or an irule input file.

There error messages can be found in the server log.

### Indexing

To improve the performance of rule execution, the new rule engine provide two level indexing on applicable rules. The first level of indexing is based on the rule name. The second level of indexing is based on rule conditions. The rule condition indexing can be demonstrate by the following example:
  
````php
  testRule(*A) {
      on (*A == "a") { ... }
      on (*A == "b") { ... }
  }
````

In this example, we have two rules with the same rule name, but different rule conditions. The first level of indexing does not improve the performance in a rule application like

````php
testRule("a")
````

However, the second level indexing does. The second level indexing works on rules with similar rule conditions. In particular, the rule conditions have to be of the form

````php
<expression> == <string>
````

All rules have to have the same number of parameters, but they may have different parameter names. The expression has to be the same for all rules modulo variable renaming, and the strings have to be different for different rules.

The rule engine indexes the rules by the string. When a rule is called, the rule engine evaluates the expression once and looks up the rule using the second level indexing.

## Improvements to irule

### ```.r``` and ```.ir``` Files

The new irule command directly accepts both .r and .ir files. The irule
command determines the format of the input file by the file extension.

### Multiple Rules in irule Input

The new rule engine supports multiple rules in the .r file when it is used as
input to the irule command. The first rule will be evaluated and all the rules
included in the .r will be available during the execution of the first rule
before the irule command returns. Only the first rule will be available for
delayed execution or remote execution.

## Backward Compatibility

### Backward Compatibility Modes

The new rule engine has backward compatible modes that allow it to run rules written for the old rule engine in the "##" syntax and a less strict grammar with little or no change. The backward compatible mode can be set to "true", "false", or "auto" using the @backwardCompatible directive. For example

````php
 @backwardCompatible "true"
 acPostProcForPut|$objPath like *.txt|writeLine(serverLog, text: $objPath)|nop
 @backwardCompatible "false"
 acPostProcForPut {
     on($objPath like "*.html") {
         writeLine("serverLog", "html: $objPath");
     }
 }
````

As shown in the example, backward compatible modes can be mixed within one code base. By default the backward compatible mode is set to ```auto```. In the ```auto``` mode, the rule engine tries to detect whether the code is written in the ```##``` syntax or the newer grammar and automatically apply backward compatibility to the ```##``` syntax.

For example, the example can be written as:

````php
 @backwardCompatible "auto"
 acPostProcForPut|$objPath like *.txt|writeLine(serverLog, text: $objPath)|nop
 acPostProcForPut {
     on($objPath like "*.html") {
         writeLine("serverLog", "html: $objPath");
     }
 }
````

or if you use the default setting,

````php
 acPostProcForPut|$objPath like *.txt|writeLine(serverLog, text: $objPath)|nop
 acPostProcForPut {
     on($objPath like "*.html") {
         writeLine("serverLog", "html: $objPath");
     }
 }
````````

### Backward Incompatibilities

There are a few exceptions in the backward compatibility modes.

#### Variables in like Expressions

In the old rule engine, the follow code matches ```$objPath``` with the pattern ```*txt```:

````php
acPostProcForPut|$objPath like *txt|nop|nop
````

In the new rule engine, even in backward compatibility modes, it matches $objPath with the content of the ```*txt``` variable, as the ```*``` character is followed by a letter. (Node: if the string is ```*.txt``` then there is no ambiguity) To match with the pattern ```*txt```, change the rule to

````php
acPostProcForPut|$objPath like \*txt|nop|nop
````

The ```\``` character escapes the ```*``` character following it and turns it from a variable prefix into a normal character.

#### Foreach with Comma Separated Strings

In the old rule engine, the following code iterates over a comma separated string:

````php
 rule||assign(*A, "x, y, z")##forEachExec(*A, writeLine(stdout, *A), nop)|nop
````

The new rule engine generates an error when running this code. This rule can be implemented as:

````php
rule {
   *A = split("x, y, z", ", ");
   foreach(*A) {
       writeLine("stdout", *A);
   }
}
````

The split function takes in two parameters, the first a string, the second a separator and then splits the string using the separator and returns a list of substrings.

#### Microservices in Rule Conditions

The old rule engine allows the following code:

````php
rule|msiDoSomething|nop|nop
````

The rule is executed only if the microservice "msiDoSomething" succeeds. A microservice is considered successful if it return an integer >=0 (usually 0) and failed if it returns an integer < 0. The new rule engine requires the rule condition to be a boolean expression which returns either true or false. We could add the following implicit conversion rules:

````php
integer >= 0 -> true
integer < 0  -> false
````

but this would be inconsistent with the conventions of C and C++, where

````php
integer != 0 -> true
integer == 0 -> false
```` 

which would lead to confusion. Therefore, we didn't include those two implicit conversion rules. The example, however, can be written in the new rule engine as follows:

````php
rule {
   on(msiDoSomething >= 0) {
   }
}
````
Note that to test for the failure case, you can use the errorcode function:

````php
rule {
   on(errorcode(msiDoSomething) < 0) {
   }
}
````

#### Expressions in irule Input Parameters

The old rule engine allows the input parameters to be either an unquoted string or an expression:

````php
 testrule||writeLine(stdout, *A *D)|nop
 *A=unquoted string%*D=0 + 1
 ruleExecOut
````

While the old rule engine returns

````php
unquoted string 1
````

The new rule engine returns

````php
unquoted string 0 + 1
````

In the new rule engine, if you use the "##" syntax, then all values of input parameters are strings, quoted or unquoted. In the newer syntax, it can be written as

````php
testrule {
   writeLine("stdout", "*A *D");
}
input *A="unquoted string", *D=0 + 1
output ruleExecOut
````

Note that rules written in the ```##``` syntax have to be saved in a ```.ir``` file and rules written in the newer notation have to be saved in a ```.r``` file.

## Converting from the ```##``` Syntax to the New Rule Engine Syntax

In the 3.0 release of iRODS, a new rule engine is included that eliminates some corner cases where it was ambiguous, such as special charaters in strings, by following the conventions of mainstream programming languages. The resulting syntax is slightly different from the old rule engine. The old rule engine provides two syntaxes for writing rules. The first syntax (the "##" syntax), as found in the core.irb file, looks like:

````php
acPostProcForPut|$objPath like *.txt|msiDataObjCopy($objPath, "$objPath.copy")|nop
````

It has the restriction that every rule must be written in one line for fast processing. The second syntax (the rulegen syntax) is a more readable form and supports multi-line rules.

````php
acPostProcForPut {
   on($objPath like *.txt) {
       msiDataObjCopy($objPath, $objPath.copy);
   }
}
````

However, rules written in the rulegen syntax have to be preprocessed using the rulegen tool into the "##" syntax before the old rule engine can process them. Therefore, they couldn't be directly included in the core.irb file. The syntax supported by the new rule is based on the rulegen syntax, with slight modifications. While we put backward compatibility at high priority, there are several concerns that lead us to make changes to the rulegen syntax in creating the new rule engine syntax.
The biggest one is to eliminate ambiguities in corner cases, which we illustrate by the following examples. Suppose that if the content of ```*A`` is a string ```"0 + 1"```, should the following expression evaluate to true or false?

````php
 *A == 0 + 1
````

This ambiguity has its root in the ambiguity whether ```"0 + 1"``` on the right hand side of the comparison is a string or an integer expression. Instead of adding exceptions to existing grammatical rules, we chose instead to streamline and simplify the rule, as other mainstream programming languages do, by requiring strings to be quoted, so that

````php
 *A == "0 + 1"
````

compares the content of *A with the string "0 + 1", and

````php
 *A == 0 + 1
````
compares the content of ```*A``` with the integer expression ```0 + 1```.

To make it easier to move rules written in the ```##``` syntax, the new rule engine provides backward compatible modes. Backward compatible modes tweak the rule engine parser and type checker so that they simulate the old rule engine when parsing the "##" syntax. In backward compatible mode, strings do not have to be quoted. Backward compatible modes work only with the ```##``` syntax. Backward compatible modes can be changed within a code base using the ```@backwardCompatible``` directive, so that users can have rules that run under backward compatible modes and not in just one code base. The backward compatible modes and their limitations are explained in Changes_and_Improvements_to_the_Rule_Language_and_the_Rule_Engine#Backward Compatibility.
The old rule engine is also included in the 3.0 release in case full backward compatibility is needed. The rule engine used by iRODS can be easily switched to the old one by making a small change in the server Makefile and rebuilding the server.

This section explains how to convert a rule written in the ```##``` syntax to the new rule engine syntax. First, we look at a rule written in the "##" syntax.

````php
 My Test Rule(*arg)|msi(*arg) && *arg like *txt|delayExec(<A></A>, copyDataObj(*objPath)##moveDataObj(*objPath), nop##nop)##remoteExec(localhost, null, writeLine(stdout, *D), nop)##assign(*A, "a, b, c")##assign(*B, *A string)##forEachExec(*A, writeLine(serverLog, *A), nop)|nop
 *obj=test.txt%*D=string
 ruleExecOut
````

Next, we convert this rule into the new rule engine syntax. If a step is written in brackets, it means that the step is the same as that for converting from the ```##``` syntax to the rulegen syntax in iRODS 2.5.

### "##" => Rulegen

#### 1. Insert newlines 

This step is optional.

````php
 My Test Rule(*arg)
     |msi(*arg) && *arg like *txt|
         delayExec(<A></A>, 
             copyDataObj(*objPath)##
             moveDataObj(*objPath), 
             nop##nop)##
         remoteExec(localhost, null, 
             writeLine(stdout, *D), 
             nop)##
         assign(*A, "a, b, c")##
         assign(*B, *A string)##
         forEachExec(*A, 
             writeLine(serverLog, *A), 
             nop)|nop
 *obj=test.txt%*D=string
 ruleExecOut
````

#### 2. Convert "|"s to rulegen syntax 

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like *txt) {
         delayExec(<A></A>, 
             copyDataObj(*objPath)##
             moveDataObj(*objPath), 
             nop##nop)##
         remoteExec(localhost, null, 
             writeLine(stdout, *D), 
             nop)##
         assign(*A, "a, b, c")##
         assign(*B, *A string)##
         forEachExec(*A, 
             writeLine(serverLog, *A), 
             nop)
     }
 }
 *obj=test.txt%*D=string
 ruleExecOut
````
 
#### 3. Convert "##"s to ";" 

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like *txt) {
         delayExec(<A></A>, 
             copyDataObj(*objPath);
             moveDataObj(*objPath), 
             nop;nop);
         remoteExec(localhost, null, 
             writeLine(stdout, *D), 
             nop);
         assign(*A, "a, b, c");
         assign(*B, *A string);
         forEachExec(*A, 
             writeLine(serverLog, *A), 
             nop)
     }
 }
 *obj=test.txt%*D=string
 ruleExecOut
````
 
#### 4. Convert delayExec, remoteExec, assign, forEachExec, etc. to rulegen syntax

In this step, you can add or remove ";"s to make it follow the C conventions.

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like *txt) {
         delay(<A></A>) { 
             copyDataObj(*objPath):::nop;
             moveDataObj(*objPath):::nop;
         }
         remote(localhost, null) { 
             writeLine(stdout, *D):::nop;
         }
         *A = "a, b, c";
         *B = *A string;
         foreach(*A) { 
             writeLine(serverLog, *A):::nop;
         }
     }
 }
 *obj=test.txt%*D=string
 ruleExecOut
````
 
#### 5. Convert input and output to rulegen syntax

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like *txt) {
         delay(<A></A>) { 
             copyDataObj(*objPath):::nop;
             moveDataObj(*objPath):::nop;
         }
         remote(localhost, null) { 
             writeLine(stdout, *D):::nop;
         }
         *A = "a, b, c";
         *B = *A string;
         foreach(*A) { 
             writeLine(serverLog, *A):::nop;
         }
     }
 }
 input *obj=test.txt, *D=string
 output ruleExecOut
````

#### 6. Delete superfluous nops

This step is optional

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like *txt) {
         delay(<A></A>) { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote(localhost, null) { 
             writeLine(stdout, *D);
         }
         *A = "a, b, c";
         *B = *A string;
         foreach(*A) { 
             writeLine(serverLog, *A);
         }
     }
 }
 input *obj=test.txt, *D=string
 output ruleExecOut
````

### Rulegen => New Rule Engine

#### 7. Quote strings

The general guideline is simple, if some text is not quoted, then it is parsed as code. If you want to pass some argument to a micro service and do not want the rule engine to interpret it, then quote the argument to turn it into a string. For example, in

````php
writeLine(serverLog, *A);
````

```serverLog``` is interpreted by the rule engine as a microservice/rule because it is not quoted. The rule engine will try to execute that microservice/rule and will pass the return value in as the argument. To pass "serverLog" as the argument, it has to be quoted.

````php
 writeLine("serverLog", *A);
````

Similarly, the arguments to remote and delay are also strings. So, they need to be quoted.
On the right hand side of an assign statement, we also need to quote the string. If the right hand side is an expression we don't quote it. For example

````php
 *A = "0 + 1"
````

assigns the string ```"0 + 1"``` to ```*A```, and

````php
 *A = 0 + 1
````

assigns ```1``` to ```*A```

On the right hand side of the like expression, we need to quote the pattern because it is also a string.
The input parameters are also strings, and we need to quote them.
The rule now looks like:

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like "*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = "a, b, c";
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````
 
#### 8. Escape special characters in strings
Special character such as ```*``` may be interpreted by the rule engine even in quoted strings. For example, in

````php
*arg like "*txt"
````

```*txt``` is considered a variable which is expanded into the string. However, what we intended to do is to use ```*txt``` as a pattern. Therefore the ```*``` should not be interpreted by the rule engine. To do this we convert it to ```\*txt```. Details on special characters can be found in the Strings section

````php
 My Test Rule(*arg) {
     on(msi(*arg) && *arg like "\*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = "a, b, c";
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````
 
9. Delete white spaces in rule names.

Rule names must be valid identifiers generated from the following regular expression

````php
 <letter> (<letter>|<digit>)*
````

where

````php
 <letter> ::= a|...|z|A|...|Z
 <digit>  ::= 0|...|9
````
 
Now, the rule looks like:
 
````php
 MyTestRule(*arg) {
     on(msi(*arg) && *arg like "\*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = "a, b, c";
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````
 
10. Use split to split the string if it is used as a collection in foreach
In the new rule engine, ```foreach``` requires the parameter to be a list or iRODS type ```GenQueryOut_PI```. The case where it is a comma separated string can be simulated using the split function:

````php
 MyTestRule(*arg) {
     on(msi(*arg) && *arg like "\*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = split("a, b, c", ", ");
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````

#### 11. Convert microservice calls in rule conditions

The micro service call
 
````php
msi(*arg)
````

in the rule condition returns an integer, if the integer is ```>= 0``` then the microservice is considered successful; otherwise it is considered failed. Therefore, when we write in the old syntax

````php
msi(*arg) && *arg like "\*txt"
````

we are not trying to compute the "logical and" with the return code of ```msi(*arg)```, but with whether ```msi(*arg)``` succeeds.

In the new rule engine, we make this explicit, and write

````php
 msi(*arg) >= 0 && *arg like "\*txt"
````

```>=``` and ```like``` have higher priority than ```&&```.

Now the rule looks like:

````php
 MyTestRule(*arg) {
     on(msi(*arg) >= 0 && *arg like "\*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = split("a, b, c", ", ");
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````

#### 12. Remove arguments from the main rule

This step only applies to the first rule in an input file to the irule command.
In the new rule engine, all the input/output variables are global variables. And the first rule is called as the "main" rule in the irule command. The "main" rule should not have any parameters.

````php
 MyTestRule {
     on(msi(*arg) >= 0 && *arg like "\*txt") {
         delay("<A></A>") { 
             copyDataObj(*objPath);
             moveDataObj(*objPath);
         }
         remote("localhost", "null") { 
             writeLine("stdout", *D);
         }
         *A = split("a, b, c", ", ");
         *B = "*A string";
         foreach(*A) { 
             writeLine("serverLog", *A);
         }
     }
 }
 input *arg="test.txt", *D="string"
 output ruleExecOut
````

Now we have a syntactically valid rule for the new rule engine. Make sure to save the rule in a ".r" file as the irule command tells whether an input is in the new rule engine syntax or the "##" syntax by the file extension.

#### Other things to consider:

The ```like``` expression only supports one type of wildcard ```*``` (as I do not know of any other wildcards that are being used"), if your rule uses other kinds of wildcard, it can be converted to the regular expression
If you still get a type error, you can try to convert the value to the correct type using one of the following functions: ```int```, ```double```, ```bool```, or ```str```. For example, if you have an error with

````php
  *A = 1;
  msi(*A);
````

and msi expects a string argument, you can do this by

````php
  msi(str(*A));
````

If you are using the msiCollectionSpider microservice, the actions have to be quoted, which can be done using [Quoting](https://wiki.irods.org/index.php/Changes_and_Improvements_to_the_Rule_Language_and_the_Rule_Engine#Quoting_Code) Code. The variables used in the actions are not accessible outside the actions. This is a result of stricter variables scopes. We will address this issue in future releases.

## Language Integrated General Query

Starting from version 3.2, the Rule Engine supports Language Integrated General Query (LIGQ). Before 3.2, if a rule needs to perform a gen query, it needs to call a sequence of micro services and manually manage the continuation index and the input and output data structures. LIGQ provides native syntax support for gen queries in the rule language and integrates automatic management of continuation index and the input and output data structures into the foreach loop. The goal is to make performing gen queries from rules easy and less error-prone.
A query expression starts with the key word SELECT and looks exactly the same as a normal gen query:

````php
SELECT META_DATA_ATTR_NAME WHERE DATA_NAME = 'data_name'
````

At runtime this query is evaluated to an object of type genQueryInp_t * genQueryOut_t. This object can be assigned to a variable:

````php
*A = SELECT META_DATA_ATTR_NAME WHERE DATA_NAME = 'data_name';
````

and iterated in a foreach loop:

````php
foreach(*Row in *A) {
    ...
}
````

Or we can skip the assignment:
````php
foreach(*Row in SELECT META_DATA_ATTR_NAME WHERE DATA_NAME = 'data_name' AND COLL_NAME = 'coll_name') {
    ...
}
````

where *Row is a keyValPair_t object that contains the current row in the result set. The break statement can be used to exit the foreach loop.

LIGQ supports the following gen query syntax:

* ```count```, ```sum```, ```order_desc, order_asc```
* ```=```, ```<>```, ```>```, ```<```, ```>=```, ```<=```, ```in```, ```between```, ```like```, ```not like```
* ```||``` and ```&&``` (added after the 3.2 release)

LIGO also provides support for using ```==``` and ```!=``` as equality predicates, in order to be consistent with the rule engine syntax.

The left hand side of comparison operators is always a column name, but the right hand side of a comparison operator is always one (or more in the case of ```between```) normal Rule Engine expression(s) which is(are) evaluated by the rule engine first. Therefore, we can use any rule engine expression on the right hand side of a comparison operator. If the right hand side operand is not a simple single quoted string or number, then the LIGO query cannot be executed from the iquery command.

One potential confusion is that the ```like``` and ```not like``` operators in the gen query syntax differ from those in the Rule Language. The right hand side operand is first evaluated by the rule engine to a string which in turn is interpreted by the qen query subsystem. Therefore, LIGQ queries do not use the same syntax for wildcards as the rule engine (unless the wildcards are in a nested rule engine expression). While the rule engine uses ```*``` for wildcards, qen query uses the standard SQL syntax for wildcards.

## Path Literals

This feature is added after the 3.2 release

A path literal starts with a slash:

````php
/tempZone/home/rods
````

A path literal is just like a string, you can use variable expansion, escape characters, etc:

````php
/*Zone/home/*User/\\.txt
````

In addition to the characters that must be escaped in a string, the following characters must also be escaped in a path literal:

````php
,(comma) ;(semicolon) )(right parenthesis)  (space)
````

A path literal can be assigned to a variable:

````php
  *H = /tempZone/home/rods
````

New path literals can be constructed from paths but it must start with "/", the rule engine automatically removes redundant leading slashes in a path:

````php
*F = /*H/foo.txt
````

Path literals can be used in various places. If a path literal points to a collection, it can be used in a foreach loop to loop over all data objects under that collection.

````php
  foreach(*D in *H) {
      ...
  }
````

A path literal can also be used in collection and data object related microservice calls:

````php
  msiCollCreate(/*H/newColl, "", *Status);
  msiRmColl(/*H/newColl, "", *Status);
````
