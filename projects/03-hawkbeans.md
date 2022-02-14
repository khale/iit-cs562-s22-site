---
title: Project 2
nav_order: 3
parent: Projects
summary: P2
---

# Project 2: The Hawkbeans JVM

Released: Monday, 02/14/2022
**Due Date: Friday, 3/11/2022 11:59PM CST**

## Overview
For this project, you will be completing the implementation of the Hawkbeans
Java Virtual Machine. Namely, you will be completing this project in four parts
by implementing components of the JVM that deal with:

1. Symbolic Resolution (linking)
2. Bytecode Interpretation
3. Exceptions
4. Garbage Collection

You will be working with the Hawkbeans codebase, a small implementation of
a reasonable subset of the JVM spec. You will receive the code in skeleton
form, which you will complete by filling in missing functionality that will
allow you to run simple Java programs.

The goals of this project are to:

- Give you more experience working with a fairly large C source tree
- Understand generally how the Java language works under the hood
- Understand how HLL features like polymorphism, object orientation, automatic
  memory management, exceptions, protection, threads, and native methods are
  implemented
- Understand the benefits of stack machine architectures
- Become acquainted with the Java VM specification
- Have fun!

While working on Hawkbeans, you will no doubt notice many similarities in the
code structure to the emulator you previously completed. You should leverage
the wisdom you gained from that project when making design decisions for this
one. You will also identify some major differences; some coding tasks here are
more open-ended than others, so you will need to think creatively.

Keep in mind that this project is still in its experimental stages. If we encounter bugs or hiccups (likely!), we will work through them together.

## Hawkbeans Overview
Hawkbeans is a small, fairly limited JVM written by your instructor. At this point, simple Java programs will run, but anything involving significant Java runtime support or double precision floating point math will likely not work. This is partly by design; this JVM will eventually be ported to the Hawknest 6502v emulator, which you previously completed. Below is a list of features not currently supported by this JVM:


- Reflection
- Native library loading (for JNI)
- Multi-threading
- JIT compilation
- Complex exceptions
- Protection and access controls (e.g., private vs. public accessors not enforced)
- Complete type checking
- User-defined class loaders
- Higher-order functions

That said, it is complete enough to give you a good understanding of how Java works under the hood once you finish writing the missing components.

## Part 1: Getting Started

You will first need to download the Hawkbeans skeleton code. As in the first
project, you'll really want to be working on a Linux box or VM. First clone the
skeleton code:

```bash
git clone https://github.com/khale/hawkbeans-skeleton.git
```

This will fetch the code from a public git repo.

## Part 2: Building and Running Hawkbeans
You can now build Hawkbeans using `make`. This builds the actual JVM. You will
also want to build the Java standard libraries which Hawkbeans will use for
runtime support. These standard libraries have been borrowed from the TinyVM
project:

```bash
make jlibs
```

This will create two new directories, `hblib` and `java`, both of which contain
several class files. You shouldn't worry about how these are used for now.

The skeleton code you're given will compile and run, but will not successfully
run a Java program yet. Your job is to get it to that point. In order to do
that, you'll need to implement some major missing functionality.

To see how to use Hawkbeans, you can run it with the `-h` option, or without any
arguments. In general, you should invoke it like this:

```bash
bin/hawkbeans-clang-debug SomeClassFile
```

