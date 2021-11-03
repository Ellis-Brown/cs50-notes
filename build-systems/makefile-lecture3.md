# The Make language
Class notes: Lecture 3 : October 28



## Recipes and the shell

Each line of a recipe (that is, each line starting with a tab following the declaration of a rule) is interpreted by `/bin/sh`, which is some kind of POSIX shell (but not always Bash)


Its not quite like a shell script:
- Each line is run in its owns shell.
    - Shell state, like variables and the working tree (using cd to get somewhere) resets with every line
- Make does its own preprocessing on each line
    - Strips the leading tab character
    - Interprets the character `$` as a Make variable substitution. (Use `$$` for a literal)
    - Allow _line continuations_ by including a backslash ( `\` ) at the end of a line.

The standard shell quoting rules apply. Use quotes whenever an argument might have spaces, even if it's a make variable

Here is an example of accessing the shell script variable `$USER`
```make 
myname: 
	echo "$$USER" > myname 
```

Here is an example that FAILS.  This is because the 2 lines run in their own shell
```make
cat-passwd:
	cd /etc/
	cat passwd
```
#
## Variables in make

Two types of variable assignment, each of which appear on a line of their own:
- **Simply Expanded**: `NAME := val`
    - Variable references and function calls in val happen _when_ the varaible is _defined_
    - Works like assignments in nearly every other language
    - When in doubt, use these.
    - Unlike the shell, it is okay to add spaces before/after the equals

- **Recursively expanded**: `NAME = val`
    - Variable references and function calls in `val` happen when the variable is _USED_.
    - Allows complex templates to be stored in a variable and used with many different values
    - Can lead to infinite recursion if you're not careful.

Expansions can occur anywhere, and use `$(VAR)` syntax to access.

Example: 
```make
EXE := main     # define the variable here, using simply expanded syntax

$(EXE): main.o log.o
	gcc main.o log.o -o $(EXE)

.PHONY: clean    # Makes sure the rule clean is called.
clean:
	rm -rf *.o $(EXE)
	
main.o: main.c log.h
	gcc -c main.c

log.o: log.c log.h
	gcc -c log.c
```

### Automatic variables

Prevent you from typing the same filenames multiple times in a rule. A few common ones:

- `$@` : name of the target of the rule

- `$^` : Names of all the prerequisites, seperated by spaces

- `$<` : Name of the first prerequesite



### Demo: Simple versus recursive expansion, using automatic variables 

```make
# Makefile

LOG_TARGET = @echo "Current target: $@" ; echo "Prerequiesites: $^"

CFLAGS := -Wall
CFLAGS := $(CFLAGS) -Werror   # Able to append a second term

main: main.o log.o
    $(LOG_TARGET)
    gcc $(CFLAGS) $^ -o $@    # Grab all prerequesites, and name the rule

main.o: main.c log.h
    $(LOG_TARGET)
    gcc $(CFLAGS) -c $< -o $@

log.o: log.c log.h
    $(LOG_TARGET)
    gcc $(CFLAGS) -c  $< -o $@
```

### Example of an infinite recursion program:
```make
Hello = Hello             # variable declaration
Hello = $(Hello) two      # variable appends self, recursively
sayhello:
	$(Hello)              # viewing varriable is an error
	@echo "hello"
```

Raises the following error message:

`Makefile:2: *** Recursive variable 'Hello' references itself (eventually).  Stop`

#

## Functions

Functions are basically varaibles that take arguments and can substitute those argumenbts in their values. Lots of built-in functions, like `word`.

- Functions can accept any number of parameters, separated by arguments.

- The replacement happens before the shell sees the variables. 

`word`, takes a position in a list of words, and a list of words
```make
print-c:
    echo "hello $(word 2, b c a)" 
```
> hello c

`subst` : takes a search term, and a replacement term
```make
print-with-replacement:
    echo $(subst needle, replacement, hello needle)
```
> hello needle


Make variables and functions are textual.

Like the shell, Make has no concept of data types. Everything is text, and Make's minimal escaping and tolerance for spaces can lead to some pretty odd constructs. Here is a code snippet from the GNU Make manual, which helps solve the characters of `space` and `comma` being read in function parameters incorrectly. 

How could we specify they are function parameters, if we need to seperate using commas and spaces?
```make
# The problem we are solving is the literals are being interpreted

comma := ,                   # comma var, used as func param
empty :=                     
space := $(empty) $(empty)   # space var, used as func param

x := a b c
bar := $(subst $(space), $(comma), $(x)) 
# bar is now `a,b,c`.
```

> Note: you do not need to always use make functions and variables. Sometimes using them is more complicated than it should be. Use recipes, not shell commands or other commands when possible. Dont use `$(shell gcc main.c -c)` or something like that, that is just turning the makefile into a glorified shell script...
#

## Macros and stages of evaluation
When building using a makefile, there are two distinct phases: "read-in" phase, and  "target-update" phase:

However, it is worth noting that knowing the explicit details of this are not necessary to use a makefile (i think)

- **Read-in phase** 
    - Reads all makefiles, including included makefiles
    - Internalizes all variables and their values and implicit and explcit rules
    - Builds a dependency graph of targets and prerequesites
- **Target-update phase**
    - Determines which targets needs to be updated
    - Run receipes necessary to update targets

# 

## Pattern rules

The following will match `make (string).o` to using the dependencies of `(string).c` and `string.h`

`%.o: %.c %.h` : Add the missing header dependency to the implict rule. You have to use automatic variables, since we dont know the exact filenames.

```make
%.o: %.c %.h:
    gcc $< -c -o $@ 
```

Demo that writes the name of a text file to the end of the text file 
```make
%.txt:
    echo "Hello I am $@" >> $@
```
