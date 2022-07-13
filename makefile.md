# Makefiles

This makefile will compile C++ files using g++. It will look for a `source` and `header` folder, which will contain the .cpp and .h files respectively. 

```bash
CC = g++
CFLAGS = -c -I./header -I./source

# collecting object file names
src = $(wildcard source/*.cpp)
src1 = $(src:.cpp=.o)

objects := $(src1:source/%=bin/%)

# Compile object files into binary
all : $(objects)
	$(CC) -o run $(objects) -larmadillo

# Generate object files by compiling .cpp and .h files
bin/%.o : source/%.cpp
	$(CC) $(CFLAGS) $?
	mv *.o bin

# Clean Recipe
.PHONY : clean
clean : 
	rm -rf all $(objects)
```
# Breaking it down
CC (Compiler Command) is g++.\
CFLAGS (Compiler Flags) are -c, since we will be using it in the recipe for creating .o files, and -I./header and -I./source to tell make where to look for files. 
```bash
CC = g++
CFLAGS = -c -I./header -I./source
```
src = collect all .cpp files in source\
src1 = src, but with all .cpp extensions replaced with .o. 
```bash
src = $(wildcard source/*.cpp)
src1 = $(src:.cpp=.o)
```
Objects is now equal to src1, but we've replaced all instances of the "source/" string with "bin/". This is because our target files will be in the bin folder. 
```bash
objects := $(src1:source/%=bin/%)
```
The all recipe. $(objects) is all of our prerequesites, which need recipe implementations. 
```bash
all : $(objects)
	$(CC) -o run $(objects) -larmadillo
```
The recipe to create all .o files. bin/%.o matches all .o files in bin. source/%.cpp matches all .cpp files in source. Moves the .o files into bin once it is done compiling it. 
```bash
bin/%.o : source/%.cpp
	$(CC) $(CFLAGS) $?
	mv *.o bin
```
Deleting all the .o files on usage of the `make` command. .PHONY is there to make sure there is no weird behavior if a file is named clean.
```bash
.PHONY : clean
clean : 
	rm -rf all $(objects)
```
# Expanded example
```bash
## HOW TO USE THIS MAKEFILE ##

## WHAT THIS MAKEFILE IS FOR
# - Building or 'making' standard C++ projects using G++. 

## FILE STRUCTURE
# This makefile will look for a HEADER file, in which all of
# your header (.h) files should be placed.
# It will also look for a SOURCE file, in which all of
# your source (.cpp) files should be placed. 
# It will put all of its compiled Object files into a BIN file. 
# Note: header files must use .h extension. 
# Note: header, source, and bin folder should be lowercase.
# Note: if using VSCODE, a c_cpp_configurations folder is also required. 

## BUILD A PROJECT
# Type 'make' in the terminal while in the same directory as the makefile. 
# That will instruct this makefile to run.
# After using 'make', you should see a 'run' binary appear in the same
# directory as this makefile. 
# You may also note the 'bin' file will have .o files in it. (object files)
# These object files are then compiled together to make the binary, run. 
# to run the binary, use './run' in the terminal. 

## CLEAN THE BUILD
# Type 'make clean' in the terminal. 
# This will remove all .o files from bin, allowing you
# to have a clean make. This is necessary when switching compiler versions, 
# switching OSs', or any other operation that would use a different compiler. 

## LINK A LIBRARY
# To link a library, change the 'LIBRARIES = ' variable to include whatever you want.
# For example, to include the linear algebra library aramdillo, simply put:
# LIBRARIES = -larmadillo
# Note: Library must be installed first. 
# Note: For vscode intellisense to work, you may have to have microsoft's
# makefile_tools extension installed. 

## USER-MODIFYABLE VARIABLES ##

# Add libraries to link here. 
LIBRARIES = 

# The name of the binary file that will be produced. 
NAME_OF_BIN = run

## USER - MODIFYABLE VARIABLES ##

### ACTUAL MAKEFILE ### USER STAY OUT ###

# CC     = The compiler we want to use. In this case, g++ from the GCC.
# CFLAGS = Flags for the compiler. These flags tell it to compile a .cpp file (-c)
# into a .o file, and to look for header and source files in the ./header and ./source
CC = g++
CFLAGS = -c -I./header

# Collects all of the source files into a single object. 
# src = all .cpp in source
# src1 = changes .cpp to .o in src.  
src = $(wildcard source/*.cpp)
src1 = $(src:.cpp=.o)

# changes the name of every object in src1 to be
# in the format bin/NAME.o, instead of source/NAME.o.
# It must be in this format, since all of our .o files
# will be in the bin folder. 
objects := $(src1:source/%=bin/%)

# The 'all' recipe, that will produce the binary to run.
# $(objects) is all of our targets, which is all of the .o files in bin.
# you can read this as an example:
#
# gcc -o run bin/main.o -larmadillo
#
# if we only had a main.cpp in source, and were including the armadillo library. 
all : $(objects)
	$(CC) -o $(NAME_OF_BIN) $(objects) $(LIBRARIES)

# Generate object files by compiling .cpp and .h files
bin/%.o : source/%.cpp
	$(CC) $(CFLAGS) $?
	mv *.o bin

# Clean Recipe
.PHONY : clean
clean : 
	rm -rf all $(objects)
```