This is exactly the same way you run Java programs using the `java` command.
There are a few options that are worth mentioning in more detail. The
`--heap-size` or `-H` option will allow you to set the JVM's heap size manually.
The default heap size is 1MB, which is pretty small. You may need to change
this when you're testing your garbage collector later on. The `--trace-gc` or `-t`
option allows you to see stats gathered by the garbage collector, i.e., how many
objects were collected, how much heap space was recovered, how long GC took
etc. By default, the GC will collect garbage every 20ms. You can change this
manually by providing a number in ms using the `--gc-interval` or `-c` options. The
`-i` option performs like it did with Hawknest, see the [Debugging](#testing-and-debugging) section for
more details.

## Part 3: Symbolic Resolution (Linking)
Java [class files](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html) are laid out in a well-structured format which makes them
relatively easy to parse. A major component of any JVM is a linker/loader
component which loads these class files into memory, parses them, and creates
internal representations of the data they encode. Every reference to a constant
value, a class, or a method in a Java program is encoded in the class file
_symbolically_ by the Java compiler. Essentially this means that each such
reference consists of a [pointer to a descriptor](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4) which gives enough information
on how to resolve it. As an example, a method reference will simply be encoded
as a number which is to be interpreted as an offset into the constant pool.
A look up to that constant pool entry will discover a pointer to a descriptor
that has more such constant pool offsets. One will be an offset to a String
descriptor which contains the bytes for the UTF-8 encoded method name, and the
other will be an offset to another String descriptor which describes the
method's parameters and return value using a [character-based encoding](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3.3). One
thing to note is that this is an analog to how dynamic linking is done for
traditional compiled programs.

There are two approaches for resolving these symbolic references. We can either
resolve them _eagerly_ when we load and parse the class, or we can resolve them
_lazily_, i.e., not until they're used. The former method will require more work
up front, and has the drawback that classes that aren't actually used may be
loaded into memory. The latter is our technique of choice. That is, we will
resolve these references the first time they're actually used. This means that
the various instructions that rely on symbolic references (e.g. `new`, `getfield`,
etc.) will not work correctly until symbolic resolution is complete.

### Coding Task 1

Your first task is to complete method and class resolution in Hawkbeans. This
will involve filling in the implementations for two functions in `src/class.c`,
`hb_resolve_class` and `hb_resolve_method`. The latter relies on the former, so
start with `hb_resolve_class`. You'll want to use the Oracle JVM docs for [class
resolution](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.1) and [method resolution](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.3) as references when writing your
implementation.

