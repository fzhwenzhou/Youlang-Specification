# Youlang Design

## About this Document
This document describes the basic grammar of Youlang. Youlang aims to be an easy-to-use system-level programming language. The implementation does not use garbage collection, thus making it suitable for system and game programming. It is greatly influenced by functional programming languages like Haskell and OCaml. However, the language itself is a multi-paradigm language. 

About the standard library, please refer to StdLibrary.md.

## Hello World
The first program of most programming languages is usually "Hello World." 
```
main ::= println "Hello, World!"
```
Here, we defined a function named "main," whose parameter list is empty, and the return type is inferred by the return statement of the function. If there is no return statement, then the return value would be void. If there are multiple return statements, the type of the return value would be a variant including all of these possible types. The details will be explained in the "Function" section. 

The function has only one statement. This statement calls a function named "println," whose only parameter is "Hello, World," a string literal. 

For more details, please read the contents in the following contents.

## Expressions
### Primitive Data Types
There are two categories of primitive data types: integers and floating point numbers. 

Signed integer types are prefixed by "i," while unsigned integer types are prefixed by "u." The integer follow the type prefix is the number of bits used to store the integer, where any bit length from $1$ to $2^{23}$ can be specified. However, typical values are usually 8, 16, 32, and 64. Using values other than typical values may lead to performance degration. Specially, "u1" and "i1" are different data types, as u1 can represent only 0 and 1, while i1 can represent only 0 and -1. Bit-lengthes larger than $2^{23}$ are unspecified and implementation defined. It is up to the compiler whether it is defined or not, and how it is defined. Specially, if "size" is followed by the prefix rather than an integer, then it would be the size of the pointer, which is machine-dependent. 

The regex expression for matching an integer type specifier is defined as follows:
```regex
(i|u)(\d+|size)
```
Floating point types are prefixed by "f," followed by the bit length. There are three bit-lengthes that have to be implemented in any YouLang compilers: 16, 32, and 64, with IEEE-754 standard. Note that, on platforms not supporting 16-bit floating point arithmetic, it may cause performance degration to use f16. Other values, like 8, 80, and 128, are unspecified and implementation defined. You need to refer to the compiler's reference to find out its usage. Specially, "bf16" is also a reserved floating point data type, which is specified for "brain floating point." However, it is the implementation to decide whether to implement it or not. 

The regex expression for floating point type is as follows:
```regex
(bf16)|f\d+
```

### Primitive Literals
Integer literals are mainly composed by four parts: sign, base, number, and type. For the sign part, "+" and "-" are valid signs, while "+" is usually omitted. For the base part, "0b," "0o," "0d," and "0x," respectively mean binary, octal, decimal, and hexadecimal, where "0d" is usually omitted. The number part cannot be omitted. It represents the exact number in the specified format. For example, if it is binary, then only 0 and 1 are permitted. If it is hexadecimal, 0-9 and A-F are permitted. In these three parts, it is case insensitive. For the last part, it defines the type of the integer. If the type is omitted, and the number is between $-2^{31}$ and $2^{31} - 1$, then the data type will be inferred as i32. Otherwise, if the number is between $-2^{63}$ and $2^{63} - 1$, then the data type will be inferred as i64. Otherwise, it would throw out an error, and you have to define the data type yourself. If the data type cannot store the previously defined integer, it will throw a compile error. 

The regex for an integer literal is defined as follows:
```regex
[+-]?(0b[01]+|0o[0-7]+|0x[0-9a-fA-F]+|(0d)?(\d+))((i|u)(\d+|size))?
```

Floating point literals are also composed by four parts: sign, number, exponent, and type. For the sign part, "+" and "-" are valid signs, while "+" is usually omitted. For the number part, it should be a floating point literal. That is, it should have at least one decimal point "." with at least one number before or after it. For the exponential part, it is an optional section starting by "E" or "e" and following by an integer literal standing for the exponent to 10. The last part is the type identifier, which stands for the type of the floating point literal. If it is omitted, then the literal is f64.



The regex for a floating point literal is defined as follows:
```regex
[+-]?(\d+\.(\d*)?|\.\d+)([eE][+-]?\d+)?((bf16)|f\d+)?
```

There are some other primitive literals that fall in this category. However, these are mainly for other purposes, and not used for arithmetic most of the time. 

Boolean literals are actually u1 integer literals. There are two boolean values: False for 0 and True for 1. 

Character literals are also unsigned integer literals. A character literal is composed by two parts: character part and type part. For the character part, a character is wrapped by two single quotes. The compiler would throw out an error if there are more than one characters in between. The character is represented by its UTF-8 value. If the type part is omitted, its type would be u32 by default to store all of the UTF-8 characters. You may specify other unsigned integer types, but if the type cannot contain the character, the compiler would throw an error.

The regex for a character literal is as follows:
```regex
'(\\'|[^'])'((i|u)(\d+|size))?
```

### Primitive Operators
The operators 