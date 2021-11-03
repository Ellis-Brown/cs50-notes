# Makefiles
All things in this lesson are using GNU make

Note: these notes were a live recording of a lecture. The offical notes released for this class can be found [here](https://bernsteinbear.com/isdt/lecture-notes/3-bld/#lecture-2). 


## Make
When you run make, it expects a file called "Makefile"

1. ### Rules
    - Target

        A file that the rule produces. Make will check the timestamp of it to see if the rule needs rebuilding.
    - Prerequeisites

        Files that the rule depends on. If any prequesite is newer than the target, the recipe runs. If a pre-requesite has its own rule, Make will run that rule first if necessary. Its up to you to get this right
    - Recipe
    
        A list of shell commands to run to generate the target. Each line runs in its own shell, so cd and setting shell variables won't persist across multiple lines.
        You can have as many lines as you want in a recipie.

2. ### Variable definitions

3. ### Special targets

> Note: A make rule will stop if an of the shell commands in the recipe end with a non-zero exit code.
## Special flags
> -s : silent mode
> preface a command with the @ sign, and it only prints the output of the recipe 

## Benefits of a build system
### Consistency
- One simple command runs arbitrary complex build rules
- Build happens identically for everyone working on the project
- Rules can be developed alongside code, tracked in version control.
### Efficiency
- Build is split up into multiple _rules_, which transforms inputs into outputs.
- Each rule only needs to run if its inputs have changed.
- Build system figures out the minimal set of rules that need to run.

## Running make
By default, typing `make` builds the first target in the file.

1. **-j[n]** _Runs up to n rules simultaneously_ (meaning, multiple threads. Dont worry about this until you are getting more than 4 seconds of building time using makefile.)

2. **-f < file >** _Reads rules from <file> instead of makefile_

3. **--silent (-s)** _Don't print commands_

> note: it is recommended to turn C, and C++ code into .o intermediate object files, which helps compile time

## .PHONY targets

A phony target is one that is not really hr name of a file, rather it is just the name for a recipe to be executed when you make an explicit request. There are two reasons to use a phony target: to avoid a conflict with a file with the same name, and to improve performance. 
If you write a rule whose recipe will not create the rarget file, the recipie will not be executed every time the target comes up for remaking.

Reason to use a phony file : You have a rule that doesn't build anything, and it may be possible to have a file with that file name.

Example usage:
```make
.PHONY: name-of-rule
```

## Comparing a shell script vs. Makefile

1. Every single file is built every single time in a shell script, such as intermediate files. Especially when optimized, this can waste a lot of time. Makefiles on the other hand can have intermediate files that were not modified and not get built at the same time.

## Example makefile
Uses the tools learned in this lesson.
```make
say-hello:
    @echo "World
    @echo "hello"


silent-gcc:
    @gcc *.c -o somecoolname
    
non-silent-gcc:
    gcc *.c -o someOtherName


with-no-dependencies:
    gcc myprogram.c -o myprogram


with-dependencies: mypprogram.c myprogram.h
    gcc mypprogram.c -o myprogram


clean: 
    rm myprogram -rf *.o
```