In general, when a constant pool entry is resolved, the implementer of the JVM
is given flexibility on what this means in practice. For our purposes, the
pointer to the constant info structure (whether it be for a class, a method,
a field, etc.) is replaced with an internal pointer to a structure representing
the appropriate entry. To indicate that the entry is resolved, we [rely on some
insider information](https://en.wikipedia.org/wiki/X86-64#Canonical_form_addresses) about virtual addresses and set the highest bit of the
pointer's address to 1 (sometimes this is called a _tagged_ pointer. For example,
for classes, this means that a `CONST_Class_info_t*` is replaced with
a `ljava_class_t*`. For methods, it means a `CONST_Methodref_info_t*` is replaced
with a `lmethod_info_t*` (both structs are defined in `include/class.h`).

#### `hb_resolve_class()`
This function is used by other parts of the JVM to resolve symbolic references
to classes, for example to access their fields or methods. It relies on an
existing class loader (there is currently only one, in
`jvm/arch/x64-linux/bootstrap_loader.c`). The goal here is to transform
a constant pool entry (a pointer to a descriptor) into a pointer to our
internal representation of classes (see the definition of `java_class_t` in
`include/class.h`).

Note that this function takes two arguments, an index into the constant pool
and a pointer to the source class. This pointer should correspond to the
currently used class, i.e., the one that was specified at the command line. Your
code should do the following:

- Check if the given index is 0. This is a special case. See the documentation.
- If not, you should derive a pointer to a [`CONSTANT_Class_info_t`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.1) struct, which
  is defined in `include/constants.h`.
- Check if the constant pool entry has been resolved already (using the
  `IS_RESOLVED()`, `MASK_RESOLVED_BIT()`, and `MARK_RESOLVED()` macros). If it's
  already resolved, you should mask off the high-order bit and return the class
  pointer directly.
- If it hasn't been resolved, you need to resolve it. First you should check
  the class map (using the class name). You can derive the class name using the
  provided `hb_get_const_str()` function. You can lookup in the classmap using
  `hb_get_class()`.
- If the class is in the class map, it should be returned and the constant pool
  entry should be replaced with a pointer to it (with the high order bit set!).
- If it's not been loaded, you need to invoke the class loader using the class
  name, add the class to the class map, [prep the class](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.2), [initialize the class](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.5),
  and overwrite the constant pool entry appropriately. You'll want to use
  `hb_add_class()`, `hb_prep_class()`, and `hb_init_class()`, all defined in
  `include/class.h`.

#### `hb_resolve_method()`

Method resolution is slightly more subtle, as what we are doing here is
implementing the machinery to support polymorphism! We cannot simply resolve
a method by just its name. We also have to consider on which Object it was
invoked and its method _signature_, i.e., the number and types of its arguments.
This is why a descriptor for a method (`lCONSTANT_Methodref_info_t`) contains two
constant pool offsets, one for the name of the method, and another for its
signature.

The general flow of this function is that we should first look and see which
class the method in the method descriptor (determined by its `class_idx`) belongs
to. This class may or may not need to be resolved, hence the order of
implementation. Once it is resolved, we iterate through each method descriptor
of that class and try to match it to the descriptor given to us (namely, do
both the method name and method parameter descriptor match?). If so, we're
done, and we set the constant pool entry to resolved and return the
`method_info_t*`. Otherwise, we cannot just fail. We now must search up the class
hierarchy to see if the method is implemented by this class's superclasses.
This should give you a hint about the algorithmic technique you choose.

Note that this method adds an extra argument, the `target_cls` argument. This
should help with the above hint. The initial call to `hb_resolve_method()` should
provide `NULL` for this argument. If it is not, it means we've failed the initial
search on the method's attached class, and we're now trying to trace back
through its superclasses.

Note, you can follow the instructions
[here](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.3),
but you don't need to worry about implementing access permissions or
exceptions. You really only need to implement the steps outlined in point (2).

## Part 4: Bytecode Interpretation

A significant component of any JVM (or any other VM architecture which
implements a virtual ISA for that matter) is an interpreter, in this case
a Java bytecode interpreter. The purpose of this component of the JVM is to
decode instructions based on their opcode, and dispatch handler functions for
each opcode. This should sound very familiar to you after writing your 6502
emulator. One major difference here is that your instruction handlers will need
to interact much more closely with other parts of the JVM (namely, class
loading, resolution, memory management, exception throwing, etc.). Another is
that because the JVM is a stack machine, you do not have to deal with
registers, making interpretation somewhat easier.

A significant portion of the bytecode interpreter is provided for you. You can
find this implementation in `src/bc_interp.c`. Note that interpretation is only
done within the context of a method! That is, there is no way to execute
bytecode in this JVM without invoking a Java method. This is a very important
distinction from other types of virtual machines.

### Coding Task 2

Your task here is to add some important functionality to the interpreter.
Namely, you will complete the implementations of the following functions:

- `istore/iload`
- `istore_<n>/iload_<n>`
- `iadd`
- `imul`
- `idiv`
- `irem`
- `isub`
- `ineg`
- `new`
- `newarray`
- `arraylength`

You should use the [Java ISA reference](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5%22) to complete the implementation of these
instructions. I will not give you much more information here, but some existing
support functions in the Hawkbeans codebase that you will want to leverage
include:

- `push_val()` and `pop_val()` functions. These are used to manipulate the current frame's operand stack.
  `hb_resolve_class()` you will need for new.
- `gc_obj_alloc` you will also need for new
- You will need `gc_array_alloc()` for `newarray`

Note within `bc_interp.c` you can access the current stack frame using
`cur_thread->cur_frame`. `cur_thread` is a globally scoped pointer that refers
to the currently executing thread (remember, there is only one at this point).

Note that in some cases you might need to look at the C structure representing
a Java object. This is the `native_obj_t` defined in `include/class.h`. You will
need this at least for `arraylength`.

One pitfall to avoid here is to be very careful with the types used by the
elements in a `var_t` (defined in `include/class.h`). For example, you may have
code like:

```C
var_t x;
x.int_val = 1;
x.int_val = -x.int_val;
```

This is actually a bug. The result will actually end up being interpreted as
`0xffffffff` instead of just `-1` since the `int_val` element of a `var_t` is actually
defined as an unsigned type (for various reasons dealing with class loading).
Be careful of this when writing your integer arithmetic routines!

Some of the required actions for these instructions have similarities that
might point towards using macros to avoid repeating code (see [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

## Part 5: Exceptions

No JVM is complete without exceptions. If you've ever had to write complex
error handling code in C using gotos and labels, you will appreciate just how
powerful the abstraction of software exceptions is. In this part, you will add
exception support to Hawkbeans. There is a skeleton in place for you; you must
just fill in the exception interface and the missing usage of it in the
bytecode interpreter.

### Coding Task 3

The interface to exceptions used by the rest of the JVM is defined in `include/exceptions.h`. The two main functions are:

- `hb_throw_exception()`: Used to throw an exception given a reference to an
  existing `Exception` object (which ultimately derives from the `Throwable`
  class). This is the function used by the JVM when a programmer explicitly
  throws an exception (see `testcode/TestNullPointerThrow.java`). This ultimately
  turns into the execution of the athrow bytecode instruction (which you will
  also implement).
- `hb_throw_and_create_excp()`: Used to throw an exception without an object
  reference. This is used when the JVM throws exceptions indirectly caused by
  the execution of some instruction. For example, if an access to an array
  element with an index past the length of an array is attempted, the JVM would
  invoke this method with the `EXCP_NULL_PTR` argument (defined in
  `include/exceptions.h`).

Your task here is to implement `hb_throw_exception()` and
`hb_throw_and_create_excp()` in `src/exceptions.c`, as well as the handler for
the athrow instruction in `src/bc_interp.c`.

I'm not going to give you too much instruction here, but the general principle
is that you need to scan the exception table for the method corresponding to the
current frame and look for a matching exception entry. If the entry is found,
we invoke the handler for that entry. If not, we pop a frame and continue
searching. If we reach the base frame and still have not found a matching
exception table entry, it means that no method explicitly provided a handler
for the exception that was thrown, so we must kill the current thread and exit.
You can expect that if the current code (which threw the exception) has a catch
clause somewhere, you should be finding its exception table within the scope of
that method!

You should look at the documentation for [exceptions](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.10) and the documentation for
[`athrow`](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.athrow) to complete your implementation.

Note, you'll want to look at and leverage these functions which already exist:

- `hb_get_or_load_class()`
- `gc_obj_alloc()`
- `hb_invoke_ctor()`
- `hb_get_class_name()`
- `hb_pop_frame()`
- `hb_get_super_class()`
- `hb_resolve_class()`
- `get_excp_str()`

You'll notice that I've given you less help in terms of skeleton code here.
This is intentional. Reading the documentation is your best bet here. You'll
probably want to break down your implementation into several support functions
that handle each subtask of exception throwing rather than trying to throw it
all into one function.

## Part 6: Garbage Collection

With the current state of Hawkbeans, if you were to write a Java program that
created many objects or arrays, you would see an `OutOfMemoryException` thrown
pretty quickly. This is mostly because the default heap size is so small. But
also because in Java, there's no way to explicitly `free()` objects that we've
allocated. This is because the JVM spec requires that there be some automatic
memory management component present in the JVM, i.e. a garbage collector.

The interesting thing is that the JVM spec does not give _any_ prescriptions on
the nature of the GC, only that it must be _present_. Therefore, we have
significant flexibility when implementing our GC. For this part, I will
describe a simple GC scheme which you will implement.

You will implement a precise garbage collector using the simple but effective
Mark-and-Sweep algorithm which we discussed in class. Recall that a precise
collector knows what a reference looks like vs. some other primitive data type.
If we don't have this information, GC becomes a bit harder because we have to
figure out which things are references and which are not. For example,
everything we're interested in scanning for GC in Hawkbeans will be a `var_t`.
Our goal is to figure out whether a given `var_t` is a reference or just some
primitive type, e.g., an `int`. It is not enough just to look at the bits to tell
the difference. One strategy is that we could add some information to a `var_t`
which would tag the type. This takes up more memory, however, and complicates
the rest of the JVM. We will use a different strategy.

Our GC will use a _reference table_ to track live object references in the JVM.
This is essentially a hash table which uses the address of a reference as its
key. When objects are allocated, and new references to them are created, those
references will be inserted into this hashtable. The GC is invoked periodically
by the bytecode interpreter after a predetermined amount of time has passed.
When the GC is invoked, a set of root objects are scanned. The GC must look at
every variable in each root object and check if it exists in the reference
table. If it does, it will mark the object the reference refers to as **live** and
will then scan that object for more references. This process continues until no
more references are found. This is essentially the _mark_ phase of the GC
algorithm.

After all root objects have been traversed, the _sweep_ phase will iterate
through the reference table, and see which references were still around during
the mark phase. Those which were not are now considered garbage and must be
collected. The details are discussed below.

### Coding Task 4
There is significantly more skeleton code for you to work with for this part.
You should start by taking a look at `gc_init()` in `src/gc.c`. This is where the GC
is initialized. The important bits here that you should look at are the calls
to `add_root()` and `init_ref_tbl()`. Each call to `add_root()` is registering a root
object which the GC shall scan during the mark phase. Each root is associated
with a callback function for scanning that particular root. Your job will be to
implement these scanning functions. The framework for them is there, you only need
fill them in.

Here are the important roots:

- The base object (this is the instantiated object corresponding to the class
  that was provided at the command line): You must scan all of the fields of
  this object for references and any reference you find, you must then descend
  into the object referred to by that reference and scan its fields as well.
- The base object reference: This is the reference to the above object. Because
  it was created before the GC was initialized, it must be special cased. The
  code for scanning this reference is already implemented.
- The class map: you must scan all classes loaded into the class map, and check
  their static fields for references. Recall these fields are those which are
  the same for every instance of the class, so you cannot access them from
  a given object instance. You can find them in the `field_vals` of
  a `java_class_t` structure.
- The base frame: This is the frame corresponding to the main function invoked
  at the beginning of the JVM's execution. This frame's local variables and
  operand stack are scanned for references to objects just like before. Each
  successive frame must also be scanned.

Note that the framework for invoking the scanning functions is already in place (see `gc_collect()`, `mark()`, and `scan_roots()`).

##### Mark
You must first finish the following functions:
- `scan_base_obj()`
- `scan_base_frame()`
- `scan_class_map()`

Note that the mark phase first sets every entry in the ref table to the state
`GC_REF_ABSENT` in the `clear_ref_tbl()` function. That way reference that are not
live will be in that state. When you scan your roots, you should set any
references you find to the state `GC_REF_PRESENT` using the `mark_ref()` function.
You can determine that a variable is indeed a reference using the
`is_valid_ref()` function.

One programming idiom you will see here that you might not recognize is the
pattern involving opaque pointers and callback functions. The `scan_X` functions
are considered callbacks because we pass them as function pointers to the
`add_roots()` registration function at initialization time. When the GC is
invoked, it will call each of these functions in turn. This pattern makes it
easy to add other GC roots later on if we want. See how each of the functions
above each takes a `void * priv_data` as an argument. This is there so that when
the callback function is invoked, it can re-derive whatever state it needs from
that pointer. This state is passed in during callback registration
(`add_root()`). You can see that each callback will be given a pointer to the
GC's state, i.e., a `gc_state_t` pointer.

##### Sweep
You now must complete the sweep phase of the collector. This means you must
fill in the `sweep()` function. This function must iterate through the reference
table (see an example of how to do this in `clear_ref_tbl()`) and collect all
entries whose status is set to `GC_REF_ABSENT`. To collect an object, you must
first free the reference table entry for it, then you must free the object
using the memory allocator's `object_free()` function. Make sure to update the
collectors stats (namely `obj_collected` and `bytes_reclaimed`). Finally, after the
object has been freed, you should `free()` the reference itself.

## Testing and Debugging
This is a big enough codebase to work with that there's really no way to make
progress without testing. You should be testing your code frequently, and even
add your own tests. For example, for each instruction that you add in [Part 4](#part-4-bytecode-interpretation),
you should write a Java test program to make sure it works. This may be
difficult for [Part 3](#part-3-symbolic-resolution-linking) since there are a lot of moving parts. Your best bet there
is to add a lot of debugging prints (using `HB_DEBUG()`) to get an idea of what
the JVM is doing. To compile a new Java program, you'll have to make sure you
have the JDK 1.8 environment installed. Once you do, you can compile a Java
program by running:

```bash
javac -g MyProgram.java
```

The `-g` flag here tells the Java compiler to add debugging information like line
numbers and local variable names to the class binary. **Don't forget that the
class name in your program must match the filename**.

You'll also probably want to get familiar with class file dumps. Probably the most informative is:

```bash
javap -v ClassFile
```

This dumps out pretty much everything in the class file, including the constant
pool items, class methods (including constructors and class initializers), and
static and instance fields.

To enable debugging prints within the Hawkbeans JVM, you can pass environment
variables to the compiler just like you did with Hawknest, only you can do so
at a finer granularity here. If you'd like to see general debugging prints, set
`MODE=DEBUG` when invoking `make`. You can then set the remaining flags depending
on which component of the JVM you'd like to debug. For example, if I'm working
in my bytecode interpreter and I want to enable debugging there, I'd set
`DEBUG_INTERP` to 1, then in my code in `src/bc_interp.c` I'd use the `BC_DEBUG()`
print macro to add debugging printouts specific to that component. These prints
will only appear when debugging is enabled. You can take a look at the `Makefile`
(see `CC_DEBUG_DEFINES`) to see which subsystems have component-specific
debugging prints.

### The Hawkbeans Debugger

I've added a line-oriented debugger much like you used in Hawknest for
Hawkbeans. This will allow you to step through a Java program, set breakpoints,
dump state etc. It isn't complete, but it should help you to find
problems. Just like with Hawknest, start Hawkbeans with the `-i` option to drop
immediately into the debugger shell. Type `help` at that shell to see what
commands you can invoke. Note that these are inspired by both `gdb` and `jdb`, the
Java debugger that ships with OpenJDK.

## Getting Help
If you're having trouble, do not hesitate to get help. Your first bet is to
come to office hours or post to the class Discord. If you need to set up an
appointment for help, I will do my best to accommodate you.

## Extra Credit

- Port Hawkbeans to Hawknest (i.e., run the JVM on the 6502)
- Complete implementation of the bytecode interpreter
- Adding proper access/permission checking in class loading
- Adding support for reflection
- Adding support for actually native libraries (currently, only a few native functions are supported, and are special-cased. You can see how this works in `src/native.c`)
- Add a JIT compiler
- Add support for multiple threads and concurrent execution (and possibly parallel execution using pthreads)
- Add more system support, e.g., for file I/O and networking
- Add support for user-defined class loaders
- Add support for random numbers
- Add sufficient support to the JVM and standard libraries to support lambda expressions (a relatively new feature in Javaland)

## Hand-In

You will hand in your code to me, as in the previous project, using our handin
script. Your code will be graded based primarily on how well it executes
a series of Java tests of my choosing. A small portion of your grade will be
determined by the cleanliness, style, and documentation of your code. I will
definitely penalize you if your code is poorly structured and not broken down
cleanly using abstraction and modularity.

Make sure to commit often. Your commits should encapsulate some logical
component that you added to the codebase. For example, once I complete my
implementation of `iadd`, I would create a commit for that. Make sure your commit
messages are sane and describe succinctly the nature of the additions.

Once you've finished, you can can hand in your code using our scripts. Only one
person from each group should submit a handin. Make sure you're either directly
connected to the IIT network or you are using a VPN before running this:

```bash
make handin
```
