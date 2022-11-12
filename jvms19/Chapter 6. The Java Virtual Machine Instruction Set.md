
Chapter 6. The Java Virtual Machine Instruction Set
===================================================

A Java Virtual Machine instruction consists of an opcode specifying the operation to be performed, followed by zero or more operands embodying values to be operated upon. This chapter gives details about the format of each Java Virtual Machine instruction and the operation it performs.

6.1. Assumptions: The Meaning of "Must"
---------------------------------------

The description of each instruction is always given in the context of Java Virtual Machine code that satisfies the static and structural constraints of [§4 (_The `class` File Format_)](jvms-4.html "Chapter 4. The class File Format"). In the description of individual Java Virtual Machine instructions, we frequently state that some situation "must" or "must not" be the case: "The _value2_ must be of type `int`." The constraints of [§4 (_The `class` File Format_)](jvms-4.html "Chapter 4. The class File Format") guarantee that all such expectations will in fact be met. If some constraint (a "must" or "must not") in an instruction description is not satisfied at run time, the behavior of the Java Virtual Machine is undefined.

The Java Virtual Machine checks that Java Virtual Machine code satisfies the static and structural constraints at link time using a `class` file verifier ([§4.10](jvms-4.html#jvms-4.10 "4.10. Verification of class Files")). Thus, a Java Virtual Machine will only attempt to execute code from valid `class` files. Performing verification at link time is attractive in that the checks are performed just once, substantially reducing the amount of work that must be done at run time. Other implementation strategies are possible, provided that they comply with _The Java Language Specification, Java SE 19 Edition_ and _The Java Virtual Machine Specification, Java SE 19 Edition_.

6.2. Reserved Opcodes
---------------------

In addition to the opcodes of the instructions specified later in this chapter, which are used in `class` files ([§4 (_The `class` File Format_)](jvms-4.html "Chapter 4. The class File Format")), three opcodes are reserved for internal use by a Java Virtual Machine implementation. If the instruction set of the Java Virtual Machine is extended in the future, these reserved opcodes are guaranteed not to be used.

Two of the reserved opcodes, numbers 254 (0xfe) and 255 (0xff), have the mnemonics _impdep1_ and _impdep2_, respectively. These instructions are intended to provide "back doors" or traps to implementation-specific functionality implemented in software and hardware, respectively. The third reserved opcode, number 202 (0xca), has the mnemonic _breakpoint_ and is intended to be used by debuggers to implement breakpoints.

Although these opcodes have been reserved, they may be used only inside a Java Virtual Machine implementation. They cannot appear in valid `class` files. Tools such as debuggers or JIT code generators ([§2.13](jvms-2.html#jvms-2.13 "2.13. Public Design, Private Implementation")) that might directly interact with Java Virtual Machine code that has been already loaded and executed may encounter these opcodes. Such tools should attempt to behave gracefully if they encounter any of these reserved instructions.

6.3. Virtual Machine Errors
---------------------------

A Java Virtual Machine implementation throws an object that is an instance of a subclass of the class `VirtualMachineError` when an internal error or resource limitation prevents it from implementing the semantics described in this chapter. This specification cannot predict where internal errors or resource limitations may be encountered and does not mandate precisely when they can be reported. Thus, any of the `VirtualMachineError` subclasses defined below may be thrown at any time during the operation of the Java Virtual Machine:

*   `InternalError`: An internal error has occurred in the Java Virtual Machine implementation because of a fault in the software implementing the virtual machine, a fault in the underlying host system software, or a fault in the hardware. This error is delivered asynchronously ([§2.10](jvms-2.html#jvms-2.10 "2.10. Exceptions")) when it is detected and may occur at any point in a program.
    
*   `OutOfMemoryError`: The Java Virtual Machine implementation has run out of either virtual or physical memory, and the automatic storage manager was unable to reclaim enough memory to satisfy an object creation request.
    
*   `StackOverflowError`: The Java Virtual Machine implementation has run out of stack space for a thread, typically because the thread is doing an unbounded number of recursive invocations as a result of a fault in the executing program.
    
*   `UnknownError`: An exception or error has occurred, but the Java Virtual Machine implementation is unable to report the actual exception or error.
    

6.4. Format of Instruction Descriptions
---------------------------------------

Java Virtual Machine instructions are represented in this chapter by entries of the form shown below, in alphabetical order and each beginning on a new page.

### mnemonic

#### Operation

Short description of the instruction

#### Format

  
_mnemonic_  
_operand1_  
_operand2_  
...  

#### Forms

_mnemonic_ = opcode

#### Operand Stack

..., _value1_, _value2_ →

..., _value3_

#### Description

A longer description detailing constraints on operand stack contents or constant pool entries, the operation performed, the type of the results, etc.

#### Linking Exceptions

If any linking exceptions may be thrown by the execution of this instruction, they are set off one to a line, in the order in which they must be thrown.

#### Run-time Exceptions

If any run-time exceptions can be thrown by the execution of an instruction, they are set off one to a line, in the order in which they must be thrown.

Other than the linking and run-time exceptions, if any, listed for an instruction, that instruction must not throw any run-time exceptions except for instances of `VirtualMachineError` or its subclasses.

#### Notes

Comments not strictly part of the specification of an instruction are set aside as notes at the end of the description.

Each cell in the instruction format diagram represents a single 8-bit byte. The instruction's _mnemonic_ is its name. Its opcode is its numeric representation and is given in both decimal and hexadecimal forms. Only the numeric representation is actually present in the Java Virtual Machine code in a `class` file.

Keep in mind that there are "operands" generated at compile time and embedded within Java Virtual Machine instructions, as well as "operands" calculated at run time and supplied on the operand stack. Although they are supplied from several different areas, all these operands represent the same thing: values to be operated upon by the Java Virtual Machine instruction being executed. By implicitly taking many of its operands from its operand stack, rather than representing them explicitly in its compiled code as additional operand bytes, register numbers, etc., the Java Virtual Machine's code stays compact.

Some instructions are presented as members of a family of related instructions sharing a single description, format, and operand stack diagram. As such, a family of instructions includes several opcodes and opcode mnemonics; only the family mnemonic appears in the instruction format diagram, and a separate forms line lists all member mnemonics and opcodes. For example, the Forms line for the _lconst\_<l>_ family of instructions, giving mnemonic and opcode information for the two instructions in that family (_lconst\_0_ and _lconst\_1_), is

_lconst\_0_ = 9 (0x9)

_lconst\_1_ = 10 (0xa)

In the description of the Java Virtual Machine instructions, the effect of an instruction's execution on the operand stack ([§2.6.2](jvms-2.html#jvms-2.6.2 "2.6.2. Operand Stacks")) of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) is represented textually, with the stack growing from left to right and each value represented separately. Thus,

..., _value1_, _value2_ →

..., _result_

shows an operation that begins by having _value2_ on top of the operand stack with _value1_ just beneath it. As a result of the execution of the instruction, _value1_ and _value2_ are popped from the operand stack and replaced by _result_ value, which has been calculated by the instruction. The remainder of the operand stack, represented by an ellipsis (...), is unaffected by the instruction's execution.

Values of types `long` and `double` are represented by a single entry on the operand stack.

In the First Edition of _The Java® Virtual Machine Specification_, values on the operand stack of types `long` and `double` were each represented in the stack diagram by two entries.

6.5. Instructions
-----------------

### _aaload_

#### Operation

Load `reference` from array

#### Format

  
_aaload_  

#### Forms

_aaload_ = 50 (0x32)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `reference`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `reference` _value_ in the component of the array at _index_ is retrieved and pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _aaload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _aaload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _aastore_

#### Operation

Store into `reference` array

#### Format

  
_aastore_  

#### Forms

_aastore_ = 83 (0x53)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `reference`. The _index_ must be of type `int`, and _value_ must be of type `reference`. The _arrayref_, _index_, and _value_ are popped from the operand stack.

If _value_ is `null`, then _value_ is stored as the component of the array at _index_.

Otherwise, _value_ is non-`null`. If the type of _value_ is assignment compatible with the type of the components of the array referenced by _arrayref_, then _value_ is stored as the component of the array at _index_.

The following rules are used to determine whether a _value_ that is not `null` is assignment compatible with the array component type. If S is the type of the object referred to by _value_, and T is the reference type of the array components, then _aastore_ determines whether assignment is compatible as follows:

*   If S is a class type, then:
    
    *   If T is a class type, then S must be the same class as T, or S must be a subclass of T;
        
    *   If T is an interface type, then S must implement interface T.
        
    
*   If S is an array type SC`[]`, that is, an array of components of type SC, then:
    
    *   If T is a class type, then T must be `Object`.
        
    *   If T is an interface type, then T must be one of the interfaces implemented by arrays (JLS §4.10.3).
        
    *   If T is an array type TC`[]`, that is, an array of components of type TC, then one of the following must be true:
        
        *   TC and SC are the same primitive type.
            
        *   TC and SC are reference types, and type SC is assignable to TC by these run-time rules.
            
        
    

#### Run-time Exceptions

If _arrayref_ is `null`, _aastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _aastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

Otherwise, if _arrayref_ is not `null` and the actual type of the non-`null` _value_ is not assignment compatible with the actual type of the components of the array, _aastore_ throws an `ArrayStoreException`.

### _aconst\_null_

#### Operation

Push `null`

#### Format

  
_aconst\_null_  

#### Forms

_aconst\_null_ = 1 (0x1)

#### Operand Stack

... →

..., `null`

#### Description

Push the `null` object `reference` onto the operand stack.

#### Notes

The Java Virtual Machine does not mandate a concrete value for `null`.

### _aload_

#### Operation

Load `reference` from local variable

#### Format

  
_aload_  
_index_  

#### Forms

_aload_ = 25 (0x19)

#### Operand Stack

... →

..., _objectref_

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at _index_ must contain a `reference`. The _objectref_ in the local variable at _index_ is pushed onto the operand stack.

#### Notes

The _aload_ instruction cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the _astore_ instruction ([§_astore_](jvms-6.html#jvms-6.5.astore "astore")) is intentional.

The _aload_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _aload\_<n>_

#### Operation

Load `reference` from local variable

#### Format

  
_aload\_<n>_  

#### Forms

_aload\_0_ = 42 (0x2a)

_aload\_1_ = 43 (0x2b)

_aload\_2_ = 44 (0x2c)

_aload\_3_ = 45 (0x2d)

#### Operand Stack

... →

..., _objectref_

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at <_n_\> must contain a `reference`. The _objectref_ in the local variable at <_n_\> is pushed onto the operand stack.

#### Notes

An _aload\_<n>_ instruction cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the corresponding _astore\_<n>_ instruction ([§_astore\_<n>_](jvms-6.html#jvms-6.5.astore_n "astore_<n>")) is intentional.

Each of the _aload\_<n>_ instructions is the same as _aload_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _anewarray_

#### Operation

Create new array of `reference`

#### Format

  
_anewarray_  
_indexbyte1_  
_indexbyte2_  

#### Forms

anewarray = 189 (0xbd)

#### Operand Stack

..., _count_ →

..., _arrayref_

#### Description

The _count_ must be of type `int`. It is popped off the operand stack. The _count_ represents the number of components of the array to be created. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a class, array, or interface type. The named class, array, or interface type is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")). A new array with components of that type, of length _count_, is allocated from the garbage-collected heap, and a `reference` _arrayref_ to this new array object is pushed onto the operand stack. All components of the new array are initialized to `null`, the default value for `reference` types ([§2.4](jvms-2.html#jvms-2.4 "2.4. Reference Types and Values")).

#### Linking Exceptions

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution") can be thrown.

#### Run-time Exceptions

Otherwise, if _count_ is less than zero, the _anewarray_ instruction throws a `NegativeArraySizeException`.

#### Notes

The _anewarray_ instruction is used to create a single dimension of an array of object references or part of a multidimensional array.

### _areturn_

#### Operation

Return `reference` from method

#### Format

  
_areturn_  

#### Forms

_areturn_ = 176 (0xb0)

#### Operand Stack

..., _objectref_ →

\[empty\]

#### Description

The _objectref_ must be of type `reference` and must refer to an object of a type that is assignment compatible (JLS §5.2) with the type represented by the return descriptor ([§4.3.3](jvms-4.html#jvms-4.3.3 "4.3.3. Method Descriptors")) of the current method. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, _objectref_ is popped from the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then reinstates the frame of the invoker and returns control to the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _areturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _areturn_ throws an `IllegalMonitorStateException`.

### _arraylength_

#### Operation

Get length of array

#### Format

  
_arraylength_  

#### Forms

_arraylength_ = 190 (0xbe)

#### Operand Stack

..., _arrayref_ →

..., _length_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array. It is popped from the operand stack. The _length_ of the array it references is determined. That _length_ is pushed onto the operand stack as an `int`.

#### Run-time Exceptions

If the _arrayref_ is `null`, the _arraylength_ instruction throws a `NullPointerException`.

### _astore_

#### Operation

Store `reference` into local variable

#### Format

  
_astore_  
_index_  

#### Forms

_astore_ = 58 (0x3a)

#### Operand Stack

..., _objectref_ →

...

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _objectref_ on the top of the operand stack must be of type `returnAddress` or of type `reference`. It is popped from the operand stack, and the value of the local variable at _index_ is set to _objectref_.

#### Notes

The _astore_ instruction is used with an _objectref_ of type `returnAddress` when implementing the `finally` clause of the Java programming language ([§3.13](jvms-3.html#jvms-3.13 "3.13. Compiling finally")).

The _aload_ instruction ([§_aload_](jvms-6.html#jvms-6.5.aload "aload")) cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the _astore_ instruction is intentional.

The _astore_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _astore\_<n>_

#### Operation

Store `reference` into local variable

#### Format

  
_astore\_<n>_  

#### Forms

_astore\_0_ = 75 (0x4b)

_astore\_1_ = 76 (0x4c)

_astore\_2_ = 77 (0x4d)

_astore\_3_ = 78 (0x4e)

#### Operand Stack

..., _objectref_ →

...

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _objectref_ on the top of the operand stack must be of type `returnAddress` or of type `reference`. It is popped from the operand stack, and the value of the local variable at <_n_\> is set to _objectref_.

#### Notes

An _astore\_<n>_ instruction is used with an _objectref_ of type `returnAddress` when implementing the `finally` clauses of the Java programming language ([§3.13](jvms-3.html#jvms-3.13 "3.13. Compiling finally")).

An _aload\_<n>_ instruction ([§_aload\_<n>_](jvms-6.html#jvms-6.5.aload_n "aload_<n>")) cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the corresponding _astore\_<n>_ instruction is intentional.

Each of the _astore\_<n>_ instructions is the same as _astore_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _athrow_

#### Operation

Throw exception or error

#### Format

  
_athrow_  

#### Forms

_athrow_ = 191 (0xbf)

#### Operand Stack

..., _objectref_ →

_objectref_

#### Description

The _objectref_ must be of type `reference` and must refer to an object that is an instance of class `Throwable` or of a subclass of `Throwable`. It is popped from the operand stack. The _objectref_ is then thrown by searching the current method ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) for the first exception handler that matches the class of _objectref_, as given by the algorithm in [§2.10](jvms-2.html#jvms-2.10 "2.10. Exceptions").

If an exception handler that matches _objectref_ is found, it contains the location of the code intended to handle this exception. The `pc` register is reset to that location, the operand stack of the current frame is cleared, _objectref_ is pushed back onto the operand stack, and execution continues.

If no matching exception handler is found in the current frame, that frame is popped. If the current frame represents an invocation of a `synchronized` method, the monitor entered or reentered on invocation of the method is exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")). Finally, the frame of its invoker is reinstated, if such a frame exists, and the _objectref_ is rethrown. If no such frame exists, the current thread exits.

#### Run-time Exceptions

If _objectref_ is `null`, _athrow_ throws a `NullPointerException` instead of _objectref_.

Otherwise, if the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the method of the current frame is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _athrow_ throws an `IllegalMonitorStateException` instead of the object previously being thrown. This can happen, for example, if an abruptly completing `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _athrow_ throws an `IllegalMonitorStateException` instead of the object previously being thrown.

#### Notes

The operand stack diagram for the _athrow_ instruction may be misleading: If a handler for this exception is matched in the current method, the _athrow_ instruction discards all the values on the operand stack, then pushes the thrown object onto the operand stack. However, if no handler is matched in the current method and the exception is thrown farther up the method invocation chain, then the operand stack of the method (if any) that handles the exception is cleared and _objectref_ is pushed onto that empty operand stack. All intervening frames from the method that threw the exception up to, but not including, the method that handles the exception are discarded.

### _baload_

#### Operation

Load `byte` or `boolean` from array

#### Format

  
_baload_  

#### Forms

_baload_ = 51 (0x33)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `byte` or of type `boolean`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `byte` _value_ in the component of the array at _index_ is retrieved, sign-extended to an `int` _value_, and pushed onto the top of the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _baload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _baload_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### Notes

The _baload_ instruction is used to load values from both `byte` and `boolean` arrays. In Oracle's Java Virtual Machine implementation, `boolean` arrays - that is, arrays of type `T_BOOLEAN` ([§2.2](jvms-2.html#jvms-2.2 "2.2. Data Types"), [§_newarray_](jvms-6.html#jvms-6.5.newarray "newarray")) - are implemented as arrays of 8-bit values. Other implementations may implement packed `boolean` arrays; the _baload_ instruction of such implementations must be used to access those arrays.

### _bastore_

#### Operation

Store into `byte` or `boolean` array

#### Format

  
_bastore_  

#### Forms

_bastore_ = 84 (0x54)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `byte` or of type `boolean`. The _index_ and the _value_ must both be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack.

If the _arrayref_ refers to an array whose components are of type `byte`, then the `int` _value_ is truncated to a `byte` and stored as the component of the array indexed by _index_.

If the _arrayref_ refers to an array whose components are of type `boolean`, then the `int` _value_ is narrowed by taking the bitwise AND of _value_ and 1; the result is stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _bastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _bastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### Notes

The _bastore_ instruction is used to store values into both `byte` and `boolean` arrays. In Oracle's Java Virtual Machine implementation, `boolean` arrays - that is, arrays of type `T_BOOLEAN` ([§2.2](jvms-2.html#jvms-2.2 "2.2. Data Types"), [§_newarray_](jvms-6.html#jvms-6.5.newarray "newarray")) - are implemented as arrays of 8-bit values. Other implementations may implement packed `boolean` arrays; in such implementations the _bastore_ instruction must be able to store `boolean` values into packed `boolean` arrays as well as `byte` values into `byte` arrays.

### _bipush_

#### Operation

Push `byte`

#### Format

  
_bipush_  
_byte_  

#### Forms

_bipush_ = 16 (0x10)

#### Operand Stack

... →

..., _value_

#### Description

The immediate _byte_ is sign-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

### _caload_

#### Operation

Load `char` from array

#### Format

  
_caload_  

#### Forms

_caload_ = 52 (0x34)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `char`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The component of the array at _index_ is retrieved and zero-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _caload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _caload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _castore_

#### Operation

Store into `char` array

#### Format

  
_castore_  

#### Forms

_castore_ = 85 (0x55)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `char`. The _index_ and the _value_ must both be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `int` _value_ is truncated to a `char` and stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _castore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _castore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _checkcast_

#### Operation

Check whether object is of given type

#### Format

  
_checkcast_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_checkcast_ = 192 (0xc0)

#### Operand Stack

..., _objectref_ →

..., _objectref_

#### Description

The _objectref_ must be of type `reference`. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a class, array, or interface type.

If _objectref_ is `null`, then the operand stack is unchanged.

Otherwise, the named class, array, or interface type is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")). If _objectref_ can be cast to the resolved class, array, or interface type, the operand stack is unchanged; otherwise, the _checkcast_ instruction throws a `ClassCastException`.

The following rules are used to determine whether an _objectref_ that is not `null` can be cast to the resolved type. If S is the type of the object referred to by _objectref_, and T is the resolved class, array, or interface type, then _checkcast_ determines whether _objectref_ can be cast to type T as follows:

*   If S is a class type, then:
    
    *   If T is a class type, then S must be the same class as T, or S must be a subclass of T;
        
    *   If T is an interface type, then S must implement interface T.
        
    
*   If S is an array type SC`[]`, that is, an array of components of type SC, then:
    
    *   If T is a class type, then T must be `Object`.
        
    *   If T is an interface type, then T must be one of the interfaces implemented by arrays (JLS §4.10.3).
        
    *   If T is an array type TC`[]`, that is, an array of components of type TC, then one of the following must be true:
        
        *   TC and SC are the same primitive type.
            
        *   TC and SC are reference types, and type SC can be cast to TC by recursive application of these rules.
            
        
    

#### Linking Exceptions

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution") can be thrown.

#### Run-time Exception

Otherwise, if _objectref_ cannot be cast to the resolved class, array, or interface type, the _checkcast_ instruction throws a `ClassCastException`.

#### Notes

The _checkcast_ instruction is very similar to the _instanceof_ instruction ([§_instanceof_](jvms-6.html#jvms-6.5.instanceof "instanceof")). It differs in its treatment of `null`, its behavior when its test fails (_checkcast_ throws an exception, _instanceof_ pushes a result code), and its effect on the operand stack.

### _d2f_

#### Operation

Convert `double` to `float`

#### Format

  
_d2f_  

#### Forms

_d2f_ = 144 (0x90)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and converted to a `float` _result_ using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The _result_ is pushed onto the operand stack.

A finite _value_ too small to be represented as a `float` is converted to a zero of the same sign; a finite _value_ too large to be represented as a `float` is converted to an infinity of the same sign. A `double` NaN is converted to a `float` NaN.

#### Notes

The _d2f_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_ and may also lose precision.

### _d2i_

#### Operation

Convert `double` to `int`

#### Format

  
_d2i_  

#### Forms

_d2i_ = 142 (0x8e)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and converted to an `int` _result_. The _result_ is pushed onto the operand stack:

*   If the _value_ is NaN, the result of the conversion is an `int` 0.
    
*   Otherwise, if the _value_ is not an infinity, it is rounded to an integer value V using the round toward zero rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If this integer value V can be represented as an `int`, then the result is the `int` value V.
    
*   Otherwise, either the _value_ must be too small (a negative value of large magnitude or negative infinity), and the result is the smallest representable value of type `int`, or the _value_ must be too large (a positive value of large magnitude or positive infinity), and the result is the largest representable value of type `int`.
    

#### Notes

The _d2i_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_ and may also lose precision.

### _d2l_

#### Operation

Convert `double` to `long`

#### Format

  
_d2l_  

#### Forms

_d2l_ = 143 (0x8f)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and converted to a `long`. The _result_ is pushed onto the operand stack:

*   If the _value_ is NaN, the result of the conversion is a `long` 0.
    
*   Otherwise, if the _value_ is not an infinity, it is rounded to an integer value V using the round toward zero rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If this integer value V can be represented as a `long`, then the result is the `long` value V.
    
*   Otherwise, either the _value_ must be too small (a negative value of large magnitude or negative infinity), and the result is the smallest representable value of type `long`, or the _value_ must be too large (a positive value of large magnitude or positive infinity), and the result is the largest representable value of type `long`.
    

#### Notes

The _d2l_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_ and may also lose precision.

### _dadd_

#### Operation

Add `double`

#### Format

  
_dadd_  

#### Forms

_dadd_ = 99 (0x63)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack. The `double` _result_ is _value1_ + _value2_. The _result_ is pushed onto the operand stack.

The result of a _dadd_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   The sum of two infinities of opposite sign is NaN.
    
*   The sum of two infinities of the same sign is the infinity of that sign.
    
*   The sum of an infinity and any finite value is equal to the infinity.
    
*   The sum of two zeroes of opposite sign is positive zero.
    
*   The sum of two zeroes of the same sign is the zero of that sign.
    
*   The sum of a zero and a nonzero finite value is equal to the nonzero value.
    
*   The sum of two nonzero finite values of the same magnitude and opposite sign is positive zero.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN and the values have the same sign or have different magnitudes, the sum is computed and rounded to the nearest representable value using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dadd_ instruction never throws a run-time exception.

### _daload_

#### Operation

Load `double` from array

#### Format

  
_daload_  

#### Forms

_daload_ = 49 (0x31)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `double`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `double` _value_ in the component of the array at _index_ is retrieved and pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _daload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _daload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _dastore_

#### Operation

Store into `double` array

#### Format

  
_dastore_  

#### Forms

_dastore_ = 82 (0x52)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `double`. The _index_ must be of type `int`, and value must be of type `double`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `double` _value_ is stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _dastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _dastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _dcmp<op>_

#### Operation

Compare `double`

#### Format

  
_dcmp<op>_  

#### Forms

_dcmpg_ = 152 (0x98)

_dcmpl_ = 151 (0x97)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and a floating-point comparison is performed:

*   If _value1_ is greater than _value2_, the `int` value 1 is pushed onto the operand stack.
    
*   Otherwise, if _value1_ is equal to _value2_, the `int` value 0 is pushed onto the operand stack.
    
*   Otherwise, if _value1_ is less than _value2_, the `int` value -1 is pushed onto the operand stack.
    
*   Otherwise, at least one of _value1_ or _value2_ is NaN. The _dcmpg_ instruction pushes the `int` value 1 onto the operand stack and the _dcmpl_ instruction pushes the `int` value -1 onto the operand stack.
    

Floating-point comparison is performed in accordance with IEEE 754. All values other than NaN are ordered, with negative infinity less than all finite values and positive infinity greater than all finite values. Positive zero and negative zero are considered equal.

#### Notes

The _dcmpg_ and _dcmpl_ instructions differ only in their treatment of a comparison involving NaN. NaN is unordered, so any `double` comparison fails if either or both of its operands are NaN. With both _dcmpg_ and _dcmpl_ available, any `double` comparison may be compiled to push the same _result_ onto the operand stack whether the comparison fails on non-NaN values or fails because it encountered a NaN. For more information, see [§3.5](jvms-3.html#jvms-3.5 "3.5. More Control Examples").

### _dconst\_<d>_

#### Operation

Push `double`

#### Format

  
_dconst\_<d>_  

#### Forms

_dconst\_0_ = 14 (0xe)

_dconst\_1_ = 15 (0xf)

#### Operand Stack

... →

..., <_d_\>

#### Description

Push the `double` constant <_d_\> (0.0 or 1.0) onto the operand stack.

### _ddiv_

#### Operation

Divide `double`

#### Format

  
_ddiv_  

#### Forms

_ddiv_ = 111 (0x6f)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack. The `double` _result_ is _value1_ / _value2_. The _result_ is pushed onto the operand stack.

The result of a _ddiv_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result is positive if both values have the same sign, negative if the values have different signs.
    
*   Division of an infinity by an infinity results in NaN.
    
*   Division of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
    
*   Division of a finite value by an infinity results in a signed zero, with the sign-producing rule just given.
    
*   Division of a zero by a zero results in NaN; division of zero by any other finite value results in a signed zero, with the sign-producing rule just given.
    
*   Division of a nonzero finite value by a zero results in a signed infinity, with the sign-producing rule just given.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN, the quotient is computed and rounded to the nearest `double` using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, division by zero, or loss of precision may occur, execution of a _ddiv_ instruction never throws a run-time exception.

### _dload_

#### Operation

Load `double` from local variable

#### Format

  
_dload_  
_index_  

#### Forms

_dload_ = 24 (0x18)

#### Operand Stack

... →

..., _value_

#### Description

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at _index_ must contain a `double`. The _value_ of the local variable at _index_ is pushed onto the operand stack.

#### Notes

The _dload_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _dload\_<n>_

#### Operation

Load `double` from local variable

#### Format

  
_dload\_<n>_  

#### Forms

_dload\_0_ = 38 (0x26)

_dload\_1_ = 39 (0x27)

_dload\_2_ = 40 (0x28)

_dload\_3_ = 41 (0x29)

#### Operand Stack

... →

..., _value_

#### Description

Both <_n_\> and <_n_\>+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at <_n_\> must contain a `double`. The _value_ of the local variable at <_n_\> is pushed onto the operand stack.

#### Notes

Each of the _dload\_<n>_ instructions is the same as _dload_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _dmul_

#### Operation

Multiply `double`

#### Format

  
_dmul_  

#### Forms

_dmul_ = 107 (0x6b)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack. The `double` _result_ is _value1_ \* _value2_. The _result_ is pushed onto the operand stack.

The result of a _dmul_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result is positive if both values have the same sign and negative if the values have different signs.
    
*   Multiplication of an infinity by a zero results in NaN.
    
*   Multiplication of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
    
*   In the remaining cases, where neither an infinity nor NaN is involved, the product is computed and rounded to the nearest representable value using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dmul_ instruction never throws a run-time exception.

### _dneg_

#### Operation

Negate `double`

#### Format

  
_dneg_  

#### Forms

_dneg_ = 119 (0x77)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The value must be of type `double`. It is popped from the operand stack. The `double` _result_ is the arithmetic negation of _value_. The _result_ is pushed onto the operand stack.

For `double` values, negation is not the same as subtraction from zero. If `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`. Unary minus merely inverts the sign of a `double`.

Special cases of interest:

*   If the operand is NaN, the result is NaN (recall that NaN has no sign).
    
    The Java Virtual Machine has not adopted the stronger requirement from the 2019 version of the IEEE 754 Standard that negation inverts the sign bit for all inputs, including NaN.
    
*   If the operand is an infinity, the result is the infinity of opposite sign.
    
*   If the operand is a zero, the result is the zero of opposite sign.
    

### _drem_

#### Operation

Remainder `double`

#### Format

  
_drem_  

#### Forms

_drem_ = 115 (0x73)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack. The `double` _result_ is calculated and pushed onto the operand stack.

The result of a _drem_ instruction is not the same as the result of the remainder operation defined by IEEE 754, due to the choice of rounding policy in the Java Virtual Machine ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The IEEE 754 remainder operation computes the remainder from a rounding division, not a truncating division, and so its behavior is _not_ analogous to that of the usual integer remainder operator. Instead, the Java Virtual Machine defines _drem_ to behave in a manner analogous to that of the integer remainder instructions _irem_ and _lrem_, with an implied division using the round toward zero rounding policy; this may be compared with the C library function `fmod`.

The result of a _drem_ instruction is governed by the following rules, which match IEEE 754 arithmetic except for how the implied division is computed:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result equals the sign of the dividend.
    
*   If the dividend is an infinity or the divisor is a zero or both, the result is NaN.
    
*   If the dividend is finite and the divisor is an infinity, the result equals the dividend.
    
*   If the dividend is a zero and the divisor is finite, the result equals the dividend.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN, the floating-point remainder _result_ from a dividend _value1_ and a divisor _value2_ is defined by the mathematical relation _result_ = _value1_ - (_value2_ \* _q_), where _q_ is an integer that is negative only if _value1_ / _value2_ is negative, and positive only if _value1_ / _value2_ is positive, and whose magnitude is as large as possible without exceeding the magnitude of the true mathematical quotient of _value1_ and _value2_.
    

Despite the fact that division by zero may occur, evaluation of a _drem_ instruction never throws a run-time exception. Overflow, underflow, or loss of precision cannot occur.

#### Notes

The IEEE 754 remainder operation may be computed by the library routine `Math.IEEEremainder` or `StrictMath.IEEEremainder`.

### _dreturn_

#### Operation

Return `double` from method

#### Format

  
_dreturn_  

#### Forms

_dreturn_ = 175 (0xaf)

#### Operand Stack

..., _value_ →

\[empty\]

#### Description

The current method must have return type `double`. The _value_ must be of type `double`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, _value_ is popped from the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _dreturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _dreturn_ throws an `IllegalMonitorStateException`.

### _dstore_

#### Operation

Store `double` into local variable

#### Format

  
_dstore_  
_index_  

#### Forms

_dstore_ = 57 (0x39)

#### Operand Stack

..., _value_ →

...

#### Description

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack. The local variables at _index_ and _index_+1 are set to _value_.

#### Notes

The _dstore_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _dstore\_<n>_

#### Operation

Store `double` into local variable

#### Format

  
_dstore\_<n>_  

#### Forms

_dstore\_0_ = 71 (0x47)

_dstore\_1_ = 72 (0x48)

_dstore\_2_ = 73 (0x49)

_dstore\_3_ = 74 (0x4a)

#### Operand Stack

..., _value_ →

...

#### Description

Both <_n_\> and <_n_\>+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack. The local variables at <_n_\> and <_n_\>+1 are set to _value_.

#### Notes

Each of the _dstore\_<n>_ instructions is the same as _dstore_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _dsub_

#### Operation

Subtract `double`

#### Format

  
_dsub_  

#### Forms

_dsub_ = 103 (0x67)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack. The `double` _result_ is _value1_ - _value2_. The _result_ is pushed onto the operand stack.

For `double` subtraction, it is always the case that `a-b` produces the same result as `a+(-b)`. However, for the _dsub_ instruction, subtraction from zero is not the same as negation, because if `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`.

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dsub_ instruction never throws a run-time exception.

### _dup_

#### Operation

Duplicate the top operand stack value

#### Format

  
_dup_  

#### Forms

_dup_ = 89 (0x59)

#### Operand Stack

..., _value_ →

..., _value_, _value_

#### Description

Duplicate the top value on the operand stack and push the duplicated value onto the operand stack.

The _dup_ instruction must not be used unless _value_ is a value of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

### _dup\_x1_

#### Operation

Duplicate the top operand stack value and insert two values down

#### Format

  
_dup\_x1_  

#### Forms

_dup\_x1_ = 90 (0x5a)

#### Operand Stack

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

#### Description

Duplicate the top value on the operand stack and insert the duplicated value two values down in the operand stack.

The _dup\_x1_ instruction must not be used unless both _value1_ and _value2_ are values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

### _dup\_x2_

#### Operation

Duplicate the top operand stack value and insert two or three values down

#### Format

  
_dup\_x2_  

#### Forms

_dup\_x2_ = 91 (0x5b)

#### Operand Stack

Form 1:

..., _value3_, _value2_, _value1_ →

..., _value1_, _value3_, _value2_, _value1_

where _value1_, _value2_, and _value3_ are all values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 2:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ is a value of a category 1 computational type and _value2_ is a value of a category 2 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Description

Duplicate the top value on the operand stack and insert the duplicated value two or three values down in the operand stack.

### _dup2_

#### Operation

Duplicate the top one or two operand stack values

#### Format

  
_dup2_  

#### Forms

_dup2_ = 92 (0x5c)

#### Operand Stack

Form 1:

..., _value2_, _value1_ →

..., _value2_, _value1_, _value2_, _value1_

where both _value1_ and _value2_ are values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 2:

..., _value_ →

..., _value_, _value_

where _value_ is a value of a category 2 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Description

Duplicate the top one or two values on the operand stack and push the duplicated value or values back onto the operand stack in the original order.

### _dup2\_x1_

#### Operation

Duplicate the top one or two operand stack values and insert two or three values down

#### Format

  
_dup2\_x1_  

#### Forms

_dup2\_x1_ = 93 (0x5d)

#### Operand Stack

Form 1:

..., _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value3_, _value2_, _value1_

where _value1_, _value2_, and _value3_ are all values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 2:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ is a value of a category 2 computational type and _value2_ is a value of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Description

Duplicate the top one or two values on the operand stack and insert the duplicated values, in the original order, one value beneath the original value or values in the operand stack.

### _dup2\_x2_

#### Operation

Duplicate the top one or two operand stack values and insert two, three, or four values down

#### Format

  
_dup2\_x2_  

#### Forms

_dup2\_x2_ = 94 (0x5e)

#### Operand Stack

Form 1:

..., _value4_, _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value4_, _value3_, _value2_, _value1_

where _value1_, _value2_, _value3_, and _value4_ are all values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 2:

..., _value3_, _value2_, _value1_ →

..., _value1_, _value3_, _value2_, _value1_

where _value1_ is a value of a category 2 computational type and _value2_ and _value3_ are both values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 3:

..., _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value3_, _value2_, _value1_

where _value1_ and _value2_ are both values of a category 1 computational type and _value3_ is a value of a category 2 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 4:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ and _value2_ are both values of a category 2 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Description

Duplicate the top one or two values on the operand stack and insert the duplicated values, in the original order, into the operand stack.

### _f2d_

#### Operation

Convert `float` to `double`

#### Format

  
_f2d_  

#### Forms

_f2d_ = 141 (0x8d)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `float`. It is popped from the operand stack and converted to a `double` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _f2d_ instruction performs a widening primitive conversion (JLS §5.1.2).

### _f2i_

#### Operation

Convert `float` to `int`

#### Format

  
_f2i_  

#### Forms

_f2i_ = 139 (0x8b)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `float`. It is popped from the operand stack and converted to an `int` _result_. The _result_ is pushed onto the operand stack:

*   If the _value_ is NaN, the _result_ of the conversion is an `int` 0.
    
*   Otherwise, if the _value_ is not an infinity, it is rounded to an integer value V using the round toward zero rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If this integer value V can be represented as an `int`, then the _result_ is the `int` value V.
    
*   Otherwise, either the _value_ must be too small (a negative value of large magnitude or negative infinity), and the _result_ is the smallest representable value of type `int`, or the _value_ must be too large (a positive value of large magnitude or positive infinity), and the _result_ is the largest representable value of type `int`.
    

#### Notes

The _f2i_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_ and may also lose precision.

### _f2l_

#### Operation

Convert `float` to `long`

#### Format

  
_f2l_  

#### Forms

_f2l_ = 140 (0x8c)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `float`. It is popped from the operand stack and converted to a `long` _result_. The _result_ is pushed onto the operand stack:

*   If the _value_ is NaN, the result of the conversion is a `long` 0.
    
*   Otherwise, if the _value_ is not an infinity, it is rounded to an integer value V using the round toward zero rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If this integer value V can be represented as a `long`, then the _result_ is the `long` value V.
    
*   Otherwise, either the _value_ must be too small (a negative value of large magnitude or negative infinity), and the _result_ is the smallest representable value of type `long`, or the _value_ must be too large (a positive value of large magnitude or positive infinity), and the _result_ is the largest representable value of type `long`.
    

#### Notes

The _f2l_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_ and may also lose precision.

### _fadd_

#### Operation

Add `float`

#### Format

  
_fadd_  

#### Forms

_fadd_ = 98 (0x62)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack. The `float` _result_ is _value1_ + _value2_. The _result_ is pushed onto the operand stack.

The result of an _fadd_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   The sum of two infinities of opposite sign is NaN.
    
*   The sum of two infinities of the same sign is the infinity of that sign.
    
*   The sum of an infinity and any finite value is equal to the infinity.
    
*   The sum of two zeroes of opposite sign is positive zero.
    
*   The sum of two zeroes of the same sign is the zero of that sign.
    
*   The sum of a zero and a nonzero finite value is equal to the nonzero value.
    
*   The sum of two nonzero finite values of the same magnitude and opposite sign is positive zero.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN and the values have the same sign or have different magnitudes, the sum is computed and rounded to the nearest representable value using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `float`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `float`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of an _fadd_ instruction never throws a run-time exception.

### _faload_

#### Operation

Load `float` from array

#### Format

  
_faload_  

#### Forms

_faload_ = 48 (0x30)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `float`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `float` value in the component of the array at _index_ is retrieved and pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _faload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _faload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _fastore_

#### Operation

Store into `float` array

#### Format

  
_fastore_  

#### Forms

_fastore_ = 81 (0x51)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `float`. The _index_ must be of type `int`, and the _value_ must be of type `float`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `float` _value_ is stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _fastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _fastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _fcmp<op>_

#### Operation

Compare `float`

#### Format

  
_fcmp<op>_  

#### Forms

_fcmpg_ = 150 (0x96)

_fcmpl_ = 149 (0x95)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack and a floating-point comparison is performed:

*   If _value1_ is greater than _value2_, the `int` value 1 is pushed onto the operand stack.
    
*   Otherwise, if _value1_ is equal to _value2_, the `int` value 0 is pushed onto the operand stack.
    
*   Otherwise, if _value1_ is less than _value2_, the `int` value -1 is pushed onto the operand stack.
    
*   Otherwise, at least one of _value1_ or _value2_ is NaN. The _fcmpg_ instruction pushes the `int` value 1 onto the operand stack and the _fcmpl_ instruction pushes the `int` value -1 onto the operand stack.
    

Floating-point comparison is performed in accordance with IEEE 754. All values other than NaN are ordered, with negative infinity less than all finite values and positive infinity greater than all finite values. Positive zero and negative zero are considered equal.

#### Notes

The _fcmpg_ and _fcmpl_ instructions differ only in their treatment of a comparison involving NaN. NaN is unordered, so any `float` comparison fails if either or both of its operands are NaN. With both _fcmpg_ and _fcmpl_ available, any `float` comparison may be compiled to push the same _result_ onto the operand stack whether the comparison fails on non-NaN values or fails because it encountered a NaN. For more information, see [§3.5](jvms-3.html#jvms-3.5 "3.5. More Control Examples").

### _fconst\_<f>_

#### Operation

Push `float`

#### Format

  
_fconst\_<f>_  

#### Forms

_fconst\_0_ = 11 (0xb)

_fconst\_1_ = 12 (0xc)

_fconst\_2_ = 13 (0xd)

#### Operand Stack

... →

..., <_f_\>

#### Description

Push the `float` constant <_f_\> (0.0, 1.0, or 2.0) onto the operand stack.

### _fdiv_

#### Operation

Divide `float`

#### Format

  
_fdiv_  

#### Forms

_fdiv_ = 110 (0x6e)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack. The `float` _result_ is _value1_ / _value2_. The _result_ is pushed onto the operand stack.

The result of an _fdiv_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result is positive if both values have the same sign, negative if the values have different signs.
    
*   Division of an infinity by an infinity results in NaN.
    
*   Division of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
    
*   Division of a finite value by an infinity results in a signed zero, with the sign-producing rule just given.
    
*   Division of a zero by a zero results in NaN; division of zero by any other finite value results in a signed zero, with the sign-producing rule just given.
    
*   Division of a nonzero finite value by a zero results in a signed infinity, with the sign-producing rule just given.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN, the quotient is computed and rounded to the nearest `float` using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `float`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `float`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, division by zero, or loss of precision may occur, execution of an _fdiv_ instruction never throws a run-time exception.

### _fload_

#### Operation

Load `float` from local variable

#### Format

  
_fload_  
_index_  

#### Forms

_fload_ = 23 (0x17)

#### Operand Stack

... →

..., _value_

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at _index_ must contain a `float`. The _value_ of the local variable at _index_ is pushed onto the operand stack.

#### Notes

The _fload_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _fload\_<n>_

#### Operation

Load `float` from local variable

#### Format

  
_fload\_<n>_  

#### Forms

_fload\_0_ = 34 (0x22)

_fload\_1_ = 35 (0x23)

_fload\_2_ = 36 (0x24)

_fload\_3_ = 37 (0x25)

#### Operand Stack

... →

..., _value_

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at <_n_\> must contain a `float`. The _value_ of the local variable at <_n_\> is pushed onto the operand stack.

#### Notes

Each of the _fload\_<n>_ instructions is the same as _fload_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _fmul_

#### Operation

Multiply `float`

#### Format

  
_fmul_  

#### Forms

_fmul_ = 106 (0x6a)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack. The `float` _result_ is _value1_ \* _value2_. The _result_ is pushed onto the operand stack.

The result of an _fmul_ instruction is governed by the rules of IEEE 754 arithmetic:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result is positive if both values have the same sign, and negative if the values have different signs.
    
*   Multiplication of an infinity by a zero results in NaN.
    
*   Multiplication of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
    
*   In the remaining cases, where neither an infinity nor NaN is involved, the product is computed and rounded to the nearest representable value using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). If the magnitude is too large to represent as a `float`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `float`, we say the operation underflows; the result is then a zero of appropriate sign.
    

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of an _fmul_ instruction never throws a run-time exception.

### _fneg_

#### Operation

Negate `float`

#### Format

  
_fneg_  

#### Forms

_fneg_ = 118 (0x76)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ must be of type `float`. It is popped from the operand stack. The `float` _result_ is the arithmetic negation of _value_. The _result_ is pushed onto the operand stack.

For `float` values, negation is not the same as subtraction from zero. If `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`. Unary minus merely inverts the sign of a `float`.

Special cases of interest:

*   If the operand is NaN, the result is NaN (recall that NaN has no sign).
    
    The Java Virtual Machine has not adopted the stronger requirement from the 2019 version of the IEEE 754 Standard that negation inverts the sign bit for all inputs, including NaN.
    
*   If the operand is an infinity, the result is the infinity of opposite sign.
    
*   If the operand is a zero, the result is the zero of opposite sign.
    

### _frem_

#### Operation

Remainder `float`

#### Format

  
_frem_  

#### Forms

_frem_ = 114 (0x72)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack. The `float` _result_ is calculated and pushed onto the operand stack.

The result of an _frem_ instruction is not the same as the result of the remainder operation defined by IEEE 754, due to the choice of rounding policy in the Java Virtual Machine ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The IEEE 754 remainder operation computes the remainder from a rounding division, not a truncating division, and so its behavior is _not_ analogous to that of the usual integer remainder operator. Instead, the Java Virtual Machine defines _frem_ to behave in a manner analogous to that of the integer remainder instructions _irem_ and _lrem_, with an implied division using the round toward zero rounding policy; this may be compared with the C library function `fmod`.

The result of an _frem_ instruction is governed by the following rules, which match IEEE 754 arithmetic except for how the implied division is computed:

*   If either _value1_ or _value2_ is NaN, the result is NaN.
    
*   If neither _value1_ nor _value2_ is NaN, the sign of the result equals the sign of the dividend.
    
*   If the dividend is an infinity or the divisor is a zero or both, the result is NaN.
    
*   If the dividend is finite and the divisor is an infinity, the result equals the dividend.
    
*   If the dividend is a zero and the divisor is finite, the result equals the dividend.
    
*   In the remaining cases, where neither operand is an infinity, a zero, or NaN, the floating-point remainder _result_ from a dividend _value1_ and a divisor _value2_ is defined by the mathematical relation _result_ = _value1_ - (_value2_ \* _q_), where _q_ is an integer that is negative only if _value1_ / _value2_ is negative, and positive only if _value1_ / _value2_ is positive, and whose magnitude is as large as possible without exceeding the magnitude of the true mathematical quotient of _value1_ and _value2_.
    

Despite the fact that division by zero may occur, evaluation of an _frem_ instruction never throws a run-time exception. Overflow, underflow, or loss of precision cannot occur.

#### Notes

The IEEE 754 remainder operation may be computed by the library routine `Math.IEEEremainder` or `StrictMath.IEEEremainder`.

### _freturn_

#### Operation

Return `float` from method

#### Format

  
_freturn_  

#### Forms

_freturn_ = 174 (0xae)

#### Operand Stack

..., _value_ →

\[empty\]

#### Description

The current method must have return type `float`. The _value_ must be of type `float`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, _value_ is popped from the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _freturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _freturn_ throws an `IllegalMonitorStateException`.

### _fstore_

#### Operation

Store `float` into local variable

#### Format

  
_fstore_  
_index_  

#### Forms

_fstore_ = 56 (0x38)

#### Operand Stack

..., _value_ →

...

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `float`. It is popped from the operand stack, and the value of the local variable at _index_ is set to _value_.

#### Notes

The _fstore_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _fstore\_<n>_

#### Operation

Store `float` into local variable

#### Format

  
_fstore\_<n>_  

#### Forms

_fstore\_0_ = 67 (0x43)

_fstore\_1_ = 68 (0x44)

_fstore\_2_ = 69 (0x45)

_fstore\_3_ = 70 (0x46)

#### Operand Stack

..., _value_ →

...

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `float`. It is popped from the operand stack, and the value of the local variable at <_n_\> is set to _value_.

#### Notes

Each of the _fstore\_<n>_ instructions is the same as _fstore_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _fsub_

#### Operation

Subtract `float`

#### Format

  
_fsub_  

#### Forms

_fsub_ = 102 (0x66)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `float`. The values are popped from the operand stack. The `float` _result_ is _value1_ - _value2_. The _result_ is pushed onto the operand stack.

For `float` subtraction, it is always the case that `a-b` produces the same result as `a+(-b)`. However, for the _fsub_ instruction, subtraction from zero is not the same as negation, because if `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`.

The Java Virtual Machine requires support of gradual underflow. Despite the fact that overflow, underflow, or loss of precision may occur, execution of an _fsub_ instruction never throws a run-time exception.

### _getfield_

#### Operation

Fetch field from object

#### Format

  
_getfield_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_getfield_ = 180 (0xb4)

#### Operand Stack

..., _objectref_ →

..., _value_

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a field ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor of the field as well as a symbolic reference to the class in which the field is to be found. The referenced field is resolved ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")).

The _objectref_, which must be of type `reference` but not an array type, is popped from the operand stack. The _value_ of the referenced field in _objectref_ is fetched and pushed onto the operand stack.

#### Linking Exceptions

During resolution of the symbolic reference to the field, any of the errors pertaining to field resolution ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")) can be thrown.

Otherwise, if the resolved field is a `static` field, _getfield_ throws an `IncompatibleClassChangeError`.

#### Run-time Exception

Otherwise, if _objectref_ is `null`, the _getfield_ instruction throws a `NullPointerException`.

#### Notes

The _getfield_ instruction cannot be used to access the `length` field of an array. The _arraylength_ instruction ([§_arraylength_](jvms-6.html#jvms-6.5.arraylength "arraylength")) is used instead.

### _getstatic_

#### Operation

Get `static` field from class

#### Format

  
_getstatic_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_getstatic_ = 178 (0xb2)

#### Operand Stack

..., →

..., _value_

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a field ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor of the field as well as a symbolic reference to the class or interface in which the field is to be found. The referenced field is resolved ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")).

On successful resolution of the field, the class or interface that declared the resolved field is initialized if that class or interface has not already been initialized ([§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization")).

The _value_ of the class or interface field is fetched and pushed onto the operand stack.

#### Linking Exceptions

During resolution of the symbolic reference to the class or interface field, any of the exceptions pertaining to field resolution ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")) can be thrown.

Otherwise, if the resolved field is not a `static` (class) field or an interface field, _getstatic_ throws an `IncompatibleClassChangeError`.

#### Run-time Exception

Otherwise, if execution of this _getstatic_ instruction causes initialization of the referenced class or interface, _getstatic_ may throw an `Error` as detailed in [§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization").

### _goto_

#### Operation

Branch always

#### Format

  
_goto_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_goto_ = 167 (0xa7)

#### Operand Stack

No change

#### Description

The unsigned bytes _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit _branchoffset_, where _branchoffset_ is (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution proceeds at that offset from the address of the opcode of this _goto_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _goto_ instruction.

### _goto\_w_

#### Operation

Branch always (wide index)

#### Format

  
_goto\_w_  
_branchbyte1_  
_branchbyte2_  
_branchbyte3_  
_branchbyte4_  

#### Forms

_goto\_w_ = 200 (0xc8)

#### Operand Stack

No change

#### Description

The unsigned bytes _branchbyte1_, _branchbyte2_, _branchbyte3_, and _branchbyte4_ are used to construct a signed 32-bit _branchoffset_, where _branchoffset_ is (_branchbyte1_ `<<` 24) | (_branchbyte2_ `<<` 16) | (_branchbyte3_ `<<` 8) | _branchbyte4_. Execution proceeds at that offset from the address of the opcode of this _goto\_w_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _goto\_w_ instruction.

#### Notes

Although the _goto\_w_ instruction takes a 4-byte branch offset, other factors limit the size of a method to 65535 bytes ([§4.11](jvms-4.html#jvms-4.11 "4.11. Limitations of the Java Virtual Machine")). This limit may be raised in a future release of the Java Virtual Machine.

### _i2b_

#### Operation

Convert `int` to `byte`

#### Format

  
_i2b_  

#### Forms

_i2b_ = 145 (0x91)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack, truncated to a `byte`, then sign-extended to an `int` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _i2b_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_. The _result_ may also not have the same sign as _value_.

### _i2c_

#### Operation

Convert `int` to `char`

#### Format

  
_i2c_  

#### Forms

_i2c_ = 146 (0x92)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack, truncated to `char`, then zero-extended to an `int` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _i2c_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_. The _result_ (which is always positive) may also not have the same sign as _value_.

### _i2d_

#### Operation

Convert `int` to `double`

#### Format

  
_i2d_  

#### Forms

_i2d_ = 135 (0x87)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack and converted to a `double` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _i2d_ instruction performs a widening primitive conversion (JLS §5.1.2). Because all values of type `int` are exactly representable by type `double`, the conversion is exact.

### _i2f_

#### Operation

Convert `int` to `float`

#### Format

  
_i2f_  

#### Forms

_i2f_ = 134 (0x86)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack and converted to a `float` _result_ using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The _result_ is pushed onto the operand stack.

#### Notes

The _i2f_ instruction performs a widening primitive conversion (JLS §5.1.2), but may result in a loss of precision because values of type `float` have only 24 significand bits.

### _i2l_

#### Operation

Convert `int` to `long`

#### Format

  
_i2l_  

#### Forms

_i2l_ = 133 (0x85)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack and sign-extended to a `long` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _i2l_ instruction performs a widening primitive conversion (JLS §5.1.2). Because all values of type `int` are exactly representable by type `long`, the conversion is exact.

### _i2s_

#### Operation

Convert `int` to `short`

#### Format

  
_i2s_  

#### Forms

_i2s_ = 147 (0x93)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack, truncated to a `short`, then sign-extended to an `int` _result_. The _result_ is pushed onto the operand stack.

#### Notes

The _i2s_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_. The _result_ may also not have the same sign as _value_.

### _iadd_

#### Operation

Add `int`

#### Format

  
_iadd_  

#### Forms

_iadd_ = 96 (0x60)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. The `int` _result_ is _value1_ + _value2_. The _result_ is pushed onto the operand stack.

The result is the 32 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `int`. If overflow occurs, then the sign of the result may not be the same as the sign of the mathematical sum of the two values.

Despite the fact that overflow may occur, execution of an _iadd_ instruction never throws a run-time exception.

### _iaload_

#### Operation

Load `int` from array

#### Format

  
_iaload_  

#### Forms

_iaload_ = 46 (0x2e)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `int`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `int` _value_ in the component of the array at _index_ is retrieved and pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _iaload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _iaload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _iand_

#### Operation

Boolean AND `int`

#### Format

  
_iand_  

#### Forms

_iand_ = 126 (0x7e)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. They are popped from the operand stack. An `int` _result_ is calculated by taking the bitwise AND (conjunction) of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _iastore_

#### Operation

Store into `int` array

#### Format

  
_iastore_  

#### Forms

_iastore_ = 79 (0x4f)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `int`. Both _index_ and _value_ must be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `int` _value_ is stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _iastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _iastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _iconst\_<i>_

#### Operation

Push `int` constant

#### Format

  
_iconst\_<i>_  

#### Forms

_iconst\_m1_ = 2 (0x2)

_iconst\_0_ = 3 (0x3)

_iconst\_1_ = 4 (0x4)

_iconst\_2_ = 5 (0x5)

_iconst\_3_ = 6 (0x6)

_iconst\_4_ = 7 (0x7)

_iconst\_5_ = 8 (0x8)

#### Operand Stack

... →

..., <_i_\>

#### Description

Push the `int` constant <_i_\> (-1, 0, 1, 2, 3, 4 or 5) onto the operand stack.

#### Notes

Each of this family of instructions is equivalent to _bipush_ <_i_\> for the respective value of <_i_\>, except that the operand <_i_\> is implicit.

### _idiv_

#### Operation

Divide `int`

#### Format

  
_idiv_  

#### Forms

_idiv_ = 108 (0x6c)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. The `int` _result_ is the value of the Java programming language expression _value1_ / _value2_ (JLS §15.17.2). The _result_ is pushed onto the operand stack.

An `int` division rounds towards 0; that is, the quotient produced for `int` values in _n_/_d_ is an `int` value _q_ whose magnitude is as large as possible while satisfying |_d_ ⋅ _q_| ≤ |_n_|. Moreover, _q_ is positive when |_n_| ≥ |_d_| and _n_ and _d_ have the same sign, but _q_ is negative when |_n_| ≥ |_d_| and _n_ and _d_ have opposite signs.

There is one special case that does not satisfy this rule: if the dividend is the negative integer of largest possible magnitude for the `int` type, and the divisor is -1, then overflow occurs, and the result is equal to the dividend. Despite the overflow, no exception is thrown in this case.

#### Run-time Exception

If the value of the divisor in an `int` division is 0, _idiv_ throws an `ArithmeticException`.

### _if\_acmp<cond>_

#### Operation

Branch if `reference` comparison succeeds

#### Format

  
_if\_acmp<cond>_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_if\_acmpeq_ = 165 (0xa5)

_if\_acmpne_ = 166 (0xa6)

#### Operand Stack

..., _value1_, _value2_ →

...

#### Description

Both _value1_ and _value2_ must be of type `reference`. They are both popped from the operand stack and compared. The results of the comparison are as follows:

*   _if\_acmpeq_ succeeds if and only if _value1_ = _value2_
    
*   _if\_acmpne_ succeeds if and only if _value1_ ≠ _value2_
    

If the comparison succeeds, the unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is calculated to be (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution then proceeds at that offset from the address of the opcode of this _if\_acmp<cond>_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _if\_acmp<cond>_ instruction.

Otherwise, if the comparison fails, execution proceeds at the address of the instruction following this _if\_acmp<cond>_ instruction.

### _if\_icmp<cond>_

#### Operation

Branch if `int` comparison succeeds

#### Format

  
_if\_icmp<cond>_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_if\_icmpeq_ = 159 (0x9f)

_if\_icmpne_ = 160 (0xa0)

_if\_icmplt_ = 161 (0xa1)

_if\_icmpge_ = 162 (0xa2)

_if\_icmpgt_ = 163 (0xa3)

_if\_icmple_ = 164 (0xa4)

#### Operand Stack

..., _value1_, _value2_ →

...

#### Description

Both _value1_ and _value2_ must be of type `int`. They are both popped from the operand stack and compared. All comparisons are signed. The results of the comparison are as follows:

*   _if\_icmpeq_ succeeds if and only if _value1_ = _value2_
    
*   _if\_icmpne_ succeeds if and only if _value1_ ≠ _value2_
    
*   _if\_icmplt_ succeeds if and only if _value1_ < _value2_
    
*   _if\_icmple_ succeeds if and only if _value1_ ≤ _value2_
    
*   _if\_icmpgt_ succeeds if and only if _value1_ > _value2_
    
*   _if\_icmpge_ succeeds if and only if _value1_ ≥ _value2_
    

If the comparison succeeds, the unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is calculated to be (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution then proceeds at that offset from the address of the opcode of this _if\_icmp<cond>_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _if\_icmp<cond>_ instruction.

Otherwise, execution proceeds at the address of the instruction following this _if\_icmp<cond>_ instruction.

### _if<cond>_

#### Operation

Branch if `int` comparison with zero succeeds

#### Format

  
_if<cond>_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_ifeq_ = 153 (0x99)

_ifne_ = 154 (0x9a)

_iflt_ = 155 (0x9b)

_ifge_ = 156 (0x9c)

_ifgt_ = 157 (0x9d)

_ifle_ = 158 (0x9e)

#### Operand Stack

..., _value_ →

...

#### Description

The _value_ must be of type `int`. It is popped from the operand stack and compared against zero. All comparisons are signed. The results of the comparisons are as follows:

*   _ifeq_ succeeds if and only if _value_ = 0
    
*   _ifne_ succeeds if and only if _value_ ≠ 0
    
*   _iflt_ succeeds if and only if _value_ < 0
    
*   _ifle_ succeeds if and only if _value_ ≤ 0
    
*   _ifgt_ succeeds if and only if _value_ > 0
    
*   _ifge_ succeeds if and only if _value_ ≥ 0
    

If the comparison succeeds, the unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is calculated to be (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution then proceeds at that offset from the address of the opcode of this _if<cond>_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _if<cond>_ instruction.

Otherwise, execution proceeds at the address of the instruction following this _if<cond>_ instruction.

### _ifnonnull_

#### Operation

Branch if `reference` not `null`

#### Format

  
_ifnonnull_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_ifnonnull_ = 199 (0xc7)

#### Operand Stack

..., _value_ →

...

#### Description

The _value_ must be of type `reference`. It is popped from the operand stack. If _value_ is not `null`, the unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is calculated to be (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution then proceeds at that offset from the address of the opcode of this _ifnonnull_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _ifnonnull_ instruction.

Otherwise, execution proceeds at the address of the instruction following this _ifnonnull_ instruction.

### _ifnull_

#### Operation

Branch if `reference` is `null`

#### Format

  
_ifnull_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_ifnull_ = 198 (0xc6)

#### Operand Stack

..., _value_ →

...

#### Description

The _value_ must of type `reference`. It is popped from the operand stack. If _value_ is `null`, the unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is calculated to be (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution then proceeds at that offset from the address of the opcode of this _ifnull_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _ifnull_ instruction.

Otherwise, execution proceeds at the address of the instruction following this _ifnull_ instruction.

### _iinc_

#### Operation

Increment local variable by constant

#### Format

  
_iinc_  
_index_  
_const_  

#### Forms

_iinc_ = 132 (0x84)

#### Operand Stack

No change

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _const_ is an immediate signed byte. The local variable at _index_ must contain an `int`. The value _const_ is first sign-extended to an `int`, and then the local variable at _index_ is incremented by that amount.

#### Notes

The _iinc_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index and to increment it by a two-byte immediate signed value.

### _iload_

#### Operation

Load `int` from local variable

#### Format

  
_iload_  
_index_  

#### Forms

_iload_ = 21 (0x15)

#### Operand Stack

... →

..., _value_

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at _index_ must contain an `int`. The _value_ of the local variable at _index_ is pushed onto the operand stack.

#### Notes

The _iload_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _iload\_<n>_

#### Operation

Load `int` from local variable

#### Format

  
_iload\_<n>_  

#### Forms

_iload\_0_ = 26 (0x1a)

_iload\_1_ = 27 (0x1b)

_iload\_2_ = 28 (0x1c)

_iload\_3_ = 29 (0x1d)

#### Operand Stack

... →

..., _value_

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at <_n_\> must contain an `int`. The _value_ of the local variable at <_n_\> is pushed onto the operand stack.

#### Notes

Each of the _iload\_<n>_ instructions is the same as _iload_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _imul_

#### Operation

Multiply `int`

#### Format

  
_imul_  

#### Forms

_imul_ = 104 (0x68)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. The `int` _result_ is _value1_ \* _value2_. The _result_ is pushed onto the operand stack.

The result is the 32 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `int`. If overflow occurs, then the sign of the result may not be the same as the sign of the mathematical multiplication of the two values.

Despite the fact that overflow may occur, execution of an _imul_ instruction never throws a run-time exception.

### _ineg_

#### Operation

Negate `int`

#### Format

  
_ineg_  

#### Forms

_ineg_ = 116 (0x74)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ must be of type `int`. It is popped from the operand stack. The `int` _result_ is the arithmetic negation of _value_, -_value_. The _result_ is pushed onto the operand stack.

For `int` values, negation is the same as subtraction from zero. Because the Java Virtual Machine uses two's-complement representation for integers and the range of two's-complement values is not symmetric, the negation of the maximum negative `int` results in that same maximum negative number. Despite the fact that overflow has occurred, no exception is thrown.

For all `int` values `x`, `-x` equals `(~x)+1`.

### _instanceof_

#### Operation

Determine if object is of given type

#### Format

  
_instanceof_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_instanceof_ = 193 (0xc1)

#### Operand Stack

..., _objectref_ →

..., _result_

#### Description

The _objectref_, which must be of type `reference`, is popped from the operand stack. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a class, array, or interface type.

If _objectref_ is `null`, the _instanceof_ instruction pushes an `int` _result_ of 0 as an `int` onto the operand stack.

Otherwise, the named class, array, or interface type is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")). If _objectref_ is an instance of the resolved class or array type, or implements the resolved interface, the _instanceof_ instruction pushes an `int` _result_ of 1 as an `int` onto the operand stack; otherwise, it pushes an `int` _result_ of 0.

The following rules are used to determine whether an _objectref_ that is not `null` is an instance of the resolved type. If S is the type of the object referred to by _objectref_, and T is the resolved class, array, or interface type, then _instanceof_ determines whether _objectref_ is an instance of T as follows:

*   If S is a class type, then:
    
    *   If T is a class type, then S must be the same class as T, or S must be a subclass of T;
        
    *   If T is an interface type, then S must implement interface T.
        
    
*   If S is an array type SC`[]`, that is, an array of components of type SC, then:
    
    *   If T is a class type, then T must be `Object`.
        
    *   If T is an interface type, then T must be one of the interfaces implemented by arrays (JLS §4.10.3).
        
    *   If T is an array type TC`[]`, that is, an array of components of type TC, then one of the following must be true:
        
        *   TC and SC are the same primitive type.
            
        *   TC and SC are reference types, and type SC can be cast to TC by these run-time rules.
            
        
    

#### Linking Exceptions

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution") can be thrown.

#### Notes

The _instanceof_ instruction is very similar to the _checkcast_ instruction ([§_checkcast_](jvms-6.html#jvms-6.5.checkcast "checkcast")). It differs in its treatment of `null`, its behavior when its test fails (_checkcast_ throws an exception, _instanceof_ pushes a result code), and its effect on the operand stack.

### _invokedynamic_

#### Operation

Invoke a dynamically-computed call site

#### Format

  
_invokedynamic_  
_indexbyte1_  
_indexbyte2_  
_0_  
_0_  

#### Forms

_invokedynamic_ = 186 (0xba)

#### Operand Stack

..., \[_arg1_, \[_arg2_ ...\]\] →

...

#### Description

First, the unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a dynamically-computed call site ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")). The values of the third and fourth operand bytes must always be zero.

The symbolic reference is resolved ([§5.4.3.6](jvms-5.html#jvms-5.4.3.6 "5.4.3.6. Dynamically-Computed Constant and Call Site Resolution")) _for this specific_ _invokedynamic_ _instruction_ to obtain a `reference` to an instance of `java.lang.invoke.CallSite`. The instance of `java.lang.invoke.CallSite` is considered "bound" to this specific _invokedynamic_ instruction.

The instance of `java.lang.invoke.CallSite` indicates a _target method handle_. The _nargs_ argument values are popped from the operand stack, and the target method handle is invoked. The invocation occurs as if by execution of an _invokevirtual_ instruction that indicates a run-time constant pool index to a symbolic reference R where:

*   R is a symbolic reference to a method of a class;
    
*   for the symbolic reference to the class in which the method is to be found, R specifies `java.lang.invoke.MethodHandle`;
    
*   for the name of the method, R specifies `invokeExact`;
    
*   for the descriptor of the method, R specifies the method descriptor in the dynamically-computed call site.
    

and where it is as if the following items were pushed, in order, onto the operand stack:

*   a `reference` to the target method handle;
    
*   the _nargs_ argument values, where the number, type, and order of the values must be consistent with the method descriptor in the dynamically-computed call site.
    

#### Linking Exceptions

During resolution of the symbolic reference to a dynamically-computed call site, any of the exceptions pertaining to dynamically-computed call site resolution can be thrown.

#### Notes

If the symbolic reference to the dynamically-computed call site can be resolved, it implies that a non-`null` `reference` to an instance of `java.lang.invoke.CallSite` is bound to the _invokedynamic_ instruction. Therefore, the target method handle, indicated by the instance of `java.lang.invoke.CallSite`, is non-`null`.

Similarly, successful resolution implies that the method descriptor in the symbolic reference is semantically equal to the type descriptor of the target method handle.

Together, these invariants mean that an _invokedynamic_ instruction which is bound to an instance of `java.lang.invoke.CallSite` never throws a `NullPointerException` or a `java.lang.invoke.WrongMethodTypeException`.

### _invokeinterface_

#### Operation

Invoke interface method

#### Format

  
_invokeinterface_  
_indexbyte1_  
_indexbyte2_  
_count_  
_0_  

#### Forms

_invokeinterface_ = 185 (0xb9)

#### Operand Stack

..., _objectref_, \[_arg1_, \[_arg2_ ...\]\] →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to an interface method ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor ([§4.3.3](jvms-4.html#jvms-4.3.3 "4.3.3. Method Descriptors")) of the interface method as well as a symbolic reference to the interface in which the interface method is to be found. The named interface method is resolved ([§5.4.3.4](jvms-5.html#jvms-5.4.3.4 "5.4.3.4. Interface Method Resolution")).

The resolved interface method must not be an instance initialization method, or the class or interface initialization method ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods"), [§2.9.2](jvms-2.html#jvms-2.9.2 "2.9.2. Class Initialization Methods")).

The _count_ operand is an unsigned byte that must not be zero. The _objectref_ must be of type `reference` and must be followed on the operand stack by _nargs_ argument values, where the number, type, and order of the values must be consistent with the descriptor of the resolved interface method. The value of the fourth operand byte must always be zero.

Let C be the class of _objectref_. A method is selected with respect to C and the resolved method ([§5.4.6](jvms-5.html#jvms-5.4.6 "5.4.6. Method Selection")). This is the _method to be invoked_.

If the method to be invoked is `synchronized`, the monitor associated with _objectref_ is entered or reentered as if by execution of a _monitorenter_ instruction ([§_monitorenter_](jvms-6.html#jvms-6.5.monitorenter "monitorenter")) in the current thread.

If the method to be invoked is not `native`, the _nargs_ argument values and _objectref_ are popped from the operand stack. A new frame is created on the Java Virtual Machine stack for the method being invoked. The _objectref_ and the argument values are consecutively made the values of local variables of the new frame, with _objectref_ in local variable 0, _arg1_ in local variable 1 (or, if _arg1_ is of type `long` or `double`, in local variables 1 and 2), and so on. The new frame is then made current, and the Java Virtual Machine `pc` is set to the opcode of the first instruction of the method to be invoked. Execution continues with the first instruction of the method.

If the method to be invoked is `native` and the platform-dependent code that implements it has not yet been bound ([§5.6](jvms-5.html#jvms-5.6 "5.6. Binding Native Method Implementations")) into the Java Virtual Machine, then that is done. The _nargs_ argument values and _objectref_ are popped from the operand stack and are passed as parameters to the code that implements the method. The parameters are passed and the code is invoked in an implementation-dependent manner. When the platform-dependent code returns:

*   If the `native` method is `synchronized`, the monitor associated with _objectref_ is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread.
    
*   If the `native` method returns a value, the return value of the platform-dependent code is converted in an implementation-dependent way to the return type of the `native` method and pushed onto the operand stack.
    

#### Linking Exceptions

During resolution of the symbolic reference to the interface method, any of the exceptions pertaining to interface method resolution ([§5.4.3.4](jvms-5.html#jvms-5.4.3.4 "5.4.3.4. Interface Method Resolution")) can be thrown.

Otherwise, if the resolved method is `static`, the _invokeinterface_ instruction throws an `IncompatibleClassChangeError`.

Note that _invokeinterface_ may refer to `private` methods declared in interfaces, including nestmate interfaces.

#### Run-time Exceptions

Otherwise, if _objectref_ is `null`, the _invokeinterface_ instruction throws a `NullPointerException`.

Otherwise, if the class of _objectref_ does not implement the resolved interface, _invokeinterface_ throws an `IncompatibleClassChangeError`.

Otherwise, if the selected method is neither `public` nor `private`, _invokeinterface_ throws an `IllegalAccessError`.

Otherwise, if the selected method is `abstract`, _invokeinterface_ throws an `AbstractMethodError`.

Otherwise, if the selected method is `native` and the code that implements the method cannot be bound, _invokeinterface_ throws an `UnsatisfiedLinkError`.

Otherwise, if no method is selected, and there are multiple maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokeinterface_ throws an `IncompatibleClassChangeError`

Otherwise, if no method is selected, and there are no maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokeinterface_ throws an `AbstractMethodError`.

#### Notes

The _count_ operand of the _invokeinterface_ instruction records a measure of the number of argument values, where an argument value of type `long` or type `double` contributes two units to the _count_ value and an argument of any other type contributes one unit. This information can also be derived from the descriptor of the selected method. The redundancy is historical.

The fourth operand byte exists to reserve space for an additional operand used in certain of Oracle's Java Virtual Machine implementations, which replace the _invokeinterface_ instruction by a specialized pseudo-instruction at run time. It must be retained for backwards compatibility.

The _nargs_ argument values and _objectref_ are not one-to-one with the first _nargs_+1 local variables. Argument values of types `long` and `double` must be stored in two consecutive local variables, thus more than _nargs_ local variables may be required to pass _nargs_ argument values to the invoked method.

The selection logic allows a non-`abstract` method declared in a superinterface to be selected. Methods in interfaces are only considered if there is no matching method in the class hierarchy. In the event that there are two non-`abstract` methods in the superinterface hierarchy, with neither more specific than the other, an error occurs; there is no attempt to disambiguate (for example, one may be the referenced method and one may be unrelated, but we do not prefer the referenced method). On the other hand, if there are many `abstract` methods but only one non-`abstract` method, the non-`abstract` method is selected (unless an `abstract` method is more specific).

### _invokespecial_

#### Operation

Invoke instance method; direct invocation of instance initialization methods and methods of the current class and its supertypes

#### Format

  
_invokespecial_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_invokespecial_ = 183 (0xb7)

#### Operand Stack

..., _objectref_, \[_arg1_, \[_arg2_ ...\]\] →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a method or an interface method ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor ([§4.3.3](jvms-4.html#jvms-4.3.3 "4.3.3. Method Descriptors")) of the method or interface method as well as a symbolic reference to the class or interface in which the method or interface method is to be found. The named method is resolved ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution"), [§5.4.3.4](jvms-5.html#jvms-5.4.3.4 "5.4.3.4. Interface Method Resolution")).

If all of the following are true, let C be the direct superclass of the current class:

*   The resolved method is not an instance initialization method ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods")).
    
*   The symbolic reference names a class (not an interface), and that class is a superclass of the current class.
    
*   The `ACC_SUPER` flag is set for the `class` file ([§4.1](jvms-4.html#jvms-4.1 "4.1. The ClassFile Structure")).
    

Otherwise, let C be the class or interface named by the symbolic reference.

The actual method to be invoked is selected by the following lookup procedure:

1.  If C contains a declaration for an instance method with the same name and descriptor as the resolved method, then it is the method to be invoked.
    
2.  Otherwise, if C is a class and has a superclass, a search for a declaration of an instance method with the same name and descriptor as the resolved method is performed, starting with the direct superclass of C and continuing with the direct superclass of that class, and so forth, until a match is found or no further superclasses exist. If a match is found, then it is the method to be invoked.
    
3.  Otherwise, if C is an interface and the class `Object` contains a declaration of a `public` instance method with the same name and descriptor as the resolved method, then it is the method to be invoked.
    
4.  Otherwise, if there is exactly one maximally-specific method ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution")) in the superinterfaces of C that matches the resolved method's name and descriptor and is not `abstract`, then it is the method to be invoked.
    

The _objectref_ must be of type `reference` and must be followed on the operand stack by _nargs_ argument values, where the number, type, and order of the values must be consistent with the descriptor of the selected instance method.

If the method is `synchronized`, the monitor associated with _objectref_ is entered or reentered as if by execution of a _monitorenter_ instruction ([§_monitorenter_](jvms-6.html#jvms-6.5.monitorenter "monitorenter")) in the current thread.

If the method is not `native`, the _nargs_ argument values and _objectref_ are popped from the operand stack. A new frame is created on the Java Virtual Machine stack for the method being invoked. The _objectref_ and the argument values are consecutively made the values of local variables of the new frame, with _objectref_ in local variable 0, _arg1_ in local variable 1 (or, if _arg1_ is of type `long` or `double`, in local variables 1 and 2), and so on. The new frame is then made current, and the Java Virtual Machine `pc` is set to the opcode of the first instruction of the method to be invoked. Execution continues with the first instruction of the method.

If the method is `native` and the platform-dependent code that implements it has not yet been bound ([§5.6](jvms-5.html#jvms-5.6 "5.6. Binding Native Method Implementations")) into the Java Virtual Machine, that is done. The _nargs_ argument values and _objectref_ are popped from the operand stack and are passed as parameters to the code that implements the method. The parameters are passed and the code is invoked in an implementation-dependent manner. When the platform-dependent code returns, the following take place:

*   If the `native` method is `synchronized`, the monitor associated with _objectref_ is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread.
    
*   If the `native` method returns a value, the return value of the platform-dependent code is converted in an implementation-dependent way to the return type of the `native` method and pushed onto the operand stack.
    

#### Linking Exceptions

During resolution of the symbolic reference to the method, any of the exceptions pertaining to method resolution ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution")) can be thrown.

Otherwise, if the resolved method is an instance initialization method, and the class in which it is declared is not the class symbolically referenced by the instruction, a `NoSuchMethodError` is thrown.

Otherwise, if the resolved method is a class (`static`) method, the _invokespecial_ instruction throws an `IncompatibleClassChangeError`.

#### Run-time Exceptions

Otherwise, if _objectref_ is `null`, the _invokespecial_ instruction throws a `NullPointerException`.

Otherwise, if step 1, step 2, or step 3 of the lookup procedure selects an `abstract` method, _invokespecial_ throws an `AbstractMethodError`.

Otherwise, if step 1, step 2, or step 3 of the lookup procedure selects a `native` method and the code that implements the method cannot be bound, _invokespecial_ throws an `UnsatisfiedLinkError`.

Otherwise, if step 4 of the lookup procedure determines there are multiple maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokespecial_ throws an `IncompatibleClassChangeError`

Otherwise, if step 4 of the lookup procedure determines there are no maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokespecial_ throws an `AbstractMethodError`.

#### Notes

The difference between the _invokespecial_ instruction and the _invokevirtual_ instruction ([§_invokevirtual_](jvms-6.html#jvms-6.5.invokevirtual "invokevirtual")) is that _invokevirtual_ invokes a method based on the class of the object. The _invokespecial_ instruction is used to directly invoke instance initialization methods ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods")) as well as methods of the current class and its supertypes.

The _invokespecial_ instruction was named `invokenonvirtual` prior to JDK release 1.0.2.

The _nargs_ argument values and _objectref_ are not one-to-one with the first _nargs_+1 local variables. Argument values of types `long` and `double` must be stored in two consecutive local variables, thus more than _nargs_ local variables may be required to pass _nargs_ argument values to the invoked method.

The _invokespecial_ instruction handles invocation of a non-`abstract` interface method, referenced either via a direct superinterface or via a superclass. In these cases, the rules for selection are essentially the same as those for _invokeinterface_ (except that the search starts from a different class).

### _invokestatic_

#### Operation

Invoke a class (`static`) method

#### Format

  
_invokestatic_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_invokestatic_ = 184 (0xb8)

#### Operand Stack

..., \[_arg1_, \[_arg2_ ...\]\] →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a method or an interface method ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor ([§4.3.3](jvms-4.html#jvms-4.3.3 "4.3.3. Method Descriptors")) of the method or interface method as well as a symbolic reference to the class or interface in which the method or interface method is to be found. The named method is resolved ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution"), [§5.4.3.4](jvms-5.html#jvms-5.4.3.4 "5.4.3.4. Interface Method Resolution")).

The resolved method must not be an instance initialization method, or the class or interface initialization method ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods"), [§2.9.2](jvms-2.html#jvms-2.9.2 "2.9.2. Class Initialization Methods")).

The resolved method must be `static`, and therefore cannot be `abstract`.

On successful resolution of the method, the class or interface that declared the resolved method is initialized if that class or interface has not already been initialized ([§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization")).

The operand stack must contain _nargs_ argument values, where the number, type, and order of the values must be consistent with the descriptor of the resolved method.

If the method is `synchronized`, the monitor associated with the resolved `Class` object is entered or reentered as if by execution of a _monitorenter_ instruction ([§_monitorenter_](jvms-6.html#jvms-6.5.monitorenter "monitorenter")) in the current thread.

If the method is not `native`, the _nargs_ argument values are popped from the operand stack. A new frame is created on the Java Virtual Machine stack for the method being invoked. The _nargs_ argument values are consecutively made the values of local variables of the new frame, with _arg1_ in local variable 0 (or, if _arg1_ is of type `long` or `double`, in local variables 0 and 1) and so on. The new frame is then made current, and the Java Virtual Machine `pc` is set to the opcode of the first instruction of the method to be invoked. Execution continues with the first instruction of the method.

If the method is `native` and the platform-dependent code that implements it has not yet been bound ([§5.6](jvms-5.html#jvms-5.6 "5.6. Binding Native Method Implementations")) into the Java Virtual Machine, that is done. The _nargs_ argument values are popped from the operand stack and are passed as parameters to the code that implements the method. The parameters are passed and the code is invoked in an implementation-dependent manner. When the platform-dependent code returns, the following take place:

*   If the `native` method is `synchronized`, the monitor associated with the resolved `Class` object is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread.
    
*   If the `native` method returns a value, the return value of the platform-dependent code is converted in an implementation-dependent way to the return type of the `native` method and pushed onto the operand stack.
    

#### Linking Exceptions

During resolution of the symbolic reference to the method, any of the exceptions pertaining to method resolution ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution")) can be thrown.

Otherwise, if the resolved method is an instance method, the _invokestatic_ instruction throws an `IncompatibleClassChangeError`.

#### Run-time Exceptions

Otherwise, if execution of this _invokestatic_ instruction causes initialization of the referenced class or interface, _invokestatic_ may throw an `Error` as detailed in [§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization").

Otherwise, if the resolved method is `native` and the code that implements the method cannot be bound, _invokestatic_ throws an `UnsatisfiedLinkError`.

#### Notes

The _nargs_ argument values are not one-to-one with the first _nargs_ local variables. Argument values of types `long` and `double` must be stored in two consecutive local variables, thus more than _nargs_ local variables may be required to pass _nargs_ argument values to the invoked method.

### _invokevirtual_

#### Operation

Invoke instance method; dispatch based on class

#### Format

  
_invokevirtual_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_invokevirtual_ = 182 (0xb6)

#### Operand Stack

..., _objectref_, \[_arg1_, \[_arg2_ ...\]\] →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a method ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor ([§4.3.3](jvms-4.html#jvms-4.3.3 "4.3.3. Method Descriptors")) of the method as well as a symbolic reference to the class in which the method is to be found. The named method is resolved ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution")).

_If the resolved method is not signature polymorphic_ ([§2.9.3](jvms-2.html#jvms-2.9.3 "2.9.3. Signature Polymorphic Methods")), then the _invokevirtual_ instruction proceeds as follows.

Let C be the class of _objectref_. A method is selected with respect to C and the resolved method ([§5.4.6](jvms-5.html#jvms-5.4.6 "5.4.6. Method Selection")). This is the _method to be invoked_.

The _objectref_ must be followed on the operand stack by _nargs_ argument values, where the number, type, and order of the values must be consistent with the descriptor of the selected instance method.

If the method to be invoked is `synchronized`, the monitor associated with _objectref_ is entered or reentered as if by execution of a _monitorenter_ instruction ([§_monitorenter_](jvms-6.html#jvms-6.5.monitorenter "monitorenter")) in the current thread.

If the method to be invoked is not `native`, the _nargs_ argument values and _objectref_ are popped from the operand stack. A new frame is created on the Java Virtual Machine stack for the method being invoked. The _objectref_ and the argument values are consecutively made the values of local variables of the new frame, with _objectref_ in local variable 0, _arg1_ in local variable 1 (or, if _arg1_ is of type `long` or `double`, in local variables 1 and 2), and so on. The new frame is then made current, and the Java Virtual Machine `pc` is set to the opcode of the first instruction of the method to be invoked. Execution continues with the first instruction of the method.

If the method to be invoked is `native` and the platform-dependent code that implements it has not yet been bound ([§5.6](jvms-5.html#jvms-5.6 "5.6. Binding Native Method Implementations")) into the Java Virtual Machine, that is done. The _nargs_ argument values and _objectref_ are popped from the operand stack and are passed as parameters to the code that implements the method. The parameters are passed and the code is invoked in an implementation-dependent manner. When the platform-dependent code returns, the following take place:

*   If the `native` method is `synchronized`, the monitor associated with _objectref_ is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread.
    
*   If the `native` method returns a value, the return value of the platform-dependent code is converted in an implementation-dependent way to the return type of the `native` method and pushed onto the operand stack.
    

_If the resolved method is signature polymorphic_ ([§2.9.3](jvms-2.html#jvms-2.9.3 "2.9.3. Signature Polymorphic Methods")), _and declared in the `java.lang.invoke.MethodHandle` class_, then the _invokevirtual_ instruction proceeds as follows, where D is the descriptor of the method symbolically referenced by the instruction.

First, a `reference` to an instance of `java.lang.invoke.MethodType` is obtained as if by resolution of a symbolic reference to a method type ([§5.4.3.5](jvms-5.html#jvms-5.4.3.5 "5.4.3.5. Method Type and Method Handle Resolution")) with the same parameter and return types as D.

*   If the named method is `invokeExact`, the instance of `java.lang.invoke.MethodType` must be semantically equal to the type descriptor of the receiving method handle _objectref_. The _method handle to be invoked_ is _objectref_.
    
*   If the named method is `invoke`, and the instance of `java.lang.invoke.MethodType` is semantically equal to the type descriptor of the receiving method handle _objectref_, then the _method handle to be invoked_ is _objectref_.
    
*   If the named method is `invoke`, and the instance of `java.lang.invoke.MethodType` is not semantically equal to the type descriptor of the receiving method handle _objectref_, then the Java Virtual Machine attempts to adjust the type descriptor of the receiving method handle, as if by invocation of the `asType` method of `java.lang.invoke.MethodHandle`, to obtain an exactly invokable method handle `m`. The _method handle to be invoked_ is `m`.
    

The _objectref_ must be followed on the operand stack by _nargs_ argument values, where the number, type, and order of the values must be consistent with the type descriptor of the method handle to be invoked. (This type descriptor will correspond to the method descriptor appropriate for the kind of the method handle to be invoked, as specified in [§5.4.3.5](jvms-5.html#jvms-5.4.3.5 "5.4.3.5. Method Type and Method Handle Resolution").)

Then, if the method handle to be invoked has bytecode behavior, the Java Virtual Machine invokes the method handle as if by execution of the bytecode behavior associated with the method handle's kind. If the kind is 5 (`REF_invokeVirtual`), 6 (`REF_invokeStatic`), 7 (`REF_invokeSpecial`), 8 (`REF_newInvokeSpecial`), or 9 (`REF_invokeInterface`), then a frame will be created and made current _in the course of executing the bytecode behavior_; however, this frame is not visible, and when the method invoked by the bytecode behavior completes (normally or abruptly), the _frame of its invoker_ is considered to be the frame for the method containing this _invokevirtual_ instruction.

Otherwise, if the method handle to be invoked has no bytecode behavior, the Java Virtual Machine invokes it in an implementation-dependent manner.

_If the resolved method is signature polymorphic and declared in the `java.lang.invoke.VarHandle` class_, then the _invokevirtual_ instruction proceeds as follows, where `N` and D are the name and descriptor of the method symbolically referenced by the instruction.

First, a `reference` to an instance of `java.lang.invoke.VarHandle.AccessMode` is obtained as if by invocation of the `valueFromMethodName` method of `java.lang.invoke.VarHandle.AccessMode` with a `String` argument denoting `N`.

Second, a `reference` to an instance of `java.lang.invoke.MethodType` is obtained as if by invocation of the `accessModeType` method of `java.lang.invoke.VarHandle` on the instance _objectref_, with the instance of `java.lang.invoke.VarHandle.AccessMode` as the argument.

Third, a `reference` to an instance of `java.lang.invoke.MethodHandle` is obtained as if by invocation of the `varHandleExactInvoker` method of `java.lang.invoke.MethodHandles` with the instance of `java.lang.invoke.VarHandle.AccessMode` as the first argument and the instance of `java.lang.invoke.MethodType` as the second argument. The resulting instance is called the _invoker method handle_.

Finally, the _nargs_ argument values and _objectref_ are popped from the operand stack, and the invoker method handle is invoked. The invocation occurs as if by execution of an _invokevirtual_ instruction that indicates a run-time constant pool index to a symbolic reference R where:

*   R is a symbolic reference to a method of a class;
    
*   for the symbolic reference to the class in which the method is to be found, R specifies `java.lang.invoke.MethodHandle`;
    
*   for the name of the method, R specifies `invoke`;
    
*   for the descriptor of the method, R specifies a return type indicated by the return descriptor of D, and specifies a first parameter type of `java.lang.invoke.VarHandle` followed by the parameter types indicated by the parameter descriptors of D (if any) in order.
    

and where it is as if the following items were pushed, in order, onto the operand stack:

*   a `reference` to the instance of `java.lang.invoke.MethodHandle` (the invoker method handle);
    
*   _objectref_;
    
*   the _nargs_ argument values, where the number, type, and order of the values must be consistent with the type descriptor of the invoker method handle.
    

#### Linking Exceptions

During resolution of the symbolic reference to the method, any of the exceptions pertaining to method resolution ([§5.4.3.3](jvms-5.html#jvms-5.4.3.3 "5.4.3.3. Method Resolution")) can be thrown.

Otherwise, if the resolved method is a class (`static`) method, the _invokevirtual_ instruction throws an `IncompatibleClassChangeError`.

Otherwise, if the resolved method is signature polymorphic and declared in the `java.lang.invoke.MethodHandle` class, then during resolution of the method type derived from the descriptor in the symbolic reference to the method, any of the exceptions pertaining to method type resolution ([§5.4.3.5](jvms-5.html#jvms-5.4.3.5 "5.4.3.5. Method Type and Method Handle Resolution")) can be thrown.

Otherwise, if the resolved method is signature polymorphic and declared in the `java.lang.invoke.VarHandle` class, then any linking exception that may arise from invocation of the invoker method handle can be thrown. No linking exceptions are thrown from invocation of the `valueFromMethodName`, `accessModeType`, and `varHandleExactInvoker` methods.

#### Run-time Exceptions

Otherwise, if _objectref_ is `null`, the _invokevirtual_ instruction throws a `NullPointerException`.

Otherwise, if the resolved method is not signature polymorphic:

*   If the selected method is `abstract`, _invokevirtual_ throws an `AbstractMethodError`.
    
*   Otherwise, if the selected method is `native` and the code that implements the method cannot be bound, _invokevirtual_ throws an `UnsatisfiedLinkError`.
    
*   Otherwise, if no method is selected, and there are multiple maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokevirtual_ throws an `IncompatibleClassChangeError`
    
*   Otherwise, if no method is selected, and there are no maximally-specific superinterface methods of C that match the resolved method's name and descriptor and are not `abstract`, _invokevirtual_ throws an `AbstractMethodError`.
    

Otherwise, if the resolved method is signature polymorphic and declared in the `java.lang.invoke.MethodHandle` class, then:

*   If the method name is `invokeExact`, and the obtained instance of `java.lang.invoke.MethodType` is not semantically equal to the type descriptor of the receiving method handle _objectref_, the _invokevirtual_ instruction throws a `java.lang.invoke.WrongMethodTypeException`.
    
*   If the method name is `invoke`, and the obtained instance of `java.lang.invoke.MethodType` is not a valid argument to the `asType` method of `java.lang.invoke.MethodHandle` invoked on the receiving method handle _objectref_, the _invokevirtual_ instruction throws a `java.lang.invoke.WrongMethodTypeException`.
    

Otherwise, if the resolved method is signature polymorphic and declared in the `java.lang.invoke.VarHandle` class, then any run-time exception that may arise from invocation of the invoker method handle can be thrown. No run-time exceptions are thrown from invocation of the `valueFromMethodName`, `accessModeType`, and `varHandleExactInvoker` methods, except `NullPointerException` if _objectref_ is `null`.

#### Notes

The _nargs_ argument values and _objectref_ are not one-to-one with the first _nargs_+1 local variables. Argument values of types `long` and `double` must be stored in two consecutive local variables, thus more than _nargs_ local variables may be required to pass _nargs_ argument values to the invoked method.

It is possible that the symbolic reference of an _invokevirtual_ instruction resolves to an interface method. In this case, it is possible that there is no overriding method in the class hierarchy, but that a non-`abstract` interface method matches the resolved method's descriptor. The selection logic matches such a method, using the same rules as for _invokeinterface_.

### _ior_

#### Operation

Boolean OR `int`

#### Format

  
_ior_  

#### Forms

_ior_ = 128 (0x80)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. They are popped from the operand stack. An `int` _result_ is calculated by taking the bitwise inclusive OR of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _irem_

#### Operation

Remainder `int`

#### Format

  
_irem_  

#### Forms

_irem_ = 112 (0x70)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. The `int` _result_ is _value1_ - (_value1_ / _value2_) \* _value2_. The _result_ is pushed onto the operand stack.

The result of the _irem_ instruction is such that `(a/b)*b + (a%b)` is equal to `a`. This identity holds even in the special case in which the dividend is the negative `int` of largest possible magnitude for its type and the divisor is -1 (the remainder is 0). It follows from this rule that the result of the remainder operation can be negative only if the dividend is negative and can be positive only if the dividend is positive. Moreover, the magnitude of the result is always less than the magnitude of the divisor.

#### Run-time Exception

If the value of the divisor for an `int` remainder operator is 0, _irem_ throws an `ArithmeticException`.

### _ireturn_

#### Operation

Return `int` from method

#### Format

  
_ireturn_  

#### Forms

_ireturn_ = 172 (0xac)

#### Operand Stack

..., _value_ →

\[empty\]

#### Description

The current method must have return type `boolean`, `byte`, `char`, `short`, or `int`. The _value_ must be of type `int`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, _value_ is popped from the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

Prior to pushing _value_ onto the operand stack of the frame of the invoker, it may have to be converted. If the return type of the invoked method was `byte`, `char`, or `short`, then _value_ is converted from `int` to the return type as if by execution of _i2b_, _i2c_, or _i2s_, respectively. If the return type of the invoked method was `boolean`, then _value_ is narrowed from `int` to `boolean` by taking the bitwise AND of _value_ and 1.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _ireturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _ireturn_ throws an `IllegalMonitorStateException`.

### _ishl_

#### Operation

Shift left `int`

#### Format

  
_ishl_  

#### Forms

_ishl_ = 120 (0x78)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. An `int` _result_ is calculated by shifting _value1_ left by _s_ bit positions, where _s_ is the value of the low 5 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

This is equivalent (even if overflow occurs) to multiplication by 2 to the power _s_. The shift distance actually used is always in the range 0 to 31, inclusive, as if _value2_ were subjected to a bitwise logical AND with the mask value 0x1f.

### _ishr_

#### Operation

Arithmetic shift right `int`

#### Format

  
_ishr_  

#### Forms

_ishr_ = 122 (0x7a)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. An `int` _result_ is calculated by shifting _value1_ right by _s_ bit positions, with sign extension, where _s_ is the value of the low 5 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

The resulting value is _floor_(_value1_ / 2_s_), where _s_ is _value2_ & 0x1f. For non-negative _value1_, this is equivalent to truncating `int` division by 2 to the power _s_. The shift distance actually used is always in the range 0 to 31, inclusive, as if _value2_ were subjected to a bitwise logical AND with the mask value 0x1f.

### _istore_

#### Operation

Store `int` into local variable

#### Format

  
_istore_  
_index_  

#### Forms

_istore_ = 54 (0x36)

#### Operand Stack

..., _value_ →

...

#### Description

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack, and the value of the local variable at _index_ is set to _value_.

#### Notes

The _istore_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _istore\_<n>_

#### Operation

Store `int` into local variable

#### Format

  
_istore\_<n>_  

#### Forms

_istore\_0_ = 59 (0x3b)

_istore\_1_ = 60 (0x3c)

_istore\_2_ = 61 (0x3d)

_istore\_3_ = 62 (0x3e)

#### Operand Stack

..., _value_ →

...

#### Description

The <_n_\> must be an index into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `int`. It is popped from the operand stack, and the value of the local variable at <_n_\> is set to _value_.

#### Notes

Each of the _istore\_<n>_ instructions is the same as _istore_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _isub_

#### Operation

Subtract `int`

#### Format

  
_isub_  

#### Forms

_isub_ = 100 (0x64)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. The `int` _result_ is _value1_ - _value2_. The _result_ is pushed onto the operand stack.

For `int` subtraction, `a-b` produces the same result as `a+(-b)`. For `int` values, subtraction from zero is the same as negation.

The result is the 32 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `int`. If overflow occurs, then the sign of the result may not be the same as the sign of the mathematical difference of the two values.

Despite the fact that overflow may occur, execution of an _isub_ instruction never throws a run-time exception.

### _iushr_

#### Operation

Logical shift right `int`

#### Format

  
_iushr_  

#### Forms

_iushr_ = 124 (0x7c)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. The values are popped from the operand stack. An `int` _result_ is calculated by shifting _value1_ right by _s_ bit positions, with zero extension, where _s_ is the value of the low 5 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

If _value1_ is positive and _s_ is _value2_ & 0x1f, the result is the same as that of _value1_ `>>` _s_; if _value1_ is negative, the result is equal to the value of the expression (_value1_ `>>` _s_) + (2 `<<` ~_s_). The addition of the (2 `<<` ~_s_) term cancels out the propagated sign bit. The shift distance actually used is always in the range 0 to 31, inclusive.

### _ixor_

#### Operation

Boolean XOR `int`

#### Format

  
_ixor_  

#### Forms

_ixor_ = 130 (0x82)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `int`. They are popped from the operand stack. An `int` _result_ is calculated by taking the bitwise exclusive OR of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _jsr_

#### Operation

Jump subroutine

#### Format

  
_jsr_  
_branchbyte1_  
_branchbyte2_  

#### Forms

_jsr_ = 168 (0xa8)

#### Operand Stack

... →

..., _address_

#### Description

The _address_ of the opcode of the instruction immediately following this _jsr_ instruction is pushed onto the operand stack as a value of type `returnAddress`. The unsigned _branchbyte1_ and _branchbyte2_ are used to construct a signed 16-bit offset, where the offset is (_branchbyte1_ `<<` 8) | _branchbyte2_. Execution proceeds at that offset from the address of this _jsr_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _jsr_ instruction.

#### Notes

Note that _jsr_ pushes the address onto the operand stack and _ret_ ([§_ret_](jvms-6.html#jvms-6.5.ret "ret")) gets it out of a local variable. This asymmetry is intentional.

In Oracle's implementation of a compiler for the Java programming language prior to Java SE 6, the _jsr_ instruction was used with the _ret_ instruction in the implementation of the `finally` clause ([§3.13](jvms-3.html#jvms-3.13 "3.13. Compiling finally"), [§4.10.2.5](jvms-4.html#jvms-4.10.2.5 "4.10.2.5. Exceptions and finally")).

### _jsr\_w_

#### Operation

Jump subroutine (wide index)

#### Format

  
_jsr\_w_  
_branchbyte1_  
_branchbyte2_  
_branchbyte3_  
_branchbyte4_  

#### Forms

_jsr\_w_ = 201 (0xc9)

#### Operand Stack

... →

..., _address_

#### Description

The _address_ of the opcode of the instruction immediately following this _jsr\_w_ instruction is pushed onto the operand stack as a value of type `returnAddress`. The unsigned _branchbyte1_, _branchbyte2_, _branchbyte3_, and _branchbyte4_ are used to construct a signed 32-bit offset, where the offset is (_branchbyte1_ `<<` 24) | (_branchbyte2_ `<<` 16) | (_branchbyte3_ `<<` 8) | _branchbyte4_. Execution proceeds at that offset from the address of this _jsr\_w_ instruction. The target address must be that of an opcode of an instruction within the method that contains this _jsr\_w_ instruction.

#### Notes

Note that _jsr\_w_ pushes the address onto the operand stack and _ret_ ([§_ret_](jvms-6.html#jvms-6.5.ret "ret")) gets it out of a local variable. This asymmetry is intentional.

In Oracle's implementation of a compiler for the Java programming language prior to Java SE 6, the _jsr\_w_ instruction was used with the _ret_ instruction in the implementation of the `finally` clause ([§3.13](jvms-3.html#jvms-3.13 "3.13. Compiling finally"), [§4.10.2.5](jvms-4.html#jvms-4.10.2.5 "4.10.2.5. Exceptions and finally")).

Although the _jsr\_w_ instruction takes a 4-byte branch offset, other factors limit the size of a method to 65535 bytes ([§4.11](jvms-4.html#jvms-4.11 "4.11. Limitations of the Java Virtual Machine")). This limit may be raised in a future release of the Java Virtual Machine.

### _l2d_

#### Operation

Convert `long` to `double`

#### Format

  
_l2d_  

#### Forms

_l2d_ = 138 (0x8a)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `long`. It is popped from the operand stack and converted to a `double` _result_ using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The _result_ is pushed onto the operand stack.

#### Notes

The _l2d_ instruction performs a widening primitive conversion (JLS §5.1.2) that may lose precision because values of type `double` have only 53 significand bits.

### _l2f_

#### Operation

Convert `long` to `float`

#### Format

  
_l2f_  

#### Forms

_l2f_ = 137 (0x89)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `long`. It is popped from the operand stack and converted to a `float` _result_ using the round to nearest rounding policy ([§2.8](jvms-2.html#jvms-2.8 "2.8. Floating-Point Arithmetic")). The _result_ is pushed onto the operand stack.

#### Notes

The _l2f_ instruction performs a widening primitive conversion (JLS §5.1.2) that may lose precision because values of type `float` have only 24 significand bits.

### _l2i_

#### Operation

Convert `long` to `int`

#### Format

  
_l2i_  

#### Forms

_l2i_ = 136 (0x88)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ on the top of the operand stack must be of type `long`. It is popped from the operand stack and converted to an `int` _result_ by taking the low-order 32 bits of the `long` value and discarding the high-order 32 bits. The _result_ is pushed onto the operand stack.

#### Notes

The _l2i_ instruction performs a narrowing primitive conversion (JLS §5.1.3). It may lose information about the overall magnitude of _value_. The _result_ may also not have the same sign as value.

### _ladd_

#### Operation

Add `long`

#### Format

  
_ladd_  

#### Forms

_ladd_ = 97 (0x61)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. The values are popped from the operand stack. The `long` _result_ is _value1_ + _value2_. The _result_ is pushed onto the operand stack.

The result is the 64 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `long`. If overflow occurs, the sign of the result may not be the same as the sign of the mathematical sum of the two values.

Despite the fact that overflow may occur, execution of an _ladd_ instruction never throws a run-time exception.

### _laload_

#### Operation

Load `long` from array

#### Format

  
_laload_  

#### Forms

_laload_ = 47 (0x2f)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `long`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `long` _value_ in the component of the array at _index_ is retrieved and pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _laload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _laload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _land_

#### Operation

Boolean AND `long`

#### Format

  
_land_  

#### Forms

_land_ = 127 (0x7f)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. They are popped from the operand stack. A `long` _result_ is calculated by taking the bitwise AND of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _lastore_

#### Operation

Store into `long` array

#### Format

  
_lastore_  

#### Forms

_lastore_ = 80 (0x50)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `long`. The _index_ must be of type `int`, and _value_ must be of type `long`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `long` _value_ is stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _lastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _lastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _lcmp_

#### Operation

Compare `long`

#### Format

  
_lcmp_  

#### Forms

_lcmp_ = 148 (0x94)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. They are both popped from the operand stack, and a signed integer comparison is performed. If _value1_ is greater than _value2_, the `int` value 1 is pushed onto the operand stack. If _value1_ is equal to _value2_, the `int` value 0 is pushed onto the operand stack. If _value1_ is less than _value2_, the `int` value -1 is pushed onto the operand stack.

### _lconst\_<l>_

#### Operation

Push `long` constant

#### Format

  
_lconst\_<l>_  

#### Forms

_lconst\_0_ = 9 (0x9)

_lconst\_1_ = 10 (0xa)

#### Operand Stack

... →

..., <_l_\>

#### Description

Push the `long` constant <_l_\> (0 or 1) onto the operand stack.

### _ldc_

#### Operation

Push item from run-time constant pool

#### Format

  
_ldc_  
_index_  

#### Forms

_ldc_ = 18 (0x12)

#### Operand Stack

... →

..., _value_

#### Description

The _index_ is an unsigned byte that must be a valid index into the run-time constant pool of the current class ([§2.5.5](jvms-2.html#jvms-2.5.5 "2.5.5. Run-Time Constant Pool")). The run-time constant pool entry at _index_ must be loadable ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), and not any of the following:

*   A numeric constant of type `long` or `double`.
    
*   A symbolic reference to a dynamically-computed constant whose field descriptor is `J` (denoting `long`) or `D` (denoting `double`).
    

If the run-time constant pool entry is a numeric constant of type `int` or `float`, then the _value_ of that numeric constant is pushed onto the operand stack as an `int` or `float`, respectively.

Otherwise, if the run-time constant pool entry is a string constant, that is, a `reference` to an instance of class `String`, then _value_, a `reference` to that instance, is pushed onto the operand stack.

Otherwise, if the run-time constant pool entry is a symbolic reference to a class or interface, then the named class or interface is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")) and _value_, a `reference` to the `Class` object representing that class or interface, is pushed onto the operand stack.

Otherwise, the run-time constant pool entry is a symbolic reference to a method type, a method handle, or a dynamically-computed constant. The symbolic reference is resolved ([§5.4.3.5](jvms-5.html#jvms-5.4.3.5 "5.4.3.5. Method Type and Method Handle Resolution"), [§5.4.3.6](jvms-5.html#jvms-5.4.3.6 "5.4.3.6. Dynamically-Computed Constant and Call Site Resolution")) and _value_, the result of resolution, is pushed onto the operand stack.

#### Linking Exceptions

During resolution of a symbolic reference, any of the exceptions pertaining to resolution of that kind of symbolic reference can be thrown.

### _ldc\_w_

#### Operation

Push item from run-time constant pool (wide index)

#### Format

  
_ldc\_w_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_ldc\_w_ = 19 (0x13)

#### Operand Stack

... →

..., _value_

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are assembled into an unsigned 16-bit index into the run-time constant pool of the current class ([§2.5.5](jvms-2.html#jvms-2.5.5 "2.5.5. Run-Time Constant Pool")), where the value of the index is calculated as (_indexbyte1_ `<<` 8) | _indexbyte2_. The index must be a valid index into the run-time constant pool of the current class. The run-time constant pool entry at the index must be loadable ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), and not any of the following:

*   A numeric constant of type `long` or `double`.
    
*   A symbolic reference to a dynamically-computed constant whose field descriptor is `J` (denoting `long`) or `D` (denoting `double`).
    

If the run-time constant pool entry is a numeric constant of type `int` or `float`, or a string constant, then _value_ is determined and pushed onto the operand stack according to the rules given for the _ldc_ instruction.

Otherwise, the run-time constant pool entry is a symbolic reference to a class, interface, method type, method handle, or dynamically-computed constant. It is resolved and _value_ is determined and pushed onto the operand stack according to the rules given for the _ldc_ instruction.

#### Linking Exceptions

During resolution of a symbolic reference, any of the exceptions pertaining to resolution of that kind of symbolic reference can be thrown.

#### Notes

The _ldc\_w_ instruction is identical to the _ldc_ instruction ([§_ldc_](jvms-6.html#jvms-6.5.ldc "ldc")) except for its wider run-time constant pool index.

### _ldc2\_w_

#### Operation

Push `long` or `double` from run-time constant pool (wide index)

#### Format

  
_ldc2\_w_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_ldc2\_w_ = 20 (0x14)

#### Operand Stack

... →

..., _value_

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are assembled into an unsigned 16-bit index into the run-time constant pool of the current class ([§2.5.5](jvms-2.html#jvms-2.5.5 "2.5.5. Run-Time Constant Pool")), where the value of the index is calculated as (_indexbyte1_ `<<` 8) | _indexbyte2_. The index must be a valid index into the run-time constant pool of the current class. The run-time constant pool entry at the index must be loadable ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), and in particular one of the following:

*   A numeric constant of type `long` or `double`.
    
*   A symbolic reference to a dynamically-computed constant whose field descriptor is `J` (denoting `long`) or `D` (denoting `double`).
    

If the run-time constant pool entry is a numeric constant of type `long` or `double`, then the _value_ of that numeric constant is pushed onto the operand stack as a `long` or `double`, respectively.

Otherwise, the run-time constant pool entry is a symbolic reference to a dynamically-computed constant. The symbolic reference is resolved ([§5.4.3.6](jvms-5.html#jvms-5.4.3.6 "5.4.3.6. Dynamically-Computed Constant and Call Site Resolution")) and _value_, the result of resolution, is pushed onto the operand stack.

#### Linking Exceptions

During resolution of a symbolic reference to a dynamically-computed constant, any of the exceptions pertaining to dynamically-computed constant resolution can be thrown.

#### Notes

Only a wide-index version of the _ldc2\_w_ instruction exists; there is no _ldc2_ instruction that pushes a `long` or `double` with a single-byte index.

### _ldiv_

#### Operation

Divide `long`

#### Format

  
_ldiv_  

#### Forms

_ldiv_ = 109 (0x6d)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. The values are popped from the operand stack. The `long` _result_ is the value of the Java programming language expression _value1_ / _value2_. The _result_ is pushed onto the operand stack.

A `long` division rounds towards 0; that is, the quotient produced for `long` values in _n_ / _d_ is a `long` value _q_ whose magnitude is as large as possible while satisfying |_d_ ⋅ _q_| ≤ |_n_|. Moreover, _q_ is positive when |_n_| ≥ |_d_| and _n_ and _d_ have the same sign, but _q_ is negative when |_n_| ≥ |_d_| and _n_ and _d_ have opposite signs.

There is one special case that does not satisfy this rule: if the dividend is the negative integer of largest possible magnitude for the `long` type and the divisor is -1, then overflow occurs and the result is equal to the dividend; despite the overflow, no exception is thrown in this case.

#### Run-time Exception

If the value of the divisor in a `long` division is 0, _ldiv_ throws an `ArithmeticException`.

### _lload_

#### Operation

Load `long` from local variable

#### Format

  
_lload_  
_index_  

#### Forms

_lload_ = 22 (0x16)

#### Operand Stack

... →

..., _value_

#### Description

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at _index_ must contain a `long`. The _value_ of the local variable at _index_ is pushed onto the operand stack.

#### Notes

The _lload_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _lload\_<n>_

#### Operation

Load `long` from local variable

#### Format

  
_lload\_<n>_  

#### Forms

_lload\_0_ = 30 (0x1e)

_lload\_1_ = 31 (0x1f)

_lload\_2_ = 32 (0x20)

_lload\_3_ = 33 (0x21)

#### Operand Stack

... →

..., _value_

#### Description

Both <_n_\> and <_n_\>+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The local variable at <_n_\> must contain a `long`. The _value_ of the local variable at <_n_\> is pushed onto the operand stack.

#### Notes

Each of the _lload\_<n>_ instructions is the same as _lload_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _lmul_

#### Operation

Multiply `long`

#### Format

  
_lmul_  

#### Forms

_lmul_ = 105 (0x69)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. The values are popped from the operand stack. The `long` _result_ is _value1_ \* _value2_. The _result_ is pushed onto the operand stack.

The result is the 64 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `long`. If overflow occurs, the sign of the result may not be the same as the sign of the mathematical multiplication of the two values.

Despite the fact that overflow may occur, execution of an _lmul_ instruction never throws a run-time exception.

### _lneg_

#### Operation

Negate `long`

#### Format

  
_lneg_  

#### Forms

_lneg_ = 117 (0x75)

#### Operand Stack

..., _value_ →

..., _result_

#### Description

The _value_ must be of type `long`. It is popped from the operand stack. The `long` _result_ is the arithmetic negation of _value_, -_value_. The _result_ is pushed onto the operand stack.

For `long` values, negation is the same as subtraction from zero. Because the Java Virtual Machine uses two's-complement representation for integers and the range of two's-complement values is not symmetric, the negation of the maximum negative `long` results in that same maximum negative number. Despite the fact that overflow has occurred, no exception is thrown.

For all `long` values `x`, `-x` equals `(~x)+1`.

### _lookupswitch_

#### Operation

Access jump table by key match and jump

#### Format

  
_lookupswitch_  
_<0-3 byte pad>_  
_defaultbyte1_  
_defaultbyte2_  
_defaultbyte3_  
_defaultbyte4_  
_npairs1_  
_npairs2_  
_npairs3_  
_npairs4_  
_match-offset pairs..._  

#### Forms

_lookupswitch_ = 171 (0xab)

#### Operand Stack

..., _key_ →

...

#### Description

A _lookupswitch_ is a variable-length instruction. Immediately after the _lookupswitch_ opcode, between zero and three bytes must act as padding, such that _defaultbyte1_ begins at an address that is a multiple of four bytes from the start of the current method (the opcode of its first instruction). Immediately after the padding follow a series of signed 32-bit values: _default_, _npairs_, and then _npairs_ pairs of signed 32-bit values. The _npairs_ must be greater than or equal to 0. Each of the _npairs_ pairs consists of an `int` _match_ and a signed 32-bit _offset_. Each of these signed 32-bit values is constructed from four unsigned bytes as (_byte1_ `<<` 24) | (_byte2_ `<<` 16) | (_byte3_ `<<` 8) | _byte4_.

The table _match-offset_ pairs of the _lookupswitch_ instruction must be sorted in increasing numerical order by _match_.

The _key_ must be of type `int` and is popped from the operand stack. The _key_ is compared against the _match_ values. If it is equal to one of them, then a target address is calculated by adding the corresponding _offset_ to the address of the opcode of this _lookupswitch_ instruction. If the _key_ does not match any of the _match_ values, the target address is calculated by adding _default_ to the address of the opcode of this _lookupswitch_ instruction. Execution then continues at the target address.

The target address that can be calculated from the _offset_ of each _match-offset_ pair, as well as the one calculated from _default_, must be the address of an opcode of an instruction within the method that contains this _lookupswitch_ instruction.

#### Notes

The alignment required of the 4-byte operands of the _lookupswitch_ instruction guarantees 4-byte alignment of those operands if and only if the method that contains the _lookupswitch_ is positioned on a 4-byte boundary.

The _match-offset_ pairs are sorted to support lookup routines that are quicker than linear search.

### _lor_

#### Operation

Boolean OR `long`

#### Format

  
_lor_  

#### Forms

_lor_ = 129 (0x81)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. They are popped from the operand stack. A `long` _result_ is calculated by taking the bitwise inclusive OR of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _lrem_

#### Operation

Remainder `long`

#### Format

  
_lrem_  

#### Forms

_lrem_ = 113 (0x71)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. The values are popped from the operand stack. The `long` _result_ is _value1_ - (_value1_ / _value2_) \* _value2_. The _result_ is pushed onto the operand stack.

The result of the _lrem_ instruction is such that `(a/b)*b + (a%b)` is equal to `a`. This identity holds even in the special case in which the dividend is the negative `long` of largest possible magnitude for its type and the divisor is -1 (the remainder is 0). It follows from this rule that the result of the remainder operation can be negative only if the dividend is negative and can be positive only if the dividend is positive; moreover, the magnitude of the result is always less than the magnitude of the divisor.

#### Run-time Exception

If the value of the divisor for a `long` remainder operator is 0, _lrem_ throws an `ArithmeticException`.

### _lreturn_

#### Operation

Return `long` from method

#### Format

  
_lreturn_  

#### Forms

_lreturn_ = 173 (0xad)

#### Operand Stack

..., _value_ →

\[empty\]

#### Description

The current method must have return type `long`. The _value_ must be of type `long`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, _value_ is popped from the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _lreturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is `synchronized`.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _lreturn_ throws an `IllegalMonitorStateException`.

### _lshl_

#### Operation

Shift left `long`

#### Format

  
_lshl_  

#### Forms

_lshl_ = 121 (0x79)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

The _value1_ must be of type `long`, and _value2_ must be of type `int`. The values are popped from the operand stack. A `long` _result_ is calculated by shifting _value1_ left by _s_ bit positions, where _s_ is the low 6 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

This is equivalent (even if overflow occurs) to multiplication by 2 to the power _s_. The shift distance actually used is therefore always in the range 0 to 63, inclusive, as if _value2_ were subjected to a bitwise logical AND with the mask value 0x3f.

### _lshr_

#### Operation

Arithmetic shift right `long`

#### Format

  
_lshr_  

#### Forms

_lshr_ = 123 (0x7b)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

The _value1_ must be of type `long`, and _value2_ must be of type `int`. The values are popped from the operand stack. A `long` _result_ is calculated by shifting _value1_ right by _s_ bit positions, with sign extension, where _s_ is the value of the low 6 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

The resulting value is _floor_(_value1_ / 2_s_), where _s_ is _value2_ & 0x3f. For non-negative _value1_, this is equivalent to truncating `long` division by 2 to the power _s_. The shift distance actually used is therefore always in the range 0 to 63, inclusive, as if _value2_ were subjected to a bitwise logical AND with the mask value 0x3f.

### _lstore_

#### Operation

Store `long` into local variable

#### Format

  
_lstore_  
_index_  

#### Forms

_lstore_ = 55 (0x37)

#### Operand Stack

..., _value_ →

...

#### Description

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `long`. It is popped from the operand stack, and the local variables at _index_ and _index_+1 are set to _value_.

#### Notes

The _lstore_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _lstore\_<n>_

#### Operation

Store `long` into local variable

#### Format

  
_lstore\_<n>_  

#### Forms

_lstore\_0_ = 63 (0x3f)

_lstore\_1_ = 64 (0x40)

_lstore\_2_ = 65 (0x41)

_lstore\_3_ = 66 (0x42)

#### Operand Stack

..., _value_ →

...

#### Description

Both <_n_\> and <_n_\>+1 must be indices into the local variable array of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")). The _value_ on the top of the operand stack must be of type `long`. It is popped from the operand stack, and the local variables at <_n_\> and <_n_\>+1 are set to _value_.

#### Notes

Each of the _lstore\_<n>_ instructions is the same as _lstore_ with an _index_ of <_n_\>, except that the operand <_n_\> is implicit.

### _lsub_

#### Operation

Subtract `long`

#### Format

  
_lsub_  

#### Forms

_lsub_ = 101 (0x65)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. The values are popped from the operand stack. The `long` _result_ is _value1_ - _value2_. The _result_ is pushed onto the operand stack.

For `long` subtraction, `a-b` produces the same result as `a+(-b)`. For `long` values, subtraction from zero is the same as negation.

The result is the 64 low-order bits of the true mathematical result in a sufficiently wide two's-complement format, represented as a value of type `long`. If overflow occurs, then the sign of the result may not be the same as the sign of the mathematical difference of the two values.

Despite the fact that overflow may occur, execution of an _lsub_ instruction never throws a run-time exception.

### _lushr_

#### Operation

Logical shift right `long`

#### Format

  
_lushr_  

#### Forms

_lushr_ = 125 (0x7d)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

The _value1_ must be of type `long`, and _value2_ must be of type `int`. The values are popped from the operand stack. A `long` _result_ is calculated by shifting _value1_ right logically by _s_ bit positions, with zero extension, where _s_ is the value of the low 6 bits of _value2_. The _result_ is pushed onto the operand stack.

#### Notes

If _value1_ is positive and _s_ is _value2_ & 0x3f, the result is the same as that of _value1_ `>>` _s_; if _value1_ is negative, the result is equal to the value of the expression (_value1_ `>>` _s_) + (2L `<<` ~_s_). The addition of the (2L `<<` ~_s_) term cancels out the propagated sign bit. The shift distance actually used is always in the range 0 to 63, inclusive.

### _lxor_

#### Operation

Boolean XOR `long`

#### Format

  
_lxor_  

#### Forms

_lxor_ = 131 (0x83)

#### Operand Stack

..., _value1_, _value2_ →

..., _result_

#### Description

Both _value1_ and _value2_ must be of type `long`. They are popped from the operand stack. A `long` _result_ is calculated by taking the bitwise exclusive OR of _value1_ and _value2_. The _result_ is pushed onto the operand stack.

### _monitorenter_

#### Operation

Enter monitor for object

#### Format

  
_monitorenter_  

#### Forms

_monitorenter_ = 194 (0xc2)

#### Operand Stack

..., _objectref_ →

...

#### Description

The _objectref_ must be of type `reference`.

Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes _monitorenter_ attempts to gain ownership of the monitor associated with _objectref_, as follows:

*   If the entry count of the monitor associated with _objectref_ is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
    
*   If the thread already owns the monitor associated with _objectref_, it reenters the monitor, incrementing its entry count.
    
*   If another thread already owns the monitor associated with _objectref_, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.
    

#### Run-time Exception

If _objectref_ is `null`, _monitorenter_ throws a `NullPointerException`.

#### Notes

A _monitorenter_ instruction may be used with one or more _monitorexit_ instructions ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) to implement a `synchronized` statement in the Java programming language ([§3.14](jvms-3.html#jvms-3.14 "3.14. Synchronization")). The _monitorenter_ and _monitorexit_ instructions are not used in the implementation of `synchronized` methods, although they can be used to provide equivalent locking semantics. Monitor entry on invocation of a `synchronized` method, and monitor exit on its return, are handled implicitly by the Java Virtual Machine's method invocation and return instructions, as if _monitorenter_ and _monitorexit_ were used.

The association of a monitor with an object may be managed in various ways that are beyond the scope of this specification. For instance, the monitor may be allocated and deallocated at the same time as the object. Alternatively, it may be dynamically allocated at the time when a thread attempts to gain exclusive access to the object and freed at some later time when no thread remains in the monitor for the object.

The synchronization constructs of the Java programming language require support for operations on monitors besides entry and exit. These include waiting on a monitor (`Object.wait`) and notifying other threads waiting on a monitor (`Object.notifyAll` and `Object.notify`). These operations are supported in the standard package `java.lang` supplied with the Java Virtual Machine. No explicit support for these operations appears in the instruction set of the Java Virtual Machine.

### _monitorexit_

#### Operation

Exit monitor for object

#### Format

  
_monitorexit_  

#### Forms

_monitorexit_ = 195 (0xc3)

#### Operand Stack

..., _objectref_ →

...

#### Description

The _objectref_ must be of type `reference`.

The thread that executes _monitorexit_ must be the owner of the monitor associated with the instance referenced by _objectref_.

The thread decrements the entry count of the monitor associated with _objectref_. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

#### Run-time Exceptions

If _objectref_ is `null`, _monitorexit_ throws a `NullPointerException`.

Otherwise, if the thread that executes _monitorexit_ is not the owner of the monitor associated with the instance referenced by _objectref_, _monitorexit_ throws an `IllegalMonitorStateException`.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the second of those rules is violated by the execution of this _monitorexit_ instruction, then _monitorexit_ throws an `IllegalMonitorStateException`.

#### Notes

One or more _monitorexit_ instructions may be used with a _monitorenter_ instruction ([§_monitorenter_](jvms-6.html#jvms-6.5.monitorenter "monitorenter")) to implement a `synchronized` statement in the Java programming language ([§3.14](jvms-3.html#jvms-3.14 "3.14. Synchronization")). The _monitorenter_ and _monitorexit_ instructions are not used in the implementation of `synchronized` methods, although they can be used to provide equivalent locking semantics.

The Java Virtual Machine supports exceptions thrown within `synchronized` methods and `synchronized` statements differently:

*   Monitor exit on normal `synchronized` method completion is handled by the Java Virtual Machine's return instructions. Monitor exit on abrupt `synchronized` method completion is handled implicitly by the Java Virtual Machine's _athrow_ instruction.
    
*   When an exception is thrown from within a `synchronized` statement, exit from the monitor entered prior to the execution of the `synchronized` statement is achieved using the Java Virtual Machine's exception handling mechanism ([§3.14](jvms-3.html#jvms-3.14 "3.14. Synchronization")).
    

### _multianewarray_

#### Operation

Create new multidimensional array

#### Format

  
_multianewarray_  
_indexbyte1_  
_indexbyte2_  
_dimensions_  

#### Forms

_multianewarray_ = 197 (0xc5)

#### Operand Stack

..., _count1_, \[_count2_, ...\] →

..., _arrayref_

#### Description

The _dimensions_ operand is an unsigned byte that must be greater than or equal to 1. It represents the number of dimensions of the array to be created. The operand stack must contain _dimensions_ values. Each such value represents the number of components in a dimension of the array to be created, must be of type `int`, and must be non-negative. The _count1_ is the desired length in the first dimension, _count2_ in the second, etc.

All of the _count_ values are popped off the operand stack. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a class, array, or interface type. The named class, array, or interface type is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")). The resulting entry must be an array class type of dimensionality greater than or equal to _dimensions_.

A new multidimensional array of the array type is allocated from the garbage-collected heap. If any _count_ value is zero, no subsequent dimensions are allocated. The components of the array in the first dimension are initialized to subarrays of the type of the second dimension, and so on. The components of the last allocated dimension of the array are initialized to the default initial value ([§2.3](jvms-2.html#jvms-2.3 "2.3. Primitive Types and Values"), [§2.4](jvms-2.html#jvms-2.4 "2.4. Reference Types and Values")) for the element type of the array type. A `reference` _arrayref_ to the new array is pushed onto the operand stack.

#### Linking Exceptions

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution") can be thrown.

Otherwise, if the current class does not have permission to access the element type of the resolved array class, _multianewarray_ throws an `IllegalAccessError`.

#### Run-time Exception

Otherwise, if any of the _dimensions_ values on the operand stack are less than zero, the _multianewarray_ instruction throws a `NegativeArraySizeException`.

#### Notes

It may be more efficient to use _newarray_ or _anewarray_ ([§_newarray_](jvms-6.html#jvms-6.5.newarray "newarray"), [§_anewarray_](jvms-6.html#jvms-6.5.anewarray "anewarray")) when creating an array of a single dimension.

The array class referenced via the run-time constant pool may have more dimensions than the _dimensions_ operand of the _multianewarray_ instruction. In that case, only the first _dimensions_ of the dimensions of the array are created.

### _new_

#### Operation

Create new object

#### Format

  
_new_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_new_ = 187 (0xbb)

#### Operand Stack

... →

..., _objectref_

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a class or interface type. The named class or interface type is resolved ([§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution")) and should result in a class type. Memory for a new instance of that class is allocated from the garbage-collected heap, and the instance variables of the new object are initialized to their default initial values ([§2.3](jvms-2.html#jvms-2.3 "2.3. Primitive Types and Values"), [§2.4](jvms-2.html#jvms-2.4 "2.4. Reference Types and Values")). The _objectref_, a `reference` to the instance, is pushed onto the operand stack.

On successful resolution of the class, it is initialized if it has not already been initialized ([§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization")).

#### Linking Exceptions

During resolution of the symbolic reference to the class or interface type, any of the exceptions documented in [§5.4.3.1](jvms-5.html#jvms-5.4.3.1 "5.4.3.1. Class and Interface Resolution") can be thrown.

Otherwise, if the symbolic reference to the class or interface type resolves to an interface or an `abstract` class, _new_ throws an `InstantiationError`.

#### Run-time Exception

Otherwise, if execution of this _new_ instruction causes initialization of the referenced class, _new_ may throw an `Error` as detailed in JLS §15.9.4.

#### Notes

The _new_ instruction does not completely create a new instance; instance creation is not completed until an instance initialization method ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods")) has been invoked on the uninitialized instance.

### _newarray_

#### Operation

Create new array

#### Format

  
_newarray_  
_atype_  

#### Forms

_newarray_ = 188 (0xbc)

#### Operand Stack

..., _count_ →

..., _arrayref_

#### Description

The _count_ must be of type `int`. It is popped off the operand stack. The _count_ represents the number of elements in the array to be created.

The _atype_ is a code that indicates the type of array to create. It must take one of the following values:

**Table 6.5.newarray-A. Array type codes**

 
| Array Type | _atype_ |
| --- | --- |
| `T_BOOLEAN` | 4 |
| `T_CHAR` | 5 |
| `T_FLOAT` | 6 |
| `T_DOUBLE` | 7 |
| `T_BYTE` | 8 |
| `T_SHORT` | 9 |
| `T_INT` | 10 |
| `T_LONG` | 11 |

  

A new array whose components are of type _atype_ and of length _count_ is allocated from the garbage-collected heap. A `reference` _arrayref_ to this new array object is pushed into the operand stack. Each of the elements of the new array is initialized to the default initial value ([§2.3](jvms-2.html#jvms-2.3 "2.3. Primitive Types and Values"), [§2.4](jvms-2.html#jvms-2.4 "2.4. Reference Types and Values")) for the element type of the array type.

#### Run-time Exception

If _count_ is less than zero, _newarray_ throws a `NegativeArraySizeException`.

#### Notes

In Oracle's Java Virtual Machine implementation, arrays of type `boolean` (_atype_ is `T_BOOLEAN`) are stored as arrays of 8-bit values and are manipulated using the _baload_ and _bastore_ instructions ([§_baload_](jvms-6.html#jvms-6.5.baload "baload"), [§_bastore_](jvms-6.html#jvms-6.5.bastore "bastore")) which also access arrays of type `byte`. Other implementations may implement packed `boolean` arrays; the _baload_ and _bastore_ instructions must still be used to access those arrays.

### _nop_

#### Operation

Do nothing

#### Format

  
_nop_  

#### Forms

_nop_ = 0 (0x0)

#### Operand Stack

No change

#### Description

Do nothing.

### _pop_

#### Operation

Pop the top operand stack value

#### Format

  
_pop_  

#### Forms

_pop_ = 87 (0x57)

#### Operand Stack

..., _value_ →

...

#### Description

Pop the top value from the operand stack.

The _pop_ instruction must not be used unless _value_ is a value of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

### _pop2_

#### Operation

Pop the top one or two operand stack values

#### Format

  
_pop2_  

#### Forms

_pop2_ = 88 (0x58)

#### Operand Stack

Form 1:

..., _value2_, _value1_ →

...

where each of _value1_ and _value2_ is a value of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

Form 2:

..., _value_ →

...

where _value_ is a value of a category 2 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Description

Pop the top one or two values from the operand stack.

### _putfield_

#### Operation

Set field in object

#### Format

  
_putfield_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_putfield_ = 181 (0xb5)

#### Operand Stack

..., _objectref_, _value_ →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a field ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor of the field as well as a symbolic reference to the class in which the field is to be found. The referenced field is resolved ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")).

The type of a _value_ stored by a _putfield_ instruction must be compatible with the descriptor of the referenced field ([§4.3.2](jvms-4.html#jvms-4.3.2 "4.3.2. Field Descriptors")). If the field descriptor type is `boolean`, `byte`, `char`, `short`, or `int`, then the _value_ must be an `int`. If the field descriptor type is `float`, `long`, or `double`, then the _value_ must be a `float`, `long`, or `double`, respectively. If the field descriptor type is a reference type, then the _value_ must be of a type that is assignment compatible (JLS §5.2) with the field descriptor type. If the field is `final`, it must be declared in the current class, and the instruction must occur in an instance initialization method of the current class ([§2.9.1](jvms-2.html#jvms-2.9.1 "2.9.1. Instance Initialization Methods")).

The _value_ and _objectref_ are popped from the operand stack.

The _objectref_ must be of type `reference` but not an array type.

If the _value_ is of type `int` and the field descriptor type is `boolean`, then the `int` _value_ is narrowed by taking the bitwise AND of _value_ and 1, resulting in _value_'. The referenced field in _objectref_ is set to _value_'.

Otherwise, the referenced field in _objectref_ is set to _value_.

#### Linking Exceptions

During resolution of the symbolic reference to the field, any of the exceptions pertaining to field resolution ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")) can be thrown.

Otherwise, if the resolved field is a `static` field, _putfield_ throws an `IncompatibleClassChangeError`.

Otherwise, if the resolved field is `final`, it must be declared in the current class, and the instruction must occur in an instance initialization method of the current class. Otherwise, an `IllegalAccessError` is thrown.

#### Run-time Exception

Otherwise, if _objectref_ is `null`, the _putfield_ instruction throws a `NullPointerException`.

### _putstatic_

#### Operation

Set static field in class

#### Format

  
_putstatic_  
_indexbyte1_  
_indexbyte2_  

#### Forms

_putstatic_ = 179 (0xb3)

#### Operand Stack

..., _value_ →

...

#### Description

The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The run-time constant pool entry at the index must be a symbolic reference to a field ([§5.1](jvms-5.html#jvms-5.1 "5.1. The Run-Time Constant Pool")), which gives the name and descriptor of the field as well as a symbolic reference to the class or interface in which the field is to be found. The referenced field is resolved ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")).

On successful resolution of the field, the class or interface that declared the resolved field is initialized if that class or interface has not already been initialized ([§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization")).

The type of a _value_ stored by a _putstatic_ instruction must be compatible with the descriptor of the referenced field ([§4.3.2](jvms-4.html#jvms-4.3.2 "4.3.2. Field Descriptors")). If the field descriptor type is `boolean`, `byte`, `char`, `short`, or `int`, then the _value_ must be an `int`. If the field descriptor type is `float`, `long`, or `double`, then the _value_ must be a `float`, `long`, or `double`, respectively. If the field descriptor type is a reference type, then the _value_ must be of a type that is assignment compatible (JLS §5.2) with the field descriptor type. If the field is `final`, it must be declared in the current class or interface, and the instruction must occur in the class or interface initialization method of the current class or interface ([§2.9.2](jvms-2.html#jvms-2.9.2 "2.9.2. Class Initialization Methods")).

The _value_ is popped from the operand stack.

If the _value_ is of type `int` and the field descriptor type is `boolean`, then the `int` _value_ is narrowed by taking the bitwise AND of _value_ and 1, resulting in _value_'. The referenced field in the class or interface is set to _value_'.

Otherwise, the referenced field in the class or interface is set to _value_.

#### Linking Exceptions

During resolution of the symbolic reference to the class or interface field, any of the exceptions pertaining to field resolution ([§5.4.3.2](jvms-5.html#jvms-5.4.3.2 "5.4.3.2. Field Resolution")) can be thrown.

Otherwise, if the resolved field is not a `static` (class) field or an interface field, _putstatic_ throws an `IncompatibleClassChangeError`.

Otherwise, if the resolved field is `final`, it must be declared in the current class or interface, and the instruction must occur in the class or interface initialization method of the current class or interface. Otherwise, an `IllegalAccessError` is thrown.

#### Run-time Exception

Otherwise, if execution of this _putstatic_ instruction causes initialization of the referenced class or interface, _putstatic_ may throw an `Error` as detailed in [§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization").

#### Notes

A _putstatic_ instruction may be used only to set the value of an interface field on the initialization of that field. Interface fields may be assigned to only once, on execution of an interface variable initialization expression when the interface is initialized ([§5.5](jvms-5.html#jvms-5.5 "5.5. Initialization"), JLS §9.3.1).

### _ret_

#### Operation

Return from subroutine

#### Format

  
_ret_  
_index_  

#### Forms

_ret_ = 169 (0xa9)

#### Operand Stack

No change

#### Description

The _index_ is an unsigned byte between 0 and 255, inclusive. The local variable at _index_ in the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) must contain a value of type `returnAddress`. The contents of the local variable are written into the Java Virtual Machine's `pc` register, and execution continues there.

#### Notes

Note that _jsr_ ([§_jsr_](jvms-6.html#jvms-6.5.jsr "jsr")) pushes the address onto the operand stack and _ret_ gets it out of a local variable. This asymmetry is intentional.

In Oracle's implementation of a compiler for the Java programming language prior to Java SE 6, the _ret_ instruction was used with the _jsr_ and _jsr\_w_ instructions ([§_jsr_](jvms-6.html#jvms-6.5.jsr "jsr"), [§_jsr\_w_](jvms-6.html#jvms-6.5.jsr_w "jsr_w")) in the implementation of the `finally` clause ([§3.13](jvms-3.html#jvms-3.13 "3.13. Compiling finally"), [§4.10.2.5](jvms-4.html#jvms-4.10.2.5 "4.10.2.5. Exceptions and finally")).

The _ret_ instruction should not be confused with the _return_ instruction ([§_return_](jvms-6.html#jvms-6.5.return "return")). A _return_ instruction returns control from a method to its invoker, without passing any value back to the invoker.

The _ret_ opcode can be used in conjunction with the _wide_ instruction ([§_wide_](jvms-6.html#jvms-6.5.wide "wide")) to access a local variable using a two-byte unsigned index.

### _return_

#### Operation

Return `void` from method

#### Format

  
_return_  

#### Forms

_return_ = 177 (0xb1)

#### Operand Stack

... →

\[empty\]

#### Description

The current method must have return type `void`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction ([§_monitorexit_](jvms-6.html#jvms-6.5.monitorexit "monitorexit")) in the current thread. If no exception is thrown, any values on the operand stack of the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")) are discarded.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

#### Run-time Exceptions

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization"), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _return_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is `synchronized`.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](jvms-2.html#jvms-2.11.10 "2.11.10. Synchronization") and if the first of those rules is violated during invocation of the current method, then _return_ throws an `IllegalMonitorStateException`.

### _saload_

#### Operation

Load `short` from array

#### Format

  
_saload_  

#### Forms

_saload_ = 53 (0x35)

#### Operand Stack

..., _arrayref_, _index_ →

..., _value_

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `short`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The component of the array at _index_ is retrieved and sign-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

#### Run-time Exceptions

If _arrayref_ is `null`, _saload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _saload_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _sastore_

#### Operation

Store into `short` array

#### Format

  
_sastore_  

#### Forms

_sastore_ = 86 (0x56)

#### Operand Stack

..., _arrayref_, _index_, _value_ →

...

#### Description

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `short`. Both _index_ and _value_ must be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `int` _value_ is truncated to a `short` and stored as the component of the array indexed by _index_.

#### Run-time Exceptions

If _arrayref_ is `null`, _sastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _sastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

### _sipush_

#### Operation

Push `short`

#### Format

  
_sipush_  
_byte1_  
_byte2_  

#### Forms

_sipush_ = 17 (0x11)

#### Operand Stack

... →

..., _value_

#### Description

The immediate unsigned _byte1_ and _byte2_ values are assembled into an intermediate `short`, where the value of the `short` is (_byte1_ `<<` 8) | _byte2_. The intermediate value is then sign-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

### _swap_

#### Operation

Swap the top two operand stack values

#### Format

  
_swap_  

#### Forms

_swap_ = 95 (0x5f)

#### Operand Stack

..., _value2_, _value1_ →

..., _value1_, _value2_

#### Description

Swap the top two values on the operand stack.

The _swap_ instruction must not be used unless _value1_ and _value2_ are both values of a category 1 computational type ([§2.11.1](jvms-2.html#jvms-2.11.1 "2.11.1. Types and the Java Virtual Machine")).

#### Notes

The Java Virtual Machine does not provide an instruction implementing a swap on operands of category 2 computational types.

### _tableswitch_

#### Operation

Access jump table by index and jump

#### Format

  
_tableswitch_  
_<0-3 byte pad>_  
_defaultbyte1_  
_defaultbyte2_  
_defaultbyte3_  
_defaultbyte4_  
_lowbyte1_  
_lowbyte2_  
_lowbyte3_  
_lowbyte4_  
_highbyte1_  
_highbyte2_  
_highbyte3_  
_highbyte4_  
_jump offsets..._  

#### Forms

_tableswitch_ = 170 (0xaa)

#### Operand Stack

..., _index_ →

...

#### Description

A _tableswitch_ is a variable-length instruction. Immediately after the _tableswitch_ opcode, between zero and three bytes must act as padding, such that _defaultbyte1_ begins at an address that is a multiple of four bytes from the start of the current method (the opcode of its first instruction). Immediately after the padding are bytes constituting three signed 32-bit values: _default_, _low_, and _high_. Immediately following are bytes constituting a series of _high_ - _low_ + 1 signed 32-bit offsets. The value _low_ must be less than or equal to _high_. The _high_ - _low_ + 1 signed 32-bit offsets are treated as a 0-based jump table. Each of these signed 32-bit values is constructed as (_byte1_ `<<` 24) | (_byte2_ `<<` 16) | (_byte3_ `<<` 8) | _byte4_.

The _index_ must be of type `int` and is popped from the operand stack. If _index_ is less than _low_ or _index_ is greater than _high_, then a target address is calculated by adding _default_ to the address of the opcode of this _tableswitch_ instruction. Otherwise, the offset at position _index_ - _low_ of the jump table is extracted. The target address is calculated by adding that offset to the address of the opcode of this _tableswitch_ instruction. Execution then continues at the target address.

The target address that can be calculated from each jump table offset, as well as the one that can be calculated from _default_, must be the address of an opcode of an instruction within the method that contains this _tableswitch_ instruction.

#### Notes

The alignment required of the 4-byte operands of the _tableswitch_ instruction guarantees 4-byte alignment of those operands if and only if the method that contains the _tableswitch_ starts on a 4-byte boundary.

### _wide_

#### Operation

Extend local variable index by additional bytes

#### Format 1

  
_wide_  
_<opcode>_  
_indexbyte1_  
_indexbyte2_  

where _<opcode>_ is one of _iload_, _fload_, _aload_, _lload_, _dload_, _istore_, _fstore_, _astore_, _lstore_, _dstore_, or _ret_

#### Format 2

  
_wide_  
_iinc_  
_indexbyte1_  
_indexbyte2_  
_constbyte1_  
_constbyte2_  

#### Forms

_wide_ = 196 (0xc4)

#### Operand Stack

Same as modified instruction

#### Description

The _wide_ instruction modifies the behavior of another instruction. It takes one of two formats, depending on the instruction being modified. The first form of the _wide_ instruction modifies one of the instructions _iload_, _fload_, _aload_, _lload_, _dload_, _istore_, _fstore_, _astore_, _lstore_, _dstore_, or _ret_ ([§_iload_](jvms-6.html#jvms-6.5.iload "iload"), [§_fload_](jvms-6.html#jvms-6.5.fload "fload"), [§_aload_](jvms-6.html#jvms-6.5.aload "aload"), [§_lload_](jvms-6.html#jvms-6.5.lload "lload"), [§_dload_](jvms-6.html#jvms-6.5.dload "dload"), [§_istore_](jvms-6.html#jvms-6.5.istore "istore"), [§_fstore_](jvms-6.html#jvms-6.5.fstore "fstore"), [§_astore_](jvms-6.html#jvms-6.5.astore "astore"), [§_lstore_](jvms-6.html#jvms-6.5.lstore "lstore"), [§_dstore_](jvms-6.html#jvms-6.5.dstore "dstore"), [§_ret_](jvms-6.html#jvms-6.5.ret "ret")). The second form applies only to the _iinc_ instruction ([§_iinc_](jvms-6.html#jvms-6.5.iinc "iinc")).

In either case, the _wide_ opcode itself is followed in the compiled code by the opcode of the instruction _wide_ modifies. In either form, two unsigned bytes _indexbyte1_ and _indexbyte2_ follow the modified opcode and are assembled into a 16-bit unsigned index to a local variable in the current frame ([§2.6](jvms-2.html#jvms-2.6 "2.6. Frames")), where the value of the index is (_indexbyte1_ `<<` 8) | _indexbyte2_. The calculated index must be an index into the local variable array of the current frame. Where the _wide_ instruction modifies an _lload_, _dload_, _lstore_, or _dstore_ instruction, the index following the calculated index (index + 1) must also be an index into the local variable array. In the second form, two immediate unsigned bytes _constbyte1_ and _constbyte2_ follow _indexbyte1_ and _indexbyte2_ in the code stream. Those bytes are also assembled into a signed 16-bit constant, where the constant is (_constbyte1_ `<<` 8) | _constbyte2_.

The widened bytecode operates as normal, except for the use of the wider index and, in the case of the second form, the larger increment range.

#### Notes

Although we say that _wide_ "modifies the behavior of another instruction," the _wide_ instruction effectively treats the bytes constituting the modified instruction as operands, denaturing the embedded instruction in the process. In the case of a modified _iinc_ instruction, one of the logical operands of the _iinc_ is not even at the normal offset from the opcode. The embedded instruction must never be executed directly; its opcode must never be the target of any control transfer instruction.
