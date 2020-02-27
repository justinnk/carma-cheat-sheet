# CARMA Specification Language (CaSL) Cheat Sheet

This document contains useful information about the syntax of the CaSL (which can be found [here](https://github.com/Quanticol/CARMA/)). It may be incomplete and contain errors. The syntax in `xtext` format can be found [here](https://github.com/Quanticol/CARMA/blob/master/XTEXT/eu.quanticol.carma.core/src/eu/quanticol/carma/core/CARMA.xtext). Tutorials with examples can be found [here](https://ieeexplore.ieee.org/abstract/document/8632456) and [here](https://link.springer.com/chapter/10.1007/978-3-319-34096-8_4).

## Structure of the File

The basic structure of a carma file is as follows:

- constants and functions
- (space definition)
- components
    - store
    - behaviour
    - inital state
- environment definition
    - global attributes
    - weights and probabilities
    - reates
    - updates
- measures

The order in which the main parts are defined is not important, although the provided one is very logical.

### Constants, Variables and Functions

Constants can be of type `int`, `real`, `bool`, `list`, `set`. Be careful when setting the values and always use `.0` at the end of reals in case they are integer.
It is possible to define records with `record` and enumerations with `enum`. Record names *should* and enum names **must** start with a capital letter.
Some examples:

```C
// integer constant
const x = 42;
// real constant
// format: ('0'..'9')+ '.' ('0' .. '9')+ ('e' ('+'|'-')? ('0'..'9')+ )?
const f = 42.5;
// boolean constant
const b = false;
// list constant
const l = [:1, 2, 3, 4:];
const ll = [:[:1, 2:], [:3, 4:]:];
const emptylist = newList(int);
// set constant
const s = {:1, 2, 3, 4, 5:};
const emptySet = newSet(int);
// records
record Vector = [real x, real y]
const position = [-1.0, 1.0];
// enumerations
enum Number = ONE, TWO, THREE;
const id = TWO;
// function
fun int to_int(real parameter){
    return int(parameter);
}
```

#### Functions

The following syntax is allowed in functions:

- assignment with `=` or `:=` to
  - variables: `var = val;`
  - lists: `list[i] = val;`
  - records: `rec.X = val;`
- Blocks with `{ // code }`
- For loops: `for x from 0 to 10 { // code }` or `for x from 0 by 2 to 10 { // code }`
- Foreach loops: `foreach x in ls { //code }`
- Branching: `if (condition) { // code }` or `if (condition) { //code } else { //code }`
- Returning a value: `return x;`

#### Variables

Variables (not store attributes) can be defined in the following way:

```C
// integer
int x;
// integer with range
int[min .. max] x;
// floating point
real f;
// boolean
bool b;
// list
list<type> l;
// sets
set<type> s;
// custom types
CustomType c;
// casting to int or real
int(0.5)
real(1)
```

### Space Definition

todo

### Components

```
component Robot(int id){
    store{
        attrib x = 42;
        attrib f = 42.5;
        attrib b = true;
    }
    behaviour{
        Initial = action[true]<>{}.Initial;
    }
    init {
        Initial
    }
}
```

#### Communication

```
// unicast send
unicast[predicate]<message1, message2, ...>{store update}
// unicast receive
unicast[predicate](message1, message2, ...){store update}
// broadcast send
broadcast*[predicate]<message1, message2, ...>{store update}
// broadcast receive
broadcast*[predicate](message1, message2, ...){store update}
```

The predicates have a boolean value. It is possible to use the `sender` and `my` context, e.g.

```
get[sender.location == my.location]<>{my.location == new_location}
```

## Environment Definition

### The `weight` Block

In case of competing receivers during a unicast communication, the weight block decides which one will receive the message. It is possible to supply atomic probabilities or use functions that depend for example on a set comprehension.

### The `prob` Block

The same as `weight`, but for broadcast. This block decides what the probability of a certain broadcast communication is. For assistance, the local stores of `sender` and `receiver` can be accessed to determince the probability.
Does allow vairable definitions.

### The `rate` Block

Does allow vairable definitions.

### The `update` Block 

Does not allow variables to be defined.

## Assignments

Assignments to variables can be made via the following syntax:

```C
var := value;
// or
var = value;
// for fields, the above or
var[i] <- value;
```

## Boolean Algebra

The common operators for boolean algebra can be used:

```C
// and
&&
// or
||
// not
!
// equals
==
// not equal
!=
```

## Ternary Operator

This inline branching operator can be used everywhere. It is important that it is surrounded by brackets.

```C
int x = (b ? 0 : 1)
```

## Measures

Measures can be defined in terms of global attributes or set comprehensions over the component states. They can have parameters.

```C
// example set comprehension: number of Robots in the initial state at location l
measure test = #{Robot[Initial] | my.loc == l};
// example global attribute
measure test2 = global.x;
// measure with parameters
measure test3(int x, int y) = #{Robot[Moving] | my.loc != x && my.loc != y}
```

## Set Comprehension

```C
// general form:
function { expression | predicate };
// function can be one of:
# // Cardinality
min // minimum
max // maximum
avg // average
```

### Component Comprehension

A special case of set comprehension for counting components in a certain state.

```
// general form
# { component[state1 | state2 | ...] | predicate };
// or
# { component[state] | predicate };
// or
# { component[*] | predicate };
```

## Randomness

It is possible to draw from several distributions:

- atomic random `RND`
- uniform `U(0, 1, 2, 3)` or `U([:0, 1, 2, 3:])`
- normal `NORMAL(mean, std)`
- weighted choice `selectFrom(0: 0.5, 1: 0.5)`
- choice `select(1, 2)`

## Builtin Functions

```C
// simulation
now // returns the current simulation time
// math
abs(x) // absolute value of x
pow(x, y) // returns x to the yth power
sqrt(x) // Returns the correctly rounded positive square root of a double value
cbrt(x) // The cubic root of arg
ceil(x) // The smallest (closest to negative infinity) double value that is greater than or equal to the argument and is equal to a mathematical integer
floor(x) // Returns the largest (closest to positive infinity) double value that is less than or equal to the argument and is equal to a mathematical integer
max(first, second) // returns the greater value of first and second
min(fist, second) // returns the smaller value of first and second
// trigonometry
sin(x) // Returns the trigonometric sine of an angle
cos(x) // Returns the trigonometric cosine of an angle
tan(x) // Returns the trigonometric tangent of an angle
asin(x) // The arc sine of x; the returned angle is in the range -pi/2 through pi/2
acos(x) // The arc cosine of x; the returned angle is in the range 0.0 through p
atan(x) // Returns the arc tangent x; the returned angle is in the range -pi/2 through pi/2
atan2(first, second) // The angle theta from the conversion of rectangular coordinates (first, second) to polar coordinates (r, theta)
// logarithms
exp(x) // Returns Euler's number e raised to the power of a double value
log(x) // Returns the natural logarithm (base e) of a double value
log10(x) // Returns the base 10 logarithm of a double value
// lists
size(l) // returns the length of a list
head(l) // returns the first element of a list
tail(l) // returns all but the first element of a list
// lambda
map(l, f)
filter(l, f)
exist(l, f)
forall(l, f)
```

### Lambda Context

For `map`, `filter`, `exist` and `forall`, a function needs to be supplied as second argument. The parameter, that should be substituted by the list value must be changed to `@`, e.g.

```
[:1, 2, 3, 4:].filter(equals(@, 2))
// returns [: 2 :]
```

## Builtin Constants

```
MAXINT // maximum integer value
MININT // minimum integer value
MAXREAL
MINREAL
true
false
none
PI // pi constant
E // euler constant
```

## Workarounds

- **Bug:** Sometimes the compilation fails when using lists as a global variable and accessing it in the update block.
- **Workaround:** try to access it wothout the `global` context.
- **Bug:** In the above case, sometimes the list must appear on the right side when setting a specific element.
- **Workaround:** `list[0] := list[0] * 0 + value;`