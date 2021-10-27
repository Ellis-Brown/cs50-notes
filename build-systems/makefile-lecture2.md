# Makefiles

All things in this lesson are using GNU make


## Make
When you run make, it expects a file called "Makefile"

1. ### Rules
    - Target

        A file that the rule produces. Make will check the timestamp of it to see if the rule needs rebuilding.
    - Prerequeisites

        Files that the rule depends on. If any prequesite is newer than the target, the recipe runs. If a pre-requesite has its own rule, Make will run that rule first if necessary. Its up to you to get this right
    - Recipe
    
        A list of shell commands to run to generate the target. Each lune runs in its own shell, so cd and setting shell variables wont persist across multiple lines.
        You can have as many lines as you want in a recipie.

2. ### Variable defintions

3. ### Special targets

> Note: A make rule will stop if an of the shell commands in the recipie end with a non-zero exit code.
## Special flags
> -s : silent mode
> preface a command with the @ sign, and it only prints the output of the recipie 

## Benefits of a build system
### Consistency
- One simple command runs arbitrary comlpex build rules
- Build happens identically for everyone working opn the project
- Rules can be developed alongside code, tracked in version control.
### Efficiency
- Build is split up into multiple _rules_, whuch transforms inputs into outputs.
- Each rule only needs to run if its inputs have changed.
- Build system figures out the minimal set of rules that need to run.

## Running make
By default, typing `make` builds the first target in the file.

1. **-j[n]** _Runs up to n rules simultaneously_ (meaning, multiple threads. Dont worry about this until you are getting more than 4 seconds of building time using makefile.)

2. **-f < file >** _Reads rules from <file> instead of makefile_

3. **--silent (-s)** _Don't print commands_

> note: it is recommended to turn C, and C++ code into .o intermediate object files, which helps compile time

## .PHONY targets

A phony target is one that is not really hr name of a file, rather it is just the name for a recipe to be executed when you make an explcit request. There are two reasons to use a phony target: to avoid a conflict with a file with the same name, and to improve performance. 
If you write a rule whose recipe will not create the rarget file, the recipie will not be executed every time the target comes up for remaking.

Reason to use a phony file : You have a rule that doesnt build anything, and it may be possible to have a file with that file name.

Example usage:

        .PHONY: name-of-rule

## Comparing a shell scrip vs. Makefile

1. Every single file is built every single time in a shell script, such as intermediate files. Especially when optimized, this can waste a lot of time. Makefiles on the other hand can have intermediate files that were not modiefied not get built at the same time.

