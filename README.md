# 230/198C Makefile Fix for Mac

## Purpose

The reason I wrote this guide is to help other Mac users compile the code natively on your machines.

## Directions

1. Install Googletest on your System (preferrably using Homebrew or some other package manager)

The command is `brew install googletest`

2. Open the Makefile in your prefered editor

3. Add the following variables to the Makefile

```makefile
CXX = g++

CXXFLAGS = -I$(IDIR) -Wall -Wextra -g -pthread -std=c++17
```

4. You will need to then change the compilation of test files

Original:
```makefile
$(ODIR)/%.o: $(TDIR)/%.cpp $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

$(TESTBIN): $(TOBJ) $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS) $(XXLIBS)
```

New:
```makefile
$(ODIR)/%.o: $(TDIR)/%.cpp $(DEPS)
	$(CXX) -c -o $@ $< $(CXXFLAGS)

$(TESTBIN): $(TOBJ) $(OBJ)
	$(CXX) -o $@ $^ $(CXXFLAGS) $(XXLIBS)
```

### The Following are extra non-neccesary steps
5. Installing Bear

The command is `brew install bear`

6. Generating `compile_commands.json` for language server auto-complete

You run `bear -- make`

if you have already ran `make` ensure that you run `make clean` before running bear

## Why do you need to do this?
Googletest the Testing Library we use is in C++, meaning that we need a C++ Compiler to compile it.
The other thing of notibility is that google-test is a library that is external to the system meaning that 
we need to install it through some other means.
I reccomend using Homebrew because it makes it easy to install and remove external packages on a Mac.
The installation instructions for Homebrew can be found [Here](https://docs.brew.sh/Installation).
After Homebrew is installed properly you run `brew install googletest`.

On Mac `/usr/bin/gcc` is symlinked to apple's clang (at `/usr/bin/clang`). 
This means that GCC assumes all of Apple Clang's behaviours because it **is not the real gcc**.

On Linux, gcc is able to discern between C and C++ files at compilation time and compile
them accordingly. However, on Apple's Clang, you need use `clang` for C files and `clang++` for C++ files.
You are able to put these in your own Makefile on a Macbook and the code should compile, however, I am unsure
if gradescope has binaries for `clang` and `clang++`. 

Luckily though, we have an out, using the fake `gcc` and fake `g++` to compile C and C++ Files respectively.

In the Makefile half the work is already done for you as `CC = gcc` is already defined, and `CFLAGS` is also defined.
All we need to do is define `CXX` for our C++ Compiler and `CXXFLAGS` for the flags we pass to that compiler.
We then add `CXX = g++` and `CXXFLAGS =  -I$(IDIR) -Wall -Wextra -g -pthread -std=c++17` to the Makefile, and half of
the work is done.

The last step we need to take is to compile our tests as C++ files.
The reason we need to do this is because the test library, Googletest, is a C++11 Library.
Given this information we need to change the compilation steps for Test to use C++.
When Creating the Object files we change `$(CC) -c -o $@ $< $(CFLAGS)` to `$(CXX) -c -o $@ $< $(CXXFLAGS)`
and when linking the libraries we change `$(CC) -o $@ $^ $(CFLAGS) $(XXLIBS)` to `$(CXX) -o $@ $^ $(CXXFLAGS) $(XXLIBS)`.
all you need to do after this is run `make` and everything should compile

Why the extra step for Bear?

Bear is used to generate the file `compile_commands.json` which is used to provide autocompletion based on the files that you compile for your projects.
I believe the VSCode extension may handle this under the hood for you, however, bear may still be useful if you want auto completion tailored to your make file.

To run bear you first need to install it using `brew install bear` or any other package manager you decide to use.
Next, you would want to run `make clean` to clean out all the directory, and allow bear to analyze the Makefile while building using
`bear -- make`. After this the `compile_commands.json` should be generated and you should be good to go.
