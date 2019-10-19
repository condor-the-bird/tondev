# Introduction

The primary purpose of the Telegram Open Network Virtual Machine (TON VM or TVM) is to execute smart-contract code in the TON Blockchain. 

TVM must support all operations required to parse incoming messages and persistent data, and to create new messages and modify persistent data. Additionally, TVM must meet the following requirements: 

- It must provide for possible future extensions and improvements while retaining backward compatibility and interoperability, because the code of a smart contract, once committed into the blockchain, must continue working in a predictable manner regardless of any future modifications to the VM. 
- It must strive to attain high “(virtual) machine code” density, so that the code of a typical smart contract occupies as little persistent blockchain storage as possible. 
- It must be completely deterministic. In other words, each run of the same code with the same input data must produce the same result, regardless of specific software and hardware used.*

The design of TVM is guided by these requirements. While this document describes a preliminary and experimental version of TVM,** the backward compatibility mechanisms built into the system allow us to be relatively unconcerned with the efficiency of the operation encoding used for TVM code in this preliminary version. TVM is not intended to be implemented in hardware (e.g., in a specialized microprocessor chip); rather, it should be implemented in software running on conventional hardware. This consideration lets us incorporate some highlevel concepts and operations in TVM that would require convoluted microcode in a hardware implementation but pose no significant problems for a software implementation. Such operations are useful for achieving high code density and minimizing the byte (or storage cell) profile of smart-contract code when deployed in the TON Blockchain.

*For example, there are no floating-point arithmetic operations (which could be efficiently implemented using hardware-supported double type on most modern CPUs) present in TVM, because the result of performing such operations is dependent on the specific underlying hardware implementation and rounding mode settings. Instead, TVM supports special integer arithmetic operations, which can be used to simulate fixed-point arithmetic if needed.

**The production version will likely require some tweaks and modifications prior to launch, which will become apparent only after using the experimental version in the test environment for some time.

# Overview

This chapter provides an overview of the main features and design principles of TVM. More detail on each topic is provided in subsequent chapters. 

## 1.0 Notation for bitstrings 

The following notation is used for bit strings *(or bitstrings)*—i.e., finite strings consisting of binary digits (bits), 0 and 1—throughout this document. 

### 1.0.1. Hexadecimal notation for bitstrings. 

When the length of a bitstring is a multiple of four, we subdivide it into groups of four bits and represent each group by one of sixteen hexadecimal digits 0–9, A–F in the usual manner: `016 ↔ 0000, 116 ↔ 0001, . . . , F16 ↔ 1111. `The resulting hexadecimal string is our equivalent representation for the original binary string. 

### 1.0.2. Bitstrings of lengths not divisible by four. 

If the length of a binary string is not divisible by four, we augment it by one 1 and several (maybe zero) 0s at the end, so that its length becomes divisible by four, and then transform it into a string of hexadecimal digits as described above. To indicate that such a transformation has taken place, a special “completion tag” _ is added to the end of the hexadecimal string. The reverse transformation (applied if the completion tag is present) consists in first replacing each hexadecimal digit by four corresponding bits, and then removing all trailing zeroes (if any) and the last 1 immediately preceding them (if the resulting bitstring is non-empty at this point). 

Notice that there are several admissible hexadecimal representations for the same bitstring. Among them, the shortest one is “canonical”. It can be deterministically obtained by the above procedure. For example, 8A corresponds to binary string 10001010, while 8A_ and 8A0_ both correspond to 100010. An empty bitstring may be represented by either ‘’, ‘8_’, ‘0_’, ‘_’, or ‘00_’. 

### 1.0.3. Emphasizing that a string is a hexadecimal representation of a bitstring. 

Sometimes we need to emphasize that a string of hexadecimal digits (with or without a _ at the end) is the hexadecimal representation of a bitstring. In such cases, we either prepend x to the resulting string (e.g., x8A), or prepend x{ and append } (e.g., x{2D9_}, which is 00101101100).

This should not be confused with hexadecimal numbers, usually prepended by 0x (e.g., 0x2D9 or 0x2d9, which is the integer 729). 

### 1.0.4. Serializing a bitstring into a sequence of octets. 

When a bitstring needs to be represented as a sequence of 8-bit bytes (octets), which take values in integers 0 . . . 255, this is achieved essentially in the same fashion as above: we split the bitstring into groups of eight bits and interpret each group as the binary representation of an integer 0 . . . 255. If the length of the bitstring is not a multiple of eight, the bitstring is augmented by a binary 1 and up to seven binary 0s before being split into groups. The fact that such a completion has been applied is usually reflected by a “completion tag” bit. For instance, 00101101100 corresponds to the sequence of two octets (0x2d, 0x90) (hexadecimal), or (45, 144) (decimal), along with a completion tag bit equal to 1 (meaning that the completion has been applied), which must be stored separately. In some cases, it is more convenient to assume the completion is enabled by default rather than store an additional completion tag bit separately. Under such conventions, 8n-bit strings are rep

## 1.1 TVM is a stack machine

First of all, TVM is a stack machine. This means that, instead of keeping values in some “variables” or “general-purpose registers”, they are kept in a (LIFO) stack, at least from the “low-level” (TVM) perspective. Most operations and user-defined functions take their arguments from the top of the stack, and replace them with their result. For example, the integer addition primitive (built-in operation) ADD does not take any arguments describing which registers or immediate values should be added together and where the result should be stored. Instead, the two top values are taken from the stack, they are added together, and their sum is pushed into the stack in their place.

