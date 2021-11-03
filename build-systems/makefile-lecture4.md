Class notes makefile lecture 4, live transcription

November 2, 2021
## Implicit rules

Implemented as pattern rules for common file extensions, such as `.c` , `.cpp`, `.o` 

Support limited customization   
- Recipe can be altered through predefined varaibles like CC, CFLAGS, LDFLAGS
- Dependencies can be altered by writing a rule definition with no recipe 

Example:
`.c` to `.o` implicit rule
```make
%.o: %.c
    $(CC) $(CPPFLAGS) $(CFLAGS) -c
```


```make
hello: main.o hello.o foo.o
```
> Note: the program has to have the same name as the .o files being built, or it would not be able to pattern match against it. 

> From make documentation : There is a catolog of Built in rules, which works for compiling certain programs, such as `c`, `pascal`, and `c++`.

Make will look for a pattern that matches the target and the prerequesite. 

Therefore, if I define the following varaibles
- `CPPFLAGS`  Compiler C++ flags. 
- `LDFLAGS` Linker flags
- ... and many others

The implict rule can be run with the set flags that we defined, even though we have the rule.

### Pitfalls of implicit rules

- Header files not automatic prerequisites, but you can add them yourselves

- Not exlicit, which some people / people value . (Hard to tell whats going on.)


### Homework-related question:
Why would you use an explicit rule when you could use an implicit rule like the one above?

Answer: Wanted build output in a folder called "$(outdir)"
Wanted to make the outdir directory if it did not exist, such as mkdir -p 

Also, uses non standard variables, like $(OUTDIR)


## Silencing commands with `@`
- When you run a command, Make prints the command (not its output- the command itself) before running it.
- Can be useful for debugging, but sometimes the command's only purpose is to print something else (eg: echo)
- To prevent the command from being printed, preface the line in its recipe with `@`

## More on varaibles: `?=` and `+=` 

- `?=` assignments
    - Only sets the variable if it has NOT been defined yet
    - Doesn't override variables that have been defined, even if they are empty.
    - Always recursively expanded

- `+=` assignemnts
    - Appends thje given text to the end of an existing variable
    - If not yet defined, recursively-expand. otherwise, same as previous

## Variable overrides & enviornments
- **Varaible overrides**
    - Example: `make CFLAGS=-g hello`
    - Not an enviornment variable: value passed as an argument to make directly
    - Sets the given Make variable to the given value
    - Causes all assignments in Makefile (=, :=, +=) to be ignored

- **Enviornment variables**
    - Example: CFLAGS = -g make hello
    - tells the shell to run make with CFLAGS set in enviornment
    - Make imports all enviornment variables as Make varaibles
    - However, assignments inMakefile override enviornment variables.
        - `=`, `:=`, overwrite completly, `+=`appends to them.
    - `?=` does NOT overwrite the variables set as env vars.
> Note: After setting the enviornment variables in the same line the command is run, the shell variables are added to the commands enviornment for the single execution of the line, and are no longer saved after. Effectively, we are setting the variable OUTSIDE of make, then make will be allowed to use those variables for their set value.
- **Built in implicit definitions**
    - Some variables have a default value , such as `$(CC) = cc`. Overwritten
    by enviornment variables.

## Default goal
```make
.DEFAULT_GOAL := some-target
```
The default goal allows you to overwrite the default behavior of typing `make` with no targets. Typically, make it runs the FIRST target rule in the makefile, but that can be set to be any target.

# Compilation

## What is compilation
In general, it is translating from one language to another. Often then output of compilation is a lower level / less experssive language.

Example: C++ code to machine code.

## What is linking?
- Combining multiple .o files into an executable

```c++
#include "SomeClass.h"
int main(int argc, char *argv[]) {
    SomeClass sc;
    sc.doFunction();
}
```
```c++
#include "SomeClass.h"
class SomeClass {
    SomeClass()              { std::cerr << "Hello "; }
    public void doFunction() { std::cerr << "World" ; }
}
```

Both lines in int main are "dangling pointers" to an undefined function **until it is linked** to **SomeClass.o**

## What is loading?
Reading on disk machine coade into data and memory.
- "Loading is the first part of loading" 

## Why not just list all C files for `gcc` ?
Single `gcc` invocations with multiple C files don't scale
Consider using `gcc file.c file2.c file3.c -o runall`

If you change one C file, you will have to complile all the c files together, meaning every single change requires all files to be compiled and linked.

### Solution: Seperate compilation and linking into seperate steps
```make
main: main.o file1.o file2.o file3.o 
    gcc $^ -o $@

main.o ... # where we may have non implicit dependencies

%.o: %.c %.h
```
- The first rule is linking, and the second/third rules are compilation

- The second rule is only needed for imlpicit dependencies such as .h files in main. If no additional things are needed, rule2 is extra.

> Note: Patterns are only used if there is no more specific rule. `make main.o` would pattern match against the second rule, not the third.

# Building large projects with Make

Goal: small, self-contained modules in a project.

- For projects with hundreds or thousands of files, those files will likely be organized into groups that make up individual components of the program
- Ideally, each group should have its build rules in a single, small file.
    - easier to find the rules you care about
    - easier to copy between projects or split into its own project

Question: **How can we achieve this, while still allowing the entire project to be built with a single command?**

### Possible solution: `recursive make`
```
.
|---lib
|    |
|    |- subsystem1
|         | - Makefile
|         | - other.c
|
|- main
|- main.c
|- main.p
|- Makefile
```
```make
all: main

main: main.o subsystems
    gcc main.o lib/subsystems1/other.o -o main

subsystems: 
    $(MAKE) -C lib/subsystem1
```
Do some recursive call to each Makefile.

**Problem**:
it is really hard for a subdirectory to depend on another, in a clean way

 There are 3 work arounds, but they all introduce comlpexity:
- Manually order all your submakes so that each directory is built after the one that depends on it
- Run multiple sbumake passes, each time building the next one the thing depends on
- Structurre your code so that each subdirectory is totally self-contained.

> Question asked: is it bad practice to have submodules depend on each other? Answer: Not always. 


### Possible solution: `include` directive

The top level Makefile uses `include` on paths to other Makefiles in subdirectories, which are written to create/remove all files based on the path form the top level module. **Problem: hard to read & maintain.**

Example:
- include directive at the top level makefile
```make
main: main.o lib/subsystem/example.o
    gcc $^ -o $@

include lib/subsystem/Makefile

clean: subsystem1-clean 
    rm -f *.o main
```
- Lower level subdirectory makefile for subsystem.
- Makefile for lib/subsystem1
 It is written knowing it is in the subdirectory.
```make
subsystem.o: example.c
    gcc -c $^ -o $@

subsystem1-clean
    rm lib/subsystem1/example.o 
```
--- 
### Case study : Linux kernel
The Linux kernel is a famous user of recursive make



Recursive Make is a good fit for projects written entirely in C:
- Cross module dependencies use header files, which don't need to be built
- Individual modules produce object files, which the higher level Make can link together
- Corner cases still require some hacks, but the volume is manageable.
    
From the "Linux Kernel Makefiles" documentation:

- "A makefile is only responsible for building objects in its own directory. Files in subdirectories should be taken care of by Makefiles in these subdirs. The build system will automatically invoke make recursively in sbudirectories, provided you let it know of them."