### 1.1.1. TVM values. 

The entities that can be stored in the TVM stack will be called TVM values, or simply values for brevity. They belong to one of several predefined value types. Each value belongs to exactly one value type. The values are always kept on the stack along with tags uniquely determining their types, and all built-in TVM operations (or primitives) only accept values of predefined types. For example, the integer addition primitive ADD accepts only two integer values, and returns one integer value as a result. One cannot supply ADD with two strings instead of two integers expecting it to concatenate these strings or to implicitly transform the strings into their decimal integer values; any attempt to do so will result in a run-time type-checking exception. 

### 1.1.2. Static typing, dynamic typing, and run-time type checking. 

In some respects TVM performs a kind of dynamic typing using run-time type checking. However, this does not make the TVM code a “dynamically typed language” like PHP or Javascript, because all primitives accept values and return results of predefined (value) types, each value belongs to strictly one type, and values are never implicitly converted from one type to another. 

If, on the other hand, one compares the TVM code to the conventional microprocessor machine code, one sees that the TVM mechanism of value tagging prevents, for example, using the address of a string as a number— or, potentially even more disastrously, using a number as the address of a string—thus eliminating the possibility of all sorts of bugs and security vulnerabilities related to invalid memory accesses, usually leading to memory corruption and segmentation faults. This property is highly desirable for a VM used to execute smart contracts in a blockchain. 

In this respect, TVM’s insistence on tagging all values with their appropriate types, instead of reinterpreting the bit sequence in a register depending on the needs of the operation it is used in, is just an additional run-time type-safety mechanism. An alternative would be to somehow analyze the smart-contract code for type correctness and type safety before allowing its execution in the VM, or even before allowing it to be uploaded into the blockchain as the code of a smart contract. 

Such a static analysis of code for a Turing-complete machine appears to be a time-consuming and non-trivial problem (likely to be equivalent to the stopping problem for Turing machines), something we would rather avoid in a blockchain smart-contract context. 

One should bear in mind that one always can implement compilers from statically typed high-level smart-contract languages into the TVM code (and we do expect that most smart contracts for TON will be written in such languages), just as one can compile statically typed languages into conventional machine code (e.g., x86 architecture). 

If the compiler works correctly, the resulting machine code will never generate any run-time type-checking exceptions. All type tags attached to values processed by TVM will always have expected values and may be safely ignored during the analysis of the resulting TVM code, apart from the fact that the run-time generation and verification of these type tags by TVM will slightly slow down the execution of the TVM code. 

### 1.1.3. Preliminary list of value types.

A preliminary list of value types supported by TVM is as follows: 

- Integer — Signed 257-bit integers, representing integer numbers in the range −2 256 . . . 2 256 − 1, as well as a special “not-a-number” value NaN. 
- Cell — A TVM cell consists of at most 1023 bits of data, and of at most four references to other cells. All persistent data (including TVM code) in the TON Blockchain is represented as a collection of TVM cells (cf. [1, 2.5.14]). • Tuple — An ordered collection of up to 255 components, having arbitrary value types, possibly distinct. May be used to represent nonpersistent values of arbitrary algebraic data types. 
- Null — A type with exactly one value ⊥, used for representing empty lists, empty branches of binary trees, absence of return value in some situations, and so on. • Slice — A TVM cell slice, or slice for short, is a contiguous “sub-cell” of an existing cell, containing some of its bits of data and some of its references. Essentially, a slice is a read-only view for a subcell of a cell. Slices are used for unpacking data previously stored (or serialized) in a cell or a tree of cells. 
- Builder — A TVM cell builder, or builder for short, is an “incomplete” cell that supports fast operations of appending bitstrings and cell references at its end. Builders are used for packing (or serializing) data from the top of the stack into new cells (e.g., before transferring them to persistent storage). 
- Continuation — Represents an “execution token” for TVM, which may be invoked (executed) later. As such, it generalizes function addresses (i.e., function pointers and references), subroutine return addresses, instruction pointer addresses, exception handler addresses, closures, partial applications, anonymous functions, and so on.

This list of value types is incomplete and may be extended in future revisions of TVM without breaking the old TVM code, due mostly to the fact that all originally defined primitives accept only values of types known to them and will fail (generate a type-checking exception) if invoked on values of new types. Furthermore, existing value types themselves can also be extended in the future: for example, 257-bit Integer might become 513-bit LongInteger , with originally defined arithmetic primitives failing if either of the arguments or the result does not fit into the original subtype Integer. Backward compatibility with respect to the introduction of new value types and extension of existing value types will be discussed in more detail later (cf. 5.1.4). 

## 1.2 Categories of TVM instructions

TVM instructions, also called primitives and sometimes (built-in) operations, are the smallest operations atomically performed by TVM that can be present in the TVM code. They fall into several categories, depending on the types of values (cf. [1.1.3](https://zeroheight.com/86757ecb2/p/77e11f/t/033c4d)) they work on. The most important of these categories are: 

- Stack (manipulation) primitives — Rearrange data in the TVM stack, so that the other primitives and user-defined functions can later be called with correct arguments. Unlike most other primitives, they are polymorphic, i.e., work with values of arbitrary types. 
- Tuple (manipulation) primitives — Construct, modify, and decompose Tuples. Similarly to the stack primitives, they are polymorphic. 
- Constant or literal primitives — Push into the stack some “constant” or “literal” values embedded into the TVM code itself, thus providing arguments to the other primitives. They are somewhat similar to stack primitives, but are less generic because they work with values of specific types. 
- Arithmetic primitives — Perform the usual integer arithmetic operations on values of type Integer. 
- Cell (manipulation) primitives — Create new cells and store data in them (cell creation primitives) or read data from previously created cells (cell parsing primitives). Because all memory and persistent storage of TVM consists of cells, these cell manipulation primitives actually correspond to “memory access instructions” of other architectures. Cell creation primitives usually work with values of type Builder, while cell parsing primitives work with Slices. 
- Continuation and control flow primitives — Create and modify Continuations, as well as execute existing Continuations in different ways, including conditional and repeated execution. 
- Custom or application-specific primitives — Efficiently perform specific high-level actions required by the application (in our case, the TON Blockchain), such as computing hash functions, performing elliptic curve cryptography, sending new blockchain messages, creating new smart contracts, and so on. These primitives correspond to standard library functions rather than microprocessor instructions. 

## 1.3 Control registers 

While TVM is a stack machine, some rarely changed values needed in almost all functions are better passed in certain special registers, and not near the top of the stack. Otherwise, a prohibitive number of stack reordering operations would be required to manage all these values. 

To this end, the TVM model includes, apart from the stack, up to 16 special control registers, denoted by c0 to c15, or c(0) to c(15). The original version of TVM makes use of only some of these registers; the rest may be supported later. 

### 1.3.1. Values kept in control registers. 

The values kept in control registers are of the same types as those kept on the stack. However, some control registers accept only values of specific types, and any attempt to load a value of a different type will lead to an exception. 

### 1.3.2. List of control registers. 

The original version of TVM defines and uses the following control registers:

- c0 — Contains the next continuation or return continuation (similar to the subroutine return address in conventional designs). This value must be a Continuation. 
- c1 — Contains the alternative (return) continuation; this value must be a Continuation. It is used in some (experimental) control flow primitives, allowing TVM to define and call “subroutines with two exit points”. • c2 — Contains the exception handler. This value is a Continuation, invoked whenever an exception is triggered. 
- c3 — Contains the current dictionary, essentially a hashmap containing the code of all functions used in the program. For reasons explained later in 4.6, this value is also a Continuation, not a Cell as one might expect. 
- c4 — Contains the root of persistent data, or simply the data. This value is a Cell. When the code of a smart contract is invoked, c4 points to the root cell of its persistent data kept in the blockchain state. If the smart contract needs to modify this data, it changes c4 before returning. 
- c5 — Contains the output actions. It is also a Cell initialized by a reference to an empty cell, but its final value is considered one of the smart contract outputs. For instance, the SENDMSG primitive, specific for the TON Blockchain, simply inserts the message into a list stored in the output actions. 
- c7 — Contains the root of temporary data. It is a Tuple, initialized by a reference to an empty Tuple before invoking the smart contract and discarded after its termination.

 More control registers may be defined in the future for specific TON Blockchain or high-level programming language purposes, if necessary.



## 1.4 Total state of TVM (SCCCG) 

The total state of TVM consists of the following components: 

Stack (cf.[ 1.1](https://zeroheight.com/86757ecb2/p/77e11f/t/08f507)) — Contains zero or more values (cf. [1.1.1](https://zeroheight.com/86757ecb2/p/77e11f/t/321ac4)), each belonging to one of value types listed in [1.1.3](https://zeroheight.com/86757ecb2/p/77e11f/t/033c4d). 

Control registers c0–c15 — Contain some specific values as described in [1.3.2](https://zeroheight.com/86757ecb2/p/77e11f/t/589c51). (Only seven control registers are used in the current version.) 

Current continuation cc — Contains the current continuation (i.e., the code that would be normally executed after the current primitive is completed). This component is similar to the instruction pointer register (ip) in other architectures. 

Current codepage cp — A special signed 16-bit integer value that selects the way the next TVM opcode will be decoded. For example, future versions of TVM might use different codepages to add new opcodes while preserving backward compatibility. 

Gas limits gas — Contains four signed 64-bit integers: the current gas limit gl , the maximal gas limit gm, the remaining gas gr, and the gas credit gc. Always 0 ≤ gl ≤ gm, gc ≥ 0, and gr ≤ gl + gc; gc is usually initialized by zero, gr is initialized by gl + gc and gradually decreases as the TVM runs. 

When gr becomes negative or if the final value of gr is less than gc, an out of gas exception is triggered. Notice that there is no “return stack” containing the return addresses of all previously called but unfinished functions. Instead, only control register c0 is used. 

The reason for this will be explained later in 4.1.9. Also notice that there are no general-purpose registers, because TVM is a stack machine (cf. [1.1](https://zeroheight.com/86757ecb2/p/77e11f/t/08f507)). So the above list, which can be summarized as “stack, control, continuation, codepage, and gas” (SCCCG), similarly to the classical SECD machine state (“stack, environment, control, dump”), is indeed the total state of TVM.

## 1.5 Integer arithmetic 

All arithmetic primitives of TVM operate on several arguments of type Integer, taken from the top of the stack, and return their results, of the same type, into the stack. Recall that Integer represents all integer values in the range −2^256 ≤ x < 2^256, and additionally contains a special value NaN (“nota-number”). 

If one of the results does not fit into the supported range of integers— or if one of the arguments is a NaN—then this result or all of the results are replaced by a NaN, and (by default) an integer overflow exception is generated. However, special “quiet” versions of arithmetic operations will simply produce NaNs and keep going. If these NaNs end up being used in a “non-quiet” arithmetic operation, or in a non-arithmetic operation, an integer overflow exception will occur. 

### 1.5.1. Absence of automatic conversion of integers. 

Notice that TVM Integer s are “mathematical” integers, and not 257-bit strings interpreted differently depending on the primitive used, as is common for other machine code designs. For example, TVM has only one multiplication primitive MUL, rather than two (MUL for unsigned multiplication and IMUL for signed multiplication) as occurs, for example, in the popular x86 architecture. 

### 1.5.2. Automatic overflow checks. 

Notice that all TVM arithmetic primitives perform overflow checks of the results. If a result does not fit into the Integer type, it is replaced by a NaN, and (usually) an exception occurs. In particular, the result is not automatically reduced modulo 2^256 or 2^257, as is common for most hardware machine code architectures. 

### 1.5.3. Custom overflow checks. 

In addition to automatic overflow checks, TVM includes custom overflow checks, performed by primitives FITS n and UFITS n, where 1 ≤ n ≤ 256. These primitives check whether the value on (the top of) the stack is an integer x in the range −2 n−1 ≤ x < 2 n−1 or 0 ≤ x < 2 n , respectively, and replace the value with a NaN and (optionally) generate an integer overflow exception if this is not the case. 

This greatly simplifies the implementation of arbitrary n-bit integer types, signed or unsigned: the programmer or the compiler must insert appropriate FITS or UFITS primitives either after each arithmetic operation (which is more reasonable, but requires more checks) or before storing computed values and returning them from functions. This is important for smart contracts, where unexpected integer overflows happen to be among the most common source of bugs. 

### 1.5.4. Reduction modulo 2^n . 

TVM also has a primitive MODPOW2 n, which reduces the integer at the top of the stack modulo 2 n , with the result ranging from 0 to 2^n − 1. 

### 1.5.5. Integer is 257-bit, not 256-bit. 

One can understand now why TVM’s Integer is (signed) 257-bit, not 256-bit. The reason is that it is the smallest integer type containing both signed 256-bit integers and unsigned 256-bit integers, which does not require automatic reinterpreting of the same 256-bit string depending on the operation used (cf. [1.5.1](https://zeroheight.com/86757ecb2/p/77e11f/t/45de2a)). 

### 1.5.6. Division and rounding. 

The most important division primitives are DIV, MOD, and DIVMOD. All of them take two numbers from the stack, x and y (y is taken from the top of the stack, and x is originally under it), compute the quotient q and remainder r of the division of x by y (i.e., two integers such that x = yq + r and |r| < |y|), and return either q, r, or both of them. If y is zero, then all of the expected results are replaced by NaNs, and (usually) an integer overflow exception is generated.

The implementation of division in TVM somewhat differs from most other implementations with regards to rounding. By default, these primitives round to −∞, meaning that q = bx/yc, and r has the same sign as y. (Most conventional implementations of division use “rounding to zero” instead, meaning that r has the same sign as x.) 

Apart from this “floor rounding”, two other rounding modes are available, called “ceiling rounding” (with q = dx/ye, and r and y having opposite signs) and “nearest rounding” (with q = bx/y + 1/2c and |r| ≤ |y|/2). These rounding modes are selected by using other division primitives, with letters C and R appended to their mnemonics. For example, DIVMODR computes both the quotient and the remainder using rounding to the nearest integer. 

### 1.5.7. Combined multiply-divide, multiply-shift, and shift-divide operations. 

To simplify implementation of fixed-point arithmetic, TVM supports combined multiply-divide, multiply-shift, and shift-divide operations with double-length (i.e., 514-bit) intermediate product. For example, MULDIVMODR takes three integer arguments from the stack, a, b, and c, first computes ab using a 514-bit intermediate result, and then divides ab by c using rounding to the nearest integer. If c is zero or if the quotient does not fit into Integer, either two NaNs are returned, or an integer overflow exception is generated, depending on whether a quiet version of the operation has been used. Otherwise, both the quotient and the remainder are pushed into the stack. 

# The Stack

This chapter contains a general discussion and comparison of register and stack machines, expanded further in Appendix C, and describes the two main classes of stack manipulation primitives employed by TVM: the basic and the compound stack manipulation primitives. An informal explanation of their sufficiency for all stack reordering required for correctly invoking other primitives and user-defined functions is also provided. Finally, the problem of efficiently implementing TVM stack manipulation primitives is discussed in [2.3](https://zeroheight.com/86757ecb2/p/6198ce/t/701240). 

## 2.1 Stack calling conventions 

A stack machine, such as TVM, uses the stack—and especially the values near the top of the stack—to pass arguments to called functions and primitives (such as built-in arithmetic operations) and receive their results. This section discusses the TVM stack calling conventions, introduces some notation, and compares TVM stack calling conventions with those of certain register machines. 

### 2.1.1. Notation for “stack registers”. 

Recall that a stack machine, as opposed to a more conventional register machine, lacks general-purpose registers. However, one can treat the values near the top of the stack as a kind of “stack registers”. We denote by s0 or s(0) the value at the top of the stack, by s1 or s(1) the value immediately under it, and so on. The total number of values in the stack is called its depth. If the depth of the stack is n, then s(0), s(1), . . . , s(n − 1) are well-defined, while s(n) and all subsequent s(i) with i > n are not. Any attempt to use s(i) with i ≥ n should produce a stack underflow exception. A compiler, or a human programmer in “TVM code”, would use these “stack registers” to hold all declared variables and intermediate values, similarly to the way general-purpose registers are used on a register machine. 

### 2.1.2. Pushing and popping values. 

When a value x is pushed into a stack of depth n, it becomes the new s0; at the same time, the old s0 becomes the new s1, the old s1—the new s2, and so on. The depth of the resulting stack is n + 1.

Similarly, when a value x is popped from a stack of depth n ≥ 1, it is the old value of s0 (i.e., the old value at the top of the stack). After this, it is removed from the stack, and the old s1 becomes the new s0 (the new value at the top of the stack), the old s2 becomes the new s1, and so on. The depth of the resulting stack is n − 1. If originally n = 0, then the stack is empty, and a value cannot be popped from it. If a primitive attempts to pop a value from an empty stack, a stack underflow exception occurs. 

### 2.1.3. Notation for hypothetical general-purpose registers. 

In order to compare stack machines with sufficiently general register machines, we will denote the general-purpose registers of a register machine by r0, r1, and so on, or by r(0),` r(1), . . . , r(n − 1)`, where n is the total number of registers. When we need a specific value of n, we will use n = 16, corresponding to the very popular x86-64 architecture. 

### 2.1.4. The top-of-stack register s0 vs. the accumulator register r0. 

Some register machine architectures require one of the arguments for most arithmetic and logical operations to reside in a special register called the accumulator. In our comparison, we will assume that the accumulator is the general-purpose register r0; otherwise we could simply renumber the registers. In this respect, the accumulator is somewhat similar to the top-ofstack “register” s0 of a stack machine, because virtually all operations of a stack machine both use s0 as one of their arguments and return their result as s0. 

### 2.1.5. Register calling conventions. 

When compiled for a register machine, high-level language functions usually receive their arguments in certain registers in a predefined order. If there are too many arguments, these functions take the remainder from the stack (yes, a register machine usually has a stack, too!). Some register calling conventions pass no arguments in registers at all, however, and only use the stack (for example, the original calling conventions used in implementations of Pascal and C, although modern implementations of C use some registers as well). 

For simplicity, we will assume that up to m ≤ n function arguments are passed in registers, and that these registers are` r0, r1, . . . , r(m − 1)`, in that order (if some other registers are used, we can simply renumber them).

> Our inclusion of r0 here creates a minor conflict with our assumption that the accumulator register, if present, is also r0; for simplicity, we will resolve this problem by assuming that the first argument to a function is passed in the accumulator. 

### 2.1.6. Order of function arguments. 

If a function or primitive requires m arguments `x1, . . . , xm`, they are pushed by the caller into the stack in the same order, starting from x1. Therefore, when the function or primitive is invoked, its first argument x1 is in s(m − 1), its second argument x2 is in s(m − 2), and so on. The last argument xm is in s0 (i.e., at the top of the stack). It is the called function or primitive’s responsibility to remove its arguments from the stack. In this respect the TVM stack calling conventions—obeyed, at least, by TMV primitives—match those of Pascal and Forth, and are the opposite of those of C (in which the arguments are pushed into the stack in the reverse order, and are removed by the caller after it regains control, not the callee). 

Of course, an implementation of a high-level language for TVM might choose some other calling conventions for its functions, different from the default ones. This might be useful for certain functions—for instance, if the total number of arguments depends on the value of the first argument, as happens for “variadic functions” such as scanf and printf. In such cases, the first one or several arguments are better passed near the top of the stack, not somewhere at some unknown location deep in the stack.

###  2.1.7. Arguments to arithmetic primitives on register machines. 

On a stack machine, built-in arithmetic primitives (such as ADD or DIVMOD) follow the same calling conventions as user-defined functions. In this respect, user-defined functions (for example, a function computing the square root of a number) might be considered as “extensions” or “custom upgrades” of the stack machine. This is one of the clearest advantages of stack machines (and of stack programming languages such as Forth) compared to register machines. In contrast, arithmetic instructions (built-in operations) on register machines usually get their parameters from general-purpose registers encoded in the full opcode. A binary operation, such as SUB, thus requires two arguments, r(i) and r(j), with i and j specified by the instruction. A register r(k) for storing the result also must be specified. Arithmetic operations can take several possible forms, depending on whether i, j, and k are allowed to take arbitrary values: 

- Three-address form — Allows the programmer to arbitrarily choose not only the two source registers r(i) and r(j), but also a separate destination register r(k). This form is common for most RISC processors, and for the XMM and AVX SIMD instruction sets in the x86-64 architecture. 
- Two-address form — Uses one of the two operand registers (usually r(i)) to store the result of an operation, so that k = i is never indicated explicitly. Only i and j are encoded inside the instruction. This is the most common form of arithmetic operations on register machines, and is quite popular on microprocessors (including the x86 family). 
- One-address form — Always takes one of the arguments from the accumulator r0, and stores the result in r0 as well; then i = k = 0, and only j needs to be specified by the instruction. This form is used by some simpler microprocessors (such as Intel 8080). Note that this flexibility is available only for built-in operations, but not for user-defined functions. In this respect, register machines are not as easily “upgradable” as stack machines.

> For instance, if one writes a function for extracting square roots, this function will always accept its argument and return its result in the same registers, in contrast with a hypothetical built-in square root instruction, which could allow the programmer to arbitrarily choose the source and destination registers. Therefore, a user-defined function is tremendously less flexible than a built-in instruction on a register machine.

### 2.1.8. Return values of functions. 

In stack machines such as TVM, when a function or primitive needs to return a result value, it simply pushes it into the stack (from which all arguments to the function have already been removed). Therefore, the caller will be able to access the result value through the top-of-stack “register” s0. This is in complete accordance with Forth calling conventions, but differs slightly from Pascal and C calling conventions, where the accumulator register r0 is normally used for the return value. 

### 2.1.9. Returning several values. 

Some functions might want to return several values `y1, . . . , yk`, with k not necessarily equal to one. In these cases, the k return values are pushed into the stack in their natural order, starting from y1. For example, the “divide with remainder” primitive DIVMOD needs to return two values, the quotient q and the remainder r. Therefore, DIVMOD pushes q and r into the stack, in that order, so that the quotient is available return values in their place, by convention leaving all deeper values intact. Then the resulting stack, again in its entirety, is returned to the caller. Most TVM primitives behave in this way, and we expect most user-defined functions to be implemented under such conventions. However, TVM provides mechanisms to specify how many arguments must be passed to a called function (cf. [4.1.10](https://zeroheight.com/86757ecb2/p/82565a/t/869715)). When these mechanisms are employed, the specified number of values are moved from the caller’s stack into the (usually initially empty) stack of the called function, while deeper values remain in the caller’s stack and are inaccessible to the callee. The caller can also specify how many return values it expects from the called function. Such argument-checking mechanisms might be useful, for example, for a library function that calls user-provided functions passed as arguments to it.

## 2.3 Efficiency of stack manipulation primitives

### 2.3.5. Absence of circular references. 

One might attempt to create a circular reference between two cells, A and B, as follows: first create A and write some data into it; then create B and write some data into it, along with a reference to previously constructed cell A; finally, add a reference to B into A. While it may seem that after this sequence of operations we obtain a cell A, which refers to B, which in turn refers to A, this is not the case. In fact, we obtain a new cell A0 , which contains a copy of the data originally stored into cell A along with a reference to cell B, which contains a reference to (the original) cell A.

 In this way the transparent copy-on-write mechanism and the “everything is a value” paradigm enable us to create new cells using only previously constructed cells, thus forbidding the appearance of circular references. This property also applies to all other data structures: for instance, the absence of circular references enables TVM to use reference counting to immediately free unused memory instead of relying on garbage collectors. Similarly, this property is crucial for storing data in the TON Blockchain. 

# 3. Cell Memory and Persistent Storage

This chapter briefly describes TVM cells, used to represent all data structures inside the TVM memory and its persistent storage, and the basic operations used to create cells, write (or serialize) data into them, and read (or deserialize) data from them. 

## 3.1 Generalities on Cells

This section presents a classification and general descriptions of cell types. 

### 3.1.1. TVM memory and persistent storage consist of cells.

 Recall that the TVM memory and persistent storage consist of (TVM) cells. Each cell contains up to 1023 bits of data and up to four references to other cells. Circular references are forbidden and cannot be created by means of TVM (cf. [2.3.5](https://zeroheight.com/86757ecb2/p/6198ce/t/3667d8)). In this way, all cells kept in TVM memory and persistent storage constitute a directed acyclic graph (DAG). 

### 3.1.2. Ordinary and exotic cells. 

Apart from the data and references, a cell has a cell type, encoded by an integer −1. . . 255. 

A cell of type −1 is called ordinary; such cells do not require any special processing. Cells of other types are called exotic, and may be loaded—automatically replaced by other cells when an attempt to deserialize them (i.e., to convert them into a Slice by a CTOS instruction) is made. 

They may also exhibit a non-trivial behavior when their hashes are computed. The most common use for exotic cells is to represent some other cells—for instance, cells present in an external library, or pruned from the original tree of cells when a Merkle proof has been created. The type of an exotic cell is stored as the first eight bits of its data. If an exotic cell has less than eight data bits, it is invalid. 

### 3.1.3. The level of a cell. 

Every cell c has another attribute Lvl(c) called its (de Brujn) level, which currently takes integer values in the range 0. . . 3.

The level of an ordinary cell is always equal to the maximum of the levels of all its children ci : 

​                                           `Lvl(c) = max Lvl(ci)` , where 1≤i≤r 

for an ordinary cell c containing r references to cells c1, . . . , cr. If r = 0, Lvl(c) = 0.

 Exotic cells may have different rules for setting their level. 

A cell’s level affects the number of higher hashes it has. More precisely, a level l cell has l higher hashes `Hash1(c), . . . , Hashl(c)` in addition to its representation hash Hash(c) = Hash∞(c). Cells of non-zero level appear inside Merkle proofs and Merkle updates, after some branches of the tree of cells representing a value of an abstract data type are pruned

### 3.1.4 Standard cell representation. 

When a cell needs to be transferred by a network protocol or stored in a disk file, it must be serialized.  The standard representation `CellRepr(c) = CellRepr∞(c)` of a cell c as an `octet (byte)` sequence is constructed as follows: 

\1. Two descriptor bytes `d1` and `d2` are serialized first. Byte` d1` equals `r+8s+32l`, where `0 ≤ r ≤ 4` is the quantity of cell references contained in the cell, `0 ≤ l ≤ 3` is the level of the cell, and `0 ≤ s ≤ 1 `is 1 for exotic cells and 0 for ordinary cells. Byte `d2` equals `bb/8c+db/8e`, where `0 ≤ b ≤ 1023` is the quantity of data bits in c. 

\2. Then the data bits are serialized as `db/8e` 8-bit octets (bytes). If b is not a multiple of eight, a binary 1 and up to six binary 0s are appended to the data bits. After that, the data is split into `db/8e` eight-bit groups, and each group is interpreted as an unsigned big-endian integer` 0 . . . 255` and stored into an octet. 

\3. Finally, each of the r cell references is represented by 32 bytes containing the 256-bit representation hash `Hash(ci)`, explained below in [3.1.5](https://zeroheight.com/86757ecb2/p/836380/t/185d5f), of the cell ci referred to. 

In this way, `2 + db/8e + 32r` bytes of `CellRepr(c)` are obtained. 



### 3.1.5. The representation hash of a cell. 

The 256-bit representation hash or simply hash `Hash(c)` of a cell c is recursively defined as the `sha256` of the standard representation of the cell c:

​                                              `Hash(c) := sha256 :=  CellRepr(c)` 

Notice that cyclic cell references are not allowed and cannot be created by means of the TVM (cf. [2.3.5](https://zeroheight.com/86757ecb2/p/6198ce/t/3667d8)), so this recursion always ends, and the representation hash of any cell is well-defined. 

### 3.1.6. The higher hashes of a cell. 

Recall that a cell c of level l has l higher hashes `Hashi(c)`, `1 ≤ i ≤ l`, as well. Exotic cells have their own rules for computing their higher hashes. Higher hashes `Hashi(c)` of an ordinary cell c are computed similarly to its representation hash, but using the higher hashes `Hashi(cj)` of its children `cj` instead of their representation hashes `Hash(cj)`. By convention, we set `Hash∞(c) := Hash(c)`, and `Hashi(c) := Hash∞(c) = Hash(c)` for all `i > l. 12` 

### 3.1.7. Types of exotic cells.

TVM currently supports the following cell types: 

**Type −1:** Ordinary cell — Contains up to 1023 bits of data and up to four cell references. 

**Type 1:** Pruned branch cell c — May have any level` 1 ≤ l ≤ 3`. It contains exactly 8 + 256l data bits: first an 8-bit integer equal to 1 (representing the cell’s type), then its l higher hashes `Hash1(c), . . . , Hashl(c)`. The level l of a pruned branch cell may be called its de Brujn index, because it determines the outer Merkle proof or Merkle update during the construction of which the branch has been pruned. An attempt to load a pruned branch cell usually leads to an exception. 

**Type 2:** Library reference cell — Always has level 0, and contains 8+256 data bits, including its 8-bit type integer 2 and the representation hash `Hash(c0)` of the library cell being referred to. When loaded, a library reference cell may be transparently replaced by the cell it refers to, if found in the current library context. 

**Type 3:** Merkle proof cell c — Has exactly one reference c1 and level `0 ≤ l ≤ 3,` which must be one less than the level of its only child c1: `Lvl(c) = max(Lvl(c1) − 1, 0)`

The 8 + 256 data bits of a Merkle proof cell contain its 8-bit type integer 3, followed by `Hash1(c1)` (assumed to be equal to `Hash(c1)` if `Lvl(c1) = 0`). The higher hashes `Hashi(c)` of c are computed similarly to the higher hashes of an ordinary cell, but with `Hashi+1(c1)` used instead of `Hashi(c1)`. When loaded, a Merkle proof cell is replaced by c1. 

**Type 4:** Merkle update cell c — Has two children c1 and c2. Its level `0 ≤ l ≤ 3` is given by `Lvl(c) = max(Lvl(c1) − 1`, `Lvl(c2) − 1, 0) (4)` 

A Merkle update behaves like a Merkle proof for both c1 and c2, and contains 8 + 256 + 256 data bits with `Hash1(c1)` and `Hash1(c2)`. However, an extra requirement is that all pruned branch cells c 0 that are descendants of c2 and are bound by c must also be descendants of `c1^13`.When a Merkle update cell is loaded, it is replaced by c2. 



### 3.2 Data manipulation instructions and cells 

The next large group of TVM instructions consists of data manipulation instructions, also known as cell manipulation instructions or simply cell instructions. They correspond to memory access instructions of other architectures. 

### 3.2.1. Classes of cell manipulation instructions. 

The TVM cell instructions are naturally subdivided into two principal classes: • Cell creation instructions or serialization instructions, used to construct new cells from values previously kept in the stack and previously constructed cells. • Cell parsing instructions or deserialization instructions, used to extract data previously stored into cells by cell creation instructions. Additionally, there are exotic cell instructions used to create and inspect exotic cells (cf. [3.1.2](https://zeroheight.com/86757ecb2/p/836380/t/527d7d)), which in particular are used to represent pruned branches of Merkle proofs and Merkle proofs themselves. 

### 3.2.2. Builder and Slice values. 

Cell creation instructions usually work with Builder values, which can be kept only in the stack (cf. [1.1.3](https://zeroheight.com/86757ecb2/p/77e11f/t/033c4d)). 

Such values represent partially constructed cells, for which fast operations for appending bitstrings, integers, other cells, and references to other cells can be defined. Similarly, cell parsing instructions make heavy use of Slice values, which represent either the remainder of a partially parsed cell, or a value (subcell) residing inside such a cell and extracted from it by a parsing instruction. 

# 4. Control Flow, Continuations, and Exceptions

This chapter describes continuations, which may represent execution tokens and exception handlers in TVM. Continuations are deeply involved with the control flow of a TVM program; in particular, subroutine calls and conditional and iterated execution are implemented in TVM using special primitives that accept one or more continuations as their arguments. We conclude this chapter with a discussion of the problem of recursion and of families of mutually recursive functions, exacerbated by the fact that cyclic references are not allowed in TVM data structures (including TVM code). 

## 4.1. Continuations and subroutines

### 4.1.10. Determining the number of arguments passed to and/or return values accepted from a subroutine. 

Similarly to JMPX and RET, CALLX also has special (rarely used) forms, which allow us to explicitly specify the number n 00 of arguments passed from the current stack to the called subroutine (by default, n 00 equals the depth of the current stack, i.e., it is passed in its entirety). Furthermore, a second number n 000 can be specified, used to set nargs of the modified cc continuation before storing it into the new c0; the new nargs equals the depth of the old stack minus n 00 plus n 000 . This means that the caller is willing to pass exactly n 00 arguments to the called subroutine, and is willing to accept exactly n 000 results in their stead. Such forms of CALLX and RET are mostly intended for library functions that accept functional arguments and want to invoke them safely. Another application is related to the “virtualization support” of TVM, which enables TVM code to run other TVM code inside a “virtual TVM machine”. Such virtualization techniques might be useful for implementing sophisticated payment channels in the TON Blockchain (cf. [5](https://zeroheight.com/86757ecb2/p/36f099)]).

# Appendix

## A

### A.1  Gas prices 

The gas price for most primitives equals the basic gas price, computed as `Pb := 10 + b + 5r`, where b is the instruction length in bits and r is the number of cell references included in the instruction. When the gas price of an instruction differs from this basic price, it is indicated in parentheses after its mnemonics, either as (x), meaning that the total gas price equals x, or as (+x), meaning Pb + x. 

Apart from integer constants, the following expressions may appear: 

- Cr — The total price of “reading” cells (i.e., transforming cell references into cell slices). Currently equal to 20 gas units per cell. 
- L — The total price of loading cells. Depends on the loading action required. 
- Bw — The total price of creating new Builders. Currently equal to 100 gas units per builder. 
- Cw — The total price of creating new Cells from Builders). Currently equal to 100 gas units per cell.

### A.4.2. Constant slices, continuations, cells, and references. 

Most of the instructions listed below push literal slices, continuations, cells, and cell references, stored as immediate arguments to the instruction. Therefore, if the immediate argument is absent or too short, an “invalid or too short opcode” exception (code 6) is thrown. 

- 88 — PUSHREF, pushes the first reference of cc.code into the stack as a Cell (and removes this reference from the current continuation). 
- 89 — PUSHREFSLICE, similar to PUSHREF, but converts the cell into a Slice. 
- 8A — PUSHREFCONT, similar to PUSHREFSLICE, but makes a simple ordinary Continuation out of the cell.
- 8Bxsss — PUSHSLICE sss, pushes the (prefix) subslice of cc.code consisting of its first 8x + 4 bits and no references (i.e., essentially a bitstring), where 0 ≤ x ≤ 15. A completion tag is assumed, meaning that all trailing zeroes and the last binary one (if present) are removed from this bitstring. If the original bitstring consists only of zeroes, an empty slice will be pushed. 
- 8B08 — PUSHSLICE x8_, pushes an empty slice (bitstring ‘’). 
- 8B04 — PUSHSLICE x4_, pushes bitstring ‘0’. 
- 8B0C — PUSHSLICE xC_, pushes bitstring ‘1’. 
- 8Crxxssss — PUSHSLICE ssss, pushes the (prefix) subslice of cc.code consisting of its first 1 ≤ r + 1 ≤ 4 references and up to first 8xx + 1 bits of data, with 0 ≤ xx ≤ 31. A completion tag is also assumed. 
- 8C01 is equivalent to PUSHREFSLICE. 
- 8Drxxsssss — PUSHSLICE sssss, pushes the subslice of cc.code consisting of 0 ≤ r ≤ 4 references and up to 8xx + 6 bits of data, with 0 ≤ xx ≤ 127. A completion tag is assumed. 
- 8DE_ — unused (reserved). 
- 8F_rxxcccc — PUSHCONT cccc, where cccc is the simple ordinary continuation made from the first 0 ≤ r ≤ 3 references and the first 0 ≤ xx ≤ 127 bytes of cc.code. 
- 9xccc — PUSHCONT ccc, pushes an x-byte continuation for 0 ≤ x ≤ 15. 



